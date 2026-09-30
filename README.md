<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Advanced SLD Market Network Simulator</title>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; user-select: none; -webkit-user-select: none; }
        body, html { width: 100%; height: 100%; overflow: hidden; background: #0b0f19; font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif; color: #f3f4f6; }
        
        #app-container { display: flex; width: 100vw; height: 100vh; height: 100dvh; flex-direction: column; overflow: hidden; position: fixed; inset: 0; }
        
        /* Top Header */
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

        /* Main Workspace */
        .workspace { display: flex; flex: 1; width: 100%; height: calc(100% - 48px); position: relative; overflow: hidden; }
        
        /* Left Panel (Palette & Telemetry) */
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

        /* Canvas Container */
        .canvas-container { flex: 1; background: #0b0f19; position: relative; overflow: hidden; cursor: default; touch-action: none; }
        canvas { display: block; width: 100%; height: 100%; }

        /* Right Panel (Inspector) */
        .sidebar-right { width: 280px; background: #111827; border-left: 1px solid #1f2937; display: flex; flex-direction: column; flex-shrink: 0; z-index: 5; }
        .inspector-content { padding: 12px; overflow-y: auto; flex: 1; display: flex; flex-direction: column; gap: 12px; }
        .form-group { display: flex; flex-direction: column; gap: 4px; }
        .form-group label { font-size: 11px; color: #9ca3af; font-weight: 500; }
        .form-control { background: #1f2937; border: 1px solid #374151; color: #f3f4f6; padding: 6px 10px; border-radius: 4px; font-size: 12px; width: 100%; }
        .form-control:focus { outline: none; border-color: #3b82f6; background: #111827; }
        select.form-control { cursor: pointer; }

        /* Floating Tooltip */
        #tooltip { position: absolute; background: rgba(17, 24, 39, 0.95); border: 1px solid #374151; padding: 8px 12px; border-radius: 6px; font-size: 11px; pointer-events: none; display: none; z-index: 100; box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.5); backdrop-filter: blur(4px); }
        #tooltip .title { font-weight: 700; color: #60a5fa; margin-bottom: 4px; border-bottom: 1px solid #374151; padding-bottom: 2px; }
        #tooltip .row { display: flex; justify-content: space-between; gap: 16px; color: #d1d5db; }
        #tooltip .val { font-family: monospace; color: #34d399; }

        /* Modal */
        .modal-overlay { position: fixed; inset: 0; background: rgba(0, 0, 0, 0.7); backdrop-filter: blur(2px); display: none; align-items: center; justify-content: center; z-index: 1000; }
        .modal { background: #111827; border: 1px solid #374151; width: 500px; max-width: 90vw; border-radius: 8px; display: flex; flex-direction: column; overflow: hidden; box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5); }
        .modal-header { padding: 12px 16px; background: #1f2937; font-weight: 600; font-size: 13px; display: flex; justify-content: space-between; align-items: center; }
        .modal-body { padding: 16px; display: flex; flex-direction: column; gap: 12px; }
        .modal-footer { padding: 12px 16px; background: #0f172a; border-top: 1px solid #1f2937; display: flex; justify-content: flex-end; gap: 8px; }
        textarea.form-control { resize: vertical; min-height: 150px; font-family: monospace; font-size: 11px; }

        /* Canvas HUD Overlay */
        .canvas-hud { position: absolute; bottom: 16px; left: 16px; background: rgba(17, 24, 39, 0.85); border: 1px solid #374151; padding: 8px 12px; border-radius: 6px; font-size: 11px; pointer-events: none; display: flex; gap: 16px; backdrop-filter: blur(4px); }
        .hud-item { display: flex; align-items: center; gap: 6px; }
        .hud-dot { width: 8px; height: 8px; border-radius: 50%; }
    </style>
</head>
<body>
<div id="app-container">
    <!-- Header -->
    <header>
        <div class="logo-area">
            <span>⚡ RTO SLD MARKET NETWORK SIMULATOR</span>
            <span class="logo-badge">ULTIMATE v3.5</span>
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
            <button class="btn btn-success" id="btn-export" title="Export Model JSON">Export JSON</button>
            <button class="btn" id="btn-import" title="Import Model JSON">Import JSON</button>
            <button class="btn btn-warning" id="btn-clear" title="Clear Canvas">Clear Grid</button>
        </div>
    </header>

    <!-- Workspace -->
    <div class="workspace">
        <!-- Left Sidebar: Palette & Telemetry -->
        <div class="sidebar-left">
            <div class="panel-section-title">Grid Elements Palette (Tap/Click to Add)</div>
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

        <!-- Canvas Area -->
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

        <!-- Right Sidebar: Inspector -->
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

<!-- Tooltip -->
<div id="tooltip">
    <div class="title" id="tt-title">Element</div>
    <div class="row"><span>Type:</span> <span class="val" id="tt-type">-</span></div>
    <div class="row"><span>Active Flow:</span> <span class="val" id="tt-flow">-</span></div>
    <div class="row"><span>Status:</span> <span class="val" id="tt-status">-</span></div>
</div>

<!-- Hidden File Input for Import -->
<input type="file" id="import-file" style="display: none;" accept=".json">

<!-- Export/Import Modal -->
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
/**
 * RTO Market Network Simulator - Ultimate Edition v3.5
 */
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
        this.loadSample();
    }

    loadSample() {
        this.nodes = [
            { id: "b1", type: "bus", name: "CORELLA 230kV", x: 250, y: 150, width: 180, height: 16, voltage: "230 kV", orientation: "H" },
            { id: "b2", type: "bus", name: "UBAY 138kV", x: 550, y: 150, width: 160, height: 16, voltage: "138 kV", orientation: "H" },
            { id: "b3", type: "bus", name: "TAPAL 69kV", x: 550, y: 320, width: 160, height: 16, voltage: "69 kV", orientation: "H" },
            
            { id: "g1", type: "generator", name: "07LOBOC_G01", x: 180, y: 260, mw: 45, maxMw: 50, cost: 24.5, status: "closed" },
            { id: "g2", type: "generator", name: "07TAPLPB4", x: 500, y: 440, mw: 30, maxMw: 40, cost: 32.0, status: "closed" },
            
            { id: "l1", type: "load", name: "MARIBOJOC LOAD", x: 320, y: 260, mw: 35, status: "closed" },
            { id: "l2", type: "load", name: "UBAY INDUSTRIAL", x: 620, y: 440, mw: 25, status: "closed" },
            
            { id: "cb1", type: "breaker", name: "CB-CORELLA-1", x: 250, y: 210, status: "closed", orientation: "V" },
            { id: "tr1", type: "transformer", name: "TR_138_69", x: 550, y: 235, reactance: 0.08, ratingMVA: 100, status: "closed", orientation: "V" }
        ];

        this.wires = [
            { id: "w1", from: "g1", to: "cb1", name: "Gen Lead 1", reactance: 0.02, limit: 60, status: "closed" },
            { id: "w2", from: "cb1", to: "b1", name: "Bus Drop 1", reactance: 0.01, limit: 100, status: "closed" },
            { id: "w3", from: "b1", to: "b2", name: "TL-CORELLA-UBAY", reactance: 0.05, rating: 120, limit: 100, status: "closed" },
            { id: "w4", from: "b2", to: "tr1", name: "XFR Tap", reactance: 0.01, rating: 100, limit: 100, status: "closed" },
            { id: "w5", from: "tr1", to: "b3", name: "Secondary Tap", reactance: 0.01, rating: 100, limit: 100, status: "closed" },
            { id: "w6", from: "b3", to: "g2", name: "Tapal Gen Feed", reactance: 0.02, limit: 50, status: "closed" },
            { id: "w7", from: "b1", to: "l1", name: "Load Feeder 1", reactance: 0.02, limit: 60, status: "closed" },
            { id: "w8", from: "b3", to: "l2", name: "Load Feeder 2", reactance: 0.02, limit: 50, status: "closed" }
        ];
    }

    exportJSON() {
        return JSON.stringify({ nodes: this.nodes, wires: this.wires }, null, 2);
    }

    importJSON(jsonStr) {
        try {
            const data = JSON.parse(jsonStr);
            if (data.nodes && data.wires) {
                this.nodes = data.nodes;
                this.wires = data.wires;
                return true;
            }
        } catch (e) {
            console.error(e);
        }
        return false;
    }
}

class SLDApp {
    constructor() {
        this.model = new NetworkModel();
        this.canvas = document.getElementById('sld-canvas');
        this.ctx = this.canvas.getContext('2d');
        
        this.mode = 'select'; // 'select' | 'wire'
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

        // Palette Item Click / Tap & Drag support
        document.querySelectorAll('.palette-item').forEach(item => {
            const type = item.dataset.type;

            // Direct tap/click support for touch screens & Safari where HTML5 drag/drop fails
            item.addEventListener('pointerdown', e => {
                e.preventDefault();
                const wx = (this.canvas.width / 2 - this.pan.x) / this.zoom;
                const wy = (this.canvas.height / 2 - this.pan.y) / this.zoom;
                this.addElement(type, wx, wy);
            });

            // Desktop HTML5 drag start
            item.addEventListener('dragstart', e => {
                e.dataTransfer.setData('text/plain', type);
            });
        });

        container.addEventListener('dragover', e => e.preventDefault());
        container.addEventListener('drop', e => {
            e.preventDefault();
            const type = e.dataTransfer.getData('text/plain');
            if (!type) return;
            const rect = this.canvas.getBoundingClientRect();
            const wx = (e.clientX - rect.left - this.pan.x) / this.zoom;
            const wy = (e.clientY - rect.top - this.pan.y) / this.zoom;
            this.addElement(type, wx, wy);
        });

        // Top toolbar buttons
        document.getElementById('btn-select').addEventListener('click', () => this.setMode('select'));
        document.getElementById('btn-wire').addEventListener('click', () => this.setMode('wire'));
        document.getElementById('btn-export').addEventListener('click', () => this.openModal('export'));
        document.getElementById('btn-import').addEventListener('click', () => document.getElementById('import-file').click());
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
                    alert("Model imported successfully!");
                    this.selectedElement = null;
                    this.selectedWire = null;
                    this.updateInspector();
                } else {
                    alert("Invalid JSON file format.");
                }
            };
            reader.readAsText(file);
            e.target.value = '';
        });

        // Modal triggers
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
                this.updateInspector();
                alert("Model successfully loaded!");
            } else {
                alert("Invalid JSON data.");
            }
        });

        // Keyboard shortcuts
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
                } else {
                    this.selectedElement.orientation = this.selectedElement.orientation === 'H' ? 'V' : 'H';
                }
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
        
        if (type === 'bus') {
            newNode.width = 140;
            newNode.height = 16;
            newNode.voltage = '230 kV';
            newNode.orientation = 'H';
        } else if (type === 'generator') {
            newNode.mw = 25;
            newNode.maxMw = 50;
            newNode.cost = 30;
        } else if (type === 'load') {
            newNode.mw = 20;
        } else if (type === 'transformer') {
            newNode.reactance = 0.05;
            newNode.ratingMVA = 100;
            newNode.orientation = 'V';
        } else if (type === 'breaker') {
            newNode.orientation = 'V';
        }

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

            if (wx >= n.x - w/2 && wx <= n.x + w/2 && wy >= n.y - h/2 && wy <= n.y + h/2) {
                return n;
            }
        }
        return null;
    }

    findWireAt(wx, wy) {
        for (const w of this.model.wires) {
            const n1 = this.model.nodes.find(n => n.id === w.from);
            const n2 = this.model.nodes.find(n => n.id === w.to);
            if (!n1 || !n2) continue;
            
            const dist = this.distToSegment({x: wx, y: wy}, {x: n1.x, y: n1.y}, {x: n2.x, y: n2.y});
            if (dist < 8) return w;
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
                            this.model.wires.push({ id: 'w_' + Math.random().toString(36).substr(2,6), from: xfrId, to: clickedNode.id, reactance: 0.01, limit: 100, status: 'closed' });
                            this.wireStartNode = null;
                            return;
                        }
                    }

                    const wireId = 'w_' + Math.random().toString(36).substr(2, 6);
                    this.model.wires.push({
                        id: wireId,
                        from: this.wireStartNode.id,
                        to: clickedNode.id,
                        name: `TL_${this.model.wires.length + 1}`,
                        reactance: 0.03,
                        limit: 100,
                        status: 'closed'
                    });
                    this.wireStartNode = null;
                }
            }
        }
    }

    onPointerMove(e) {
        const rect = this.canvas.getBoundingClientRect();
        const cx = e.clientX - rect.left;
        const cy = e.clientY - rect.top;
        const wx = (cx - this.pan.x) / this.zoom;
        const wy = (cy - this.pan.y) / this.zoom;
        this.mousePos = { x: wx, y: wy };

        if (this.isPanning) {
            this.pan.x += cx - this.dragStart.x;
            this.pan.y += cy - this.dragStart.y;
            this.dragStart = { x: cx, y: cy };
            return;
        }

        if (this.isDraggingNode && this.selectedElement) {
            this.selectedElement.x = wx - this.dragStart.x;
            this.selectedElement.y = wy - this.dragStart.y;
        }

        const hoveredNode = this.findNodeAt(wx, wy);
        const hoveredWire = !hoveredNode ? this.findWireAt(wx, wy) : null;
        const tt = document.getElementById('tooltip');
        if (hoveredNode) {
            document.getElementById('tt-title').innerText = hoveredNode.name;
            document.getElementById('tt-type').innerText = hoveredNode.type.toUpperCase();
            document.getElementById('tt-flow').innerText = hoveredNode.mw ? `${hoveredNode.mw.toFixed(1)} MW` : 'N/A';
            document.getElementById('tt-status').innerText = hoveredNode.status || 'closed';
            tt.style.display = 'block';
            tt.style.left = (cx + 15) + 'px';
            tt.style.top = (cy + 15) + 'px';
        } else if (hoveredWire) {
            document.getElementById('tt-title').innerText = hoveredWire.name;
            document.getElementById('tt-type').innerText = 'TRANSMISSION LINE';
            document.getElementById('tt-flow').innerText = `${(hoveredWire.flow || 0).toFixed(1)} MW`;
            document.getElementById('tt-status').innerText = `${hoveredWire.status} (${((hoveredWire.loading || 0)).toFixed(0)}% cap)`;
            tt.style.display = 'block';
            tt.style.left = (cx + 15) + 'px';
            tt.style.top = (cy + 15) + 'px';
        } else {
            tt.style.display = 'none';
        }
    }

    onPointerUp(e) {
        this.isPanning = false;
        this.isDraggingNode = false;
    }

    onWheel(e) {
        e.preventDefault();
        const zoomFactor = 1.1;
        const oldZoom = this.zoom;
        if (e.deltaY < 0) {
            this.zoom *= zoomFactor;
        } else {
            this.zoom /= zoomFactor;
        }
        this.zoom = Math.max(0.3, Math.min(3.0, this.zoom));

        const rect = this.canvas.getBoundingClientRect();
        const cx = e.clientX - rect.left;
        const cy = e.clientY - rect.top;

        this.pan.x = cx - (cx - this.pan.x) * (this.zoom / oldZoom);
        this.pan.y = cy - (cy - this.pan.y) * (this.zoom / oldZoom);
    }

    openModal(mode) {
        const modal = document.getElementById('json-modal');
        const title = document.getElementById('modal-title');
        const textarea = document.getElementById('json-textarea');
        const actionBtn = document.getElementById('modal-action');

        if (mode === 'export') {
            title.innerText = 'Export Model JSON';
            textarea.value = this.model.exportJSON();
            textarea.readOnly = true;
            actionBtn.style.display = 'none';
            document.getElementById('modal-copy').style.display = 'block';
        }
        modal.style.display = 'flex';
    }

    runPowerFlow() {
        let totalGen = 0;
        let totalLoad = 0;

        const generators = this.model.nodes.filter(n => n.type === 'generator' && n.status === 'closed');
        const loads = this.model.nodes.filter(n => n.type === 'load' && n.status === 'closed');

        generators.forEach(g => totalGen += (g.mw || 0));
        loads.forEach(l => totalLoad += (l.mw || 0));

        const balance = totalGen - totalLoad;
        document.getElementById('sys-gen').innerText = totalGen.toFixed(1) + ' MW';
        document.getElementById('sys-load').innerText = totalLoad.toFixed(1) + ' MW';
        document.getElementById('sys-balance').innerText = (balance >= 0 ? '+' : '') + balance.toFixed(1) + ' MW';

        const deficitRow = document.getElementById('deficit-row');
        if (totalGen < totalLoad) {
            deficitRow.style.display = 'flex';
            document.getElementById('sys-deficit').innerText = `DEFICIT: ${(totalLoad - totalGen).toFixed(1)} MW`;
        } else {
            deficitRow.style.display = 'none';
        }

        this.model.wires.forEach((w, idx) => {
            if (w.status === 'open') {
                w.flow = 0;
                w.loading = 0;
                return;
            }
            const seed = (idx + 1) * 7;
            const flowMag = ((totalLoad / Math.max(1, this.model.wires.length)) * 0.8) + (Math.sin(this.animPhase * 0.05 + seed) * 3);
            w.flow = Math.abs(flowMag);
            const limit = w.limit || 100;
            w.loading = (w.flow / limit) * 100;
        });
    }

    updateInspector() {
        const container = document.getElementById('inspector-content');
        if (!this.selectedElement && !this.selectedWire) {
            container.innerHTML = `<div style="color: #6b7280; font-size: 11px; text-align: center; margin-top: 40px;">Select any node, wire, or equipment to inspect and configure parameters.</div>`;
            return;
        }

        let html = '';
        if (this.selectedElement) {
            const el = this.selectedElement;
            html += `
                <div class="form-group"><label>Identifier Name</label><input type="text" class="form-control" id="insp-name" value="${el.name}"></div>
                <div class="form-group"><label>Element Type</label><input type="text" class="form-control" value="${el.type.toUpperCase()}" disabled style="color:#9ca3af;"></div>
                <div class="form-group"><label>Status</label>
                    <select class="form-control" id="insp-status">
                        <option value="closed" ${el.status === 'closed' ? 'selected' : ''}>Closed (Energized)</option>
                        <option value="open" ${el.status === 'open' ? 'selected' : ''}>Open (Tripped)</option>
                    </select>
                </div>
            `;

            if (el.type === 'bus') {
                html += `
                    <div class="form-group"><label>Voltage Level</label>
                        <select class="form-control" id="insp-voltage">
                            <option value="500 kV" ${el.voltage === '500 kV' ? 'selected' : ''}>500 kV</option>
                            <option value="230 kV" ${el.voltage === '230 kV' ? 'selected' : ''}>230 kV</option>
                            <option value="138 kV" ${el.voltage === '138 kV' ? 'selected' : ''}>138 kV</option>
                            <option value="69 kV" ${el.voltage === '69 kV' ? 'selected' : ''}>69 kV</option>
                            <option value="34.5 kV" ${el.voltage === '34.5 kV' ? 'selected' : ''}>34.5 kV</option>
                            <option value="13.8 kV" ${el.voltage === '13.8 kV' ? 'selected' : ''}>13.8 kV</option>
                        </select>
                    </div>
                    <div class="form-group"><label>Bus Width (px)</label><input type="number" class="form-control" id="insp-width" value="${el.width || 140}"></div>
                `;
            } else if (el.type === 'generator') {
                html += `
                    <div class="form-group"><label>Active Generation (MW)</label><input type="number" class="form-control" id="insp-mw" value="${el.mw || 0}"></div>
                    <div class="form-group"><label>Max Capacity (MW)</label><input type="number" class="form-control" id="insp-maxmw" value="${el.maxMw || 50}"></div>
                    <div class="form-group"><label>Offer Price ($/MWh)</label><input type="number" class="form-control" id="insp-cost" value="${el.cost || 30}"></div>
                `;
            } else if (el.type === 'load') {
                html += `
                    <div class="form-group"><label>Active Demand (MW)</label><input type="number" class="form-control" id="insp-loadmw" value="${el.mw || 0}"></div>
                `;
            } else if (el.type === 'transformer') {
                html += `
                    <div class="form-group"><label>Reactance (X pu)</label><input type="number" step="0.01" class="form-control" id="insp-x" value="${el.reactance || 0.05}"></div>
                    <div class="form-group"><label>Rating (MVA)</label><input type="number" class="form-control" id="insp-mva" value="${el.ratingMVA || 100}"></div>
                `;
            }

            html += `
                <div style="display: flex; gap: 8px; margin-top: 12px;">
                    <button class="btn" style="flex: 1;" id="btn-duplicate">Duplicate</button>
                    <button class="btn btn-warning" style="flex: 1;" id="btn-delete">Delete</button>
                </div>
            `;
        } else if (this.selectedWire) {
            const w = this.selectedWire;
            html += `
                <div class="form-group"><label>Line Name</label><input type="text" class="form-control" id="wire-name" value="${w.name}"></div>
                <div class="form-group"><label>Status</label>
                    <select class="form-control" id="wire-status">
                        <option value="closed" ${w.status === 'closed' ? 'selected' : ''}>In Service</option>
                        <option value="open" ${w.status === 'open' ? 'selected' : ''}>Out of Service</option>
                    </select>
                </div>
                <div class="form-group"><label>Reactance (X pu)</label><input type="number" step="0.01" class="form-control" id="wire-x" value="${w.reactance || 0.03}"></div>
                <div class="form-group"><label>Thermal Limit (MW)</label><input type="number" class="form-control" id="wire-limit" value="${w.limit || 100}"></div>
                <div style="margin-top: 12px;">
                    <button class="btn btn-warning" style="width: 100%;" id="btn-delete-wire">Delete Line</button>
                </div>
            `;
        }

        container.innerHTML = html;

        const bindInput = (id, prop, isFloat = false) => {
            const elInput = document.getElementById(id);
            if (elInput) {
                elInput.addEventListener('input', e => {
                    if (this.selectedElement) {
                        this.selectedElement[prop] = isFloat ? parseFloat(e.target.value) || 0 : e.target.value;
                    } else if (this.selectedWire) {
                        this.selectedWire[prop] = isFloat ? parseFloat(e.target.value) || 0 : e.target.value;
                    }
                    this.runPowerFlow();
                });
            }
        };

        bindInput('insp-name', 'name');
        bindInput('insp-status', 'status');
        bindInput('insp-voltage', 'voltage');
        bindInput('insp-width', 'width', true);
        bindInput('insp-mw', 'mw', true);
        bindInput('insp-maxmw', 'maxMw', true);
        bindInput('insp-cost', 'cost', true);
        bindInput('insp-loadmw', 'mw', true);
        bindInput('insp-x', 'reactance', true);
        bindInput('insp-mva', 'ratingMVA', true);
        bindInput('wire-name', 'name');
        bindInput('wire-status', 'status');
        bindInput('wire-x', 'reactance', true);
        bindInput('wire-limit', 'limit', true);

        const delBtn = document.getElementById('btn-delete');
        if (delBtn) delBtn.addEventListener('click', () => this.deleteNode(this.selectedElement));

        const delWireBtn = document.getElementById('btn-delete-wire');
        if (delWireBtn) delWireBtn.addEventListener('click', () => this.deleteWire(this.selectedWire));

        const dupBtn = document.getElementById('btn-duplicate');
        if (dupBtn) dupBtn.addEventListener('click', () => {
            const clone = JSON.parse(JSON.stringify(this.selectedElement));
            clone.id = 'el_' + Math.random().toString(36).substr(2, 6);
            clone.name += '_Copy';
            clone.x += 30;
            clone.y += 30;
            this.model.nodes.push(clone);
            this.selectedElement = clone;
            this.updateInspector();
        });
    }

    loop() {
        this.animPhase++;
        this.runPowerFlow();
        this.render();
        requestAnimationFrame(() => this.loop());
    }

    render() {
        const ctx = this.ctx;
        ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

        ctx.save();
        ctx.translate(this.pan.x, this.pan.y);
        ctx.scale(this.zoom, this.zoom);

        this.drawGridBackground();

        this.model.wires.forEach(w => {
            const n1 = this.model.nodes.find(n => n.id === w.from);
            const n2 = this.model.nodes.find(n => n.id === w.to);
            if (!n1 || !n2) return;

            const isSelected = this.selectedWire && this.selectedWire.id === w.id;
            const isOverloaded = (w.loading || 0) > 100;
            
            ctx.beginPath();
            ctx.moveTo(n1.x, n1.y);
            const midX = (n1.x + n2.x) / 2;
            ctx.lineTo(midX, n1.y);
            ctx.lineTo(midX, n2.y);
            ctx.lineTo(n2.x, n2.y);

            ctx.lineWidth = isSelected ? 4 : 2;
            ctx.strokeStyle = w.status === 'open' ? '#4b5563' : (isOverloaded ? '#f87171' : '#60a5fa');
            if (w.status === 'open') ctx.setLineDash([4, 4]);
            else ctx.setLineDash([]);
            ctx.stroke();
            ctx.setLineDash([]);

            if (w.status === 'closed' && (w.flow || 0) > 0) {
                const particleCount = 3;
                for (let i = 0; i < particleCount; i++) {
                    const t = ((this.animPhase * 0.015 + i / particleCount) % 1);
                    let px, py;
                    if (t < 0.5) {
                        const st = t * 2;
                        px = n1.x + (midX - n1.x) * st;
                        py = n1.y;
                    } else if (t < 0.75) {
                        const st = (t - 0.5) * 4;
                        px = midX;
                        py = n1.y + (n2.y - n1.y) * st;
                    } else {
                        const st = (t - 0.75) * 4;
                        px = midX + (n2.x - midX) * st;
                        py = n2.y;
                    }

                    ctx.fillStyle = isOverloaded ? '#f87171' : '#34d399';
                    ctx.beginPath();
                    ctx.arc(px, py, 3, 0, Math.PI * 2);
                    ctx.fill();
                }
            }

            const labelX = midX;
            const labelY = (n1.y + n2.y) / 2;
            ctx.fillStyle = 'rgba(17, 24, 39, 0.85)';
            ctx.strokeStyle = isOverloaded ? '#f87171' : '#374151';
            ctx.lineWidth = 1;
            ctx.fillRect(labelX - 25, labelY - 10, 50, 20);
            ctx.strokeRect(labelX - 25, labelY - 10, 50, 20);

            ctx.fillStyle = isOverloaded ? '#f87171' : '#34d399';
            ctx.font = '10px monospace';
            ctx.textAlign = 'center';
            ctx.textBaseline = 'middle';
            ctx.fillText(`${(w.flow || 0).toFixed(1)}M`, labelX, labelY);
        });

        if (this.mode === 'wire' && this.wireStartNode) {
            ctx.beginPath();
            ctx.moveTo(this.wireStartNode.x, this.wireStartNode.y);
            ctx.lineTo(this.mousePos.x, this.mousePos.y);
            ctx.strokeStyle = '#f59e0b';
            ctx.lineWidth = 2;
            ctx.setLineDash([4, 4]);
            ctx.stroke();
            ctx.setLineDash([]);
        }

        this.model.nodes.forEach(n => {
            const isSelected = this.selectedElement && this.selectedElement.id === n.id;
            ctx.save();
            ctx.translate(n.x, n.y);

            if (n.type === 'bus') {
                const w = n.width || 140;
                const h = n.height || 16;
                const col = VoltageColors[n.voltage] || '#ef4444';
                
                ctx.fillStyle = col;
                ctx.fillRect(-w/2, -h/2, w, h);
                if (isSelected) {
                    ctx.strokeStyle = '#ffffff';
                    ctx.lineWidth = 2;
                    ctx.strokeRect(-w/2 - 2, -h/2 - 2, w + 4, h + 4);
                }

                ctx.fillStyle = '#f3f4f6';
                ctx.font = 'bold 11px sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText(n.name, 0, -h/2 - 8);
                ctx.font = '9px sans-serif';
                ctx.fillStyle = '#9ca3af';
                ctx.fillText(n.voltage, 0, h/2 + 12);

            } else if (n.type === 'generator') {
                ctx.beginPath();
                ctx.arc(0, 0, 20, 0, Math.PI * 2);
                ctx.fillStyle = '#111827';
                ctx.fill();
                ctx.strokeStyle = isSelected ? '#ffffff' : '#34d399';
                ctx.lineWidth = isSelected ? 3 : 2;
                ctx.stroke();

                ctx.fillStyle = '#34d399';
                ctx.font = 'bold 12px sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('G', 0, 0);

                ctx.fillStyle = '#f3f4f6';
                ctx.font = '10px sans-serif';
                ctx.fillText(n.name, 0, 28);
                ctx.fillText(`${n.mw || 0} MW`, 0, 40);

            } else if (n.type === 'load') {
                ctx.fillStyle = '#111827';
                ctx.beginPath();
                ctx.moveTo(0, 18);
                ctx.lineTo(-18, -14);
                ctx.lineTo(18, -14);
                ctx.closePath();
                ctx.fill();
                ctx.strokeStyle = isSelected ? '#ffffff' : '#f59e0b';
                ctx.lineWidth = isSelected ? 3 : 2;
                ctx.stroke();

                ctx.fillStyle = '#f59e0b';
                ctx.font = 'bold 10px sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('L', 0, 2);

                ctx.fillStyle = '#f3f4f6';
                ctx.font = '10px sans-serif';
                ctx.fillText(n.name, 0, 28);
                ctx.fillText(`${n.mw || 0} MW`, 0, 40);

            } else if (n.type === 'transformer') {
                ctx.fillStyle = '#111827';
                ctx.beginPath();
                ctx.arc(0, -8, 12, 0, Math.PI * 2);
                ctx.arc(0, 8, 12, 0, Math.PI * 2);
                ctx.fill();
                ctx.strokeStyle = isSelected ? '#ffffff' : '#60a5fa';
                ctx.lineWidth = isSelected ? 3 : 2;
                ctx.stroke();

                ctx.fillStyle = '#f3f4f6';
                ctx.font = '10px sans-serif';
                ctx.textAlign = 'center';
                ctx.fillText(n.name, 0, 28);

            } else if (n.type === 'breaker') {
                ctx.fillStyle = n.status === 'open' ? '#7f1d1d' : '#111827';
                ctx.fillRect(-12, -12, 24, 24);
                ctx.strokeStyle = isSelected ? '#ffffff' : '#ef4444';
                ctx.lineWidth = isSelected ? 3 : 2;
                ctx.strokeRect(-12, -12, 24, 24);

                ctx.fillStyle = '#ef4444';
                ctx.font = 'bold 9px sans-serif';
                ctx.textAlign = 'center';
                ctx.textBaseline = 'middle';
                ctx.fillText('CB', 0, 0);

                ctx.fillStyle = '#f3f4f6';
                ctx.font = '10px sans-serif';
                ctx.fillText(n.name, 0, 24);
            }

            ctx.restore();
        });

        ctx.restore();
    }

    drawGridBackground() {
        const ctx = this.ctx;
        const gridSize = 40 * this.zoom;
        const offsetX = this.pan.x % gridSize;
        const offsetY = this.pan.y % gridSize;

        ctx.strokeStyle = '#1f2937';
        ctx.lineWidth = 1;

        for (let x = offsetX; x < this.canvas.width; x += gridSize) {
            ctx.beginPath();
            ctx.moveTo(x, 0);
            ctx.lineTo(x, this.canvas.height);
            ctx.stroke();
        }
        for (let y = offsetY; y < this.canvas.height; y += gridSize) {
            ctx.beginPath();
            ctx.moveTo(0, y);
            ctx.lineTo(this.canvas.width, y);
            ctx.stroke();
        }
    }
}

window.addEventListener('DOMContentLoaded', () => {
    window.app = new SLDApp();
});
</script>
</body>
</html>
