const VoltageColors = {
    "500 kV": "#3b82f6",
    "230 kV": "#ef4444",
    "138 kV": "#f97316",
    "69 kV": "#06b6d4",
    "34.5 kV": "#10b981",
    "13.8 kV": "#a855f7"
};

class NetworkModel {
    constructor() {
        this.nodes = [];
        this.wires = [];
    }
    exportJSON() { return JSON.stringify({ nodes: this.nodes, wires: this.wires }, null, 2); }
    importJSON(jsonStr) {
        try {
            const data = JSON.parse(jsonStr);
            if (data.nodes && data.wires) {
                this.nodes = data.nodes;
                this.wires = data.wires;
                return true;
            }
        } catch (e) { console.error(e); }
        return false;
    }
}

class SLDApp {
    constructor() {
        this.model = new NetworkModel();
        this.canvas = document.getElementById('sld-canvas');
        this.ctx = this.canvas.getContext('2d');
        
        this.mode = 'select'; 
        this.selectedElement = null;
        this.selectedWire = null;
        
        this.pan = { x: 0, y: 0 };
        this.zoom = 1.0;
        
        this.isPanning = false;
        this.isDraggingNode = false;
        this.dragStart = { x: 0, y: 0 };
        this.wireStartNode = null;
        this.mousePos = { x: 0, y: 0 };

        this.animPhase = 0;
        this.autoSolve = true;
        
        this.initEvents();
        this.resize();
        this.loop();
    }

