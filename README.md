<html lang="en" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IEMOP Visayas Grid Market Network Model SLD & Flow Monitoring</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    colors: {
                        v500: '#8b5cf6', // 500 kV Purple
                        v230: '#ef4444', // 230 kV Red
                        v138: '#f97316', // 138 kV Orange
                        v69:  '#06b6d4', // 69 kV Cyan
                        v13:  '#10b981', // 13.8 kV Green
                    }
                }
            }
        }
    </script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .mono { font-family: 'JetBrains Mono', monospace; }
        
        /* Line Flow Animation Particles */
        .flow-particle {
            stroke-dasharray: 6, 6;
            animation: flowMove 1.5s linear infinite;
        }
        @keyframes flowMove {
            from { stroke-dashoffset: 24; }
            to { stroke-dashoffset: 0; }
        }
        .flow-particle-reverse {
            stroke-dasharray: 6, 6;
            animation: flowMoveRev 1.5s linear infinite;
        }
        @keyframes flowMoveRev {
            from { stroke-dashoffset: 0; }
            to { stroke-dashoffset: 24; }
        }
        
        /* Glassmorphism overlays */
        .glass-panel {
            background: rgba(15, 23, 42, 0.85);
            backdrop-filter: blur(8px);
        }
        .light .glass-panel {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(8px);
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col transition-colors duration-300">

    <!-- Top Navigation / IEMOP Market Operator Header -->
    <header class="bg-slate-900 dark:bg-slate-900 border-b border-slate-800 px-6 py-3.5 flex flex-wrap justify-between items-center sticky top-0 z-50 shadow-md">
        <div class="flex items-center space-x-4">
            <div class="bg-cyan-500/10 p-2 rounded-lg border border-cyan-500/30">
                <svg class="w-6 h-6 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"/>
                </svg>
            </div>
            <div>
                <div class="flex items-center space-x-2">
                    <h1 class="text-base font-bold tracking-wide text-white">INDEPENDENT ELECTRICITY MARKET OPERATOR OF THE PHILIPPINES</h1>
                    <span class="text-xs bg-cyan-500/20 text-cyan-300 font-mono px-2 py-0.5 rounded border border-cyan-500/30">IEMOP MNM 2026</span>
                </div>
                <p class="text-xs text-slate-400">Visayas Grid Bus-Oriented Market Network Model — Real-Time Market Flow & LMP Monitor</p>
            </div>
        </div>

        <div class="flex items-center space-x-4 text-xs mt-2 sm:mt-0">
            <!-- Market Status Badges -->
            <div class="bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700">
                <span class="text-slate-400 block text-[10px] font-semibold">MARKET RUN</span>
                <span class="mono font-bold text-emerald-400 text-sm">RTD 5-MIN #142</span>
            </div>
            <div class="bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700">
                <span class="text-slate-400 block text-[10px] font-semibold">VISAYAS SYSTEM LMP</span>
                <span id="hdr-avg-lmp" class="mono font-bold text-amber-400 text-sm">₱4,285.50 / MWh</span>
            </div>
            <div class="bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700 text-center">
                <span class="text-slate-400 block text-[10px] font-semibold">NEXT INTERVAL</span>
                <span id="poll-timer" class="mono font-bold text-cyan-400 text-sm">04:45</span>
            </div>
            <!-- Theme Toggle -->
            <button onclick="toggleDarkMode()" class="p-2 rounded-lg bg-slate-800 border border-slate-700 text-slate-300 hover:bg-slate-700 transition" title="Toggle Light/Dark Theme">
                <svg id="theme-icon" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"/>
                </svg>
            </button>
        </div>
    </header>

    <!-- Navigation Bar / View Switcher -->
    <nav class="bg-slate-900/80 border-b border-slate-800 px-6 py-2 flex justify-between items-center">
        <div class="flex space-x-3">
            <button onclick="switchTab('sld')" id="btn-sld" class="px-4 py-1.5 rounded-md text-xs font-semibold bg-cyan-500 text-slate-950 transition shadow">
                Graphical Market SLD Diagram
            </button>
            <button onclick="switchTab('market-summary')" id="btn-market-summary" class="px-4 py-1.5 rounded-md text-xs font-medium text-slate-300 hover:bg-slate-800 transition">
                Regional Telemetry (Sheet 4)
            </button>
            <button onclick="switchTab('definitions')" id="btn-definitions" class="px-4 py-1.5 rounded-md text-xs font-medium text-slate-300 hover:bg-slate-800 transition">
                Market Definitions (Sheet 2)
            </button>
            <button onclick="switchTab('register')" id="btn-register" class="px-4 py-1.5 rounded-md text-xs font-medium text-slate-300 hover:bg-slate-800 transition">
                Substation Register (Sheet 3)
            </button>
        </div>

        <!-- Market Controls & Simulation Feed -->
        <div class="flex items-center space-x-3 text-xs">
            <span class="text-slate-400 font-medium">Market Dispatch Mode:</span>
            <select id="dispatch-scenario" onchange="changeScenario(this.value)" class="bg-slate-800 border border-slate-700 text-slate-200 rounded px-2.5 py-1 focus:outline-none focus:border-cyan-500">
                <option value="normal">Real-Time Dispatch (RTD Live)</option>
                <option value="peak">Peak Demand Simulation (2,779 MW)</option>
                <option value="congestion">Congestion Test Mode (Panay Export Limit)</option>
            </select>
        </div>
    </nav>

    <!-- Main Workspace -->
    <main class="flex-1 relative overflow-hidden flex flex-col">

        <!-- TAB 1: GRAPHICAL SINGLE LINE DIAGRAM CANVAS -->
        <div id="tab-sld" class="flex-1 relative bg-slate-950 dark:bg-slate-950 overflow-hidden flex">
            
            <!-- Voltage Level Legend & Flow Controls Overlay -->
            <div class="absolute top-4 left-4 z-20 glass-panel p-3.5 rounded-xl border border-slate-800 text-xs shadow-xl space-y-2 max-w-xs">
                <div class="font-bold text-slate-200 border-b border-slate-700/60 pb-1.5 flex justify-between items-center">
                    <span>VOLTAGE LEGEND (IEMOP)</span>
                    <span class="text-[10px] text-slate-400 font-mono">MNM 2026-08</span>
                </div>
                <div class="grid grid-cols-2 gap-2 text-[11px] font-mono">
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v500 inline-block"></span><span>500 kV Transmission</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v230 inline-block"></span><span>230 kV Transmission</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v138 inline-block"></span><span>138 kV Transmission</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v69 inline-block"></span><span>69 kV Sub-trans</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v13 inline-block"></span><span>13.8 kV Gen/Local</span></div>
                </div>
                <div class="pt-2 border-t border-slate-700/60 text-[10px] text-slate-400">
                    <span class="text-cyan-400 font-semibold">Interactive:</span> Hover/Click any node or generator to view market dispatch & LMP.
                </div>
            </div>

            <!-- Canvas Zoom Controls -->
            <div class="absolute bottom-6 left-4 z-20 flex space-x-2">
                <button onclick="zoomCanvas(1.2)" class="bg-slate-800 hover:bg-slate-700 text-slate-200 p-2 rounded-lg border border-slate-700 shadow font-mono font-bold">+</button>
                <button onclick="zoomCanvas(0.8)" class="bg-slate-800 hover:bg-slate-700 text-slate-200 p-2 rounded-lg border border-slate-700 shadow font-mono font-bold">-</button>
                <button onclick="resetZoom()" class="bg-slate-800 hover:bg-slate-700 text-slate-200 px-3 py-2 rounded-lg border border-slate-700 shadow text-xs font-semibold">Reset View</button>
            </div>

            <!-- Inter-Island Corridor Summary Floating Bar -->
            <div class="absolute bottom-6 right-6 z-20 glass-panel p-3.5 rounded-xl border border-slate-800 shadow-xl hidden md:block">
                <div class="text-[11px] font-bold text-slate-300 mb-2 uppercase tracking-wide">Major Market Corridors (MW Transfer)</div>
                <div class="flex space-x-4 text-xs font-mono">
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">PANAY ↔ NEGROS</span>
                        <span id="flow-val-pan-neg" class="font-bold text-emerald-400">124.20 MW</span>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">NEGROS ↔ CEBU (230kV)</span>
                        <span id="flow-val-neg-ceb" class="font-bold text-rose-400">-76.92 MW</span>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">LEYTE ↔ CEBU</span>
                        <span id="flow-val-ley-ceb" class="font-bold text-emerald-400">55.27 MW</span>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">MINDANAO ↔ VISAYAS</span>
                        <span id="flow-val-min-vis" class="font-bold text-amber-400">179.05 MW</span>
                    </div>
                </div>
            </div>

            <!-- SVG Single Line Diagram Container -->
            <div id="svg-viewport" class="w-full h-full cursor-grab active:cursor-grabbing overflow-hidden flex items-center justify-center">
                <svg id="sld-canvas" viewBox="0 0 1600 1000" class="w-full h-full transition-transform duration-100 ease-out">
                    <defs>
                        <!-- Glow filter for energized buses -->
                        <filter id="glow-230" x="-20%" y="-20%" width="140%" height="140%">
                            <feGaussianBlur stdDeviation="3" result="blur" />
                            <feComposite in="SourceGraphic" in2="blur" operator="over" />
                        </filter>
                    </defs>

                    <!-- Background Grid Pattern -->
                    <rect width="1600" height="1000" fill="#030712"/>
                    <g opacity="0.05" stroke="#ffffff" stroke-width="0.5">
                        <path d="M 0 100 H 1600 M 0 200 H 1600 M 0 300 H 1600 M 0 400 H 1600 M 0 500 H 1600 M 0 600 H 1600 M 0 700 H 1600 M 0 800 H 1600 M 0 900 H 1600" />
                        <path d="M 100 0 V 1000 M 200 0 V 1000 M 300 0 V 1000 M 400 0 V 1000 M 500 0 V 1000 M 600 0 V 1000 M 700 0 V 1000 M 800 0 V 1000 M 900 0 V 1000 M 1000 0 V 1000 M 1100 0 V 1000 M 1200 0 V 1000 M 1300 0 V 1000 M 1400 0 V 1000 M 1500 0 V 1000" />
                    </g>

                    <!-- REGIONAL BOUNDARY BOXES (Matching IEMOP MNM SLD PDF) -->
                    <!-- PANAY REGION -->
                    <rect x="30" y="30" width="310" height="910" fill="#0f172a" fill-opacity="0.4" stroke="#f97316" stroke-width="1.5" stroke-dasharray="6,4" rx="8"/>
                    <text x="45" y="55" fill="#f97316" font-size="16" font-weight="bold" font-family="Inter">PANAY SUB-GRID</text>
                    
                    <!-- NEGROS REGION -->
                    <rect x="360" y="30" width="310" height="910" fill="#0f172a" fill-opacity="0.4" stroke="#ef4444" stroke-width="1.5" stroke-dasharray="6,4" rx="8"/>
                    <text x="375" y="55" fill="#ef4444" font-size="16" font-weight="bold" font-family="Inter">NEGROS SUB-GRID</text>

                    <!-- CEBU REGION -->
                    <rect x="690" y="30" width="310" height="910" fill="#0f172a" fill-opacity="0.4" stroke="#06b6d4" stroke-width="1.5" stroke-dasharray="6,4" rx="8"/>
                    <text x="705" y="55" fill="#06b6d4" font-size="16" font-weight="bold" font-family="Inter">CEBU SUB-GRID</text>

                    <!-- LEYTE - SAMAR REGION -->
                    <rect x="1020" y="30" width="310" height="650" fill="#0f172a" fill-opacity="0.4" stroke="#10b981" stroke-width="1.5" stroke-dasharray="6,4" rx="8"/>
                    <text x="1035" y="55" fill="#10b981" font-size="16" font-weight="bold" font-family="Inter">LEYTE–SAMAR SUB-GRID</text>

                    <!-- BOHOL REGION -->
                    <rect x="1020" y="700" width="310" height="240" fill="#0f172a" fill-opacity="0.4" stroke="#8b5cf6" stroke-width="1.5" stroke-dasharray="6,4" rx="8"/>
                    <text x="1035" y="725" fill="#8b5cf6" font-size="16" font-weight="bold" font-family="Inter">BOHOL SUB-GRID</text>

                    <!-- INTERCONNECTION CORRIDOR LINES & ANIMATED FLOW PARTICLES -->
                    <!-- Panay <-> Negros Submarine Cable Line -->
                    <path id="line-pan-neg" d="M 310 160 L 380 160" stroke="#f97316" stroke-width="3" />
                    <path d="M 310 160 L 380 160" stroke="#38bdf8" stroke-width="3" class="flow-particle" />

                    <!-- Negros <-> Cebu 138kV / 230kV Tie Lines -->
                    <path d="M 640 280 L 710 280" stroke="#ef4444" stroke-width="3" />
                    <path d="M 640 280 L 710 280" stroke="#ef4444" stroke-width="3" class="flow-particle-reverse" />

                    <!-- Cebu <-> Leyte Submarine Line -->
                    <path d="M 970 200 L 1040 200" stroke="#ef4444" stroke-width="3" />
                    <path d="M 970 200 L 1040 200" stroke="#38bdf8" stroke-width="3" class="flow-particle" />

                    <!-- Leyte <-> Bohol Tie Line -->
                    <path d="M 1170 660 L 1170 720" stroke="#f97316" stroke-width="3" />
                    <path d="M 1170 660 L 1170 720" stroke="#38bdf8" stroke-width="3" class="flow-particle" />

                    <!-- Mindanao -> Visayas Link (Bottom Right) -->
                    <path d="M 1350 820 L 1290 820" stroke="#f59e0b" stroke-width="4" stroke-dasharray="4,2" />

                    <!-- SUBSTATION BUSES & GENERATOR NODES (Interactive SVG Elements) -->
                    
                    <!-- PANAY SUBSTATIONS -->
                    <!-- NABAS 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('NABAS', 'Panay', '138 kV', 360.44, 4210.50)">
                        <rect x="50" y="100" width="180" height="10" fill="#f97316" rx="2"/>
                        <text x="50" y="92" fill="#e2e8f0" font-size="11" font-weight="bold">NABAS 138kV</text>
                        <!-- Connected Gen -->
                        <circle cx="90" cy="140" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="90" y="144" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="90" y1="110" x2="90" y2="128" stroke="#10b981" stroke-width="2"/>
                        <text x="110" y="144" fill="#94a3b8" font-size="10">PN3/NPP (112 MW)</text>
                    </g>

                    <!-- BAROTAC 230kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('BAROTAC', 'Panay', '230 kV', 185.00, 4280.00)">
                        <rect x="50" y="240" width="220" height="12" fill="#ef4444" rx="2" filter="url(#glow-230)"/>
                        <text x="50" y="232" fill="#e2e8f0" font-size="11" font-weight="bold">BAROTAC 230kV HUB</text>
                        <line x1="100" y1="200" x2="100" y2="240" stroke="#f97316" stroke-width="2.5"/>
                    </g>

                    <!-- DINGLE 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('DINGLE', 'Panay', '138 kV', 95.00, 4190.00)">
                        <rect x="50" y="380" width="180" height="10" fill="#f97316" rx="2"/>
                        <text x="50" y="372" fill="#e2e8f0" font-size="11" font-weight="bold">DINGLE 138kV</text>
                        <circle cx="100" cy="420" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="100" y="424" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="100" y1="390" x2="100" y2="408" stroke="#10b981" stroke-width="2"/>
                        <text x="120" y="424" fill="#94a3b8" font-size="10">CSH/HPP 3</text>
                    </g>

                    <!-- ILOILO / PEDC 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('ILOILO / PEDC', 'Panay', '138 kV', 210.00, 4310.00)">
                        <rect x="50" y="520" width="200" height="10" fill="#f97316" rx="2"/>
                        <text x="50" y="512" fill="#e2e8f0" font-size="11" font-weight="bold">ILOILO / PEDC 138kV</text>
                        <circle cx="120" cy="560" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="120" y="564" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="120" y1="530" x2="120" y2="548" stroke="#10b981" stroke-width="2"/>
                        <text x="140" y="564" fill="#94a3b8" font-size="10">PEDC Coal (164 MW)</text>
                    </g>

                    <!-- NEGROS SUBSTATIONS -->
                    <!-- BACOLOD 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('BACOLOD', 'Negros', '138 kV', 240.00, 4150.00)">
                        <rect x="380" y="150" width="200" height="10" fill="#f97316" rx="2"/>
                        <text x="380" y="142" fill="#e2e8f0" font-size="11" font-weight="bold">BACOLOD / GAHIT 138kV</text>
                    </g>

                    <!-- CADIZ 138kV (Solar Hub) -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('CADIZ', 'Negros', '138 kV', 132.50, 3980.00)">
                        <rect x="380" y="270" width="220" height="10" fill="#f97316" rx="2"/>
                        <text x="380" y="262" fill="#e2e8f0" font-size="11" font-weight="bold">CADIZ SOLAR 138kV</text>
                        <circle cx="480" cy="310" r="12" fill="#0f172a" stroke="#eab308" stroke-width="2"/>
                        <text x="480" y="314" fill="#eab308" font-size="10" text-anchor="middle" font-weight="bold">S</text>
                        <line x1="480" y1="280" x2="480" y2="298" stroke="#eab308" stroke-width="2"/>
                        <text x="500" y="314" fill="#94a3b8" font-size="10">Cadiz Solar (132 MW)</text>
                    </g>

                    <!-- CALATRAVA 230kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('CALATRAVA', 'Negros', '230 kV', 310.00, 4120.00)">
                        <rect x="380" y="420" width="220" height="12" fill="#ef4444" rx="2" filter="url(#glow-230)"/>
                        <text x="380" y="412" fill="#e2e8f0" font-size="11" font-weight="bold">CALATRAVA 230kV HUB</text>
                    </g>

                    <!-- CEBU SUBSTATIONS -->
                    <!-- MAGDUGO / NAGA 230kV BACKBONE -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('MAGDUGO / NAGA', 'Cebu', '230 kV', 580.00, 4450.00)">
                        <rect x="710" y="180" width="240" height="14" fill="#ef4444" rx="2" filter="url(#glow-230)"/>
                        <text x="710" y="170" fill="#e2e8f0" font-size="11" font-weight="bold">MAGDUGO - NAGA 230kV MAIN</text>
                    </g>

                    <!-- TOLEDO 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('TOLEDO', 'Cebu', '138 kV', 340.00, 4420.00)">
                        <rect x="710" y="340" width="200" height="10" fill="#f97316" rx="2"/>
                        <text x="710" y="332" fill="#e2e8f0" font-size="11" font-weight="bold">TOLEDO 138kV</text>
                        <circle cx="800" cy="380" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="800" y="384" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="800" y1="350" x2="800" y2="368" stroke="#10b981" stroke-width="2"/>
                        <text x="820" y="384" fill="#94a3b8" font-size="10">TBE/TSO Thermal</text>
                    </g>

                    <!-- COLON / METRO CEBU 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('COLON / MANDAUE', 'Cebu', '138 kV', 0.00, 4520.00)">
                        <rect x="710" y="500" width="220" height="10" fill="#f97316" rx="2"/>
                        <text x="710" y="492" fill="#e2e8f0" font-size="11" font-weight="bold">COLON / MANDAUE (METRO LOAD)</text>
                    </g>

                    <!-- LEYTE SUBSTATIONS -->
                    <!-- TABANGO 230kV (HVDC INTERFACE) -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('TABANGO', 'Leyte-Samar', '230 kV', 0.00, 3850.00)">
                        <rect x="1040" y="180" width="220" height="12" fill="#ef4444" rx="2" filter="url(#glow-230)"/>
                        <text x="1040" y="172" fill="#e2e8f0" font-size="11" font-weight="bold">TABANGO 230kV HUB</text>
                    </g>

                    <!-- KANANGA GEOTHERMAL 138/230kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('KANANGA', 'Leyte-Samar', '230 kV', 520.00, 3790.00)">
                        <rect x="1040" y="320" width="240" height="12" fill="#ef4444" rx="2" filter="url(#glow-230)"/>
                        <text x="1040" y="312" fill="#e2e8f0" font-size="11" font-weight="bold">KANANGA GEOTHERMAL HUB</text>
                        <circle cx="1140" cy="360" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="1140" y="364" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="1140" y1="332" x2="1140" y2="348" stroke="#10b981" stroke-width="2"/>
                        <text x="1160" y="364" fill="#94a3b8" font-size="10">Tongonan Geo (520 MW)</text>
                    </g>

                    <!-- BOHOL SUBSTATIONS -->
                    <!-- TAGBILARAN 69kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('TAGBILARAN', 'Bohol', '69 kV', 3.34, 4610.00)">
                        <rect x="1040" y="760" width="200" height="8" fill="#06b6d4" rx="2"/>
                        <text x="1040" y="752" fill="#e2e8f0" font-size="11" font-weight="bold">TAGBILARAN 69kV</text>
                    </g>

                    <!-- UBAY 138kV -->
                    <g class="substation-node cursor-pointer" onclick="inspectNode('UBAY', 'Bohol', '138 kV', 0.00, 4580.00)">
                        <rect x="1040" y="860" width="200" height="10" fill="#f97316" rx="2"/>
                        <text x="1040" y="852" fill="#e2e8f0" font-size="11" font-weight="bold">UBAY 138kV TIE</text>
                    </g>
                </svg>
            </div>

            <!-- SUBSTATION INSPECTOR SIDE DRAWER -->
            <aside id="inspector-drawer" class="absolute top-0 right-0 h-full w-80 glass-panel border-l border-slate-800 p-5 shadow-2xl transition-transform transform translate-x-full z-30 flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-700 pb-3 mb-4">
                        <div>
                            <h3 id="drawer-node-name" class="font-bold text-white text-base">NABAS SUBSTATION</h3>
                            <span id="drawer-node-region" class="text-xs text-orange-400 font-semibold">Panay Grid</span>
                        </div>
                        <button onclick="closeDrawer()" class="text-slate-400 hover:text-white p-1">✕</button>
                    </div>

                    <div class="space-y-4 text-xs">
                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-1.5">
                            <div class="text-slate-400">BUS VOLTAGE / ROLE</div>
                            <div id="drawer-node-role" class="mono font-bold text-cyan-400 text-sm">138 kV Sub-transmission</div>
                        </div>

                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-1.5">
                            <div class="text-slate-400">LOCATIONAL MARGINAL PRICE (LMP)</div>
                            <div id="drawer-node-lmp" class="mono font-bold text-amber-400 text-base">₱4,210.50 / MWh</div>
                        </div>

                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-1.5">
                            <div class="text-slate-400">ACTIVE GENERATION DISPATCH</div>
                            <div id="drawer-node-gen" class="mono font-bold text-emerald-400 text-sm">360.44 MW</div>
                        </div>

                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-2">
                            <div class="text-slate-400 font-semibold">CONNECTED TRANSMISSION CORRIDORS</div>
                            <ul class="space-y-1.5 text-slate-300">
                                <li class="flex justify-between"><span>Panit-an Line 1:</span><span class="mono text-emerald-400">84.2 MW</span></li>
                                <li class="flex justify-between"><span>Barotac 230kV Tie:</span><span class="mono text-emerald-400">142.1 MW</span></li>
                            </ul>
                        </div>
                    </div>
                </div>

                <div class="pt-4 border-t border-slate-800">
                    <button class="w-full bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-semibold py-2 rounded-lg text-xs transition">
                        Export Node Dispatch Log
                    </button>
                </div>
            </aside>
        </div>

        <!-- TAB 2: MARKET SUMMARY TELEMETRY (SHEET 4) -->
        <div id="tab-market-summary" class="hidden p-6 overflow-y-auto space-y-6">
            <h2 class="text-lg font-bold text-white">Market Telemetry & Regional Input Summary (Excel Sheet 4)</h2>

            <!-- Regional Telemetry Grid -->
            <div class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-5 gap-4">
                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-orange-400 text-sm">PANAY</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">360.44 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">486.08 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">-125.64 MW</span></div>
                        <div class="flex justify-between"><span>Frequency:</span><span class="mono text-cyan-400">60.338 Hz</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-rose-400 text-sm">NEGROS</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">646.11 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">411.58 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">+234.53 MW</span></div>
                        <div class="flex justify-between"><span>Frequency:</span><span class="mono text-cyan-400">60.350 Hz</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-cyan-400 text-sm">CEBU</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">642.05 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">1,107.30 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">-465.25 MW</span></div>
                        <div class="flex justify-between"><span>Frequency:</span><span class="mono text-cyan-400">60.350 Hz</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-emerald-400 text-sm">LEYTE–SAMAR</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">581.37 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">279.28 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">+302.09 MW</span></div>
                        <div class="flex justify-between"><span>Frequency:</span><span class="mono text-cyan-400">60.354 Hz</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-purple-400 text-sm">BOHOL</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">3.34 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">122.26 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">-118.92 MW</span></div>
                        <div class="flex justify-between"><span>Frequency:</span><span class="mono text-cyan-400">60.354 Hz</span></div>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 3: DEFINITIONS (SHEET 2) -->
        <div id="tab-definitions" class="hidden p-6 overflow-y-auto space-y-4">
            <div class="flex justify-between items-center">
                <h2 class="text-lg font-bold text-white">SLD / EMS / SCADA Detail Definitions (Sheet 2)</h2>
                <input type="text" id="search-defs" placeholder="Search parameters..." oninput="filterDefs(this.value)" class="bg-slate-900 border border-slate-800 rounded-lg px-4 py-2 text-xs w-64 focus:outline-none focus:border-cyan-500">
            </div>
            <div class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden shadow">
                <table class="w-full text-left text-xs border-collapse">
                    <thead>
                        <tr class="bg-slate-800 text-slate-400 uppercase tracking-wider border-b border-slate-700">
                            <th class="p-3">Item / Label</th>
                            <th class="p-3">Category</th>
                            <th class="p-3">Definition</th>
                            <th class="p-3">What Operator Reads</th>
                            <th class="p-3">Unit</th>
                        </tr>
                    </thead>
                    <tbody id="defs-tbody" class="divide-y divide-slate-800 text-slate-300">
                        <!-- Populated by JS -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- TAB 4: ELEMENT REGISTER (SHEET 3) -->
        <div id="tab-register" class="hidden p-6 overflow-y-auto space-y-4">
            <div class="flex justify-between items-center">
                <h2 class="text-lg font-bold text-white">Visible Element SLD Register (Sheet 3)</h2>
                <input type="text" id="search-reg" placeholder="Search substations..." oninput="filterReg(this.value)" class="bg-slate-900 border border-slate-800 rounded-lg px-4 py-2 text-xs w-64 focus:outline-none focus:border-cyan-500">
            </div>
            <div class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden shadow">
                <table class="w-full text-left text-xs border-collapse">
                    <thead>
                        <tr class="bg-slate-800 text-slate-400 uppercase tracking-wider border-b border-slate-700">
                            <th class="p-3">Region</th>
                            <th class="p-3">Substation / Label</th>
                            <th class="p-3">Type</th>
                            <th class="p-3">Voltage / Role</th>
                            <th class="p-3">Explanation</th>
                        </tr>
                    </thead>
                    <tbody id="reg-tbody" class="divide-y divide-slate-800 text-slate-300">
                        <!-- Populated by JS -->
                    </tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- Footer Status -->
    <footer class="bg-slate-900 border-t border-slate-800 py-2.5 px-6 text-center text-xs text-slate-500 flex justify-between items-center">
        <span>IEMOP Visayas Single Line Diagram — Bus-Oriented Market Network Model (MNM 2026-08)</span>
        <span class="mono text-cyan-400">Data Source: Sheet 2, 3, 4 Embedded Excel Telemetry</span>
    </footer>

    <!-- JavaScript Logic & Telemetry Engine -->
    <script>
        // Data from Sheet 2 (Definitions)
        const DEFS = [
            { item: "SLD", cat: "Diagram", def: "Single Line Diagram; simplified multi-bus power network representation.", reads: "Network topology and line connectivity.", unit: "Graphical" },
            { item: "Bus / Busbar", cat: "Equipment", def: "Electrical node where transmission lines, transformers, and generators connect.", reads: "Voltage magnitude and angle.", unit: "kV" },
            { item: "230 kV Bus", cat: "Voltage", def: "High-voltage transmission bus for bulk inter-area power transfer.", reads: "Bulk power flow and corridor limits.", unit: "kV" },
            { item: "138 kV Bus", cat: "Voltage", def: "Sub-transmission / primary distribution grid backbone.", reads: "Regional load transfer.", unit: "kV" },
            { item: "LMP", cat: "Market", def: "Locational Marginal Price; cost of supplying next MW at specific node.", reads: "Energy + Congestion + Loss price components.", unit: "₱/MWh" }
        ];

        // Data from Sheet 3 (Element Register)
        const REGS = [
            { reg: "PANAY", name: "NABAS", type: "Substation", role: "138 kV Area", exp: "Northern Panay connection feeding wind/solar generation." },
            { reg: "PANAY", name: "BAROTAC", type: "Substation", role: "230/138 kV Hub", exp: "Major Panay bulk transmission hub." },
            { reg: "NEGROS", name: "CADIZ", type: "Substation", role: "138 kV Solar Hub", exp: "High solar penetration plant collector bus." },
            { reg: "CEBU", name: "MAGDUGO / NAGA", type: "Corridor", role: "230 kV Backbone", exp: "Cebu main high-voltage transmission corridor." },
            { reg: "LEYTE", name: "KANANGA", type: "Substation", role: "230 kV Geothermal", exp: "Bulk geothermal generation collector node." }
        ];

        // Canvas Zoom & Pan Control
        let scale = 1;
        let translateX = 0;
        let translateY = 0;
        const canvas = document.getElementById('sld-canvas');

        function zoomCanvas(factor) {
            scale *= factor;
            scale = Math.min(Math.max(0.6, scale), 3);
            applyTransform();
        }

        function resetZoom() {
            scale = 1;
            translateX = 0;
            translateY = 0;
            applyTransform();
        }

        function applyTransform() {
            canvas.style.transform = `translate(${translateX}px, ${translateY}px) scale(${scale})`;
        }

        // Substation Drawer Inspector
        function inspectNode(name, region, role, gen, lmp) {
            document.getElementById('drawer-node-name').innerText = name + " SUBSTATION";
            document.getElementById('drawer-node-region').innerText = region + " Sub-grid";
            document.getElementById('drawer-node-role').innerText = role;
            document.getElementById('drawer-node-gen').innerText = gen + " MW";
            document.getElementById('drawer-node-lmp').innerText = "₱" + lmp.toLocaleString('en-US', {minimumFractionDigits: 2}) + " / MWh";
            document.getElementById('inspector-drawer').classList.remove('translate-x-full');
        }

        function closeDrawer() {
            document.getElementById('inspector-drawer').classList.add('translate-x-full');
        }

        // Tab Switcher Logic
        function switchTab(tab) {
            ['sld', 'market-summary', 'definitions', 'register'].forEach(t => {
                document.getElementById(`tab-${t}`).classList.add('hidden');
                document.getElementById(`btn-${t}`).classList.remove('bg-cyan-500', 'text-slate-950', 'font-semibold');
                document.getElementById(`btn-${t}`).classList.add('text-slate-300');
            });
            document.getElementById(`tab-${tab}`).classList.remove('hidden');
            document.getElementById(`btn-${tab}`).classList.add('bg-cyan-500', 'text-slate-950', 'font-semibold');
        }

        // Theme Toggle
        function toggleDarkMode() {
            document.documentElement.classList.toggle('dark');
        }

        // Render Tables
        function renderTables() {
            document.getElementById('defs-tbody').innerHTML = DEFS.map(d => `
                <tr class="hover:bg-slate-800/50">
                    <td class="p-3 font-bold text-white">${d.item}</td>
                    <td class="p-3 text-cyan-400">${d.cat}</td>
                    <td class="p-3">${d.def}</td>
                    <td class="p-3 text-slate-400">${d.reads}</td>
                    <td class="p-3 mono text-amber-400">${d.unit}</td>
                </tr>
            `).join('');

            document.getElementById('reg-tbody').innerHTML = REGS.map(r => `
                <tr class="hover:bg-slate-800/50">
                    <td class="p-3 font-bold text-orange-400">${r.reg}</td>
                    <td class="p-3 font-bold text-white">${r.name}</td>
                    <td class="p-3">${r.type}</td>
                    <td class="p-3 mono text-cyan-400">${r.role}</td>
                    <td class="p-3 text-slate-300">${r.exp}</td>
                </tr>
            `).join('');
        }

        renderTables();
    </script>
</body>
</html>
