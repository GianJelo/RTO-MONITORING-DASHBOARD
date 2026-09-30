<html lang="en" class="dark fixed inset-0 w-full h-full overflow-hidden bg-slate-950 text-slate-200">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no, viewport-fit=cover">
  <title>SLD Market Network Simulator Pro</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: { extend: { colors: { panel: '#0f172a', canvas: '#020617', kv500: '#3b82f6', kv230: '#ef4444', kv138: '#f97316', kv69: '#06b6d4' } } }
    }
  </script>
  <style>
    html, body {
      position: fixed; top: 0; left: 0; right: 0; bottom: 0;
      width: 100%; height: 100dvh; margin: 0; padding: 0; overflow: hidden;
      overscroll-behavior: none; -webkit-touch-callout: none;
    }
    .no-touch { touch-action: none; -webkit-user-select: none; user-select: none; }
    input, select { -webkit-user-select: auto; user-select: auto; touch-action: auto; }
    
    .canvas-bg {
      background-color: #020617;
      background-image: radial-gradient(circle, #334155 1px, transparent 1px);
      background-size: 40px 40px;
    }

    @keyframes flowFwd { from { stroke-dashoffset: 24; } to { stroke-dashoffset: 0; } }
    @keyframes flowRev { from { stroke-dashoffset: -24; } to { stroke-dashoffset: 0; } }
    .flow-anim { stroke-dasharray: 4 8; stroke-linecap: round; }
    .flow-anim.fwd { animation: flowFwd linear infinite; }
    .flow-anim.rev { animation: flowRev linear infinite; }
    
    @keyframes pulseWarning {
      0%, 100% { filter: drop-shadow(0 0 4px #ef4444); stroke: #ef4444; }
      50% { filter: drop-shadow(0 0 12px #f87171); stroke: #f87171; }
    }
    .overload { animation: pulseWarning 0.8s infinite; stroke-width: 5px !important; stroke-dasharray: none; }
    
    @keyframes flashText {
      0%, 100% { opacity: 1; color: #ef4444; }
      50% { opacity: 0.3; color: #f87171; }
    }
    .deficit-warn { animation: flashText 1s infinite; font-weight: bold; }

    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-thumb { background: #334155; border-radius: 3px; }
    .resizer { width: 6px; background: #1e293b; cursor: col-resize; flex-shrink: 0; z-index: 40; }
    .resizer:hover, .resizer:active { background: #6366f1; }
  </style>
</head>
<body class="flex flex-col font-sans no-touch text-sm">

  <header class="h-14 border-b border-slate-700 bg-panel px-4 flex items-center justify-between z-30 shrink-0 shadow-lg">
    <div class="flex items-center gap-3">
      <div class="w-8 h-8 rounded bg-indigo-600 flex items-center justify-center font-bold text-xs text-white shadow-[0_0_10px_rgba(79,70,229,0.5)]">SLD</div>
      <h1 class="font-bold text-sm tracking-wide hidden sm:block text-slate-300">MARKET NETWORK SIMULATOR</h1>
    </div>
    <div class="flex items-center gap-2 text-xs">
      <button id="btnSelect" class="px-4 py-2 rounded bg-indigo-600 text-white font-semibold flex items-center gap-1 shadow transition" onclick="setMode('SELECT')">Select (V)</button>
      <button id="btnWire" class="px-4 py-2 rounded border border-slate-600 text-slate-300 hover:bg-slate-700 font-semibold relative transition" onclick="setMode('WIRE')">
        Wire (W)
        <span id="wireBadge" class="hidden absolute -top-1 -right-1 w-2.5 h-2.5 bg-red-500 rounded-full animate-ping"></span>
      </button>
      <div class="w-px h-5 bg-slate-600 mx-1"></div>
      <button class="px-3 py-2 rounded bg-emerald-700 hover:bg-emerald-600 text-white font-semibold shadow" onclick="exportData()">Export</button>
      <button class="px-3 py-2 rounded bg-amber-700 hover:bg-amber-600 text-white font-semibold shadow" onclick="document.getElementById('fileUpload').click()">Import</button>
      <input type="file" id="fileUpload" class="hidden" accept=".json" onchange="importData(event)">
      <button class="px-3 py-2 rounded border border-red-900 text-red-400 hover:bg-red-900/50 hidden sm:block ml-2" onclick="clearCanvas()">Clear</button>
    </div>
  </header>

  <div class="flex-1 flex overflow-hidden w-full h-full relative">
    <aside id="leftPanel" class="w-[240px] bg-panel flex flex-col z-20 shrink-0 border-r border-slate-800">
      <div class="p-3 border-b border-slate-700 font-bold text-[10px] text-slate-400 uppercase tracking-widest bg-slate-900/80">Elements</div>
      <div class="p-3 grid grid-cols-2 gap-3 flex-1 overflow-y-auto pointer-events-auto" style="touch-action: pan-y;">
        <button onclick="spawn('BUS')" class="p-3 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-2 transition hover:scale-105">
          <div class="w-10 h-2 bg-kv230 rounded-sm"></div><span class="text-[10px] font-medium">Busbar</span>
        </button>
        <button onclick="spawn('GEN')" class="p-3 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-2 transition hover:scale-105">
          <div class="w-7 h-7 rounded-full border-2 border-emerald-500 text-emerald-500 flex items-center justify-center font-bold text-xs">G</div>
          <span class="text-[10px] font-medium">Generator</span>
        </button>
        <button onclick="spawn('LOAD')" class="p-3 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-2 transition hover:scale-105">
          <div class="w-0 h-0 border-l-[10px] border-r-[10px] border-t-[14px] border-l-transparent border-r-transparent border-t-amber-500"></div>
          <span class="text-[10px] font-medium">Load</span>
        </button>
        <button onclick="spawn('XFMR')" class="p-3 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-2 transition hover:scale-105">
          <div class="flex items-center"><div class="w-5 h-5 rounded-full border-2 border-orange-400 -mr-2"></div><div class="w-5 h-5 rounded-full border-2 border-orange-400"></div></div>
          <span class="text-[10px] font-medium">Transformer</span>
        </button>
        <button onclick="spawn('BREAKER')" class="p-3 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded flex flex-col items-center gap-2 transition hover:scale-105">
          <div class="w-6 h-6 border-2 border-red-500 bg-red-900 rounded flex items-center justify-center text-[9px] font-bold text-red-400">CB</div>
          <span class="text-[10px] font-medium">Breaker</span>
        </button>
      </div>
      
      <div class="p-4 border-t border-slate-700 bg-slate-900 text-xs shrink-0 font-mono">
        <div class="text-[10px] text-slate-500 font-bold mb-2 uppercase">Live System Balance</div>
        <div class="flex justify-between mb-1"><span>Generation Cap:</span><span id="lblGenCap" class="text-emerald-400">0 MW</span></div>
        <div class="flex justify-between mb-1"><span>System Demand:</span><span id="lblLoad" class="text-amber-400">0 MW</span></div>
        <div class="flex justify-between pt-1 border-t border-slate-800 mt-1"><span>Active Output:</span><span id="lblGenAct" class="text-blue-400">0 MW</span></div>
        <div id="lblDeficit" class="hidden mt-2 text-center deficit-warn text-[10px] bg-red-950/50 py-1 rounded border border-red-900">SYSTEM DEFICIT - LOAD SHEDDING</div>
      </div>
    </aside>

    <div class="resizer hidden md:block" id="resizerLeft"></div>

    <main id="viewportContainer" class="flex-1 relative canvas-bg overflow-hidden cursor-grab active:cursor-grabbing no-touch">
      <svg id="canvas" class="w-full h-full absolute inset-0 block pointer-events-none">
        <g id="transformGroup" transform="translate(0,0) scale(1)">
          <g id="layer-wires"></g>
          <g id="layer-flow"></g>
          <g id="layer-wire-labels"></g>
          <g id="layer-buses"></g>
          <g id="layer-components"></g>
          <g id="layer-labels"></g>
          <line id="activeWire" x1="0" y1="0" x2="0" y2="0" stroke="#f43f5e" stroke-width="2.5" stroke-dasharray="6 4" class="hidden drop-shadow-[0_0_5px_#f43f5e]" />
        </g>
      </svg>
    </main>

    <div class="resizer hidden md:block" id="resizerRight"></div>

    <aside id="rightPanel" class="w-[280px] bg-panel flex flex-col z-20 shrink-0 border-l border-slate-800">
      <div class="p-3 border-b border-slate-700 font-bold text-[10px] text-slate-400 uppercase tracking-widest flex justify-between items-center bg-slate-900/80">
        <span>Inspector</span><span id="insType" class="text-indigo-400 border border-indigo-700/50 bg-indigo-900/30 px-1.5 py-0.5 rounded">NONE</span>
      </div>
      
      <div id="inspectorPanel" class="p-4 overflow-y-auto flex-1 text-xs pointer-events-auto" style="touch-action: pan-y;">
        <div id="inspector-none" class="text-center py-10 text-slate-500">Select an element or wire.<br><br><b>R</b> to Rotate<br><b>Del</b> to Delete</div>
        <div id="inspector-form" class="hidden flex flex-col gap-3">
          
          <div id="wrap-name"><label class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Name</label>
            <input id="prop-name" type="text" oninput="updateProp('name', this.value)" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-slate-200">
          </div>

          <div id="wrap-kv" class="hidden"><label class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Voltage</label>
            <select id="prop-kv" onchange="updateProp('kv', this.value)" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-slate-200">
              <option value="500">500 kV</option><option value="230">230 kV</option><option value="138">138 kV</option><option value="69">69 kV</option>
            </select>
          </div>

          <div id="wrap-len" class="hidden"><label class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Bus Length (px)</label>
            <input id="prop-len" type="number" oninput="updateProp('len', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-slate-200">
          </div>

          <div id="wrap-cap" class="hidden"><label class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Max Capacity (MW)</label>
            <input id="prop-cap" type="number" oninput="updateProp('pMax', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-emerald-300">
          </div>

          <div id="wrap-pmw" class="hidden"><label id="lbl-pmw" class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Demand (MW)</label>
            <input id="prop-pmw" type="number" oninput="updateProp('pMW', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-amber-300">
          </div>

          <div id="wrap-xpu" class="hidden"><label class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Reactance (X pu)</label>
            <input id="prop-xpu" type="number" step="0.001" oninput="updateProp('xpu', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-slate-200">
          </div>

          <div id="wrap-limit" class="hidden"><label class="block font-bold text-slate-500 mb-1 text-[10px] uppercase">Thermal Limit (MW)</label>
            <input id="prop-limit" type="number" oninput="updateProp('limit', Number(this.value))" class="w-full bg-slate-900 border border-slate-700 rounded px-2 py-1.5 outline-none focus:border-indigo-500 text-red-300">
          </div>

          <div id="wrap-rotate" class="hidden mt-2">
            <button onclick="toggleRotation()" class="w-full py-1.5 bg-slate-800 hover:bg-slate-700 border border-slate-600 rounded font-medium transition"><span id="txt-rotate">Orientation</span></button>
          </div>

          <div id="wrap-breaker" class="hidden mt-2">
            <button id="btn-breaker" onclick="toggleBreaker()" class="w-full py-2 font-bold rounded border shadow-sm transition tracking-widest text-[10px]"></button>
          </div>

          <div class="mt-4 pt-3 border-t border-slate-800 flex flex-col gap-2 shrink-0">
            <button onclick="duplicateElem()" class="w-full bg-indigo-900/30 text-indigo-400 hover:bg-indigo-900/50 py-1.5 rounded border border-indigo-800/50 transition font-bold">Duplicate</button>
            <button onclick="deleteElem()" class="w-full bg-red-900/30 text-red-400 hover:bg-red-900/50 py-1.5 rounded border border-red-800/50 transition font-bold">Delete</button>
          </div>
        </div>
      </div>
    </aside>
  </div>

  <div id="toastContainer" class="absolute top-16 left-1/2 transform -translate-x-1/2 z-50 flex flex-col gap-2 pointer-events-none"></div>

  <script>
    const V_COLORS = { '500':'#3b82f6', '230':'#ef4444', '138':'#f97316', '69':'#06b6d4' };
    let state = { buses: [], components: [], wires: [] };
    
    let mode = 'SELECT'; 
    let selId = null, selType = null;
    let wireSrc = null;
    
    let vp = { x: 0, y: 0, scale: 1, isDrag: false, startX: 0, startY: 0 };
    let dragEl = null;

    const $ = id => document.getElementById(id);
    const createSVG = (tag, attrs) => { const el = document.createElementNS('http://www.w3.org/2000/svg', tag); for(let k in attrs) el.setAttribute(k, attrs[k]); return el; };
    const genId = prefix => prefix + '_' + Math.random().toString(36).substr(2,6).toUpperCase();

    function showToast(msg, isErr=false) {
      const t = document.createElement('div');
      t.className = `px-4 py-2 rounded shadow-lg text-sm font-bold border transition-opacity duration-500 ` + (isErr ? 'bg-red-900 border-red-600 text-red-200' : 'bg-emerald-900 border-emerald-600 text-emerald-200');
      t.innerText = msg; $('toastContainer').appendChild(t);
      setTimeout(() => { t.style.opacity = '0'; setTimeout(()=>t.remove(), 500); }, 2500);
    }

    function initSplitters() {
      let isR = null; const L = $('resizerLeft'), R = $('resizerRight');
      L.addEventListener('pointerdown', e => { isR = 'L'; e.preventDefault(); L.setPointerCapture(e.pointerId); });
      R.addEventListener('pointerdown', e => { isR = 'R'; e.preventDefault(); R.setPointerCapture(e.pointerId); });
      window.addEventListener('pointermove', e => {
        if(!isR) return;
        if(isR === 'L') { let w = e.clientX; if(w > 180 && w < 400) $('leftPanel').style.width = w + 'px'; }
        if(isR === 'R') { let w = document.body.clientWidth - e.clientX; if(w > 200 && w < 400) $('rightPanel').style.width = w + 'px'; }
      });
      window.addEventListener('pointerup', () => isR = null);
    }
    initSplitters();

    const getB = id => state.buses.find(b => b.id === id);
    const getC = id => state.components.find(c => c.id === id);
    const getW = id => state.wires.find(w => w.id === id);

    function getRawPos(termId) {
      if(getB(termId)) return { isBus: true, id: termId };
      let [cId, port] = termId.split('_T'); let c = getC(cId); if(!c) return {x:0, y:0};
      if(['XFMR', 'BREAKER'].includes(c.type)) {
        if(c.rot === 'V') return port === '1' ? {x: c.x, y: c.y - 24} : {x: c.x, y: c.y + 24};
        else return port === '1' ? {x: c.x - 24, y: c.y} : {x: c.x + 24, y: c.y};
      } else return {x: c.x, y: c.y - 18};
    }

    function getBusPoint(bus, px, py) {
      if(!bus) return {x: px, y: py};
      let hw = bus.rot === 'H' ? bus.len/2 : 6, hh = bus.rot === 'H' ? 6 : bus.len/2;
      return { x: Math.max(bus.x-hw, Math.min(px, bus.x+hw)), y: Math.max(bus.y-hh, Math.min(py, bus.y+hh)) };
    }

    function getWireCoords(w) {
      let p1 = getRawPos(w.src), p2 = getRawPos(w.tgt);
      if(p1.isBus && p2.isBus) { let b1 = getB(p1.id), b2 = getB(p2.id); return {x1: b1.x, y1: b1.y, x2: b2.x, y2: b2.y}; }
      if(p1.isBus) { let cp = getBusPoint(getB(p1.id), p2.x, p2.y); return {x1: cp.x, y1: cp.y, x2: p2.x, y2: p2.y}; }
      if(p2.isBus) { let cp = getBusPoint(getB(p2.id), p1.x, p1.y); return {x1: p1.x, y1: p1.y, x2: cp.x, y2: cp.y}; }
      return {x1: p1.x, y1: p1.y, x2: p2.x, y2: p2.y};
    }

    function distPointToSegment(px, py, x1, y1, x2, y2) {
      let l2 = (x2-x1)**2 + (y2-y1)**2;
      if(l2 === 0) return Math.hypot(px-x1, py-y1);
      let t = Math.max(0, Math.min(1, ((px-x1)*(x2-x1) + (py-y1)*(y2-y1)) / l2));
      return Math.hypot(px - (x1 + t*(x2-x1)), py - (y1 + t*(y2-y1)));
    }

    function solvePowerFlow() {
      let parent = {};
      const find = i => parent[i] === i ? i : (parent[i] = find(parent[i]));
      const union = (i, j) => { if(!parent[i]) parent[i]=i; if(!parent[j]) parent[j]=j; let ri = find(i), rj = find(j); if(ri !== rj) parent[ri] = rj; };

      state.buses.forEach(b => parent[b.id] = b.id);
      state.components.forEach(c => {
        parent[`${c.id}_T1`] = `${c.id}_T1`;
        if(['XFMR', 'BREAKER'].includes(c.type)) parent[`${c.id}_T2`] = `${c.id}_T2`;
      });
      
      // Zero impedance branches (Closed breakers)
      state.components.forEach(c => { if(c.type==='BREAKER' && c.status==='CLOSED') union(`${c.id}_T1`, `${c.id}_T2`); });

      let nodes = {};
      Object.keys(parent).forEach(t => {
        let r = find(t); if(!nodes[r]) nodes[r] = { id: r, pNet: 0, pLoad: 0, maxGen: 0, theta: 0, terms: [] };
        nodes[r].terms.push(t);
      });

      // Tally Gen/Load mapping to nodes
      let sysLoad = 0, sysGenCap = 0;
      state.components.forEach(c => {
        let r1 = find(`${c.id}_T1`);
        if(c.type === 'GEN') { nodes[r1].maxGen += Number(c.pMax||0); sysGenCap += Number(c.pMax||0); }
        if(c.type === 'LOAD') { nodes[r1].pLoad += Number(c.pMW||0); sysLoad += Number(c.pMW||0); }
      });

      // Treat Wires & Transformers as Branches
      let branches = [];
      state.wires.forEach(w => {
        let r1 = find(w.src), r2 = find(w.tgt);
        let x = Math.max(Number(w.xpu)||0.01, 0.0001);
        if(r1 !== r2) branches.push({ id: w.id, r1, r2, x, type: 'WIRE' });
      });
      state.components.forEach(c => {
        if(c.type === 'XFMR') {
          let r1 = find(`${c.id}_T1`), r2 = find(`${c.id}_T2`);
          let x = Math.max(Number(c.xpu)||0.01, 0.0001);
          if(r1 !== r2) branches.push({ id: c.id, r1, r2, x, type: 'XFMR' });
        }
      });

      // Detect Islands & Dispatch
      let adj = {}; Object.keys(nodes).forEach(k => adj[k] = []);
      branches.forEach(b => { adj[b.r1].push(b.r2); adj[b.r2].push(b.r1); });
      
      let visited = new Set(), islands = [];
      Object.keys(nodes).forEach(st => {
        if(!visited.has(st)) {
          let isl = { nodes: [], load: 0, genCap: 0 };
          let q = [st]; visited.add(st);
          while(q.length > 0) {
            let curr = q.shift(); isl.nodes.push(curr);
            isl.load += nodes[curr].pLoad; isl.genCap += nodes[curr].maxGen;
            adj[curr].forEach(n => { if(!visited.has(n)) { visited.add(n); q.push(n); } });
          }
          islands.push(isl);
        }
      });

      let sysGenAct = 0;
      islands.forEach(isl => {
        let ratio = isl.genCap > 0 ? Math.min(1, isl.load / isl.genCap) : 0;
        isl.nodes.forEach(nid => {
          let n = nodes[nid];
          let actGen = n.maxGen * ratio;
          sysGenAct += actGen;
          n.pNet = actGen - n.pLoad; // Net injection
        });

        if(isl.genCap > 0) {
          let slackId = isl.nodes.reduce((a,b) => nodes[a].maxGen > nodes[b].maxGen ? a : b);
          nodes[slackId].isSlack = true;
          
          // Gauss-Seidel
          let act = isl.nodes.map(id => nodes[id]);
          for(let iter=0; iter<100; iter++) {
            let mxD = 0;
            act.forEach(n => {
              if(n.isSlack) return;
              let sumInvX = 0, sumTh = 0;
              branches.forEach(br => {
                if(br.r1 === n.id || br.r2 === n.id) {
                  let nbr = nodes[br.r1 === n.id ? br.r2 : br.r1];
                  let inv = 1 / br.x;
                  sumInvX += inv; sumTh += nbr.theta * inv;
                }
              });
              if(sumInvX > 0) {
                let old = n.theta; n.theta = ((n.pNet/100) + sumTh) / sumInvX;
                mxD = Math.max(mxD, Math.abs(n.theta - old));
              }
            });
            if(mxD < 1e-5) break;
          }
        }
      });

      // Compute Flows
      let termTh = {};
      Object.keys(nodes).forEach(nid => nodes[nid].terms.forEach(t => termTh[t] = nodes[nid].theta));

      state.wires.forEach(w => {
        w.t1 = termTh[w.src]; w.t2 = termTh[w.tgt];
        w.energized = w.t1 !== undefined && w.t2 !== undefined;
        w.pFlow = w.energized ? ((w.t1 - w.t2) / Math.max(Number(w.xpu)||0.01, 0.0001)) * 100 : 0;
      });

      state.components.forEach(c => {
        c.energized = termTh[`${c.id}_T1`] !== undefined;
        if(c.type === 'XFMR' && c.energized) {
          c.pFlow = ((termTh[`${c.id}_T1`] - termTh[`${c.id}_T2`]) / Math.max(Number(c.xpu)||0.01, 0.0001)) * 100;
        } else if (c.type === 'GEN') {
          c.actGen = c.energized ? Number(c.pMax||0) * (islands.find(i=>i.nodes.includes(find(`${c.id}_T1`))).load / Math.max(0.001, islands.find(i=>i.nodes.includes(find(`${c.id}_T1`))).genCap)) : 0;
          if(c.actGen > c.pMax) c.actGen = c.pMax;
        }
      });
      state.buses.forEach(b => b.energized = termTh[b.id] !== undefined);

      $('lblGenCap').textContent = sysGenCap.toFixed(1) + ' MW';
      $('lblLoad').textContent = sysLoad.toFixed(1) + ' MW';
      $('lblGenAct').textContent = sysGenAct.toFixed(1) + ' MW';
      
      let defWarn = $('lblDeficit');
      if(sysLoad > sysGenCap && sysLoad > 0) defWarn.classList.remove('hidden'); else defWarn.classList.add('hidden');
    }

    function render() {
      solvePowerFlow();
      const L = { w: $('layer-wires'), f: $('layer-flow'), wl: $('layer-wire-labels'), c: $('layer-components'), b: $('layer-buses'), lbl: $('layer-labels') };
      for(let k in L) L[k].innerHTML = '';

      // Render Transmission Wires
      state.wires.forEach(w => {
        let p = getWireCoords(w);
        let isSel = selType === 'WIRE' && selId === w.id;
        let isOverload = Math.abs(w.pFlow) > Number(w.limit);
        
        let gW = createSVG('g', { class: 'cursor-pointer pointer-events-auto group' });
        
        // Base wire line (thicker invisible for easy clicking)
        gW.appendChild(createSVG('line', { x1: p.x1, y1: p.y1, x2: p.x2, y2: p.y2, stroke: 'transparent', 'stroke-width': 15 }));
        
        // Visible Wire
        let visWire = createSVG('line', { x1: p.x1, y1: p.y1, x2: p.x2, y2: p.y2, stroke: isSel?'#cbd5e1':(w.energized?'#64748b':'#334155'), 'stroke-width': isSel?4:3 });
        if(isOverload && w.energized) visWire.setAttribute('class', 'overload');
        gW.appendChild(visWire);

        // Flow particles
        if(w.energized && Math.abs(w.pFlow) > 0.5 && !isOverload) {
          let isFwd = w.pFlow > 0;
          let f = createSVG('line', { x1: p.x1, y1: p.y1, x2: p.x2, y2: p.y2, stroke: '#38bdf8', 'stroke-width': 3, class: `flow-anim pointer-events-none ${isFwd?'fwd':'rev'}` });
          f.style.animationDuration = Math.max(0.2, 2 - Math.abs(w.pFlow)*0.005) + 's';
          L.f.appendChild(f);
        }

        // Wire Label (Midpoint badge)
        if(w.energized || isSel) {
          let mx = (p.x1 + p.x2)/2, my = (p.y1 + p.y2)/2;
          let badge = createSVG('rect', { x: mx-24, y: my-9, width: 48, height: 18, rx: 4, fill: isOverload?'#7f1d1d':'#0f172a', stroke: isOverload?'#f87171':'#3b82f6', 'stroke-width':1, class:'pointer-events-none' });
          let txt = createSVG('text', { x: mx, y: my+3, fill: isOverload?'#fecaca':'#93c5fd', 'font-size': 9, 'font-weight': 'bold', 'text-anchor':'middle', class:'pointer-events-none' });
          txt.textContent = Math.abs(w.pFlow).toFixed(1) + ' MW';
          L.wl.appendChild(badge); L.wl.appendChild(txt);
          
          let nm = createSVG('text', { x: mx, y: my-12, fill: '#64748b', 'font-size': 9, 'font-weight': 'bold', 'text-anchor':'middle', class:'pointer-events-none' });
          nm.textContent = w.name; L.wl.appendChild(nm);
        }
        
        gW.addEventListener('pointerdown', e => startDrag(e, 'WIRE', w.id));
        L.w.appendChild(gW);
      });

      // Render Buses
      state.buses.forEach(b => {
        let isSel = selType === 'BUS' && selId === b.id;
        let w = b.rot === 'H' ? b.len : 12, h = b.rot === 'H' ? 12 : b.len;
        let col = b.energized ? (V_COLORS[b.kv] || '#94a3b8') : '#334155';
        
        let g = createSVG('g', { class: 'pointer-events-auto cursor-pointer hover:brightness-125 transition' });
        g.addEventListener('pointerdown', e => startDrag(e, 'BUS', b.id));
        g.appendChild(createSVG('rect', { x: b.x-w/2, y: b.y-h/2, width: w, height: h, rx: 3, fill: col, stroke: isSel?'#fff':'none', 'stroke-width':3 }));
        
        if(b.name !== 'TAP' && b.len > 20) {
          let t = createSVG('text', { x: b.x, y: b.y-h/2-5, fill: '#cbd5e1', 'font-size': 11, 'text-anchor':'middle', class:'font-bold pointer-events-none' });
          t.textContent = b.name; L.lbl.appendChild(t);
        }
        L.b.appendChild(g);
      });

      // Render Components
      state.components.forEach(c => {
        let isSel = selType === 'COMP' && selId === c.id;
        let gMain = createSVG('g', { transform: `translate(${c.x}, ${c.y})`, class: 'pointer-events-auto cursor-pointer hover:drop-shadow-lg' });
        gMain.addEventListener('pointerdown', e => startDrag(e, 'COMP', c.id));
        
        let gR = createSVG('g', { transform: c.rot==='V' ? 'rotate(90)' : '' });
        gMain.appendChild(gR);

        if(isSel) gR.appendChild(createSVG('rect', { x: -30, y: -25, width: 60, height: 50, rx: 6, fill: 'none', stroke: '#818cf8', 'stroke-width': 2, 'stroke-dasharray': '4 4' }));

        let strokeCol = c.energized ? '#94a3b8' : '#475569';
        
        if(c.type === 'GEN') {
          strokeCol = c.energized ? '#34d399' : '#475569';
          gR.appendChild(createSVG('circle', { cx:0, cy:0, r:16, fill:'#064e3b', stroke: strokeCol, 'stroke-width':2 }));
          let t=createSVG('text', {x:0, y:5, fill:strokeCol, 'font-size':14, 'font-weight':'bold', 'text-anchor':'middle'}); t.textContent='G'; gR.appendChild(t);
        } else if (c.type === 'LOAD') {
          strokeCol = c.energized ? '#fbbf24' : '#475569';
          gR.appendChild(createSVG('polygon', { points: "-14,-14 14,-14 0,14", fill:'#78350f', stroke: strokeCol, 'stroke-width':2 }));
        } else if (c.type === 'XFMR') {
          strokeCol = c.energized ? '#fb923c' : '#475569';
          gR.appendChild(createSVG('circle', { cx:-7, cy:0, r:11, fill:'#1e293b', stroke: strokeCol, 'stroke-width':2 }));
          gR.appendChild(createSVG('circle', { cx:7, cy:0, r:11, fill:'none', stroke: strokeCol, 'stroke-width':2 }));
          if(c.energized && Math.abs(c.pFlow) > 1e-4) {
             let isFwd = c.pFlow > 0;
             let f = createSVG('line', { x1: -16, y1: 0, x2: 16, y2: 0, stroke: '#fff', 'stroke-width': 2, class: `flow-anim pointer-events-none drop-shadow-[0_0_2px_#fff] ${isFwd?'fwd':'rev'}` });
             gR.appendChild(f);
          }
        } else if (c.type === 'BREAKER') {
          let isO = c.status === 'OPEN';
          gR.appendChild(createSVG('rect', { x:-12, y:-12, width:24, height:24, rx:2, fill:isO?'#7f1d1d':'#14532d', stroke:isO?'#f87171':'#4ade80', 'stroke-width':2 }));
          let t=createSVG('text', {x:0, y:4, fill:'#fff', 'font-size':12, 'font-weight':'bold', 'text-anchor':'middle', transform:c.rot==='V'?'rotate(-90)':''}); 
          t.textContent=isO?'O':'C'; gR.appendChild(t);
        }

        const dt = (x,y) => gR.appendChild(createSVG('circle', { cx:x, cy:y, r:3, fill:'#64748b' }));
        if(['GEN', 'LOAD'].includes(c.type)) dt(0,-18); else { dt(-24,0); dt(24,0); }

        let nameT = createSVG('text', { x:0, y: c.rot==='V'?36:30, fill:'#94a3b8', 'font-size':10, 'text-anchor':'middle', class:'font-mono pointer-events-none' });
        nameT.textContent = c.name; gMain.appendChild(nameT);

        if(c.energized && ['GEN','LOAD'].includes(c.type)) {
          let vY = c.rot==='V'? -26 : -18;
          let valT = createSVG('text', { x:0, y:vY, fill:'#fff', 'font-size':11, 'font-weight':'bold', 'text-anchor':'middle', class:'pointer-events-none' });
          valT.setAttribute('paint-order', 'stroke'); valT.setAttribute('stroke', '#020617'); valT.setAttribute('stroke-width', '3px');
          
          if(c.type === 'GEN') { valT.textContent = `+${(c.actGen||0).toFixed(1)} MW`; valT.setAttribute('fill', '#34d399'); }
          if(c.type === 'LOAD') { valT.textContent = `-${c.pMW} MW`; valT.setAttribute('fill', '#fbbf24'); }
          gMain.appendChild(valT);
        }
        L.c.appendChild(gMain);
      });
    }

    const cont = $('viewportContainer');
    
    function setMode(m) {
      mode = m; wireSrc = null;
      $('activeWire').classList.add('hidden');
      $('btnSelect').className = `px-4 py-2 rounded font-semibold flex items-center gap-1 transition ${mode==='SELECT'?'bg-indigo-600 text-white shadow':'border border-slate-600 text-slate-300 hover:bg-slate-700'}`;
      $('btnWire').className = `px-4 py-2 rounded font-semibold relative transition ${mode==='WIRE'?'bg-indigo-600 text-white shadow':'border border-slate-600 text-slate-300 hover:bg-slate-700'}`;
      $('wireBadge').style.display = mode==='WIRE'?'block':'none';
    }

    window.addEventListener('keydown', e => {
      if(e.target.tagName === 'INPUT' || e.target.tagName === 'SELECT') return;
      if(e.key.toLowerCase() === 'v') setMode('SELECT');
      if(e.key.toLowerCase() === 'w') setMode('WIRE');
      if(e.key.toLowerCase() === 'r' && selId) toggleRotation();
      if(e.key === 'Delete' || e.key === 'Backspace') deleteElem();
    });

    function spawn(type) {
      let x = -vp.x/vp.scale + cont.clientWidth/2, y = -vp.y/vp.scale + cont.clientHeight/2;
      if(type === 'BUS') state.buses.push({ id: genId('B'), name: `BUS ${state.buses.length+1}`, x, y, len: 150, rot: 'H', kv: '230' });
      else {
        let c = { id: genId('C'), type, name: `${type}_${state.components.length+1}`, x, y, rot: 'H' };
        if(type === 'GEN') c.pMax = 200;
        if(type === 'LOAD') c.pMW = 50;
        if(type === 'XFMR') { c.xpu = 0.05; c.limit = 500; }
        if(type === 'BREAKER') c.status = 'CLOSED';
        state.components.push(c);
      }
      setMode('SELECT'); render();
    }

    function getFreePort(type, id) {
      if(type === 'BUS') return id;
      let used = new Set(); state.wires.forEach(w => { used.add(w.src); used.add(w.tgt); });
      let c = getC(id);
      if(['GEN', 'LOAD'].includes(c.type)) return used.has(`${id}_T1`) ? null : `${id}_T1`;
      if(!used.has(`${id}_T1`)) return `${id}_T1`;
      if(!used.has(`${id}_T2`)) return `${id}_T2`;
      return null;
    }

    function startDrag(e, type, id) {
      e.stopPropagation(); e.preventDefault();
      if(mode === 'WIRE') {
        if(type === 'WIRE') return; // Can't start wire on wire
        if(!wireSrc) {
          let p = getFreePort(type, id);
          if(p) wireSrc = p; else showToast("No open ports", true);
        } else {
          let t = getFreePort(type, id);
          if(t && t !== wireSrc) { 
            state.wires.push({ id: genId('W'), name: `TL-${state.wires.length+1}`, src: wireSrc, tgt: t, xpu: 0.02, limit: 300 }); 
            setMode('SELECT'); render(); 
          } else wireSrc = null;
        }
        return;
      }
      
      selectEl(type, id);
      if(type === 'WIRE') return; // Don't drag wires
      
      let el = type === 'BUS' ? getB(id) : getC(id);
      let r = cont.getBoundingClientRect();
      dragEl = { el, ox: el.x - ((e.clientX - r.left - vp.x) / vp.scale), oy: el.y - ((e.clientY - r.top - vp.y) / vp.scale) };
      cont.setPointerCapture(e.pointerId);
    }

    cont.addEventListener('pointerdown', e => {
      if(e.target.tagName === 'svg' || e.target.id === 'viewportContainer') {
        if(mode === 'SELECT') {
          vp.isDrag = true; vp.startX = e.clientX - vp.x; vp.startY = e.clientY - vp.y;
          selectEl(null, null); cont.setPointerCapture(e.pointerId);
        } else if (mode === 'WIRE' && wireSrc) { wireSrc = null; $('activeWire').classList.add('hidden'); }
      }
    });

    window.addEventListener('pointermove', e => {
      let r = cont.getBoundingClientRect();
      let mx = (e.clientX - r.left - vp.x) / vp.scale, my = (e.clientY - r.top - vp.y) / vp.scale;

      if(vp.isDrag) {
        vp.x = e.clientX - vp.startX; vp.y = e.clientY - vp.startY;
        $('transformGroup').setAttribute('transform', `translate(${vp.x},${vp.y}) scale(${vp.scale})`);
      } else if (dragEl) {
        dragEl.el.x = mx + dragEl.ox; dragEl.el.y = my + dragEl.oy; render();
      }

      if(mode === 'WIRE' && wireSrc) {
        let p1 = getRawPos(wireSrc), aw = $('activeWire');
        aw.setAttribute('x1', p1.isBus ? getB(p1.id).x : p1.x); aw.setAttribute('y1', p1.isBus ? getB(p1.id).y : p1.y);
        aw.setAttribute('x2', mx); aw.setAttribute('y2', my); aw.classList.remove('hidden');
      }
    });

    window.addEventListener('pointerup', e => { vp.isDrag = false; dragEl = null; try{cont.releasePointerCapture(e.pointerId)}catch(e){} });

    cont.addEventListener('wheel', e => {
      e.preventDefault(); let os = vp.scale;
      vp.scale = Math.min(Math.max(0.2, vp.scale + (e.deltaY<0 ? 0.05 : -0.05)), 3);
      let r = cont.getBoundingClientRect(), mx = e.clientX - r.left, my = e.clientY - r.top;
      vp.x = mx - (mx - vp.x) * (vp.scale / os); vp.y = my - (my - vp.y) * (vp.scale / os);
      $('transformGroup').setAttribute('transform', `translate(${vp.x},${vp.y}) scale(${vp.scale})`);
    }, { passive: false });

    function selectEl(type, id) {
      selType = type; selId = id;
      if(!id) {
        $('inspector-none').classList.remove('hidden'); $('inspector-form').classList.add('hidden');
        $('insType').textContent = 'NONE';
      } else {
        $('inspector-none').classList.add('hidden'); $('inspector-form').classList.remove('hidden');
        let el = type === 'BUS' ? getB(id) : type === 'WIRE' ? getW(id) : getC(id);
        $('insType').textContent = type === 'BUS' ? 'BUSBAR' : type === 'WIRE' ? 'TRANS. LINE' : el.type;
        ['wrap-kv','wrap-len','wrap-cap','wrap-pmw','wrap-xpu','wrap-limit','wrap-rotate','wrap-breaker'].forEach(w => $(w).classList.add('hidden'));
        
        $('prop-name').value = el.name;
        if(type === 'BUS') {
          $('wrap-kv').classList.remove('hidden'); $('prop-kv').value = el.kv;
          $('wrap-len').classList.remove('hidden'); $('prop-len').value = el.len;
          $('wrap-rotate').classList.remove('hidden'); $('txt-rotate').textContent = el.rot==='H'?'Horizontal':'Vertical';
        } else if (type === 'WIRE') {
          $('wrap-xpu').classList.remove('hidden'); $('prop-xpu').value = el.xpu; 
          $('wrap-limit').classList.remove('hidden'); $('prop-limit').value = el.limit;
        } else {
          if(['XFMR','BREAKER'].includes(el.type)) { $('wrap-rotate').classList.remove('hidden'); $('txt-rotate').textContent = el.rot==='H'?'Horizontal':'Vertical'; }
          if(el.type === 'GEN') { $('wrap-cap').classList.remove('hidden'); $('prop-cap').value = el.pMax; }
          if(el.type === 'LOAD') { $('wrap-pmw').classList.remove('hidden'); $('prop-pmw').value = el.pMW; }
          if(el.type === 'XFMR') { $('wrap-xpu').classList.remove('hidden'); $('prop-xpu').value = el.xpu; $('wrap-limit').classList.remove('hidden'); $('prop-limit').value = el.limit; }
          if(el.type === 'BREAKER') { $('wrap-breaker').classList.remove('hidden'); updBrkBtn(el.status); }
        }
      }
      render();
    }

    function updateProp(key, val) {
      if(!selId) return;
      let el = selType === 'BUS' ? getB(selId) : selType === 'WIRE' ? getW(selId) : getC(selId);
      if(el) { el[key] = val; render(); }
    }
    
    function toggleRotation() { let el=selType==='BUS'?getB(selId):getC(selId); if(el){el.rot=el.rot==='H'?'V':'H'; selectEl(selType, selId);} }
    function toggleBreaker() { let el=getC(selId); if(el && el.type==='BREAKER'){ el.status=el.status==='CLOSED'?'OPEN':'CLOSED'; updBrkBtn(el.status); render(); } }
    function updBrkBtn(st) { let b=$('btn-breaker'); b.textContent = st==='CLOSED'?'TRIP (OPEN)':'CLOSE BREAKER'; b.className = `w-full py-2 font-bold rounded border shadow-sm transition tracking-widest text-[10px] ${st==='CLOSED'?'bg-red-900/40 text-red-400 border-red-800':'bg-emerald-900/40 text-emerald-400 border-emerald-800'}`; }

    window.duplicateElem = function() {
      if(!selId || selType === 'WIRE') { showToast("Cannot duplicate wires directly.", true); return; }
      let orig = selType === 'BUS' ? getB(selId) : getC(selId);
      let dup = JSON.parse(JSON.stringify(orig));
      dup.id = genId(selType==='BUS'?'B':'C'); dup.x+=40; dup.y+=40; dup.name += ' (Copy)';
      if(selType==='BUS') state.buses.push(dup); else state.components.push(dup);
      selectEl(selType, dup.id);
    };

    window.deleteElem = function() {
      if(!selId) return;
      if(selType === 'WIRE') { state.wires = state.wires.filter(w => w.id !== selId); }
      else {
        state.wires = state.wires.filter(w => !w.src.startsWith(selId) && !w.tgt.startsWith(selId));
        if(selType === 'BUS') state.buses = state.buses.filter(b => b.id !== selId);
        else state.components = state.components.filter(c => c.id !== selId);
      }
      selectEl(null, null);
    };

    window.exportData = function() {
      const blob = new Blob([JSON.stringify(state, null, 2)], { type: 'application/json' });
      const url = URL.createObjectURL(blob); const a = document.createElement('a');
      a.href = url; a.download = 'market_model.json'; document.body.appendChild(a); a.click();
      setTimeout(() => { document.body.removeChild(a); window.URL.revokeObjectURL(url); }, 0);
      showToast("Model Exported.");
    };

    window.importData = function(e) {
      let file = e.target.files[0]; if(!file) return;
      let fr = new FileReader();
      fr.onload = ev => { try { let res = JSON.parse(ev.target.result); if(res.buses) { state=res; selectEl(null,null); showToast("Model Loaded!"); } } catch(err) { showToast("Invalid File", true); } };
      fr.readAsText(file); e.target.value = '';
    };
    window.clearCanvas = function() { if(confirm("Clear everything?")) { state = {buses:[], components:[], wires:[]}; selectEl(null,null); } };

    // Boot Load Model
    window.onload = () => {
      vp.x = window.innerWidth/2 - 250; vp.y = window.innerHeight/2 - 200;
      $('transformGroup').setAttribute('transform', `translate(${vp.x},${vp.y}) scale(1)`);
      
      state.buses = [
        {id:'B1', name:'BUS 1 (230kV)', x:100, y:150, len:200, rot:'V', kv:'230'},
        {id:'B2', name:'BUS 2 (138kV)', x:400, y:150, len:200, rot:'V', kv:'138'}
      ];
      state.components = [
        {id:'G1', type:'GEN', name:'PLANT A', x:0, y:150, rot:'H', pMax:250},
        {id:'X1', type:'XFMR', name:'TR_01', x:250, y:100, rot:'H', xpu:0.05, limit:300},
        {id:'LD1', type:'LOAD', name:'CITY D', x:500, y:150, rot:'H', pMW:280} // 280 > 250 to trigger deficit warning
      ];
      state.wires = [
        {id:'W1', name:'TL-GEN', src:'G1_T1', tgt:'B1', xpu:0.01, limit: 300}, 
        {id:'W2', name:'TL-XFMR-PRI', src:'B1', tgt:'X1_T1', xpu:0.01, limit: 300},
        {id:'W3', name:'TL-XFMR-SEC', src:'X1_T2', tgt:'B2', xpu:0.01, limit: 300}, 
        {id:'W4', name:'TL-MAIN', src:'B1', tgt:'B2', xpu:0.1, limit: 120}, // Direct line, gets overloaded
        {id:'W6', name:'TL-LOAD', src:'B2', tgt:'LD1_T1', xpu:0.01, limit: 300}
      ];
      render();
    };
  </script>
</body>
</html>