    initEvents() {
        window.addEventListener('resize', () => this.resize());
        const container = document.getElementById('canvas-container');
        container.addEventListener('pointerdown', e => this.onPointerDown(e));
        container.addEventListener('pointermove', e => this.onPointerMove(e));
        container.addEventListener('pointerup', e => this.onPointerUp(e));
        container.addEventListener('wheel', e => this.onWheel(e), { passive: false });

        document.querySelectorAll('.palette-item').forEach(item => {
            const type = item.dataset.type;
            item.addEventListener('pointerdown', e => {
                e.preventDefault();
                const wx = (this.canvas.width / 2 - this.pan.x) / this.zoom;
                const wy = (this.canvas.height / 2 - this.pan.y) / this.zoom;
                this.addElement(type, wx, wy);
            });
            item.addEventListener('dragstart', e => { e.dataTransfer.setData('text/plain', type); });
        });

        document.getElementById('btn-select').addEventListener('click', () => this.setMode('select'));
        document.getElementById('btn-wire').addEventListener('click', () => this.setMode('wire'));
        document.getElementById('btn-export').addEventListener('click', () => this.openModal('export'));
        document.getElementById('btn-import').addEventListener('click', () => document.getElementById('import-file').click());
        document.getElementById('btn-inject').addEventListener('click', () => document.getElementById('inject-file').click());
        document.getElementById('btn-limits').addEventListener('click', () => document.getElementById('limits-file').click());
        
        const btnSolve = document.getElementById('btn-solver-toggle');
        btnSolve.addEventListener('click', () => {
            this.autoSolve = !this.autoSolve;
            if (this.autoSolve) {
                btnSolve.className = 'btn btn-success';
                document.getElementById('txt-solver').innerText = 'Auto-Solve: ON';
                document.getElementById('icon-solver').innerHTML = '<path d="M5 3v18l15-9-15-9z"/>'; 
                this.runPowerFlow();
            } else {
                btnSolve.className = 'btn btn-warning';
                document.getElementById('txt-solver').innerText = 'Auto-Solve: OFF';
                document.getElementById('icon-solver').innerHTML = '<rect x="6" y="4" width="4" height="16"/><rect x="14" y="4" width="4" height="16"/>'; 
                
                document.getElementById('sys-gen').innerText = 'OFF';
                document.getElementById('sys-load').innerText = 'OFF';
                document.getElementById('sys-balance').innerText = 'OFF';
                document.getElementById('deficit-row').style.display = 'none';
            }
            this.updateInspector();
        });

        document.getElementById('btn-clear').addEventListener('click', () => {
            if (confirm("Clear entire model?")) {
                this.model.nodes = [];
                this.model.wires = [];
                this.selectedElement = null;
                this.selectedWire = null;
                this.updateInspector();
            }
        });

        document.getElementById('import-file').addEventListener('change', e => {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = (evt) => {
                if (this.model.importJSON(evt.target.result)) {
                    this.selectedElement = null;
                    this.selectedWire = null;
                    if (this.autoSolve) this.runPowerFlow(); 
                    this.updateInspector();
                    alert("Model imported successfully!");
                } else { alert("Invalid JSON file format."); }
            };
            reader.readAsText(file);
            e.target.value = '';
        });

        // 1. LIVE DATA INJECTOR
        document.getElementById('inject-file').addEventListener('change', e => {
            const file = e.target.files[0];
            if (!file) return;
            const ext = file.name.split('.').pop().toLowerCase();
            const reader = new FileReader();
            reader.onload = (evt) => {
                let updated = 0;
                if (ext === 'csv') {
                    const lines = evt.target.result.split(/\r?\n/).filter(l => l.trim().length > 0);
                    if (lines.length > 0) {
                        const headers = lines[0].split(',').map(h => h.trim().toLowerCase());
                        const nameIdx = headers.indexOf('name');
                        const valIdx = headers.indexOf('value');
                        
                        if (nameIdx > -1 && valIdx > -1) {
                            this.model.nodes.forEach(n => { if (n.type === 'generator' || n.type === 'load') n.mw = 0; });
                            for (let i = 1; i < lines.length; i++) {
                                const cols = lines[i].split(',').map(c => c.trim());
                                if (cols.length <= Math.max(nameIdx, valIdx)) continue;
                                const name = cols[nameIdx];
                                const val = parseFloat(cols[valIdx]);
                                let node = this.model.nodes.find(n => n.name === name);
                                if (node && !isNaN(val)) { node.mw = val; updated++; }
                            }
                            alert(`Successfully injected CSV Live Data to ${updated} elements.`);
                        } else { alert("Error: CSV must contain exact headers: 'Name' and 'Value'."); }
                    }
                } else {
                    try {
                        const data = JSON.parse(evt.target.result);
                        this.model.nodes.forEach(n => { if (n.type === 'generator' || n.type === 'load') n.mw = 0; });
                        const processLiveNode = (item, key) => {
                            let name = item.name || key;
                            let node = this.model.nodes.find(n => n.name === name || n.id === name);
                            if (node) {
                                if (item.mw !== undefined) node.mw = parseFloat(item.mw);
                                if (item.value !== undefined) node.mw = parseFloat(item.value);
                                if (item.status !== undefined) node.status = item.status;
                                updated++;
                            }
                        };
                        if (Array.isArray(data)) data.forEach(item => processLiveNode(item));
                        else for (let key in data) processLiveNode(data[key], key);
                        alert(`Successfully applied JSON Live Data to ${updated} elements.`);
                    } catch (err) { alert("Invalid JSON format for data injection."); }
                }
                if (this.autoSolve) this.runPowerFlow();
                this.updateInspector();
            };
            reader.readAsText(file);
            e.target.value = ''; 
        });

        // 2. LIMITS DATA INJECTOR
        document.getElementById('limits-file').addEventListener('change', e => {
            const file = e.target.files[0];
            if (!file) return;
            const ext = file.name.split('.').pop().toLowerCase();
            const reader = new FileReader();
            reader.onload = (evt) => {
                let updated = 0;
                if (ext === 'csv') {
                    const lines = evt.target.result.split(/\r?\n/).filter(l => l.trim().length > 0);
                    if (lines.length > 0) {
                        const headers = lines[0].split(',').map(h => h.trim().toLowerCase());
                        const nameIdx = headers.indexOf('name');
                        let valIdx = headers.indexOf('limit');
                        if (valIdx === -1) valIdx = headers.indexOf('value'); 
                        
                        if (nameIdx > -1 && valIdx > -1) {
                            for (let i = 1; i < lines.length; i++) {
                                const cols = lines[i].split(',').map(c => c.trim());
                                if (cols.length <= Math.max(nameIdx, valIdx)) continue;
                                const name = cols[nameIdx];
                                const val = parseFloat(cols[valIdx]);
                                
                                if (!isNaN(val)) {
                                    let isUpdated = false;
                                    let node = this.model.nodes.find(n => n.name === name);
                                    if (node && node.type === 'transformer') { node.ratingMVA = val; isUpdated = true; } 
                                    else {
                                        let wire = this.model.wires.find(w => w.name === name);
                                        if (wire) { wire.limit = val; isUpdated = true; }
                                    }
                                    if (isUpdated) updated++;
                                }
                            }
                            alert(`Successfully applied Thermal Limits to ${updated} lines/transformers.`);
                        } else { alert("Error: Limits CSV must contain 'Name' and 'Limit' (or 'Value') headers."); }
                    }
                } else {
                    try {
                        const data = JSON.parse(evt.target.result);
                        const processLimitItem = (item, key) => {
                            let name = item.name || key;
                            let limitVal = item.limit !== undefined ? parseFloat(item.limit) : (item.value !== undefined ? parseFloat(item.value) : NaN);
                            if (!isNaN(limitVal)) {
                                let isUpdated = false;
                                let node = this.model.nodes.find(n => n.name === name || n.id === name);
                                if (node && node.type === 'transformer') { node.ratingMVA = limitVal; isUpdated = true; } 
                                else {
                                    let wire = this.model.wires.find(w => w.name === name || w.id === name);
                                    if (wire) { wire.limit = limitVal; isUpdated = true; }
                                }
                                if (isUpdated) updated++;
                            }
                        };
                        if (Array.isArray(data)) data.forEach(item => processLimitItem(item));
                        else for (let key in data
