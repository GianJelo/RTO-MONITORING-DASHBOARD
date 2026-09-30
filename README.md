<html lang="en" class="h-full bg-slate-950 text-slate-100 dark select-none">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SLD Power Grid Builder & Network Flow Simulator</title>
  <!-- Tailwind CSS CDN -->
  <script src="https://cdn.tailwindcss.com"></script>
  <!-- Phosphor Icons -->
  <script src="https://unpkg.com/@phosphor-icons/web"></script>
  <script>
    tailwind.config = {
      darkMode: 'class',
      theme: {
        extend: {
          colors: {
            grid: {
              bg: '#0B0F17',
              panel: '#111827',
              border: '#1F2937',
              bus230: '#EF4444',
              bus138: '#F97316',
              bus69: '#10B981',
              bus13: '#3B82F6'
            }
          },
          fontFamily: {
            mono: ['JetBrains Mono', 'ui-monospace', 'monospace'],
            sans: ['Inter', 'system-ui', 'sans-serif']
          }
        }
      }
    }
  </script>
  <style>
    @keyframes flowParticle {
      from { stroke-dashoffset: 24; }
      to { stroke-dashoffset: 0; }
    }
    .flow-line {
      stroke-dasharray: 6, 6;
      animation: flowParticle 0.7s linear infinite;
    }
    .flow-reverse {
      animation-direction: reverse;
    }
    .bg-grid-dots {
      background-image: radial-gradient(circle, #1e293b 1px, transparent 1px);
      background-size: 24px 24px;
    }
    ::-webkit-scrollbar { width: 6px; height: 6px; }
    ::-webkit-scrollbar-track { background: #0b0f17; }
    ::-webkit-scrollbar-thumb { background: #1e293b; border-radius: 3px; }
    ::-webkit-scrollbar-thumb:hover { background: #334155; }
  </style>
</head>
<body class="h-full flex flex-col font-sans overflow-hidden bg-grid-bg text-slate-200">

  <header class="h-14 border-b border-grid-border bg-grid-panel px-4 flex items-center justify-between z-30 shrink-0">
    <div class="flex items-center space-x-3">
      <div class="p-2 bg-blue-600/20 text-blue-400 rounded-lg border border-blue-500/30">
        <i class="ph-bold ph-circuit text-xl"></i>
      </div>
      <div>
        <h1 class="font-bold text-slate-100 text-sm md:text-base flex items-center gap-2">
          SLD NETWORK BUILDER & SOLVER
          <span class="text-[10px] font-mono px-2 py-0.5 rounded bg-emerald-500/10 text-emerald-400 border border-emerald-500/20">LIVE POWER FLOW</span>
        </h1>
        <p class="text-xs text-slate-400 hidden sm:block">Market Model Single Line Diagram Canvas & Auto-Solver</p>
      </div>
    </div>

    <!-- Quick Actions Toolbar -->
    <div class="flex items-center space-x-2">
      <button id="wireToolBtn" class="px-3 py-1.5 bg-slate-800 hover:bg-slate-700 text-slate-200 border border-slate-700 rounded-lg text-xs font-semibold transition flex items-center gap-1.5">
        <i class="ph-bold ph-line-segments text-base text-blue-400"></i>
        <span id="wireToolText">Connect Mode (Off)</span>
      </button>

      <button id="solveFlowBtn" class="px-3 py-1.5 bg-blue-600 hover:bg-blue-500 text-white font-medium rounded-lg text-xs transition flex items-center gap-1.5 shadow-lg shadow-blue-600/20">
        <i class="ph-bold ph-arrows-clockwise text-sm"></i>
        <span>Solve Grid Flow</span>
      </button>

      <div class="h-5 w-px bg-slate-800 my-auto"></div>

      <button id="loadSampleBtn" class="px-3 py-1.5 bg-slate-800 hover:bg-slate-700 text-slate-300 rounded-lg text-xs font-medium transition flex items-center gap-1">
        <i class="ph-bold ph-folder-open text-sm"></i> Sample 5-Bus
      </button>

      <button id="clearCanvasBtn" class="px-3 py-1.5 bg-red-950/40 hover:bg-red-900/50 text-red-300 border border-red-800/40 rounded-lg text-xs transition flex items-center gap-1">
        <i class="ph-bold ph-trash text-sm"></i> Clear
      </button>

      <button id="exportJsonBtn" class="p-2 text-slate-400 hover:text-white bg-slate-800 hover:bg-slate-700 rounded-lg transition" title="Export JSON">
        <i class="ph-bold ph-download-simple text-base"></i>
      </button>

      <button id="importJsonBtn" class="p-2 text-slate-400 hover:text-white bg-slate-800 hover:bg-slate-700 rounded-lg transition" title="Import JSON">
        <i class="ph-bold ph-upload-simple text-base"></i>
      </button>
      <input type="file" id="importFileInput" accept=".json" class="hidden">
    </div>
  </header>

  <div class="flex-1 flex overflow-hidden relative">

    <!-- Left Toolbar Palette -->
    <aside class="w-64 border-r border-grid-border bg-grid-panel p-3 flex flex-col shrink-0 z-20 space-y-4">
      <div>
        <h3 class="text-xs font-bold text-slate-400 uppercase tracking-wider font-mono mb-2">Grid Component Palette</h3>
        <p class="text-[11px] text-slate-500 mb-3">Click any element below to drop it onto the canvas canvas area:</p>

        <div class="space-y-2">
          <button onclick="spawnElement('BUS')" class="w-full p-2.5 bg-slate-900 hover:bg-slate-800 border border-slate-800 rounded-xl flex items-center justify-between text-xs transition group">
            <div class="flex items-center gap-2.5">
              <div class="w-6 h-1.5 bg-red-500 rounded"></div>
              <span class="font-medium text-slate-200">Substation Bus</span>
            </div>
            <i class="ph-bold ph-plus text-slate-500 group-hover:text-blue-400"></i>
          </button>

          <button onclick="spawnElement('GENERATOR')" class="w-full p-2.5 bg-slate-900 hover:bg-slate-800 border border-slate-800 rounded-xl flex items-center justify-between text-xs transition group">
            <div class="flex items-center gap-2.5">
              <div class="w-6 h-6 rounded-full border-2 border-emerald-400 flex items-center justify-center font-bold text-[10px] text-emerald-400">G</div>
              <span class="font-medium text-slate-200">Power Generator</span>
            </div>
            <i class="ph-bold ph-plus text-slate-500 group-hover:text-blue-400"></i>
          </button>

          <button onclick="spawnElement('LOAD')" class="w-full p-2.5 bg-slate-900 hover:bg-slate-800 border border-slate-800 rounded-xl flex items-center justify-between text-xs transition group">
            <div class="flex items-center gap-2.5">
              <div class="w-6 h-6 rounded border-2 border-amber-400 flex items-center justify-center font-bold text-[10px] text-amber-400">L</div>
              <span class="font-medium text-slate-200">System Load</span>
            </div>
            <i class="ph-bold ph-plus text-slate-500 group-hover:text-blue-400"></i>
          </button>

          <button onclick="spawnElement('TRANSFORMER')" class="w-full p-2.5 bg-slate-900 hover:bg-slate-800 border border-slate-800 rounded-xl flex items-center justify-between text-xs transition group">
            <div class="flex items-center gap-2.5">
              <div class="w-6 h-6 flex items-center justify-center text-cyan-400 font-bold text-sm">88</div>
              <span class="font-medium text-slate-200">Transformer</span>
            </div>
            <i class="ph-bold ph-plus text-slate-500 group-hover:text-blue-400"></i>
          </button>

          <button onclick="spawnElement('BREAKER')" class="w-full p-2.5 bg-slate-900 hover:bg-slate-800 border border-slate-800 rounded-xl flex items-center justify-between text-xs transition group">
            <div class="flex items-center gap-2.5">
              <div class="w-5 h-5 bg-emerald-950 border border-emerald-500 rounded text-[9px] font-bold text-emerald-400 flex items-center justify-center">CB</div>
              <span class="font-medium text-slate-200">Circuit Breaker</span>
            </div>
            <i class="ph-bold ph-plus text-slate-500 group-hover:text-blue-400"></i>
          </button>
        </div>
      </div>

      <hr class="border-slate-800">

      <div>
        <h3 class="text-xs font-bold text-slate-400 uppercase tracking-wider font-mono mb-2">Instructions</h3>
        <ul class="text-[11px] text-slate-400 space-y-1.5 list-disc pl-4">
          <li><strong>Drag</strong> components to position them.</li>
          <li>Click <strong>Connect Mode</strong> then click two buses (or Bus & Gen/Load) to wire them together.</li>
          <li>Click any element to edit its <strong>MW, Reactance, or Rating</strong> in the Inspector.</li>
          <li>Toggle Breakers OPEN/CLOSED to observe real-time power redistribution.</li>
        </ul>
      </div>

      <!-- System Summary Stats -->
      <div class="mt-auto bg-slate-900/80 p-3 rounded-xl border border-slate-800 text-xs space-y-1.5">
        <div class="flex justify-between text-slate-400">
          <span>Total MW Generation:</span>
          <span id="summaryTotalGen" class="font-mono text-emerald-400 font-bold">0.0 MW</span>
        </div>
        <div class="flex justify-between text-slate-400">
          <span>Total MW Load:</span>
          <span id="summaryTotalLoad" class="font-mono text-amber-400 font-bold">0.0 MW</span>
        </div>
        <div class="flex justify-between text-slate-400">
          <span>Active Lines:</span>
          <span id="summaryActiveLines" class="font-mono text-blue-400 font-bold">0</span>
        </div>
      </div>
    </aside>

    <!-- Canvas SVG Drawing Surface -->
    <main id="canvasContainer" class="flex-1 relative overflow-hidden bg-grid-bg bg-grid-dots cursor-grab active:cursor-grabbing">
      
      <!-- Floating Canvas Overlay Controls -->
      <div class="absolute top-4 left-4 z-10 flex gap-2">
        <div class="bg-grid-panel/90 backdrop-blur border border-grid-border rounded-lg p-1 flex gap-1 shadow-xl">
          <button id="zoomInBtn" class="p-1.5 hover:bg-slate-800 rounded text-slate-300" title="Zoom In">
            <i class="ph-bold ph-magnifying-glass-plus text-base"></i>
          </button>
          <button id="zoomOutBtn" class="p-1.5 hover:bg-slate-800 rounded text-slate-300" title="Zoom Out">
            <i class="ph-bold ph-magnifying-glass-minus text-base"></i>
          </button>
          <button id="resetViewBtn" class="p-1.5 hover:bg-slate-800 rounded text-slate-300" title="Fit View">
            <i class="ph-bold ph-arrows-out-line text-base"></i>
          </button>
        </div>

        <div id="connectionStatusBadge" class="hidden bg-blue-600/20 text-blue-400 border border-blue-500/30 px-3 py-1.5 rounded-lg text-xs font-semibold items-center gap-2 shadow-xl animate-pulse">
          <i class="ph-bold ph-plugs"></i>
          <span>Click first node to connect...</span>
        </div>
      </div>

      <!-- Dynamic SVG Viewport -->
      <svg id="sldSvg" class="w-full h-full min-h-full min-w-full">
        <g id="viewportGroup" transform="translate(40, 40) scale(1)">
          <!-- Dynamic rendering layers -->
          <g id="linesLayer"></g>
          <g id="transformersLayer"></g>
          <g id="shuntsLayer"></g>
          <g id="breakersLayer"></g>
          <g id="busesLayer"></g>
          <g id="generatorsLayer"></g>
          <g id="loadsLayer"></g>
          <g id="interactiveConnectLayer"></g>
        </g>
      </svg>
    </main>

    <!-- Right Side Inspector Panel -->
    <aside class="w-80 border-l border-grid-border bg-grid-panel flex flex-col shrink-0 z-20 shadow-2xl">
      <div class="p-3.5 border-b border-grid-border flex items-center justify-between bg-slate-900/50">
        <h2 class="text-xs font-bold text-slate-300 uppercase tracking-wider font-mono flex items-center gap-1.5">
          <i class="ph-bold ph-sliders-horizontal text-blue-400"></i> Element Inspector
        </h2>
        <span id="inspectTypeBadge" class="text-[10px] font-mono px-2 py-0.5 rounded bg-slate-800 text-slate-400 border border-slate-700">NONE</span>
      </div>

      <div id="inspectorContent" class="flex-1 overflow-y-auto p-4 space-y-4">
        <div id="emptyInspectState" class="text-center py-12 px-4 border border-dashed border-slate-800 rounded-xl">
          <i class="ph-duotone ph-cursor-click text-4xl text-slate-600 mb-2"></i>
          <h3 class="text-xs font-semibold text-slate-400">No Element Selected</h3>
          <p class="text-[11px] text-slate-500 mt-1">Click any element on the canvas to edit its properties or view computed power flow.</p>
        </div>

        <div id="inspectForm" class="hidden space-y-4">
          <!-- Dynamically generated controls -->
        </div>
      </div>

      <!-- Real-time Event Logger -->
      <div class="border-t border-grid-border bg-slate-950 p-3 h-32 flex flex-col">
        <div class="text-[11px] font-mono text-slate-400 mb-1 flex items-center justify-between font-semibold">
          <span><i class="ph-bold ph-terminal text-blue-400"></i> Power Flow Diagnostics</span>
          <button id="clearLogBtn" class="text-[9px] text-slate-500 hover:text-slate-300">Clear</button>
        </div>
        <div id="eventLog" class="flex-1 overflow-y-auto font-mono text-[10px] space-y-1 text-slate-400 bg-slate-900/50 rounded-lg p-2 border border-slate-800/80">
          <div class="text-slate-500">[READY] Grid solver engine initialized.</div>
        </div>
      </div>
    </aside>

  </div>

  <script>
    // System Data Model State
    let network = {
      buses: [],
      generators: [],
      loads: [],
      lines: [],
      transformers: [],
      breakers: []
    };

    let selectedElement = null;
    let wiringMode = false;
    let wireSource = null;

    // Viewport State
    let zoomLevel = 1.0;
    let panX = 40;
    let panY = 40;
    let isPanning = false;
    let startPanX = 0;
    let startPanY = 0;

    // Element Dragging State
    let draggingNode = null;
    let dragOffsetX = 0;
    let dragOffsetY = 0;

    // Sample 5-Bus Power System Preset
    const SAMPLE_5BUS_MODEL = {
      buses: [
        { id: "BUS_1", name: "Bus 1 (Corella 230kV)", kv: 230, x: 100, y: 150 },
        { id: "BUS_2", name: "Bus 2 (Ubay Tie 230kV)", kv: 230, x: 450, y: 150 },
        { id: "BUS_3", name: "Bus 3 (Corella 138kV)", kv: 138, x: 100, y: 380 },
        { id: "BUS_4", name: "Bus 4 (Tapal 69kV)", kv: 69, x: 450, y: 380 },
        { id: "BUS_5", name: "Bus 5 (Gen Bus 13.8kV)", kv: 13.8, x: 275, y: 520 }
      ],
      generators: [
        { id: "GEN_1", name: "Corella Thermal", busId: "BUS_1", pMW: 120, pMax: 200, cost: 25 },
        { id: "GEN_2", name: "Ubay Hydro", busId: "BUS_2", pMW: 80, pMax: 150, cost: 15 },
        { id: "GEN_3", name: "Bohol Solar", busId: "BUS_5", pMW: 45, pMax: 60, cost: 5 }
      ],
      loads: [
        { id: "LOAD_1", name: "City Center Load", busId: "BUS_3", pMW: 110 },
        { id: "LOAD_2", name: "Industrial Zone", busId: "BUS_4", pMW: 95 },
        { id: "LOAD_3", name: "Residential Load", busId: "BUS_2", pMW: 40 }
      ],
      lines: [
        { id: "LINE_1_2", name: "230kV Tie Line 1-2", fromBus: "BUS_1", toBus: "BUS_2", xPu: 0.05, maxMW: 100 },
        { id: "LINE_3_4", name: "138kV Substation Line 3-4", fromBus: "BUS_3", toBus: "BUS_4", xPu: 0.08, maxMW: 80 }
      ],
      transformers: [
        { id: "TR_1_3", name: "TR1 (230/138kV)", fromBus: "BUS_1", toBus: "BUS_3", xPu: 0.03, mva: 150 },
        { id: "TR_2_4", name: "TR2 (230/69kV)", fromBus: "BUS_2", toBus: "BUS_4", xPu: 0.04, mva: 120 },
        { id: "TR_4_5", name: "TR3 (69/13.8kV)", fromBus: "BUS_4", toBus: "BUS_5", xPu: 0.02, mva: 80 }
      ],
      breakers: [
        { id: "CB_1_2", name: "Breaker Line 1-2", elementId: "LINE_1_2", status: "CLOSED" },
        { id: "CB_TR1", name: "Breaker TR 1-3", elementId: "TR_1_3", status: "CLOSED" }
      ]
    };

    // Voltage Color Palette
    function getVoltageColor(kv) {
      if (kv >= 230) return '#EF4444'; // Red
      if (kv >= 138) return '#F97316'; // Orange
      if (kv >= 69) return '#10B981';  // Green
      return '#3B82F6';                // Blue
    }

    // --- POWER FLOW SOLVER ENGINE ---
    function solvePowerFlow() {
      // 1. Identify Connected Components and Bus Active Injection
      const nBuses = network.buses.length;
      if (nBuses === 0) return;

      const busMap = {};
      network.buses.forEach((b, idx) => {
        busMap[b.id] = idx;
        b.pGen = 0;
        b.pLoad = 0;
        b.pNet = 0;
        b.theta = 0; // Voltage Angle in Radians
        b.energized = false;
      });

      // Sum Generations
      network.generators.forEach(g => {
        if (busMap[g.busId] !== undefined) {
          network.buses[busMap[g.busId]].pGen += parseFloat(g.pMW || 0);
        }
      });

      // Sum Loads
      network.loads.forEach(l => {
        if (busMap[l.busId] !== undefined) {
          network.buses[busMap[l.busId]].pLoad += parseFloat(l.pMW || 0);
        }
      });

      // Compute Net Active Power Injection at each bus
      network.buses.forEach(b => {
        b.pNet = b.pGen - b.pLoad;
      });

      // Collect Active Network Branches (Lines & Transformers) that are NOT isolated by OPEN Breakers
      const branches = [];

      network.lines.forEach(line => {
        const cb = network.breakers.find(b => b.elementId === line.id);
        const isOpen = cb && cb.status === 'OPEN';
        line.isOpen = isOpen;
        if (!isOpen && busMap[line.fromBus] !== undefined && busMap[line.toBus] !== undefined) {
          branches.push({
            id: line.id,
            type: 'LINE',
            from: busMap[line.fromBus],
            to: busMap[line.toBus],
            x: parseFloat(line.xPu) || 0.05,
            maxMW: parseFloat(line.maxMW) || 100,
            ref: line
          });
        } else {
          line.pFlow = 0;
          line.loadingPct = 0;
        }
      });

      network.transformers.forEach(tr => {
        const cb = network.breakers.find(b => b.elementId === tr.id);
        const isOpen = cb && cb.status === 'OPEN';
        tr.isOpen = isOpen;
        if (!isOpen && busMap[tr.fromBus] !== undefined && busMap[tr.toBus] !== undefined) {
          branches.push({
            id: tr.id,
            type: 'TRANSFORMER',
            from: busMap[tr.fromBus],
            to: busMap[tr.toBus],
            x: parseFloat(tr.xPu) || 0.03,
            maxMW: parseFloat(tr.mva) || 100,
            ref: tr
          });
        } else {
          tr.pFlow = 0;
          tr.loadingPct = 0;
        }
      });

      // Mark Energized Buses via Graph Traversal starting from Slack Bus (Bus 0 or Bus with active Gen)
      const adj = Array.from({ length: nBuses }, () => []);
      branches.forEach(br => {
        adj[br.from].push(br.to);
        adj[br.to].push(br.from);
      });

      const queue = [0]; // Slack bus
      if (nBuses > 0) network.buses[0].energized = true;

      while (queue.length > 0) {
        const curr = queue.shift();
        adj[curr].forEach(neighbor => {
          if (!network.buses[neighbor].energized) {
            network.buses[neighbor].energized = true;
            queue.push(neighbor);
          }
        });
      }

      // Linear DC Power Flow Formulation: [B] * [Theta] = [P_net]
      // Approximated Angle Solver using Gauss-Seidel Method for robust live canvas updating
      const B = Array.from({ length: nBuses }, () => Array(nBuses).fill(0));

      branches.forEach(br => {
        const b_ij = 1.0 / (br.x || 0.01);
        B[br.from][br.from] += b_ij;
        B[br.to][br.to] += b_ij;
        B[br.from][br.to] -= b_ij;
        B[br.to][br.from] -= b_ij;
      });

      // Solve for angles (Bus 0 is reference angle theta_0 = 0)
      for (let iter = 0; iter < 40; iter++) {
        for (let i = 1; i < nBuses; i++) {
          if (!network.buses[i].energized) continue;
          let sumBTheta = 0;
          let sumB = 0;
          for (let j = 0; j < nBuses; j++) {
            if (i !== j) {
              const b_ij = B[i][j];
              sumBTheta -= b_ij * network.buses[j].theta;
              sumB -= b_ij;
            }
          }
          if (sumB > 0) {
            network.buses[i].theta = (network.buses[i].pNet / 100.0 + sumBTheta) / sumB;
          }
        }
      }

      // Compute Branch Power Flow P_ij = (theta_i - theta_j) / X_ij
      branches.forEach(br => {
        const thetaI = network.buses[br.from].theta;
        const thetaJ = network.buses[br.to].theta;
        const flowMW = ((thetaI - thetaJ) / br.x) * 100.0; // Scaled to MW

        br.ref.pFlow = Math.abs(flowMW);
        br.ref.flowDir = flowMW >= 0 ? 1 : -1; // 1: From -> To, -1: To -> From
        br.ref.loadingPct = Math.min(999, Math.round((Math.abs(flowMW) / br.maxMW) * 100));

        if (br.ref.loadingPct > 100) {
          logEvent(`OVERLOAD DETECTED: ${br.ref.name || br.ref.id} flow (${br.ref.pFlow.toFixed(1)} MW) exceeds capacity (${br.maxMW} MW)`, 'WARN');
        }
      });

      updateSummaryStats();
    }

    function updateSummaryStats() {
      const totalGen = network.generators.reduce((sum, g) => sum + parseFloat(g.pMW || 0), 0);
      const totalLoad = network.loads.reduce((sum, l) => sum + parseFloat(l.pMW || 0), 0);
      const activeLines = network.lines.filter(l => !l.isOpen).length + network.transformers.filter(t => !t.isOpen).length;

      document.getElementById('summaryTotalGen').textContent = `${totalGen.toFixed(1)} MW`;
      document.getElementById('summaryTotalLoad').textContent = `${totalLoad.toFixed(1)} MW`;
      document.getElementById('summaryActiveLines').textContent = activeLines;
    }

    // --- RENDER FUNCTION ---
    function renderCanvas() {
      solvePowerFlow();

      const linesG = document.getElementById('linesLayer');
      const trsG = document.getElementById('transformersLayer');
      const breakersG = document.getElementById('breakersLayer');
      const busesG = document.getElementById('busesLayer');
      const gensG = document.getElementById('generatorsLayer');
      const loadsG = document.getElementById('loadsLayer');

      linesG.innerHTML = '';
      trsG.innerHTML = '';
      breakersG.innerHTML = '';
      busesG.innerHTML = '';
      gensG.innerHTML = '';
      loadsG.innerHTML = '';

      const busMap = {};
      network.buses.forEach(b => busMap[b.id] = b);

      // 1. Render Transmission Lines
      network.lines.forEach(line => {
        const b1 = busMap[line.fromBus];
        const b2 = busMap[line.toBus];
        if (!b1 || !b2) return;

        const isOverloaded = line.loadingPct > 100;
        const color = line.isOpen ? '#475569' : (isOverloaded ? '#EF4444' : '#38BDF8');

        const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.setAttribute('class', 'cursor-pointer');
        g.onclick = (e) => { e.stopPropagation(); selectElement('LINE', line); };

        const path = document.createElementNS('http://www.w3.org/2000/svg', 'line');
        path.setAttribute('x1', b1.x + 40); path.setAttribute('y1', b1.y + 4);
        path.setAttribute('x2', b2.x + 40); path.setAttribute('y2', b2.y + 4);
        path.setAttribute('stroke', color);
        path.setAttribute('stroke-width', isOverloaded ? '4' : '2.5');
        if (isOverloaded) path.setAttribute('class', 'animate-pulse');

        g.appendChild(path);

        // Animated Power Flow Particles
        if (!line.isOpen && line.pFlow > 0.1) {
          const flowPath = document.createElementNS('http://www.w3.org/2000/svg', 'line');
          flowPath.setAttribute('x1', b1.x + 40); flowPath.setAttribute('y1', b1.y + 4);
          flowPath.setAttribute('x2', b2.x + 40); flowPath.setAttribute('y2', b2.y + 4);
          flowPath.setAttribute('stroke', '#FFFFFF');
          flowPath.setAttribute('stroke-width', '2');
          flowPath.setAttribute('class', `flow-line ${line.flowDir < 0 ? 'flow-reverse' : ''}`);
          g.appendChild(flowPath);
        }

        // Flow MW Label
        const midX = (b1.x + b2.x) / 2 + 40;
        const midY = (b1.y + b2.y) / 2;
        const text = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        text.setAttribute('x', midX); text.setAttribute('y', midY - 8);
        text.setAttribute('fill', isOverloaded ? '#EF4444' : '#94A3B8');
        text.setAttribute('font-size', '10');
        text.setAttribute('font-mono', 'true');
        text.setAttribute('text-anchor', 'middle');
        text.textContent = `${(line.pFlow || 0).toFixed(1)} MW (${line.loadingPct || 0}%)`;

        g.appendChild(text);
        linesG.appendChild(g);
      });

      // 2. Render Transformers
      network.transformers.forEach(tr => {
        const b1 = busMap[tr.fromBus];
        const b2 = busMap[tr.toBus];
        if (!b1 || !b2) return;

        const isOverloaded = tr.loadingPct > 100;
        const color = tr.isOpen ? '#475569' : (isOverloaded ? '#EF4444' : '#06B6D4');
        const midX = (b1.x + b2.x) / 2 + 40;
        const midY = (b1.y + b2.y) / 2 + 4;

        const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.setAttribute('class', 'cursor-pointer');
        g.onclick = (e) => { e.stopPropagation(); selectElement('TRANSFORMER', tr); };

        const line1 = document.createElementNS('http://www.w3.org/2000/svg', 'line');
        line1.setAttribute('x1', b1.x + 40); line1.setAttribute('y1', b1.y + 4);
        line1.setAttribute('x2', midX); line1.setAttribute('y2', midY);
        line1.setAttribute('stroke', color); line1.setAttribute('stroke-width', '2');

        const line2 = document.createElementNS('http://www.w3.org/2000/svg', 'line');
        line2.setAttribute('x1', midX); line2.setAttribute('y1', midY);
        line2.setAttribute('x2', b2.x + 40); line2.setAttribute('y2', b2.y + 4);
        line2.setAttribute('stroke', color); line2.setAttribute('stroke-width', '2');

        // Dual interlocking circles
        const c1 = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
        c1.setAttribute('cx', midX - 6); c1.setAttribute('cy', midY); c1.setAttribute('r', '8');
        c1.setAttribute('fill', '#0F172A'); c1.setAttribute('stroke', color); c1.setAttribute('stroke-width', '2');

        const c2 = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
        c2.setAttribute('cx', midX + 6); c2.setAttribute('cy', midY); c2.setAttribute('r', '8');
        c2.setAttribute('fill', '#0F172A'); c2.setAttribute('stroke', color); c2.setAttribute('stroke-width', '2');

        const label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        label.setAttribute('x', midX); label.setAttribute('y', midY + 20);
        label.setAttribute('fill', '#94A3B8'); label.setAttribute('font-size', '9');
        label.setAttribute('font-mono', 'true'); label.setAttribute('text-anchor', 'middle');
        label.textContent = `${(tr.pFlow || 0).toFixed(1)} MW (${tr.loadingPct || 0}%)`;

        g.appendChild(line1); g.appendChild(line2);
        g.appendChild(c1); g.appendChild(c2); g.appendChild(label);
        trsG.appendChild(g);
      });

      // 3. Render Circuit Breakers
      network.breakers.forEach(cb => {
        const isClosed = cb.status === 'CLOSED';
        const targetLine = network.lines.find(l => l.id === cb.elementId) || network.transformers.find(t => t.id === cb.elementId);
        if (!targetLine) return;

        const b1 = busMap[targetLine.fromBus];
        const b2 = busMap[targetLine.toBus];
        if (!b1 || !b2) return;

        const cbX = (b1.x * 0.7 + b2.x * 0.3) + 40;
        const cbY = (b1.y * 0.7 + b2.y * 0.3) + 4;

        const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.setAttribute('transform', `translate(${cbX}, ${cbY})`);
        g.setAttribute('class', 'cursor-pointer');
        g.onclick = (e) => { e.stopPropagation(); toggleBreaker(cb.id); selectElement('BREAKER', cb); };

        const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
        rect.setAttribute('x', '-10'); rect.setAttribute('y', '-10');
        rect.setAttribute('width', '20'); rect.setAttribute('height', '20');
        rect.setAttribute('rx', '4');
        rect.setAttribute('fill', isClosed ? '#064E3B' : '#7F1D1D');
        rect.setAttribute('stroke', isClosed ? '#10B981' : '#EF4444');
        rect.setAttribute('stroke-width', '2');

        const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        txt.setAttribute('text-anchor', 'middle'); txt.setAttribute('y', '3.5');
        txt.setAttribute('fill', '#FFFFFF'); txt.setAttribute('font-size', '8');
        txt.setAttribute('font-weight', 'bold');
        txt.textContent = isClosed ? 'CB' : 'X';

        g.appendChild(rect); g.appendChild(txt);
        breakersG.appendChild(g);
      });

      // 4. Render Buses
      network.buses.forEach(bus => {
        const color = bus.energized ? getVoltageColor(bus.kv) : '#475569';
        const isSelected = selectedElement && selectedElement.data.id === bus.id;

        const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.setAttribute('class', 'cursor-move');
        g.onmousedown = (e) => startDragNode(e, 'BUS', bus);
        g.onclick = (e) => { e.stopPropagation(); handleNodeClick('BUS', bus); };

        const bar = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
        bar.setAttribute('x', bus.x); bar.setAttribute('y', bus.y);
        bar.setAttribute('width', '80'); bar.setAttribute('height', '8');
        bar.setAttribute('rx', '4');
        bar.setAttribute('fill', color);
        if (isSelected) {
          bar.setAttribute('stroke', '#FFFFFF');
          bar.setAttribute('stroke-width', '2');
        }

        const label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        label.setAttribute('x', bus.x + 40); label.setAttribute('y', bus.y - 8);
        label.setAttribute('fill', '#F8FAFC'); label.setAttribute('font-size', '11');
        label.setAttribute('font-weight', 'bold'); label.setAttribute('text-anchor', 'middle');
        label.textContent = bus.name;

        const sub = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        sub.setAttribute('x', bus.x + 40); sub.setAttribute('y', bus.y + 20);
        sub.setAttribute('fill', '#94A3B8'); sub.setAttribute('font-size', '9');
        sub.setAttribute('font-mono', 'true'); sub.setAttribute('text-anchor', 'middle');
        sub.textContent = `${bus.kv}kV | Net: ${bus.pNet.toFixed(1)}MW`;

        g.appendChild(bar); g.appendChild(label); g.appendChild(sub);
        busesG.appendChild(g);
      });

      // 5. Render Generators
      network.generators.forEach(gen => {
        const bus = busMap[gen.busId];
        if (!bus) return;

        const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.setAttribute('class', 'cursor-pointer');
        g.onclick = (e) => { e.stopPropagation(); selectElement('GENERATOR', gen); };

        const gx = bus.x + 40;
        const gy = bus.y - 35;

        const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
        line.setAttribute('x1', gx); line.setAttribute('y1', gy + 12);
        line.setAttribute('x2', gx); line.setAttribute('y2', bus.y);
        line.setAttribute('stroke', '#10B981'); line.setAttribute('stroke-width', '2');

        const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
        circle.setAttribute('cx', gx); circle.setAttribute('cy', gy); circle.setAttribute('r', '12');
        circle.setAttribute('fill', '#064E3B'); circle.setAttribute('stroke', '#10B981'); circle.setAttribute('stroke-width', '2');

        const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        txt.setAttribute('x', gx); txt.setAttribute('y', gy + 4);
        txt.setAttribute('text-anchor', 'middle'); txt.setAttribute('fill', '#10B981');
        txt.setAttribute('font-size', '10'); txt.setAttribute('font-weight', 'bold');
        txt.textContent = 'G';

        const label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        label.setAttribute('x', gx); label.setAttribute('y', gy - 16);
        label.setAttribute('text-anchor', 'middle'); label.setAttribute('fill', '#10B981');
        label.setAttribute('font-size', '9'); label.setAttribute('font-mono', 'true');
        label.textContent = `${gen.pMW}MW`;

        g.appendChild(line); g.appendChild(circle); g.appendChild(txt); g.appendChild(label);
        gensG.appendChild(g);
      });

      // 6. Render Loads
      network.loads.forEach(load => {
        const bus = busMap[load.busId];
        if (!bus) return;

        const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
        g.setAttribute('class', 'cursor-pointer');
        g.onclick = (e) => { e.stopPropagation(); selectElement('LOAD', load); };

        const lx = bus.x + 40;
        const ly = bus.y + 35;

        const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
        line.setAttribute('x1', lx); line.setAttribute('y1', bus.y + 8);
        line.setAttribute('x2', lx); line.setAttribute('y2', ly - 10);
        line.setAttribute('stroke', '#F59E0B'); line.setAttribute('stroke-width', '2');

        const poly = document.createElementNS('http://www.w3.org/2000/svg', 'polygon');
        poly.setAttribute('points', `${lx-10},${ly-10} ${lx+10},${ly-10} ${lx},${ly+8}`);
        poly.setAttribute('fill', '#78350F'); poly.setAttribute('stroke', '#F59E0B'); poly.setAttribute('stroke-width', '2');

        const label = document.createElementNS('http://www.w3.org/2000/svg', 'text');
        label.setAttribute('x', lx); label.setAttribute('y', ly + 20);
        label.setAttribute('text-anchor', 'middle'); label.setAttribute('fill', '#F59E0B');
        label.setAttribute('font-size', '9'); label.setAttribute('font-mono', 'true');
        label.textContent = `${load.pMW}MW`;

        g.appendChild(line); g.appendChild(poly); g.appendChild(label);
        loadsG.appendChild(g);
      });
    }

    // --- INTERACTION & INSPECTOR HANDLERS ---
    function selectElement(type, data) {
      selectedElement = { type, data };
      const emptyState = document.getElementById('emptyInspectState');
      const form = document.getElementById('inspectForm');
      const badge = document.getElementById('inspectTypeBadge');

      emptyState.classList.add('hidden');
      form.classList.remove('hidden');
      badge.textContent = type;

      let fieldsHtml = `
        <div>
          <label class="text-[11px] text-slate-400 font-semibold">Element Identifier</label>
          <input type="text" value="${data.name || data.id}" onchange="updateAttr('name', this.value)" 
            class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1 focus:outline-none focus:border-blue-500">
        </div>
      `;

      if (type === 'BUS') {
        fieldsHtml += `
          <div>
            <label class="text-[11px] text-slate-400 font-semibold">Nominal Voltage (kV)</label>
            <input type="number" value="${data.kv}" onchange="updateAttr('kv', parseFloat(this.value))" 
              class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1">
          </div>
          <div class="bg-slate-900 p-2.5 rounded-lg border border-slate-800 text-xs space-y-1 font-mono">
            <div class="flex justify-between"><span>Calculated Angle:</span><span class="text-blue-400">${(data.theta || 0).toFixed(3)} rad</span></div>
            <div class="flex justify-between"><span>Net MW Injection:</span><span class="text-emerald-400">${(data.pNet || 0).toFixed(1)} MW</span></div>
          </div>
        `;
      } else if (type === 'GENERATOR') {
        fieldsHtml += `
          <div>
            <label class="text-[11px] text-slate-400 font-semibold flex justify-between">
              <span>Active Dispatch (MW)</span>
              <span class="text-emerald-400 font-mono">${data.pMW} MW</span>
            </label>
            <input type="range" min="0" max="${data.pMax}" step="1" value="${data.pMW}" oninput="updateAttr('pMW', parseFloat(this.value))" 
              class="w-full accent-emerald-500 bg-slate-800 h-1.5 rounded-lg appearance-none cursor-pointer mt-2">
          </div>
          <div>
            <label class="text-[11px] text-slate-400 font-semibold">Max Capacity Pmax (MW)</label>
            <input type="number" value="${data.pMax}" onchange="updateAttr('pMax', parseFloat(this.value))" 
              class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1">
          </div>
          <div>
            <label class="text-[11px] text-slate-400 font-semibold">Marginal Cost ($/MWh)</label>
            <input type="number" value="${data.cost || 20}" onchange="updateAttr('cost', parseFloat(this.value))" 
              class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1">
          </div>
        `;
      } else if (type === 'LOAD') {
        fieldsHtml += `
          <div>
            <label class="text-[11px] text-slate-400 font-semibold">Active Demand (MW)</label>
            <input type="number" value="${data.pMW}" onchange="updateAttr('pMW', parseFloat(this.value))" 
              class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1">
          </div>
        `;
      } else if (type === 'LINE') {
        fieldsHtml += `
          <div>
            <label class="text-[11px] text-slate-400 font-semibold">Per-Unit Reactance X (pu)</label>
            <input type="number" step="0.01" value="${data.xPu}" onchange="updateAttr('xPu', parseFloat(this.value))" 
              class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1">
          </div>
          <div>
            <label class="text-[11px] text-slate-400 font-semibold">Thermal Rating Limit (MW)</label>
            <input type="number" value="${data.maxMW}" onchange="updateAttr('maxMW', parseFloat(this.value))" 
              class="w-full bg-slate-900 border border-slate-800 rounded-lg p-2 text-xs font-mono text-slate-200 mt-1">
          </div>
          <div class="bg-slate-900 p-2.5 rounded-lg border border-slate-800 text-xs space-y-1 font-mono">
            <div class="flex justify-between"><span>Computed Active Flow:</span><span class="text-blue-400">${(data.pFlow || 0).toFixed(1)} MW</span></div>
            <div class="flex justify-between"><span>Loading Percentage:</span><span class="${data.loadingPct > 100 ? 'text-red-400 font-bold' : 'text-emerald-400'}">${data.loadingPct || 0}%</span></div>
          </div>
        `;
      } else if (type === 'BREAKER') {
        fieldsHtml += `
          <button onclick="toggleBreaker('${data.id}')" class="w-full py-2.5 px-3 rounded-xl text-xs font-semibold flex items-center justify-center gap-2 ${data.status === 'CLOSED' ? 'bg-red-500/20 text-red-400 border border-red-500/30' : 'bg-emerald-500/20 text-emerald-400 border border-emerald-500/30'}">
            <i class="ph-bold ph-power"></i> ${data.status === 'CLOSED' ? 'TRIP / OPEN BREAKER' : 'CLOSE BREAKER'}
          </button>
        `;
      }

      fieldsHtml += `
        <button onclick="deleteSelectedElement()" class="w-full py-2 px-3 bg-red-950/40 hover:bg-red-900/50 text-red-300 border border-red-800/40 rounded-xl text-xs font-medium transition flex items-center justify-center gap-1.5 mt-4">
          <i class="ph-bold ph-trash"></i> Delete Element
        </button>
      `;

      form.innerHTML = fieldsHtml;
      renderCanvas();
    }

    function updateAttr(key, val) {
      if (selectedElement && selectedElement.data) {
        selectedElement.data[key] = val;
        renderCanvas();
      }
    }

    function toggleBreaker(cbId) {
      const cb = network.breakers.find(b => b.id === cbId);
      if (cb) {
        cb.status = cb.status === 'CLOSED' ? 'OPEN' : 'CLOSED';
        logEvent(`Circuit Breaker ${cb.id} switched to ${cb.status}`, cb.status === 'OPEN' ? 'WARN' : 'INFO');
        renderCanvas();
        if (selectedElement && selectedElement.data.id === cbId) selectElement('BREAKER', cb);
      }
    }

    function spawnElement(type) {
      const id = `${type}_${Date.now().toString().slice(-4)}`;
      if (type === 'BUS') {
        network.buses.push({ id, name: `Bus ${network.buses.length + 1}`, kv: 230, x: 200, y: 200 });
      } else if (type === 'GENERATOR' && network.buses.length > 0) {
        network.generators.push({ id, name: `Gen ${id}`, busId: network.buses[0].id, pMW: 50, pMax: 100, cost: 20 });
      } else if (type === 'LOAD' && network.buses.length > 0) {
        network.loads.push({ id, name: `Load ${id}`, busId: network.buses[0].id, pMW: 40 });
      } else if (type === 'TRANSFORMER' && network.buses.length >= 2) {
        network.transformers.push({ id, name: `TR ${id}`, fromBus: network.buses[0].id, toBus: network.buses[1].id, xPu: 0.04, mva: 100 });
      } else if (type === 'BREAKER' && network.lines.length > 0) {
        network.breakers.push({ id, name: `CB ${id}`, elementId: network.lines[0].id, status: 'CLOSED' });
      } else {
        logEvent('Need at least 1 or 2 buses created before placing attached components.', 'WARN');
        return;
      }
      logEvent(`Created new element: ${id}`);
      renderCanvas();
    }

    function deleteSelectedElement() {
      if (!selectedElement) return;
      const { type, data } = selectedElement;

      if (type === 'BUS') network.buses = network.buses.filter(b => b.id !== data.id);
      else if (type === 'GENERATOR') network.generators = network.generators.filter(g => g.id !== data.id);
      else if (type === 'LOAD') network.loads = network.loads.filter(l => l.id !== data.id);
      else if (type === 'LINE') network.lines = network.lines.filter(l => l.id !== data.id);
      else if (type === 'TRANSFORMER') network.transformers = network.transformers.filter(t => t.id !== data.id);
      else if (type === 'BREAKER') network.breakers = network.breakers.filter(c => c.id !== data.id);

      selectedElement = null;
      document.getElementById('emptyInspectState').classList.remove('hidden');
      document.getElementById('inspectForm').classList.add('hidden');
      renderCanvas();
    }

    // Wiring Mode Manager
    function handleNodeClick(type, node) {
      if (!wiringMode) return;
      if (!wireSource) {
        wireSource = node;
        document.getElementById('connectionStatusBadge').children[1].textContent = `Connected from ${node.name}. Click target bus...`;
      } else {
        if (wireSource.id !== node.id) {
          const id = `LINE_${Date.now().toString().slice(-4)}`;
          network.lines.push({
            id,
            name: `Line ${wireSource.name} - ${node.name}`,
            fromBus: wireSource.id,
            toBus: node.id,
            xPu: 0.05,
            maxMW: 100
          });
          logEvent(`Wired transmission path: ${wireSource.name} <-> ${node.name}`);
        }
        wireSource = null;
        toggleWiringMode(false);
        renderCanvas();
      }
    }

    function toggleWiringMode(active) {
      wiringMode = active;
      const btn = document.getElementById('wireToolBtn');
      const badge = document.getElementById('connectionStatusBadge');
      const text = document.getElementById('wireToolText');

      if (wiringMode) {
        btn.classList.replace('bg-slate-800', 'bg-blue-600');
        badge.classList.remove('hidden');
        badge.classList.add('flex');
        text.textContent = 'Connect Mode (Active)';
      } else {
        btn.classList.replace('bg-blue-600', 'bg-slate-800');
        badge.classList.add('hidden');
        badge.classList.remove('flex');
        text.textContent = 'Connect Mode (Off)';
        wireSource = null;
      }
    }

    function startDragNode(e, type, node) {
      if (wiringMode) return;
      draggingNode = node;
      startPanX = e.clientX - node.x;
      startPanY = e.clientY - node.y;
    }

    function logEvent(msg, type = 'INFO') {
      const container = document.getElementById('eventLog');
      const time = new Date().toLocaleTimeString();
      const div = document.createElement('div');
      div.className = type === 'WARN' ? 'text-amber-400 font-bold' : 'text-slate-300';
      div.textContent = `[${time}] ${msg}`;
      container.appendChild(div);
      container.scrollTop = container.scrollHeight;
    }

    // Pan & Zoom Setup
    function initViewportPanZoom() {
      const container = document.getElementById('canvasContainer');
      const viewportGroup = document.getElementById('viewportGroup');

      function updateTransform() {
        viewportGroup.setAttribute('transform', `translate(${panX}, ${panY}) scale(${zoomLevel})`);
      }

      container.addEventListener('mousedown', (e) => {
        if (e.target.closest('.cursor-pointer') || e.target.closest('.cursor-move')) return;
        isPanning = true;
        startPanX = e.clientX - panX;
        startPanY = e.clientY - panY;
      });

      window.addEventListener('mousemove', (e) => {
        if (draggingNode) {
          draggingNode.x = e.clientX - startPanX;
          draggingNode.y = e.clientY - startPanY;
          renderCanvas();
        } else if (isPanning) {
          panX = e.clientX - startPanX;
          panY = e.clientY - startPanY;
          updateTransform();
        }
      });

      window.addEventListener('mouseup', () => {
        isPanning = false;
        draggingNode = null;
      });

      container.addEventListener('wheel', (e) => {
        e.preventDefault();
        const factor = e.deltaY < 0 ? 1.1 : 0.9;
        zoomLevel = Math.min(Math.max(0.3, zoomLevel * factor), 2.5);
        updateTransform();
      }, { passive: false });

      document.getElementById('zoomInBtn').onclick = () => { zoomLevel = Math.min(2.5, zoomLevel * 1.2); updateTransform(); };
      document.getElementById('zoomOutBtn').onclick = () => { zoomLevel = Math.max(0.3, zoomLevel / 1.2); updateTransform(); };
      document.getElementById('resetViewBtn').onclick = () => { zoomLevel = 1.0; panX = 40; panY = 40; updateTransform(); };
    }

    // Initializer
    window.onload = function() {
      initViewportPanZoom();

      // Load initial Sample 5-Bus Model
      network = JSON.parse(JSON.stringify(SAMPLE_5BUS_MODEL));

      document.getElementById('solveFlowBtn').onclick = () => { renderCanvas(); logEvent('Power flow manually solved.', 'INFO'); };
      document.getElementById('wireToolBtn').onclick = () => toggleWiringMode(!wiringMode);
      document.getElementById('loadSampleBtn').onclick = () => { network = JSON.parse(JSON.stringify(SAMPLE_5BUS_MODEL)); renderCanvas(); logEvent('Loaded Sample 5-Bus Grid Model.'); };
      document.getElementById('clearCanvasBtn').onclick = () => { network = { buses: [], generators: [], loads: [], lines: [], transformers: [], breakers: [] }; renderCanvas(); logEvent('Canvas cleared.'); };
      document.getElementById('clearLogBtn').onclick = () => { document.getElementById('eventLog').innerHTML = ''; };

      // JSON Export/Import
      document.getElementById('exportJsonBtn').onclick = () => {
        const str = "data:text/json;charset=utf-8," + encodeURIComponent(JSON.stringify(network, null, 2));
        const a = document.createElement('a');
        a.setAttribute("href", str);
        a.setAttribute("download", "network_model.json");
        document.body.appendChild(a);
        a.click();
        a.remove();
      };

      const importInput = document.getElementById('importFileInput');
      document.getElementById('importJsonBtn').onclick = () => importInput.click();
      importInput.onchange = (e) => {
        const file = e.target.files[0];
        if (file) {
          const reader = new FileReader();
          reader.onload = (evt) => {
            network = JSON.parse(evt.target.result);
            renderCanvas();
            logEvent('Imported external JSON network model.');
          };
          reader.readAsText(file);
        }
      };

      renderCanvas();
    };
  </script>
</body>
</html>
