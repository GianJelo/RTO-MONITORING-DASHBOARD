<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Advanced SLD Market Network Simulator - JS KCL Solver</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #0b0f19; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; color: #f3f4f6; }
        
        #app-container { display: flex; width: 100vw; height: 100vh; height: 100dvh; flex-direction: column; overflow: hidden; position: fixed; inset: 0; }
        
        header { height: 48px; background: #111827; border-bottom: 1px solid #1f2937; display: flex; align-items: center; justify-content: space-between; padding: 0 16px; flex-shrink: 0; z-index: 10; }
        .logo-area { display: flex; align-items: center; gap: 8px; font-weight: 700; font-size: 15px; letter-spacing: 0.5px; color: #60a5fa; }
        .logo-badge { background: #1d4ed8; color: white; padding: 2px 6px; border-radius: 4px; font-size: 10px; font-weight: 600; }
        .header-tools { display: flex; gap: 8px; align-items: center; }
        
        .btn { background: #1f2937; color: #d1d5db; border: 1px solid #374151; padding: 6px 12px; border-radius: 6px; font-size: 12px; font-weight: 500; cursor: pointer; transition: all 0.2s; display: flex; align-items: center; gap: 6px; }
        .btn:hover { background: #374151; color: white; border-color: #4b5563; }
        .btn.active { background: #2563eb; color: white; border-color: #3b82f6; }
        .btn-success { background: #065f46; color: #d1fae5; border-color: #047857; }
        .btn-success:hover { background: #047857; color: white; }
        .btn-warning { background: #9a3412; color: #ffedd5; border-color: #c2410c; }
        .btn-warning:hover { background: #c2410c; color: white; }
        .btn-primary { background: #1d4ed8; color: white; border-color: #2563eb; }
        .btn-primary:hover { background: #2563eb; }
        .btn-info { background: #0369a1; color: white; border-color: #0284c7; }
        .btn-info:hover { background: #0284c7; }

        .workspace { display: flex; flex: 1; width: 100%; height: calc(100% - 48px); position: relative; overflow: hidden; }
        
        .sidebar-left { width: 260px; background: #111827; border-right: 1px solid #1f2937; display: flex; flex-direction: column; flex-shrink: 0; z-index: 5; }
        .panel-section-title { font-size: 10px; font-weight: 700; text-transform: uppercase; color: #9ca3af; letter-spacing: 0.8px; padding: 10px 12px 6px 12px; }
        .palette-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; padding: 8px 12px; }
        .palette-item { background: #1f2937; border: 1px solid #374151; border-radius: 6px; padding: 10px; display: flex; flex-direction: column; align-items: center; gap: 6px; cursor: pointer; transition: all 0.2s; touch-action: manipulation; }
        .palette-item:hover { background: #374151; border-color: #60a5fa; transform: translateY(-1px); }
        .palette-item span { font-size: 11px; color: #d1d5db; font-weight: 500; }
        .palette-icon { width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; pointer-events: none; }

        .telemetry-box { background: #0f172a; border-top: 1px solid #1f2937; padding: 12px; margin-top: auto; font-size: 11px; display: flex; flex-direction: column; gap: 6px; }
        .telemetry-row { display: flex; justify-content: space-between; align-items: center; }
        .telemetry-label { color: #9ca3af; }
        .telemetry-val { font-family: monospace; font-weight: 600; color: #34d399; }
        .telemetry-val.warning { color: #f87171; animation: pulse 1s infinite; }
        @keyframes pulse { 0% { opacity: 1; } 50% { opacity: 0.5; } 100% { opacity: 1; } }

        .canvas-container { flex: 1; background: #0b0f19; position: relative; overflow: hidden; cursor: default; touch-action: none; }
        canvas { display: block; width: 100%; height: 100%; }

        .sidebar-right { width: 280px; background: #111827; border-left: 1px solid #1f2937; display: flex; flex-direction: column; flex-shrink: 0; z-index: 5; }
        .inspector-content { padding: 12px; overflow-y: auto; flex: 1; display: flex; flex-direction: column; gap: 12px; }
        .form-group { display: flex; flex-direction: column; gap: 4px; }
        .form-group label { font-size: 11px; color: #9ca3af; font-weight: 500; }
        .form-control { background: #1f2937; border: 1px solid #374151; color: #f3f4f6; padding: 6px 10px; border-radius: 4px; font-size: 12px; width: 100%; }
        .form-control:focus { outline: none; border-color: #3b82f6; background: #111827; }
        select.form-control { cursor: pointer; }

        #tooltip { position: absolute; background: rgba(17, 24, 39, 0.95); border: 1px solid #374151; padding: 8px 12px; border-radius: 6px; font-size: 11px; pointer-events: none; display: none; z-index: 100; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.5); backdrop-filter: blur(4px); }
        #tooltip .title { font-weight: 700; color: #60a5fa; margin-bottom: 4px; border-bottom: 1px solid #374151; padding-bottom: 2px; }
        #tooltip .row { display: flex; justify-content: space-between; gap: 16px; color: #d1d5db; }
        #tooltip .val { font-family: monospace; color: #34d399; }
        #tooltip .val.overload { color: #f87171; font-weight: 700; }

        .modal-overlay { position: fixed; inset: 0; background: rgba(0, 0, 0, 0.7); backdrop-filter: blur(2px); display: none; align-items: center; justify-content: center; z-index: 1000; }
        .modal { background: #111827; border: 1px solid #374151; width: 500px; max-width: 90vw; border-radius: 8px; display: flex; flex-direction: column; overflow: hidden; box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5); }
        .modal-header { padding: 12px 16px; background: #1f2937; font-weight: 600; font-size: 13px; display: flex; justify-content: space-between; align-items: center; }
        .modal-body { padding: 16px; display: flex; flex-direction: column; gap: 12px; }
        .modal-footer { padding: 12px 16px; background: #0f172a; border-top: 1px solid #1f2937; display: flex; justify-content: flex-end; gap: 8px; }
        textarea.form-control { resize: vertical; min-height: 150px; font-family: monospace; font-size: 11px; }

        .canvas-hud { position: absolute; bottom: 16px; left: 16px; background: rgba(17, 24, 39, 0.85); border: 1px solid #374151; padding: 8px 12px; border-radius: 6px; font-size: 11px; pointer-events: none; display: flex; gap: 16px; backdrop-filter: blur(4px); }
        .hud-item { display: flex; align-items: center; gap: 6px; }
        .hud-dot { width: 8px; height: 8px; border-radius: 50%; }
    </style>
</head>
<body>
<div id="app-container">
    <header>
        <div class="logo-area">
            <span>⚡ RTO JAVASCRIPT KCL NETWORK SOLVER</span>
            <span class="logo-badge">FINAL v8.5</span>
        </div>
        <div class="header-tools">
            <button class="btn active" id="btn-select" title="Select & Move (V)">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 3l7.07 16.97 2.51-7.39 7.39-2.51L3 3z"/></svg>
                Select
            </button>
            <button class="btn" id="btn-wire" title="Connect Wire / Line (W)">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="5" cy="5" r="3"/><circle cx="19" cy="19" r="3"/><path d="M5 8v4a6 6 0 0 0 6 6h2"/></svg>
                Wire Tool
            </button>
            <div style="width: 1px; height: 20px; background: #374151; margin: 0 4px;"></div>
            
            <button class="btn btn-success" id="btn-solver-toggle" title="Toggle Auto Power Flow Solver">
                <svg id="icon-solver" width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M5 3v18l15-9-15-9z"/></svg>
                <span id="txt-solver">Auto-Solve: ON</span>
            </button>

            <button class="btn btn-primary" id="btn-inject" title="Inject Active Power Data (Generators & Loads)">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v-4"/><polyline points="17 8 12 3 7 8"/><line x1="12" y1="3" x2="12" y2="15"/></svg>
                Live Data
            </button>

            <button class="btn btn-info" id="btn-limits" title="Inject Thermal Limits (Lines & Transformers)">
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 2v20M17 5H9.5a3.5 3.5 0 0 0 0 7h5a3.5 3.5 0 0 1 0 7H6"/></svg>
                Limits Data
            </button>
            
            <button class="btn" id="btn-export" title="Export Model Schema JSON">Export</button>
            <button class="btn" id="btn-import" title="Import Model Schema JSON">Import</button>
            <button class="btn btn-warning" id="btn-clear" title="Clear Canvas">Clear</button>
        </div>
    </header>

    <div class="workspace">
        <div class="sidebar-left">
            <div class="panel-section-title">Grid Elements Palette</div>
            <div class="palette-grid">
                <div class="palette-item" draggable="true" data-type="bus">
                    <div class="palette-icon"><svg width="24" height="24" viewBox="0 0 24 24"><rect x="2" y="10" width="20" height="4" rx="2" fill="#ef4444"/></svg></div>
                    <span>Busbar</span>
                </div>
                <div class="palette-item" draggable="true" data-type="generator">
                    <div class="palette-icon"><svg width="24" height="24" viewBox="0 0 24 24"><circle cx="12" cy="12" r="9" fill="none" stroke="#34d399" stroke-width="2"/><text x="12" y="16" font-size="12" fill="#34d399" text-anchor="middle" font-weight="bold">G</text></svg></div>
                    <span>Generator</span>
                </div>
                <div class="palette-item" draggable="true" data-type="load">
                    <div class="palette-icon"><svg width="24" height="24" viewBox="0 0 24 24"><polygon points="12,20 3,6 21,6" fill="#f59e0b"/></svg></div>
                    <span>Load</span>
                </div>
                <div class="palette-item" draggable="true" data-type="transformer">
                    <div class="palette-icon"><svg width="24" height="24" viewBox="0 0 24 24"><circle cx="12" cy="8" r="5" fill="none" stroke="#60a5fa" stroke-width="2"/><circle cx="12" cy="16" r="5" fill="none" stroke="#60a5fa" stroke-width="2"/></svg></div>
                    <span>Transformer</span>
                </div>
                <div class="palette-item" draggable="true" data-type="breaker">
                    <div class="palette-icon"><svg width="24" height="24" viewBox="0 0 24 24"><rect x="6" y="8" width="12" height="8" rx="2" fill="#ef4444"/><line x1="12" y1="2" x2="12" y2="8" stroke="#ef4444" stroke-width="2"/><line x1="12" y1="16" x2="12" y2="22" stroke="#ef4444" stroke-width="2"/></svg></div>
                    <span>Breaker</span>
                </div>
            </div>

            <div class="panel-section-title">System Status</div>
            <div class="telemetry-box">
                <div class="telemetry-row">
                    <span class="telemetry-label">Total Generation:</span>
                    <span class="telemetry-val" id="sys-gen">0.0 MW</span>
                </div>
                <div class="telemetry-row">
                    <span class="telemetry-label">Total Demand:</span>
                    <span class="telemetry-val" id="sys-load">0.0 MW</span>
                </div>
                <div class="telemetry-row">
                    <span class="telemetry-label">Balance / Mismatch:</span>
                    <span class="telemetry-val" id="sys-balance">0.0 MW</span>
                </div>
                <div class="telemetry-row" id="deficit-row" style="display: none;">
                    <span class="telemetry-label" style="color: #f87171;">System Deficit:</span>
                    <span class="telemetry-val warning" id="sys-deficit">LOAD SHEDDING</span>
                </div>
            </div>
        </div>

        <div class="canvas-container" id="canvas-container">
            <canvas id="sld-canvas"></canvas>
            <div class="canvas-hud">
                <div class="hud-item"><div class="hud-dot" style="background:#3b82f6;"></div><span>500 kV</span></div>
                <div class="hud-item"><div class="hud-dot" style="background:#ef4444;"></div><span>230 kV</span></div>
                <div class="hud-item"><div class="hud-dot" style="background:#f97316;"></div><span>138 kV</span></div>
                <div class="hud-item"><div class="hud-dot" style="background:#06b6d4;"></div><span>69 kV</span></div>
                <div class="hud-item"><div class="hud-dot" style="background:#10b981;"></div><span>&lt;40 kV</span></div>
            </div>
        </div>

        <div class="sidebar-right">
            <div class="panel-section-title">Inspector</div>
            <div class="inspector-content" id="inspector-content">
                <div style="color: #6b7280; font-size: 11px; text-align: center; margin-top: 40px;">
                    Select any node, wire, or equipment to inspect and configure parameters.
                </div>
            </div>
        </div>
    </div>
</div>

<div id="tooltip">
    <div class="title" id="tt-title">Element</div>
    <div class="row"><span>Type:</span> <span class="val" id="tt-type">-</span></div>
    <div class="row"><span>Active Flow:</span> <span class="val" id="tt-flow">-</span></div>
    <div class="row"><span>Status:</span> <span class="val" id="tt-status">-</span></div>
</div>

<input type="file" id="import-file" style="display: none;" accept=".json">
<input type="file" id="inject-file" style="display: none;" accept=".json, .csv">
<input type="file" id="limits-file" style="display: none;" accept=".json, .csv">

<div class="modal-overlay" id="json-modal">
    <div class="modal">
        <div class="modal-header">
            <span id="modal-title">Export Model JSON</span>
            <button class="btn" style="padding: 2px 6px;" id="modal-close">✕</button>
        </div>
        <div class="modal-body">
            <textarea class="form-control" id="json-textarea"></textarea>
        </div>
        <div class="modal-footer">
            <button class="btn" id="modal-copy">Copy to Clipboard</button>
            <button class="btn btn-success" id="modal-action">Confirm Import</button>
        </div>
    </div>
</div>

<script>
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
                        else for (let key in data) processLimitItem(data[key], key);
                        alert(`Successfully applied JSON Thermal Limits to ${updated} lines/transformers.`);
                    } catch (err) { alert("Invalid JSON format for limits injection."); }
                }
                if (this.autoSolve) this.runPowerFlow();
                this.updateInspector();
            };
            reader.readAsText(file);
            e.target.value = ''; 
        });

        document.getElementById('modal-close').addEventListener('click', () => document.getElementById('json-modal').style.display = 'none');
        document.getElementById('modal-copy').addEventListener('click', () => {
            navigator.clipboard.writeText(document.getElementById('json-textarea').value);
            alert("Copied to clipboard!");
        });
        document.getElementById('modal-action').addEventListener('click', () => {
            const txt = document.getElementById('json-textarea').value;
            if (this.model.importJSON(txt)) {
                document.getElementById('json-modal').style.display = 'none';
                this.selectedElement = null;
                this.selectedWire = null;
                if (this.autoSolve) this.runPowerFlow();
                this.updateInspector();
                alert("Model successfully loaded!");
            } else { alert("Invalid JSON data."); }
        });

        window.addEventListener('keydown', e => {
            if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') return;
            if (e.key.toLowerCase() === 'v') this.setMode('select');
            if (e.key.toLowerCase() === 'w') this.setMode('wire');
            if (e.key.toLowerCase() === 'r' && this.selectedElement) {
                if (this.selectedElement.type === 'bus') {
                    this.selectedElement.orientation = this.selectedElement.orientation === 'H' ? 'V' : 'H';
                    const w = this.selectedElement.width;
                    this.selectedElement.width = this.selectedElement.height;
                    this.selectedElement.height = w;
                } else { this.selectedElement.orientation = this.selectedElement.orientation === 'H' ? 'V' : 'H'; }
                this.updateInspector();
            }
            if ((e.key === 'Delete' || e.key === 'Backspace')) {
                if (this.selectedElement) this.deleteNode(this.selectedElement);
                if (this.selectedWire) this.deleteWire(this.selectedWire);
            }
        });
    }

    resize() {
        const container = document.getElementById('canvas-container');
        this.canvas.width = container.clientWidth;
        this.canvas.height = container.clientHeight;
    }

    setMode(m) {
        this.mode = m;
        document.getElementById('btn-select').classList.toggle('active', m === 'select');
        document.getElementById('btn-wire').classList.toggle('active', m === 'wire');
        this.wireStartNode = null;
    }

    addElement(type, x, y) {
        const id = 'el_' + Math.random().toString(36).substr(2, 6);
        let newNode = { id, type, name: `${type.toUpperCase()}_${this.model.nodes.length + 1}`, x: x, y: y, status: 'closed' };
        
        if (type === 'bus') { newNode.width = 140; newNode.height = 16; newNode.voltage = '230 kV'; newNode.orientation = 'H'; } 
        else if (type === 'generator') { newNode.mw = 25; newNode.maxMw = 50; } 
        else if (type === 'load') { newNode.mw = 20; } 
        else if (type === 'transformer') { newNode.reactance = 0.05; newNode.ratingMVA = 100; newNode.orientation = 'V'; } 
        else if (type === 'breaker') { newNode.orientation = 'V'; }

        this.model.nodes.push(newNode);
        this.selectedElement = newNode;
        this.selectedWire = null;
        this.updateInspector();
    }

    deleteNode(node) {
        this.model.nodes = this.model.nodes.filter(n => n.id !== node.id);
        this.model.wires = this.model.wires.filter(w => w.from !== node.id && w.to !== node.id);
        this.selectedElement = null;
        this.updateInspector();
    }

    deleteWire(wire) {
        this.model.wires = this.model.wires.filter(w => w.id !== wire.id);
        this.selectedWire = null;
        this.updateInspector();
    }

    findNodeAt(wx, wy) {
        for (let i = this.model.nodes.length - 1; i >= 0; i--) {
            const n = this.model.nodes[i];
            let w = 50, h = 50;
            if (n.type === 'bus') { w = n.width || 120; h = n.height || 16; }
            else if (n.type === 'generator' || n.type === 'load') { w = 40; h = 40; }
            else if (n.type === 'transformer' || n.type === 'breaker') { w = 36; h = 36; }

            if (wx >= n.x - w/2 && wx <= n.x + w/2 && wy >= n.y - h/2 && wy <= n.y + h/2) return n;
        }
        return null;
    }

    getWireGroups() {
        let groups = {};
        this.model.wires.forEach(w => {
            let key = [w.from, w.to].sort().join('-');
            if (!groups[key]) groups[key] = [];
            groups[key].push(w);
        });
        return groups;
    }

    findWireAt(wx, wy) {
        const groups = this.getWireGroups();
        for (const w of this.model.wires) {
            const n1 = this.model.nodes.find(n => n.id === w.from);
            const n2 = this.model.nodes.find(n => n.id === w.to);
            if (!n1 || !n2) continue;
            
            const group = groups[[n1.id, n2.id].sort().join('-')];
            const index = group.indexOf(w);
            const offset = (index - (group.length - 1) / 2) * 20;

            let p1 = this.getConnectionPoint(n1, n2, offset);
            let p2 = this.getConnectionPoint(n2, n1, offset);
            
            let drawMidX = (p1.x + p2.x) / 2;
            if (n1.type !== 'bus' && n2.type !== 'bus') drawMidX += offset;

            const d1 = this.distToSegment({x: wx, y: wy}, {x: p1.x, y: p1.y}, {x: drawMidX, y: p1.y});
            const d2 = this.distToSegment({x: wx, y: wy}, {x: drawMidX, y: p1.y}, {x: drawMidX, y: p2.y});
            const d3 = this.distToSegment({x: wx, y: wy}, {x: drawMidX, y: p2.y}, {x: p2.x, y: p2.y});
            
            if (d1 < 8 || d2 < 8 || d3 < 8) return w;
        }
        return null;
    }
    
    distToSegment(p, a, b) {
        const l2 = (b.x - a.x)*(b.x - a.x) + (b.y - a.y)*(b.y - a.y);
        if (l2 === 0) return Math.hypot(p.x - a.x, p.y - a.y);
        let t = ((p.x - a.x)*(b.x - a.x) + (p.y - a.y)*(b.y - a.y)) / l2;
        t = Math.max(0, Math.min(1, t));
        return Math.hypot(p.x - (a.x + t*(b.x - a.x)), p.y - (a.y + t*(b.y - a.y)));
    }

    getConnectionPoint(node, otherNode, offset = 0) {
        let p = { x: node.x, y: node.y };
        if (node.type === 'bus') {
            let w = node.width || 140; 
            let isH = node.orientation !== 'V'; 
            if (isH) {
                p.x = Math.max(node.x - w/2 + 8, Math.min(node.x + w/2 - 8, otherNode.x));
                p.x += offset; 
                p.x = Math.max(node.x - w/2 + 4, Math.min(node.x + w/2 - 4, p.x));
            } else {
                p.y = Math.max(node.y - w/2 + 8, Math.min(node.y + w/2 - 8, otherNode.y));
                p.y += offset;
                p.y = Math.max(node.y - w/2 + 4, Math.min(node.y + w/2 - 4, p.y));
            }
        }
        return p;
    }

    onPointerDown(e) {
        const rect = this.canvas.getBoundingClientRect();
        const cx = e.clientX - rect.left;
        const cy = e.clientY - rect.top;
        const wx = (cx - this.pan.x) / this.zoom;
        const wy = (cy - this.pan.y) / this.zoom;

        this.mousePos = { x: wx, y: wy };

        if (e.button === 1 || (e.button === 0 && e.shiftKey)) {
            this.isPanning = true;
            this.dragStart = { x: cx, y: cy };
            return;
        }

        const clickedNode = this.findNodeAt(wx, wy);

        if (this.mode === 'select') {
            if (clickedNode) {
                this.selectedElement = clickedNode;
                this.selectedWire = null;
                this.isDraggingNode = true;
                this.dragStart = { x: wx - clickedNode.x, y: wy - clickedNode.y };
            } else {
                const clickedWire = this.findWireAt(wx, wy);
                if (clickedWire) {
                    this.selectedWire = clickedWire;
                    this.selectedElement = null;
                } else {
                    this.selectedElement = null;
                    this.selectedWire = null;
                }
            }
            this.updateInspector();
        } else if (this.mode === 'wire') {
            if (clickedNode) {
                if (!this.wireStartNode) {
                    this.wireStartNode = clickedNode;
                } else if (this.wireStartNode.id !== clickedNode.id) {
                    if (this.wireStartNode.type === 'bus' && clickedNode.type === 'bus') {
                        if (this.wireStartNode.voltage !== clickedNode.voltage) {
                            const midX = (this.wireStartNode.x + clickedNode.x) / 2;
                            const midY = (this.wireStartNode.y + clickedNode.y) / 2;
                            const xfrId = 'el_' + Math.random().toString(36).substr(2, 6);
                            this.model.nodes.push({
                                id: xfrId, type: 'transformer', name: `TR_${this.wireStartNode.voltage}_${clickedNode.voltage}`,
                                x: midX, y: midY, reactance: 0.05, ratingMVA: 100, status: 'closed', orientation: 'V'
                            });
                            
                            this.model.wires.push({ id: 'w_' + Math.random().toString(36).substr(2,6), from: this.wireStartNode.id, to: xfrId, reactance: 0.01, limit: 100, status: 'closed' });
                            this.model.wires.push({ id: 'w_' + Math.random().toString(3
