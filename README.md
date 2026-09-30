<html lang="en" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
  <title>SLD Market Simulator</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            panel: '#0f172a',
            canvas: '#020617',
            kv500: '#0000FF',
            kv230: '#ED3237',
            kv138: '#F58220',
            kv69:  '#66FFFF',
            kv40:  '#66CC33',
            kv7:   '#993399'
          }
        }
      }
    }
  </script>
  <style>
    @keyframes dashFlow { from { stroke-dashoffset: 20; } to { stroke-dashoffset: 0; } }
    @keyframes pulseWarning {
      0%, 100% { stroke: #ef4444; filter: drop-shadow(0 0 6px #ef4444); }
      50% { stroke: #f87171; filter: drop-shadow(0 0 12px #ef4444); }
    }
    @keyframes toastFadeInOut {
      0% { opacity: 0; transform: translateY(-20px); }
      10% { opacity: 1; transform: translateY(0); }
      90% { opacity: 1; transform: translateY(0); }
      100% { opacity: 0; transform: translateY(-20px); }
    }
    .flow-line { stroke-dasharray: 8 8; animation: dashFlow 1s linear infinite; }
    .overload { animation: pulseWarning 1s ease-in-out infinite !important; stroke-width: 3px !important; }
    .canvas-bg { background-image: radial-gradient(circle, #334155 1px, transparent 1px); background-size: 30px 30px; }
    .toast { animation: toastFadeInOut 3s ease-in-out forwards; }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #0f172a; }
    ::-webkit-scrollbar-thumb { background: #334155; border-radius: 3px; }
    
    /* Resizer styling */
    .resizer {
      width: 4px;
      background: #1e293b;
      cursor: col-resize;
      transition: background 0.2s;
      z-index: 40;
    }
    .resizer:hover, .resizer:active { background: #6366f1; }
  </style>
</head>
<body class="h-screen w-screen overflow-hidden bg-canvas text-slate-200 font-sans flex flex-col select-none relative">

  <header class="h-10 border-b border-slate-700 bg-panel px-3 flex items-center justify-between z-30 shrink-0">
    <div class="flex items-center gap-2">
      <div class="w-6 h-6 rounded bg-indigo-600 flex items-center justify-center font-bold font-mono text-white text-[10px] shadow-[0_0_8px_rgba(79,70,229,0.5)]">SLD</div>
      <h1 class="font-bold text-xs tracking-wide hidden sm:block text-slate-300">MARKET SIMULATOR</h1>
    </div>

    <div class="flex items-center gap-1.5 text-xs">
      <button id="btnSelect" class="px-2.5 py-1 rounded bg-indigo-600 text-white font-medium transition flex items-center gap-1 shadow-lg" onclick="setMode('SELECT')">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 15l-2 5L9 9l11 4-5 2zm0 0l5 5M7.188 2.239l.777 2.897M5.136 7.965l-2.898-.777M13.95 4.05l-2.122 2.122m-5.657 5.656l-2.12 2.122"></path></svg>
        Select (V)
      </button>
      <button id="btnWire" class="px-2.5 py-1 rounded border border-slate-600 hover:bg-slate-700 text-slate-300 font-medium transition relative flex items-center gap-1" onclick="setMode('WIRE')">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"></path></svg>
        Wire (W)
        <span id="wireBadge" class="hidden absolute -top-1 -right-1 w-2 h-2 bg-red-500 rounded-full animate-ping"></span>
      </button>

      <div class="w-px h-4 bg-slate-600 mx-1"></div>

      <button class="px-2 py-1 rounded bg-emerald-600/80 hover:bg-emerald-500 text-white font-medium transition flex items-center gap-1" onclick="exportData()" title="Save Model to JSON File">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"></path></svg>
        Export
      </button>
      <button class="px-2 py-1 rounded bg-amber-600/80 hover:bg-amber-500 text-white font-medium transition flex items-center gap-1" onclick="document.getElementById('fileUpload').click()" title="Load Model from JSON File">
        <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l-4-4m0 0L8 8m4-4v12"></path></svg>
        Import
      </button>
      <input type="file" id="fileUpload" class="hidden" accept=".json" onchange="importData(event)">
      
      <div class="w-px h-4 bg-slate-600 mx-0.5"></div>
      <button class="px-2 py-1 rounded border border-red-900/50 text-red-400 hover:bg-red-900/50 transition" onclick="clearCanvas()" title="Clear Canvas">🗑️</button>
    </div>
  </header>

  <div class="flex-1 flex overflow-hidden relative">
    
    <!-- Left Sidebar: Palette -->
    <aside id="leftPanel" class="w-[200px] bg-panel flex flex-col z-20 shrink-0">
      <div class="p-2 border-b border-slate-700 font-bold text-[10px] text-slate-400 uppercase tracking-wider">Palette</div>
      <div class="p-2 grid grid-cols-2 gap-1.5 flex-1 content-start overflow-y-auto">
        <!-- Palette Buttons -->
        <button onclick="spawnComponent('BUS')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-1.5 transition group">
          <div class="w-10 h-1.5 bg-kv230 rounded-sm group-hover:scale-110 transition-transform"></div>
          <span class="text-[10px] font-medium">Busbar</span>
        </button>
        <button onclick="spawnComponent('GEN')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-1.5 transition group">
          <div class="w-6 h-6 rounded-full border-2 border-green-500 text-green-500 flex items-center justify-center text-[10px] font-bold group-hover:scale-110 transition-transform">G</div>
          <span class="text-[10px] font-medium">Generator</span>
        </button>
        <button onclick="spawnComponent('LOAD')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-1.5 transition group">
          <div class="w-0 h-0 border-l-[10px] border-r-[10px] border-t-[14px] border-l-transparent border-r-transparent border-t-amber-500 group-hover:scale-110 transition-transform"></div>
          <span class="text-[10px] font-medium">Load</span>
        </button>
        <button onclick="spawnComponent('LINE')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-1.5 transition group">
          <div class="w-10 h-0.5 bg-blue-400 relative group-hover:scale-110 transition-transform">
            <div class="absolute -top-1 left-1/2 w-2.5 h-2.5 bg-blue-400 rounded-sm transform -translate-x-1/2"></div>
          </div>
          <span class="text-[10px] font-medium">Trans. Line</span>
        </button>
        <button onclick="spawnComponent('XFMR')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-1.5 transition group">
          <div class="flex items-center group-hover:scale-110 transition-transform">
            <div class="w-5 h-5 rounded-full border-2 border-orange-400 -mr-2"></div>
            <div class="w-5 h-5 rounded-full border-2 border-orange-400"></div>
          </div>
          <span class="text-[10px] font-medium">Transformer</span>
        </button>
        <button onclick="spawnComponent('BREAKER')" class="p-2 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-1.5 transition group">
          <div class="w-5 h-5 border-2 border-red-500 bg-red-900 rounded flex items-center justify-center font-bold text-[8px] text-red-400 group-hover:scale-110 transition-transform">CB</div>
          <span class="text-[10px] font-medium">Breaker</span>
        </button>
      </div>
      
      <!-- Live Stats Panel -->
      <div class="p-3 border-t border-slate-700 bg-slate-900 text-[11px] font-mono shrink-0">
        <div class="font-bold text-slate-400 mb-1.5 uppercase">Live Stats</div>
        <div class="flex justify-between text-slate-300 mb-0.5"><span>Gen:</span> <span id="lblGen" class="text-green-400 font-bold">0.0 MW</span></div>
        <div class="flex justify-between text-slate-300 mb-0.5"><span>Load:</span> <span id="lblLoad" class="text-amber-400 font-bold">0.0 MW</span></div>
        <div class="flex justify-between text-slate-300"><span>Loss:</span> <span id="lblLoss" class="text-red-400 font-bold">0.0 MW</span></div>
      </div>
    </aside>

    <div class="resizer" id="resizerLeft"></div>

    <main id="viewportContainer" class="flex-1 relative canvas-bg overflow-hidden cursor-grab active:cursor-grabbing min-w-[200px]">
      <svg id="canvas" class="w-full h-full absolute inset-0 font-sans">
        <g id="transformGroup" transform="translate(0,0) scale(1)">
          <g id="layer-wires"></g>
          <g id="layer-flow"></g>
          <g id="layer-components"></g>
          <g id="layer-buses"></g>
          <g id="layer-snap-hints"></g>
          <g id="layer-live-labels"></g> 
          <!-- Active wiring line -->
          <line id="activeWire" x1="0" y1="0" x2="0" y2="0" stroke="#f43f5e" stroke-width="2.5" stroke-dasharray="6 4" class="hidden pointer-events-none drop-shadow-[0_0_5px_#f43f5e]" />
          <circle id="snapIndicator" cx="0" cy="0" r="8" fill="none" stroke="#22c55e" stroke-width="2" class="hidden pointer-events-none drop-shadow-[0_0_5px_#22c55e]" />
        </g>
      </svg>
      <!-- HTML Overlay Tooltip -->
      <div id="tooltip" class="absolute hidden bg-slate-800 border border-slate-600 rounded shadow-xl p-2.5 text-xs pointer-events-none z-50 w-44 text-slate-300 transition-opacity duration-150"></div>
    </main>

    <div class="resizer" id="resizerRight"></div>

    <aside id="rightPanel" class="w-[240px] bg-panel flex flex-col z-20 shrink-0">
      <div class="p-2 border-b border-slate-700 font-bold text-[10px] text-slate-400 uppercase tracking-wider flex justify-between items-center">
        <span>Inspector</span>
        <span id="insType" class="text-indigo-400 bg-indigo-900/30 px-1.5 py-0.5 rounded">NONE</span>
      </div>
      
      <div id="inspectorPanel" class="p-3 overflow-y-auto flex-1 text-[11px] font-mono">
        <!-- Empty State -->
        <div id="inspector-none" class="text-center py-10 text-slate-500 font-sans text-xs">
          Select an element to inspect and edit.<br><br><b>R</b> to rotate.<br><b>Del</b> to remove.
        </div>

        <!-- Stable Form State (Never destroyed, preserves focus) -->
        <div id="inspector-form" class="hidden flex flex-col gap-3">
          
          <div id="wrap-name">
            <label class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Identifier Name</label>
            <input id="prop-name" type="text" oninput="updateParam('name', this.value)" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
          </div>

          <div id="wrap-kv" class="hidden">
            <label class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Voltage Level</label>
            <select id="prop-kv" onchange="updateParam('kv', this.value)" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
              <option value="500">500 kV (Blue)</option>
              <option value="230">230 kV (Red)</option>
              <option value="138">138 kV (Orange)</option>
              <option value="69">69 kV (Cyan)</option>
              <option value="40">40 kV (Green)</option>
              <option value="7">7 kV (Purple)</option>
            </select>
          </div>

          <div id="wrap-len" class="hidden">
            <label class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Bus Length (px)</label>
            <input id="prop-len" type="number" oninput="updateParam('len', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
          </div>

          <div id="wrap-pmw" class="hidden">
            <label id="lbl-pmw" class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Active Power (MW)</label>
            <input id="prop-pmw" type="number" oninput="updateParam('pMW', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
          </div>

          <div id="wrap-maxmw" class="hidden">
            <label class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Max Capacity (MW)</label>
            <input id="prop-maxmw" type="number" oninput="updateParam('maxMW', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
          </div>

          <div id="wrap-xpu" class="hidden">
            <label class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Reactance (X p.u.)</label>
            <input id="prop-xpu" type="number" oninput="updateParam('xpu', Number(this.value))" step="0.01" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
          </div>

          <div id="wrap-limit" class="hidden">
            <label class="block font-bold text-slate-400 mb-1 uppercase tracking-wider">Thermal Limit (MW)</label>
            <input id="prop-limit" type="number" oninput="updateParam('limit', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 focus:border-indigo-500 outline-none text-slate-200 transition">
          </div>

          <div id="wrap-rotate" class="hidden mt-2">
            <button id="btn-rotate" onclick="toggleRotate()" class="w-full bg-slate-800 hover:bg-slate-700 py-1.5 rounded border border-slate-600 transition flex justify-center items-center gap-1.5">
              <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15"></path></svg>
              <span id="txt-rotate">Orientation</span>
            </button>
          </div>

          <div id="wrap-breaker" class="hidden mt-2">
            <button id="btn-breaker" onclick="toggleBreaker()" class="w-full py-2 font-bold rounded border shadow-sm transition"></button>
          </div>

          <div class="mt-2 pt-3 border-t border-slate-700">
            <button onclick="deleteElement()" class="w-full bg-red-900/20 text-red-500 hover:bg-red-900/40 py-1.5 rounded border border-red-900/50 transition font-bold flex justify-center items-center gap-1">
               <svg class="w-3.5 h-3.5" fill="none" stroke="currentColor" viewBox="0 0 24 24"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16"></path></svg>
               Delete Element
            </button>
          </div>
        </div>
      </div>
    </aside>

  </div>

  <!-- Toast Container -->
  <div id="toastContainer" class="absolute top-12 left-1/2 transform -translate-x-1/2 z-50 flex flex-col gap-2 pointer-events-none"></div>

  <script>
    // System Constants & State
    const V_COLORS = { '500':'#0000FF', '230':'#ED3237', '138':'#F58220', '69':'#66FFFF', '40':'#66CC33', '7':'#993399' };
    let state = { buses: [], components: [], wires: [] };
    
    let mode = 'SELECT'; 
    let selectedId = null;
    let selectedType = null; 
    let wireSourceTerm = null; 
    let currentHoverId = null;
    let vp = { x: 50, y: 50, scale: 1, isDragging: false, startX: 0, startY: 0 };
    let dragElement = null; 
    let snapTarget = null; 

    // DOM Utilities
    const $ = id => document.getElementById(id);
    const genId = prefix => prefix + '_' + Math.random().toString(36).substr(2, 6).toUpperCase();
    const createSVG = (tag, attrs) => {
      const el = document.createElementNS('http://www.w3.org/2000/svg', tag);
      for (let k in attrs) el.setAttribute(k, attrs[k]);
      return el;
    };

    function showToast(msg, type='info') {
      const colors = type==='error'?'bg-red-900 border-red-500 text-red-200' : 'bg-emerald-900 border-emerald-500 text-emerald-200';
      const t = document.createElement('div');
      t.className = `toast px-3 py-1.5 rounded border ${colors} shadow-lg text-xs font-medium`;
      t.innerText = msg;
      $('toastContainer').appendChild(t);
      setTimeout(() => t.remove(), 3000);
    }

    // Getters
    const getBus = id => state.buses.find(b => b.id === id);
    const getComp = id => state.components.find(c => c.id === id);

    // Resizer Logic (Draggable Gutters)
    function initSplitters() {
      let isResizing = null;
      $('resizerLeft').onmousedown = e => { isResizing = 'left'; e.preventDefault(); };
      $('resizerRight').onmousedown = e => { isResizing = 'right'; e.preventDefault(); };
      window.addEventListener('mousemove', e => {
        if (!isResizing) return;
        if (isResizing === 'left') {
          let w = e.clientX;
          if (w > 120 && w < 400) $('leftPanel').style.width = w + 'px';
        } else if (isResizing === 'right') {
          let w = document.body.clientWidth - e.clientX;
          if (w > 150 && w < 500) $('rightPanel').style.width = w + 'px';
        }
      });
      window.addEventListener('mouseup', () => isResizing = null);
    }
    initSplitters();

    // Wiring Coordinates Math
    function getRawTerminalPos(termId) {
      if(getBus(termId)) return { isBus: true, busId: termId, type: 'BUS' };
      let [compId, port] = termId.split('_T');
      let c = getComp(compId);
      if(!c) return {x:0, y:0};
      let isVert = c.rot === 'V';
      if(['LINE', 'XFMR', 'BREAKER'].includes(c.type)) {
        if(isVert) return port === '1' ? {x: c.x, y: c.y - 24} : {x: c.x, y: c.y + 24};
        else       return port === '1' ? {x: c.x - 24, y: c.y} : {x: c.x + 24, y: c.y};
      } else return {x: c.x, y: c.y - 18}; 
    }

    function getClosestPointOnBus(bus, px, py) {
      if(!bus) return {x: px, y: py};
      let hw = bus.rot === 'H' ? bus.len / 2 : 6;
      let hh = bus.rot === 'H' ? 6 : bus.len / 2;
      return {
        x: Math.max(bus.x - hw, Math.min(px, bus.x + hw)),
        y: Math.max(bus.y - hh, Math.min(py, bus.y + hh))
      };
    }

    function resolveWireCoordinates(srcTerm, tgtTerm) {
      let p1 = getRawTerminalPos(srcTerm);
      let p2 = getRawTerminalPos(tgtTerm);
      if(p1.isBus && p2.isBus) { let b1 = getBus(p1.busId), b2 = getBus(p2.busId); return { x1: b1.x, y1: b1.y, x2: b2.x, y2: b2.y }; }
      if(p1.isBus) { let cp = getClosestPointOnBus(getBus(p1.busId), p2.x, p2.y); return { x1: cp.x, y1: cp.y, x2: p2.x, y2: p2.y }; }
      if(p2.isBus) { let cp = getClosestPointOnBus(getBus(p2.busId), p1.x, p1.y); return { x1: p1.x, y1: p1.y, x2: cp.x, y2: cp.y }; }
      return { x1: p1.x, y1: p1.y, x2: p2.x, y2: p2.y };
    }

    function dist2(v, w) { return (v.x - w.x)**2 + (v.y - w.y)**2; }
    function distToSegmentSquared(p, v, w) {
      let l2 = dist2(v, w);
      if (l2 === 0) return dist2(p, v);
      let t = Math.max(0, Math.min(1, ((p.x - v.x) * (w.x - v.x) + (p.y - v.y) * (w.y - v.y)) / l2));
      return { dist2: dist2(p, { x: v.x + t * (w.x - v.x), y: v.y + t * (w.y - v.y) }), point: { x: v.x + t * (w.x - v.x), y: v.y + t * (w.y - v.y) } };
    }

    function findSnapTarget(mx, my) {
      let best = null; let minDist = 225; // 15px radius squared
      state.buses.forEach(b => {
        let hw = b.rot === 'H' ? b.len / 2 : 6, hh = b.rot === 'H' ? 6 : b.len / 2;
        let cx = Math.max(b.x - hw, Math.min(mx, b.x + hw)), cy = Math.max(b.y - hh, Math.min(my, b.y + hh));
        let d2 = dist2({x:mx, y:my}, {x:cx, y:cy});
        if(d2 < minDist) { minDist = d2; best = { type: 'term', id: b.id, x: cx, y: cy }; }
      });
      state.components.forEach(c => {
        let t1 = getRawTerminalPos(`${c.id}_T1`), d1 = dist2({x:mx, y:my}, t1);
        if(d1 < minDist) { minDist = d1; best = { type: 'term', id: `${c.id}_T1`, x: t1.x, y: t1.y }; }
        if(['LINE', 'XFMR', 'BREAKER'].includes(c.type)) {
          let t2 = getRawTerminalPos(`${c.id}_T2`), d2 = dist2({x:mx, y:my}, t2);
          if(d2 < minDist) { minDist = d2; best = { type: 'term', id: `${c.id}_T2`, x: t2.x, y: t2.y }; }
        }
      });
      if(best) return best;
      minDist = 100;
      state.wires.forEach(w => {
        if(wireSourceTerm && (w.src === wireSourceTerm || w.tgt === wireSourceTerm)) return; 
        let c = resolveWireCoordinates(w.src, w.tgt);
        let res = distToSegmentSquared({x:mx, y:my}, {x:c.x1, y:c.y1}, {x:c.x2, y:c.y2});
        if(res.dist2 < minDist) { minDist = res.dist2; best = { type: 'wire', id: w.id, x: res.point.x, y: res.point.y, wireObj: w }; }
      });
      return best;
    }

    function getAvailableTerminal(type, id) {
      if(type === 'BUS') return id; 
      let c = getComp(id), usedTerms = new Set();
      state.wires.forEach(w => { if(w.src.startsWith(id)) usedTerms.add(w.src); if(w.tgt.startsWith(id)) usedTerms.add(w.tgt); });
      if(['GEN', 'LOAD'].includes(c.type)) return usedTerms.has(`${id}_T1`) ? null : `${id}_T1`;
      if(!usedTerms.has(`${id}_T1`)) return `${id}_T1`;
      if(!usedTerms.has(`${id}_T2`)) return `${id}_T2`;
      return null; 
    }

    // DC Power Flow Solver Engine
    function solvePowerFlow() {
      let totalGen = 0, totalLoad = 0;
      let parent = {};
      const find = i => parent[i] === i ? i : (parent[i] = find(parent[i]));
      const union = (i, j) => { if(!parent[i]) parent[i] = i; if(!parent[j]) parent[j] = j; parent[find(i)] = find(j); };

      state.buses.forEach(b => parent[b.id] = b.id);
      state.components.forEach(c => {
        parent[`${c.id}_T1`] = `${c.id}_T1`;
        if(['LINE', 'XFMR', 'BREAKER'].includes(c.type)) parent[`${c.id}_T2`] = `${c.id}_T2`;
      });
      state.wires.forEach(w => union(w.src, w.tgt));

      let nodeMap = {}; 
      Object.keys(parent).forEach(term => {
        let root = find(term);
        if(!nodeMap[root]) nodeMap[root] = { id: root, pNet: 0, theta: 0, energized: false, isSlack: false };
      });

      let branches = [];
      state.components.forEach(c => {
        let r1 = find(`${c.id}_T1`);
        if(c.type === 'GEN') { nodeMap[r1].pNet += Number(c.pMW); nodeMap[r1].energized = true; nodeMap[r1].isSlack = true; totalGen += Number(c.pMW); }
        else if(c.type === 'LOAD') { nodeMap[r1].pNet -= Number(c.pMW); totalLoad += Number(c.pMW); }
        else if(['LINE', 'XFMR', 'BREAKER'].includes(c.type)) {
          if(c.type === 'BREAKER' && c.status === 'OPEN') return; 
          let x = (c.type === 'BREAKER') ? 0.0001 : Math.max(Number(c.xpu)||0.01, 0.0001);
          branches.push({ cId: c.id, r1, r2: find(`${c.id}_T2`), x });
        }
      });

      $('lblGen').textContent = totalGen.toFixed(1) + ' MW';
      $('lblLoad').textContent = totalLoad.toFixed(1) + ' MW';
      $('lblLoss').textContent = Math.max(0, (totalGen - totalLoad)).toFixed(1) + ' MW';

      // BFS Energization
      let q = Object.values(nodeMap).filter(n => n.energized);
      while(q.length > 0) {
        let curr = q.shift();
        branches.forEach(br => {
          if(br.r1 === curr.id && !nodeMap[br.r2].energized) { nodeMap[br.r2].energized = true; q.push(nodeMap[br.r2]); }
          if(br.r2 === curr.id && !nodeMap[br.r1].energized) { nodeMap[br.r1].energized = true; q.push(nodeMap[br.r1]); }
        });
      }

      // Gauss-Seidel Math
      let activeNodes = Object.values(nodeMap).filter(n => n.energized);
      if(activeNodes.length > 0) {
        let slacks = activeNodes.filter(n => n.isSlack);
        slacks.forEach(s => s.theta = 0);
        if(slacks.length === 0) activeNodes[0].theta = 0; 

        for(let iter = 0; iter < 100; iter++) {
          activeNodes.forEach(node => {
            if(node.isSlack) return; 
            let sumInvX = 0, sumThetaInvX = 0;
            branches.forEach(br => {
              if(br.r1 === node.id || br.r2 === node.id) {
                let neighbor = nodeMap[br.r1 === node.id ? br.r2 : br.r1];
                if(neighbor && neighbor.energized) {
                  let invX = 1.0 / br.x; sumInvX += invX; sumThetaInvX += neighbor.theta * invX;
                }
              }
            });
            if(sumInvX > 0) node.theta = ((node.pNet / 100.0) + sumThetaInvX) / sumInvX;
          });
        }
      }

      // Map back to canvas
      state.components.forEach(c => { c.pFlow = 0; c.energized = false; });
      state.buses.forEach(b => b.energized = nodeMap[find(b.id)]?.energized || false);

      branches.forEach(br => {
        let flowPu = (nodeMap[br.r1].theta - nodeMap[br.r2].theta) / br.x;
        let comp = getComp(br.cId);
        if(comp) { comp.pFlow = flowPu * 100.0; comp.energized = true; comp.flowingT1toT2 = flowPu > 0; }
      });
      state.components.forEach(c => { if(['GEN', 'LOAD'].includes(c.type)) c.energized = nodeMap[find(`${c.id}_T1`)]?.energized || false; });
    }

    // Canvas Graphics Renderer
    function render() {
      solvePowerFlow();
      const layers = { wires: $('layer-wires'), flow: $('layer-flow'), comps: $('layer-components'), buses: $('layer-buses'), labels: $('layer-live-labels') };
      for(let k in layers) layers[k].innerHTML = '';

      // Wires
      state.wires.forEach(w => {
        let c = resolveWireCoordinates(w.src, w.tgt);
        layers.wires.appendChild(createSVG('line', { x1: c.x1, y1: c.y1, x2: c.x2, y2: c.y2, stroke: '#64748b', 'stroke-width': 2.5 }));
      });

      // Buses
      state.buses.forEach(b => {
        const isSel = (selectedType === 'BUS' && selectedId === b.id);
        const w = b.rot === 'H' ? b.len : 12, h = b.rot === 'H' ? 12 : b.len;
        const color = b.energized ? (V_COLORS[b.kv] || '#888') : '#475569';
        
        let g = createSVG('g', { class: 'cursor-move hover:brightness-110 transition-all' });
        g.onmousedown = e => startDrag(e, 'BUS', b.id);
        g.onclick = e => selectElement(e, 'BUS', b.id);
        g.onmouseenter = e => showHoverTooltip(e, 'BUS', b.id);
        g.onmouseleave = hideHoverTooltip;

        g.appendChild(createSVG('rect', { x: b.x - w/2, y: b.y - h/2, width: w, height: h, rx: 3, fill: color, stroke: isSel ? '#ffffff' : 'none', 'stroke-width': 3 }));
        if(b.len > 20) {
          let txt = createSVG('text', { x: b.x, y: b.y - h/2 - 6, fill: '#e2e8f0', 'font-size': 11, 'text-anchor': 'middle', class: 'pointer-events-none font-bold' });
          txt.textContent = b.name; layers.labels.appendChild(txt);
        }
        layers.buses.appendChild(g);
      });

      // Components
      state.components.forEach(c => {
        const isSel = (selectedType === 'COMP' && selectedId === c.id);
        let gMain = createSVG('g', { transform: `translate(${c.x}, ${c.y})`, class: 'cursor-pointer hover:drop-shadow-lg transition-all' });
        let gRot = createSVG('g', { transform: c.rot === 'V' ? 'rotate(90)' : 'rotate(0)' });
        gMain.appendChild(gRot);
        gMain.onmousedown = e => startDrag(e, 'COMP', c.id);
        gMain.onclick = e => selectElement(e, 'COMP', c.id);
        gMain.onmouseenter = e => showHoverTooltip(e, 'COMP', c.id);
        gMain.onmouseleave = hideHoverTooltip;

        if(isSel) gRot.appendChild(createSVG('rect', { x: -35, y: -25, width: 70, height: 50, rx: 6, fill: 'none', stroke: '#818cf8', 'stroke-width': 2, 'stroke-dasharray': '4 4' }));

        if (c.type === 'GEN') {
          gRot.appendChild(createSVG('circle', { cx: 0, cy: 0, r: 16, fill: '#064e3b', stroke: c.energized?'#34d399':'#64748b', 'stroke-width': 2 }));
          let t = createSVG('text', { x: 0, y: 5, fill: c.energized?'#34d399':'#64748b', 'font-size': 14, 'text-anchor': 'middle', 'font-weight': 'bold' });
          t.textContent = 'G'; gRot.appendChild(t);
        }
        else if (c.type === 'LOAD') { gRot.appendChild(createSVG('polygon', { points: "-14,-14 14,-14 0,14", fill: '#78350f', stroke: c.energized?'#fbbf24':'#64748b', 'stroke-width': 2 })); }
        else if (c.type === 'LINE') {
          gRot.appendChild(createSVG('rect', { x: -16, y: -8, width: 32, height: 16, rx: 2, fill: '#1e3a8a', stroke: '#60a5fa', 'stroke-width': 2 }));
          gRot.appendChild(createSVG('line', { x1: -16, y1: 0, x2: 16, y2: 0, stroke: '#93c5fd', 'stroke-width': 2 }));
        }
        else if (c.type === 'XFMR') {
          gRot.appendChild(createSVG('circle', { cx: -7, cy: 0, r: 11, fill: '#1e293b', stroke: '#fb923c', 'stroke-width': 2 }));
          gRot.appendChild(createSVG('circle', { cx: 7, cy: 0, r: 11, fill: 'none', stroke: '#fb923c', 'stroke-width': 2 }));
        }
        else if (c.type === 'BREAKER') {
          let isOpen = c.status === 'OPEN';
          gRot.appendChild(createSVG('rect', { x: -12, y: -12, width: 24, height: 24, rx: 2, fill: isOpen ? '#7f1d1d' : '#14532d', stroke: isOpen ? '#f87171' : '#4ade80', 'stroke-width': 2 }));
          let t = createSVG('text', { x: 0, y: 4, fill: '#fff', 'font-size': 12, 'text-anchor': 'middle', 'font-weight': 'bold', transform: c.rot==='V'?'rotate(-90)':'rotate(0)' });
          t.textContent = isOpen ? 'O' : 'C'; gRot.appendChild(t);
        }

        const drawTerm = (x, y) => gRot.appendChild(createSVG('circle', { cx: x, cy: y, r: 3.5, fill: '#94a3b8' }));
        if(['GEN', 'LOAD'].includes(c.type)) drawTerm(0, -18);
        else { drawTerm(-24, 0); drawTerm(24, 0); }

        let lName = createSVG('text', { x: 0, y: c.rot==='V'? 36 : 30, fill: '#cbd5e1', 'font-size': 10, 'text-anchor': 'middle', class: 'font-mono pointer-events-none drop-shadow-md' });
        lName.textContent = c.name; gMain.appendChild(lName);

        if(c.energized) {
          let lVal = createSVG('text', { x: 0, y: c.rot==='V'? -28 : -20, fill: '#fff', 'font-size': 11, 'font-weight': 'bold', 'text-anchor': 'middle', class: 'pointer-events-none' });
          lVal.setAttribute('paint-order', 'stroke'); lVal.setAttribute('stroke', '#020617'); lVal.setAttribute('stroke-width', '3px');
          if(c.type === 'GEN') { lVal.textContent = `+${c.pMW} MW`; lVal.setAttribute('fill', '#4ade80'); }
          else if(c.type === 'LOAD') { lVal.textContent = `-${c.pMW} MW`; lVal.setAttribute('fill', '#fbbf24'); }
          else if(['LINE', 'XFMR'].includes(c.type)) {
            let absFlow = Math.abs(c.pFlow || 0); let overloaded = absFlow > Number(c.limit);
            lVal.textContent = `${absFlow.toFixed(1)} MW`; lVal.setAttribute('fill', overloaded ? '#f87171' : '#38bdf8');
            if(overloaded) gRot.appendChild(createSVG('rect', { x: -20, y: -16, width: 40, height: 32, rx:4, fill: 'none', stroke: '#ef4444', 'stroke-width': 3, class: 'overload' }));
            if(absFlow > 0.1) {
              let speed = Math.max(0.2, 1.5 - (absFlow / 500)); 
              let flowLine = createSVG('line', { x1: c.flowingT1toT2 ? -15 : 15, y1: 0, x2: c.flowingT1toT2 ? 15 : -15, y2: 0, stroke: '#ffffff', 'stroke-width': 2, class: 'flow-line pointer-events-none drop-shadow-[0_0_3px_#fff]' });
              flowLine.style.animationDuration = `${speed}s`; gRot.appendChild(flowLine);
            }
          }
          gMain.appendChild(lVal);
        }
        layers.comps.appendChild(gMain);
      });
    }

    // Core Interactions
    const container = $('viewportContainer');
    function setMode(newMode) {
      mode = newMode; wireSourceTerm = null; snapTarget = null;
      $('snapIndicator').classList.add('hidden'); $('activeWire').classList.add('hidden');
      $('btnSelect').className = `px-2.5 py-1 rounded font-medium transition flex items-center gap-1 ${mode==='SELECT' ? 'bg-indigo-600 text-white shadow-lg' : 'border border-slate-600 hover:bg-slate-700 text-slate-300'}`;
      $('btnWire').className = `px-2.5 py-1 rounded font-medium transition relative flex items-center gap-1 ${mode==='WIRE' ? 'bg-indigo-600 text-white shadow-lg' : 'border border-slate-600 hover:bg-slate-700 text-slate-300'}`;
      $('wireBadge').style.display = mode==='WIRE' ? 'block' : 'none';
    }

    function spawnComponent(type) {
      let x = -vp.x / vp.scale + 200, y = -vp.y / vp.scale + 200;
      if(type === 'BUS') state.buses.push({ id: genId('B'), name: `Bus ${state.buses.length+1}`, x, y, len: 150, rot: 'H', kv: '230' });
      else {
        let comp = { id: genId('C'), type, name: `${type} ${state.components.length+1}`, x, y, rot: 'H' };
        if(type === 'GEN') { comp.pMW = 100; comp.maxMW = 200; }
        if(type === 'LOAD') { comp.pMW = 50; }
        if(['LINE', 'XFMR', 'BREAKER'].includes(type)) { comp.xpu = type === 'XFMR' ? 0.05 : 0.1; comp.limit = 500; }
        if(type === 'BREAKER') comp.status = 'CLOSED';
        state.components.push(comp);
      }
      setMode('SELECT'); render();
    }

    function selectElement(e, type, id) {
      e.stopPropagation();
      if(mode === 'WIRE') { handleWiring(type, id, e.clientX, e.clientY); return; }
      selectedType = type; selectedId = id;
      populateInspector(); // Bind inputs without destroying DOM
      render();
    }

    function populateInspector() {
      if(!selectedId) {
        $('insType').textContent = 'NONE';
        $('inspector-none').classList.remove('hidden');
        $('inspector-form').classList.add('hidden');
        return;
      }

      $('inspector-none').classList.add('hidden');
      $('inspector-form').classList.remove('hidden');
      
      let el = selectedType === 'BUS' ? getBus(selectedId) : getComp(selectedId);
      $('insType').textContent = selectedType === 'BUS' ? 'BUSBAR' : el.type;
      
      // Reset visibility
      const fields = ['wrap-kv', 'wrap-len', 'wrap-pmw', 'wrap-maxmw', 'wrap-xpu', 'wrap-limit', 'wrap-rotate', 'wrap-breaker'];
      fields.forEach(f => $(f).classList.add('hidden'));

      // Populate Name
      $('prop-name').value = el.name || '';

      if (selectedType === 'BUS') {
        $('wrap-kv').classList.remove('hidden'); $('prop-kv').value = el.kv;
        $('wrap-len').classList.remove('hidden'); $('prop-len').value = el.len;
        $('wrap-rotate').classList.remove('hidden'); $('txt-rotate').textContent = el.rot === 'H' ? 'Horizontal ↔' : 'Vertical ↕';
      } else {
        if (['BREAKER', 'LINE', 'XFMR'].includes(el.type)) {
           $('wrap-rotate').classList.remove('hidden'); $('txt-rotate').textContent = el.rot === 'H' ? 'Horizontal ↔' : 'Vertical ↕';
        }
        if (el.type === 'GEN') {
          $('wrap-pmw').classList.remove('hidden'); $('lbl-pmw').textContent = "Active Power (MW)"; $('prop-pmw').value = el.pMW;
          $('wrap-maxmw').classList.remove('hidden'); $('prop-maxmw').value = el.maxMW;
        } else if (el.type === 'LOAD') {
          $('wrap-pmw').classList.remove('hidden'); $('lbl-pmw').textContent = "Demand (MW)"; $('prop-pmw').value = el.pMW;
        } else if (['LINE', 'XFMR'].includes(el.type)) {
          $('wrap-xpu').classList.remove('hidden'); $('prop-xpu').value = el.xpu;
          $('wrap-limit').classList.remove('hidden'); $('prop-limit').value = el.limit;
        } else if (el.type === 'BREAKER') {
          $('wrap-breaker').classList.remove('hidden');
          updateBreakerButton(el.status);
        }
      }
    }

    function updateParam(key, val) {
      if(!selectedId) return;
      let el = selectedType === 'BUS' ? getBus(selectedId) : getComp(selectedId);
      if(el) { el[key] = val; render(); }
    }

    function toggleRotate() {
      let el = selectedType === 'BUS' ? getBus(selectedId) : getComp(selectedId);
      if(el) {
        el.rot = el.rot === 'H' ? 'V' : 'H';
        $('txt-rotate').textContent = el.rot === 'H' ? 'Horizontal ↔' : 'Vertical ↕';
        render();
      }
    }

    function toggleBreaker() {
      let el = getComp(selectedId);
      if(el && el.type === 'BREAKER') {
        el.status = el.status === 'CLOSED' ? 'OPEN' : 'CLOSED';
        updateBreakerButton(el.status);
        render();
      }
    }
    
    function updateBreakerButton(status) {
      const btn = $('btn-breaker');
      if (status === 'CLOSED') {
         btn.className = "w-full py-2 font-bold rounded border shadow-sm transition bg-red-900/30 text-red-400 border-red-800";
         btn.textContent = "TRIP (OPEN)";
      } else {
         btn.className = "w-full py-2 font-bold rounded border shadow-sm transition bg-green-900/30 text-green-400 border-green-800";
         btn.textContent = "CLOSE BREAKER";
      }
    }

    window.deleteElement = function() {
      if(!selectedId) return;
      state.wires = state.wires.filter(w => !w.src.startsWith(selectedId) && !w.tgt.startsWith(selectedId));
      if(selectedType === 'BUS') state.buses = state.buses.filter(b => b.id !== selectedId);
      else state.components = state.components.filter(c => c.id !== selectedId);
      selectedId = null; hideHoverTooltip(); populateInspector(); render();
    };

    // Wiring Logic (T-Tap preserved)
    function handleWiring(type, id, cx, cy) {
      if(!wireSourceTerm) {
        if(snapTarget && snapTarget.type === 'term') wireSourceTerm = snapTarget.id;
        else {
           let term = getAvailableTerminal(type, id);
           if(!term) { showToast("No available ports.", "error"); return; }
           wireSourceTerm = term;
        }
      } else {
        let tgt = null;
        if(snapTarget) {
          if(snapTarget.type === 'term') tgt = snapTarget.id;
          else if (snapTarget.type === 'wire') {
             let jId = genId('J');
             state.buses.push({ id: jId, name: `TAP`, x: snapTarget.x, y: snapTarget.y, len: 14, rot: 'H', kv: '230' }); 
             let oldWire = snapTarget.wireObj;
             state.wires = state.wires.filter(w => w.id !== oldWire.id);
             state.wires.push({ id: genId('W'), src: oldWire.src, tgt: jId }, { id: genId('W'), src: jId, tgt: oldWire.tgt });
             tgt = jId; showToast("Line Tapped!");
          }
        } else tgt = getAvailableTerminal(type, id);

        if(tgt) {
          if(wireSourceTerm === tgt) { wireSourceTerm = null; $('activeWire').classList.add('hidden'); return; }
          state.wires.push({ id: genId('W'), src: wireSourceTerm, tgt: tgt });
          wireSourceTerm = null; $('activeWire').classList.add('hidden'); setMode('SELECT'); render();
        }
      }
    }

    // Mouse & Keyboard Interactions
    container.addEventListener('mousedown', e => {
      if(e.target.tagName === 'svg' || e.target.id === 'viewportContainer') {
        if(mode === 'SELECT') {
          vp.isDragging = true; vp.startX = e.clientX - vp.x; vp.startY = e.clientY - vp.y;
          selectedId = null; selectedType = null; populateInspector(); render();
        } else if (mode === 'WIRE' && snapTarget && snapTarget.type === 'wire' && wireSourceTerm) {
          handleWiring(null, null, e.clientX, e.clientY);
        } else if (mode === 'WIRE' && wireSourceTerm) {
           wireSourceTerm = null; $('activeWire').classList.add('hidden');
        }
      }
    });

    function startDrag(e, type, id) {
      if(mode !== 'SELECT') return;
      e.stopPropagation();
      let el = type === 'BUS' ? getBus(id) : getComp(id);
      let rect = container.getBoundingClientRect();
      let mx = (e.clientX - rect.left - vp.x) / vp.scale;
      let my = (e.clientY - rect.top - vp.y) / vp.scale;
      dragElement = { obj: el, offsetX: el.x - mx, offsetY: el.y - my };
      selectElement(e, type, id);
    }

    window.addEventListener('mousemove', e => {
      let rect = container.getBoundingClientRect();
      let mx = (e.clientX - rect.left - vp.x) / vp.scale, my = (e.clientY - rect.top - vp.y) / vp.scale;

      if(vp.isDragging) {
        vp.x = e.clientX - vp.startX; vp.y = e.clientY - vp.startY;
        $('transformGroup').setAttribute('transform', `translate(${vp.x}, ${vp.y}) scale(${vp.scale})`);
      } else if (dragElement) {
        dragElement.obj.x = mx + dragElement.offsetX; dragElement.obj.y = my + dragElement.offsetY; render();
      }

      if(mode === 'WIRE') {
        snapTarget = findSnapTarget(mx, my);
        let ind = $('snapIndicator');
        if(snapTarget) { ind.setAttribute('cx', snapTarget.x); ind.setAttribute('cy', snapTarget.y); ind.setAttribute('stroke', snapTarget.type === 'wire' ? '#f59e0b' : '#22c55e'); ind.classList.remove('hidden'); }
        else ind.classList.add('hidden');
        if(wireSourceTerm) {
          let p1 = getRawTerminalPos(wireSourceTerm), aw = $('activeWire');
          aw.setAttribute('x1', p1.isBus ? getBus(p1.busId).x : p1.x); aw.setAttribute('y1', p1.isBus ? getBus(p1.busId).y : p1.y);
          aw.setAttribute('x2', snapTarget ? snapTarget.x : mx); aw.setAttribute('y2', snapTarget ? snapTarget.y : my); aw.classList.remove('hidden');
        }
      }
      if(currentHoverId) { let tt = $('tooltip'); tt.style.left = (e.clientX + 15) + 'px'; tt.style.top = (e.clientY + 15) + 'px'; }
    });

    window.addEventListener('mouseup', () => { vp.isDragging = false; dragElement = null; });
    container.addEventListener('wheel', e => {
      e.preventDefault();
      let oldScale = vp.scale;
      vp.scale = Math.min(Math.max(0.1, vp.scale + (e.deltaY < 0 ? 0.05 : -0.05)), 4);
      let rect = container.getBoundingClientRect(), mx = e.clientX - rect.left, my = e.clientY - rect.top;
      vp.x = mx - (mx - vp.x) * (vp.scale / oldScale); vp.y = my - (my - vp.y) * (vp.scale / oldScale);
      $('transformGroup').setAttribute('transform', `translate(${vp.x}, ${vp.y}) scale(${vp.scale})`);
    });

    window.addEventListener('keydown', e => {
      if(e.target.tagName === 'INPUT' || e.target.tagName === 'SELECT') return;
      if(e.key.toLowerCase() === 'r' && selectedId) toggleRotate();
      else if (e.key.toLowerCase() === 'v') setMode('SELECT');
      else if (e.key.toLowerCase() === 'w') setMode('WIRE');
      else if (e.key === 'Delete' || e.key === 'Backspace') deleteElement();
    });

    function showHoverTooltip(e, type, id) {
      if(mode === 'WIRE' || vp.isDragging || dragElement) return;
      currentHoverId = id; const tt = $('tooltip'); let html = '';
      if(type === 'BUS') {
        let b = getBus(id); html = `<strong>${b.name}</strong><br><span class="text-slate-400">Type: Busbar (${b.kv}kV)</span><br>Status: ${b.energized?'<span class="text-green-400">Energized</span>':'<span class="text-slate-500">Dead</span>'}`;
      } else {
        let c = getComp(id); html = `<strong>${c.name}</strong><br><span class="text-slate-400">Type: ${c.type}</span><br>Status: ${c.energized?'<span class="text-green-400">Live</span>':'<span class="text-slate-500">Dead</span>'}`;
        if(c.type === 'GEN') html += `<br>Gen: ${c.pMW} / ${c.maxMW} MW`; if(c.type === 'LOAD') html += `<br>Demand: ${c.pMW} MW`;
        if(['LINE', 'XFMR'].includes(c.type)) html += `<br>Flow: ${Math.abs(c.pFlow||0).toFixed(1)} MW<br>Limit: ${c.limit} MW`;
      }
      tt.innerHTML = html; tt.style.left = (e.clientX + 15) + 'px'; tt.style.top = (e.clientY + 15) + 'px'; tt.classList.remove('hidden');
    }
    function hideHoverTooltip() { currentHoverId = null; $('tooltip').classList.add('hidden'); }

    // System I/O
    function exportData() {
      const a = document.createElement('a');
      a.href = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(state, null, 2));
      a.download = 'grid_model.json'; a.click(); showToast("Exported Successfully!");
    }
    function importData(e) {
      const file = e.target.files[0]; if(!file) return;
      const reader = new FileReader();
      reader.onload = e => {
        try { 
          let loaded = JSON.parse(e.target.result); 
          if(loaded.buses && loaded.components && loaded.wires) {
            state = loaded; vp.x = 50; vp.y = 50; vp.scale = 1;
            $('transformGroup').setAttribute('transform', `translate(${vp.x}, ${vp.y}) scale(${vp.scale})`);
            selectedId = null; populateInspector(); render(); showToast("Model Loaded!");
          } else throw new Error();
        } catch(err) { showToast("Invalid JSON Schema.", "error"); }
      };
      reader.readAsText(file); e.target.value = '';
    }
    function clearCanvas() { if(confirm('Clear the entire network?')) { state = {buses:[], components:[], wires:[]}; selectedId=null; populateInspector(); render(); } }

    window.onload = () => {
      state.buses = [
        { id: 'B1', name: 'MAIN BUS 1', x: 200, y: 150, len: 400, rot: 'H', kv: '500' },
        { id: 'B2', name: 'MAIN BUS 2', x: 200, y: 450, len: 400, rot: 'H', kv: '500' },
        { id: 'B3', name: 'SUB 230kV', x: 700, y: 300, len: 200, rot: 'V', kv: '230' }
      ];
      state.components = [
        { id: 'CB1', type: 'BREAKER', name: 'TIE 1', x: 100, y: 225, status: 'CLOSED', rot: 'V' },
        { id: 'CB2', type: 'BREAKER', name: 'TIE 2', x: 100, y: 300, status: 'CLOSED', rot: 'V' },
        { id: 'CB3', type: 'BREAKER', name: 'TIE 3', x: 100, y: 375, status: 'CLOSED', rot: 'V' },
        { id: 'G1', type: 'GEN', name: 'UNIT_01', x: 200, y: 300, pMW: 600, maxMW: 800, rot: 'H' },
        { id: 'X1', type: 'XFMR', name: 'T_500_230', x: 300, y: 300, xpu: 0.05, limit: 1000, rot: 'H' },
        { id: 'L1', type: 'LINE', name: 'TL_230_01', x: 500, y: 300, xpu: 0.05, limit: 800, rot: 'H' },
        { id: 'D1', type: 'LOAD', name: 'DIST_A', x: 800, y: 300, pMW: 450, rot: 'H' }
      ];
      state.wires = [
        { id: 'W1', src: 'B1', tgt: 'CB1_T1' }, { id: 'W2', src: 'CB1_T2', tgt: 'CB2_T1' },
        { id: 'W3', src: 'CB2_T2', tgt: 'CB3_T1' }, { id: 'W4', src: 'CB3_T2', tgt: 'B2' },
        { id: 'W5', src: 'G1_T1', tgt: 'CB1_T2' }, { id: 'W6', src: 'X1_T1', tgt: 'CB2_T2' }, 
        { id: 'W7', src: 'X1_T2', tgt: 'L1_T1' }, { id: 'W8', src: 'L1_T2', tgt: 'B3' },
        { id: 'W9', src: 'B3', tgt: 'D1_T1' }
      ];
      render();
    };
  </script>
</body>
</html>
