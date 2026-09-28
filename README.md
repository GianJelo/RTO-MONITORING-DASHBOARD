<html lang="en" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IEMOP Visayas Market Network Model (MNM) Full SLD & Flow Monitor</title>
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

        /* Power Flow Particle Animations */
        .flow-line-normal {
            stroke-dasharray: 6, 6;
            animation: flowForward 1.2s linear infinite;
        }
        .flow-line-heavy {
            stroke-dasharray: 8, 4;
            animation: flowForward 0.7s linear infinite;
        }
        .flow-line-reverse {
            stroke-dasharray: 6, 6;
            animation: flowReverse 1.2s linear infinite;
        }

        @keyframes flowForward {
            from { stroke-dashoffset: 24; }
            to { stroke-dashoffset: 0; }
        }
        @keyframes flowReverse {
            from { stroke-dashoffset: 0; }
            to { stroke-dashoffset: 24; }
        }

        /* Glassmorphism Panel Overlay */
        .glass-panel {
            background: rgba(15, 23, 42, 0.88);
            backdrop-filter: blur(10px);
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col transition-colors duration-300">

    <!-- Header Navigation Bar -->
    <header class="bg-slate-900 border-b border-slate-800 px-6 py-3 flex flex-wrap justify-between items-center sticky top-0 z-50 shadow-lg">
        <div class="flex items-center space-x-3">
            <div class="bg-cyan-500/10 p-2 rounded-lg border border-cyan-500/30">
                <svg class="w-6 h-6 text-cyan-400" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 10V3L4 14h7v7l9-11h-7z"/>
                </svg>
            </div>
            <div>
                <div class="flex items-center space-x-2">
                    <h1 class="text-base font-bold tracking-wide text-white">INDEPENDENT ELECTRICITY MARKET OPERATOR OF THE PHILIPPINES</h1>
                    <span class="text-xs bg-cyan-500/20 text-cyan-300 font-mono px-2 py-0.5 rounded border border-cyan-500/30">MNM 2026-08-17</span>
                </div>
                <p class="text-xs text-slate-400">BUS-ORIENTED SINGLE LINE DIAGRAM — MARKET NETWORK MODEL (VISAYAS GRID)</p>
            </div>
        </div>

        <div class="flex items-center space-x-4 text-xs mt-2 sm:mt-0">
            <div class="bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700">
                <span class="text-slate-400 block text-[10px] font-semibold">INTERVAL RUN</span>
                <span class="mono font-bold text-emerald-400 text-sm">RTD 5-MIN #142</span>
            </div>
            <div class="bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700">
                <span class="text-slate-400 block text-[10px] font-semibold">AVERAGE MARKET LMP</span>
                <span id="hdr-avg-lmp" class="mono font-bold text-amber-400 text-sm">₱4,285.50 / MWh</span>
            </div>
            <div class="bg-slate-800 px-3 py-1.5 rounded-lg border border-slate-700 text-center">
                <span class="text-slate-400 block text-[10px] font-semibold">NEXT DISPATCH</span>
                <span id="poll-timer" class="mono font-bold text-cyan-400 text-sm">04:42</span>
            </div>
            <button onclick="toggleDarkMode()" class="p-2 rounded-lg bg-slate-800 border border-slate-700 text-slate-300 hover:bg-slate-700 transition" title="Toggle Light/Dark Mode">
                <svg id="theme-icon" class="w-4 h-4" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z"/>
                </svg>
            </button>
        </div>
    </header>

    <!-- Navigation Tabs Bar -->
    <nav class="bg-slate-900/80 border-b border-slate-800 px-6 py-2 flex flex-wrap justify-between items-center">
        <div class="flex space-x-3">
            <button onclick="switchTab('mnm-graphics')" id="btn-mnm-graphics" class="px-4 py-1.5 rounded-md text-xs font-semibold bg-cyan-500 text-slate-950 shadow transition">
                Graphical Market SLD (All Circuits)
            </button>
            <button onclick="switchTab('circuit-matrix')" id="btn-circuit-matrix" class="px-4 py-1.5 rounded-md text-xs font-medium text-slate-300 hover:bg-slate-800 transition">
                Circuits & Line Flow Matrix
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

        <!-- Scenario Selector -->
        <div class="flex items-center space-x-3 text-xs mt-2 sm:mt-0">
            <span class="text-slate-400 font-medium">Market Dispatch Mode:</span>
            <select id="dispatch-scenario" onchange="changeScenario(this.value)" class="bg-slate-800 border border-slate-700 text-slate-200 rounded px-2.5 py-1 focus:outline-none focus:border-cyan-500">
                <option value="normal">Real-Time Market Dispatch (Normal Flow)</option>
                <option value="peak">Peak Load Scenario (2,779.37 MW)</option>
                <option value="congestion">Panay–Negros Line Congestion Stress Test</option>
            </select>
        </div>
    </nav>

    <!-- Main Content Workspace -->
    <main class="flex-1 relative overflow-hidden flex flex-col">

        <!-- TAB 1: GRAPHICAL SINGLE LINE DIAGRAM -->
        <div id="tab-mnm-graphics" class="flex-1 relative bg-slate-950 overflow-hidden flex">
            
            <!-- Floating Voltage Legend & Info Overlay -->
            <div class="absolute top-4 left-4 z-20 glass-panel p-4 rounded-xl border border-slate-800 text-xs shadow-2xl space-y-3 max-w-xs">
                <div class="font-bold text-slate-100 border-b border-slate-700/60 pb-1.5 flex justify-between items-center">
                    <span>MNM VOLTAGE LEGEND</span>
                    <span class="text-[10px] text-cyan-400 font-mono">IEMOP STANDARD</span>
                </div>
                <div class="grid grid-cols-2 gap-2 text-[11px] font-mono">
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v500 inline-block"></span><span>500 kV Grid</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v230 inline-block"></span><span>230 kV Backbone</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v138 inline-block"></span><span>138 kV Primary</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v69 inline-block"></span><span>69 kV Sub-trans</span></div>
                    <div class="flex items-center space-x-2"><span class="w-3 h-3 rounded-full bg-v13 inline-block"></span><span>13.8 kV Gen/Local</span></div>
                </div>
                <div class="pt-2 border-t border-slate-700/60 text-[10px] text-slate-400">
                    <span class="text-emerald-400 font-semibold">Active Flows:</span> Moving particle speed reflects circuit MW transfer intensity.
                </div>
            </div>

            <!-- Canvas Viewport Controls -->
            <div class="absolute bottom-6 left-4 z-20 flex space-x-2">
                <button onclick="zoomCanvas(1.25)" class="bg-slate-800 hover:bg-slate-700 text-slate-200 p-2.5 rounded-lg border border-slate-700 shadow font-mono font-bold text-sm">+</button>
                <button onclick="zoomCanvas(0.8)" class="bg-slate-800 hover:bg-slate-700 text-slate-200 p-2.5 rounded-lg border border-slate-700 shadow font-mono font-bold text-sm">-</button>
                <button onclick="resetZoom()" class="bg-slate-800 hover:bg-slate-700 text-slate-200 px-3.5 py-2 rounded-lg border border-slate-700 shadow text-xs font-semibold">Reset View</button>
            </div>

            <!-- Inter-Regional Flow Summary Bar -->
            <div class="absolute bottom-6 right-6 z-20 glass-panel p-3.5 rounded-xl border border-slate-800 shadow-xl hidden md:block">
                <div class="text-[11px] font-bold text-slate-300 mb-2 uppercase tracking-wide">Inter-Regional Tie Line Transfers</div>
                <div class="flex space-x-3 text-xs font-mono">
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">PANAY ↔ NEGROS</span>
                        <span id="summary-pan-neg" class="font-bold text-emerald-400">124.20 MW</span>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">NEGROS ↔ CEBU</span>
                        <span id="summary-neg-ceb" class="font-bold text-rose-400">-76.92 MW</span>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">LEYTE ↔ CEBU</span>
                        <span id="summary-ley-ceb" class="font-bold text-emerald-400">55.27 MW</span>
                    </div>
                    <div class="bg-slate-900/80 p-2 rounded border border-slate-800 text-center">
                        <span class="text-slate-400 text-[10px] block">MINDANAO ↔ VISAYAS</span>
                        <span id="summary-min-vis" class="font-bold text-amber-400">179.05 MW</span>
                    </div>
                </div>
            </div>

            <!-- SVG Market Network Canvas -->
            <div id="svg-viewport" class="w-full h-full cursor-grab active:cursor-grabbing overflow-hidden flex items-center justify-center">
                <svg id="sld-canvas" viewBox="0 0 1700 1000" class="w-full h-full transition-transform duration-100 ease-out">
                    
                    <!-- Background Dark Grid Pattern -->
                    <rect width="1700" height="1000" fill="#030712"/>
                    <g opacity="0.04" stroke="#ffffff" stroke-width="0.5">
                        <path d="M 0 100 H 1700 M 0 200 H 1700 M 0 300 H 1700 M 0 400 H 1700 M 0 500 H 1700 M 0 600 H 1700 M 0 700 H 1700 M 0 800 H 1700 M 0 900 H 1700" />
                        <path d="M 100 0 V 1000 M 200 0 V 1000 M 300 0 V 1000 M 400 0 V 1000 M 500 0 V 1000 M 600 0 V 1000 M 700 0 V 1000 M 800 0 V 1000 M 900 0 V 1000 M 1000 0 V 1000 M 1100 0 V 1000 M 1200 0 V 1000 M 1300 0 V 1000 M 1400 0 V 1000 M 1500 0 V 1000 M 1600 0 V 1000" />
                    </g>

                    <!-- REGIONAL SUB-GRID BOUNDARIES -->
                    <!-- PANAY (08) -->
                    <rect x="30" y="30" width="320" height="920" fill="#0f172a" fill-opacity="0.3" stroke="#f97316" stroke-width="1.5" stroke-dasharray="6,4" rx="10"/>
                    <text x="45" y="55" fill="#f97316" font-size="16" font-weight="bold" font-family="Inter">PANAY SUB-GRID (REGION 08)</text>

                    <!-- NEGROS (06) -->
                    <rect x="370" y="30" width="320" height="920" fill="#0f172a" fill-opacity="0.3" stroke="#ef4444" stroke-width="1.5" stroke-dasharray="6,4" rx="10"/>
                    <text x="385" y="55" fill="#ef4444" font-size="16" font-weight="bold" font-family="Inter">NEGROS SUB-GRID (REGION 06)</text>

                    <!-- CEBU (05) -->
                    <rect x="710" y="30" width="320" height="920" fill="#0f172a" fill-opacity="0.3" stroke="#06b6d4" stroke-width="1.5" stroke-dasharray="6,4" rx="10"/>
                    <text x="725" y="55" fill="#06b6d4" font-size="16" font-weight="bold" font-family="Inter">CEBU SUB-GRID (REGION 05)</text>

                    <!-- LEYTE - SAMAR (04) -->
                    <rect x="1050" y="30" width="320" height="660" fill="#0f172a" fill-opacity="0.3" stroke="#10b981" stroke-width="1.5" stroke-dasharray="6,4" rx="10"/>
                    <text x="1065" y="55" fill="#10b981" font-size="16" font-weight="bold" font-family="Inter">LEYTE–SAMAR SUB-GRID (REGION 04)</text>

                    <!-- BOHOL (07) -->
                    <rect x="1050" y="710" width="320" height="240" fill="#0f172a" fill-opacity="0.3" stroke="#8b5cf6" stroke-width="1.5" stroke-dasharray="6,4" rx="10"/>
                    <text x="1065" y="735" fill="#8b5cf6" font-size="16" font-weight="bold" font-family="Inter">BOHOL SUB-GRID (REGION 07)</text>

                    <!-- INTER-REGION TIE LINE CORRIDORS & ANIMATED PARTICLES -->
                    <!-- Panay <-> Negros Submarine Cable (138kV) -->
                    <path d="M 320 180 L 400 180" stroke="#f97316" stroke-width="3" />
                    <path d="M 320 180 L 400 180" stroke="#38bdf8" stroke-width="3" class="flow-line-normal" />

                    <!-- Negros <-> Cebu Tie Lines (138kV & 230kV) -->
                    <path d="M 660 280 L 740 280" stroke="#ef4444" stroke-width="3" />
                    <path d="M 660 280 L 740 280" stroke="#ef4444" stroke-width="3" class="flow-line-reverse" />
                    
                    <path d="M 660 480 L 740 480" stroke="#f97316" stroke-width="2.5" />
                    <path d="M 660 480 L 740 480" stroke="#38bdf8" stroke-width="2.5" class="flow-line-normal" />

                    <!-- Cebu <-> Leyte Tie Lines (230kV Submarine) -->
                    <path d="M 1000 220 L 1080 220" stroke="#ef4444" stroke-width="3" />
                    <path d="M 1000 220 L 1080 220" stroke="#38bdf8" stroke-width="3" class="flow-line-heavy" />

                    <!-- Leyte <-> Bohol Interconnection -->
                    <path d="M 1210 670 L 1210 730" stroke="#f97316" stroke-width="3" />
                    <path d="M 1210 670 L 1210 730" stroke="#38bdf8" stroke-width="3" class="flow-line-normal" />

                    <!-- Mindanao -> Visayas Interconnection (Bottom Right Link) -->
                    <path d="M 1390 830 L 1330 830" stroke="#f59e0b" stroke-width="3.5" stroke-dasharray="4,2" />

                    <!-- PANAY SUBSTATIONS & CIRCUITS (08) -->
                    <!-- 08NABAS 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('08NABAS', 'PANAY', '138 kV', 360.44, 4210.50)">
                        <rect x="50" y="110" width="200" height="10" fill="#f97316" rx="2"/>
                        <text x="50" y="102" fill="#e2e8f0" font-size="11" font-weight="bold">08NABAS 138kV</text>
                        <!-- Connected Plants -->
                        <circle cx="90" cy="150" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="90" y="154" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="90" y1="120" x2="90" y2="138" stroke="#10b981" stroke-width="2"/>
                        <text x="110" y="154" fill="#94a3b8" font-size="10">08NABASDPP_U01 (112 MW)</text>
                    </g>

                    <!-- 08BAROTAC 230kV HUB -->
                    <g class="cursor-pointer" onclick="inspectSubstation('08BAROTAC', 'PANAY', '230 kV', 185.00, 4280.00)">
                        <rect x="50" y="260" width="230" height="12" fill="#ef4444" rx="2"/>
                        <text x="50" y="252" fill="#e2e8f0" font-size="11" font-weight="bold">08BAROTAC 230kV MAIN HUB</text>
                        <line x1="100" y1="210" x2="100" y2="260" stroke="#f97316" stroke-width="2.5"/>
                    </g>

                    <!-- 08DINGLE 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('08DINGLE', 'PANAY', '138 kV', 95.00, 4190.00)">
                        <rect x="50" y="410" width="200" height="10" fill="#f97316" rx="2"/>
                        <text x="50" y="402" fill="#e2e8f0" font-size="11" font-weight="bold">08DINGLE 138kV</text>
                        <circle cx="100" cy="450" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="100" y="454" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="100" y1="420" x2="100" y2="438" stroke="#10b981" stroke-width="2"/>
                        <text x="120" y="454" fill="#94a3b8" font-size="10">08PDPP3_S01 (95 MW)</text>
                    </g>

                    <!-- 08ILOILO / PEDC 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('08ILOILO', 'PANAY', '138 kV', 210.00, 4310.00)">
                        <rect x="50" y="560" width="220" height="10" fill="#f97316" rx="2"/>
                        <text x="50" y="552" fill="#e2e8f0" font-size="11" font-weight="bold">08ILOILO / PEDC 138kV</text>
                        <circle cx="120" cy="600" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="120" y="604" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="120" y1="570" x2="120" y2="588" stroke="#10b981" stroke-width="2"/>
                        <text x="140" y="604" fill="#94a3b8" font-size="10">08PEDC_U01 (164 MW)</text>
                    </g>

                    <!-- NEGROS SUBSTATIONS & CIRCUITS (06) -->
                    <!-- 06BACOLOD 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('06BACOLOD', 'NEGROS', '138 kV', 240.00, 4150.00)">
                        <rect x="390" y="170" width="210" height="10" fill="#f97316" rx="2"/>
                        <text x="390" y="162" fill="#e2e8f0" font-size="11" font-weight="bold">06BACOLOD 138kV</text>
                    </g>

                    <!-- 06CADIZ SOLAR HUB 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('06CADIZ', 'NEGROS', '138 kV', 132.50, 3980.00)">
                        <rect x="390" y="290" width="230" height="10" fill="#f97316" rx="2"/>
                        <text x="390" y="282" fill="#e2e8f0" font-size="11" font-weight="bold">06CADIZ SOLAR 138kV</text>
                        <circle cx="490" cy="330" r="12" fill="#0f172a" stroke="#eab308" stroke-width="2"/>
                        <text x="490" y="334" fill="#eab308" font-size="10" text-anchor="middle" font-weight="bold">S</text>
                        <line x1="490" y1="300" x2="490" y2="318" stroke="#eab308" stroke-width="2"/>
                        <text x="510" y="334" fill="#94a3b8" font-size="10">06CADSOL_G01 (132 MW)</text>
                    </g>

                    <!-- 06CALATRAVA 230kV HUB -->
                    <g class="cursor-pointer" onclick="inspectSubstation('06CALATRAVA', 'NEGROS', '230 kV', 310.00, 4120.00)">
                        <rect x="390" y="440" width="230" height="12" fill="#ef4444" rx="2"/>
                        <text x="390" y="432" fill="#e2e8f0" font-size="11" font-weight="bold">06CALATRAVA 230kV HUB</text>
                    </g>

                    <!-- CEBU SUBSTATIONS & CIRCUITS (05) -->
                    <!-- 05MAGDUGO / NAGA 230kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('05MAGDUGO', 'CEBU', '230 kV', 580.00, 4450.00)">
                        <rect x="730" y="190" width="250" height="14" fill="#ef4444" rx="2"/>
                        <text x="730" y="180" fill="#e2e8f0" font-size="11" font-weight="bold">05MAGDUGO - NAGA 230kV MAIN</text>
                    </g>

                    <!-- 05TOLEDO 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('05TOLEDO', 'CEBU', '138 kV', 340.00, 4420.00)">
                        <rect x="730" y="360" width="210" height="10" fill="#f97316" rx="2"/>
                        <text x="730" y="352" fill="#e2e8f0" font-size="11" font-weight="bold">05TOLEDO 138kV</text>
                        <circle cx="820" cy="400" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="820" y="404" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="820" y1="370" x2="820" y2="388" stroke="#10b981" stroke-width="2"/>
                        <text x="840" y="404" fill="#94a3b8" font-size="10">05CEDC_U01 (246 MW)</text>
                    </g>

                    <!-- LEYTE SUBSTATIONS & CIRCUITS (04) -->
                    <!-- 04TABANGO 230kV HUB -->
                    <g class="cursor-pointer" onclick="inspectSubstation('04TABAN', 'LEYTE-SAMAR', '230 kV', 0.00, 3850.00)">
                        <rect x="1070" y="200" width="230" height="12" fill="#ef4444" rx="2"/>
                        <text x="1070" y="192" fill="#e2e8f0" font-size="11" font-weight="bold">04TABAN 230kV HUB</text>
                    </g>

                    <!-- 04KANANGA GEOTHERMAL 230kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('04TONGONAN', 'LEYTE-SAMAR', '230 kV', 520.00, 3790.00)">
                        <rect x="1070" y="350" width="250" height="12" fill="#ef4444" rx="2"/>
                        <text x="1070" y="342" fill="#e2e8f0" font-size="11" font-weight="bold">04TONGONAN GEOTHERMAL</text>
                        <circle cx="1170" cy="390" r="12" fill="#0f172a" stroke="#10b981" stroke-width="2"/>
                        <text x="1170" y="394" fill="#10b981" font-size="10" text-anchor="middle" font-weight="bold">G</text>
                        <line x1="1170" y1="362" x2="1170" y2="378" stroke="#10b981" stroke-width="2"/>
                        <text x="1190" y="394" fill="#94a3b8" font-size="10">04LGPP_G01 (520 MW)</text>
                    </g>

                    <!-- BOHOL SUBSTATIONS & CIRCUITS (07) -->
                    <!-- 07UBAY 138kV -->
                    <g class="cursor-pointer" onclick="inspectSubstation('07UBAY', 'BOHOL', '138 kV', 0.00, 4580.00)">
                        <rect x="1070" y="770" width="220" height="10" fill="#f97316" rx="2"/>
                        <text x="1070" y="762" fill="#e2e8f0" font-size="11" font-weight="bold">07UBAY 138kV</text>
                    </g>
                </svg>
            </div>

            <!-- SUBSTATION INSPECTOR SIDE DRAWER -->
            <aside id="inspector-drawer" class="absolute top-0 right-0 h-full w-80 glass-panel border-l border-slate-800 p-5 shadow-2xl transition-transform transform translate-x-full z-30 flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-700 pb-3 mb-4">
                        <div>
                            <h3 id="drawer-node-name" class="font-bold text-white text-base">SUBSTATION INSPECTOR</h3>
                            <span id="drawer-node-region" class="text-xs text-cyan-400 font-semibold">Grid Region</span>
                        </div>
                        <button onclick="closeDrawer()" class="text-slate-400 hover:text-white p-1">✕</button>
                    </div>

                    <div class="space-y-4 text-xs">
                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-1">
                            <div class="text-slate-400">BUS VOLTAGE ROLE</div>
                            <div id="drawer-node-role" class="mono font-bold text-cyan-400 text-sm">230 kV Main Transmission</div>
                        </div>

                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-1">
                            <div class="text-slate-400">LOCATIONAL MARGINAL PRICE (LMP)</div>
                            <div id="drawer-node-lmp" class="mono font-bold text-amber-400 text-base">₱4,280.00 / MWh</div>
                        </div>

                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-1">
                            <div class="text-slate-400">CONNECTED ACTIVE GENERATION</div>
                            <div id="drawer-node-gen" class="mono font-bold text-emerald-400 text-sm">185.00 MW</div>
                        </div>

                        <div class="bg-slate-900/80 p-3 rounded-lg border border-slate-800 space-y-2">
                            <div class="text-slate-400 font-semibold">CONNECTED CIRCUITS (MNM MODEL)</div>
                            <ul id="drawer-circuits-list" class="space-y-1.5 text-slate-300 mono text-[11px]">
                                <!-- Populated dynamically -->
                            </ul>
                        </div>
                    </div>
                </div>

                <div class="pt-4 border-t border-slate-800">
                    <button class="w-full bg-cyan-500 hover:bg-cyan-400 text-slate-950 font-semibold py-2 rounded-lg text-xs transition">
                        Export Circuit Dispatch Log
                    </button>
                </div>
            </aside>
        </div>

        <!-- TAB 2: CIRCUITS & LINE FLOW MATRIX -->
        <div id="tab-circuit-matrix" class="hidden p-6 overflow-y-auto space-y-4">
            <div class="flex justify-between items-center">
                <h2 class="text-base font-bold text-white">Full Market Network Model (MNM) Circuit & Line Flow Register</h2>
                <input type="text" id="search-circuits" placeholder="Filter circuits (e.g. 08BAROTAC, 05CEBU)..." oninput="filterCircuits(this.value)" class="bg-slate-900 border border-slate-800 rounded-lg px-4 py-1.5 text-xs w-72 focus:outline-none focus:border-cyan-500">
            </div>
            <div class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden shadow">
                <table class="w-full text-left text-xs border-collapse">
                    <thead>
                        <tr class="bg-slate-800 text-slate-400 uppercase tracking-wider border-b border-slate-700">
                            <th class="p-3">Region</th>
                            <th class="p-3">Circuit / Line Code</th>
                            <th class="p-3">Type / Voltage</th>
                            <th class="p-3">Active Transfer (MW)</th>
                            <th class="p-3">Loading Status</th>
                        </tr>
                    </thead>
                    <tbody id="circuits-tbody" class="divide-y divide-slate-800 text-slate-300 mono">
                        <!-- Populated by JS -->
                    </tbody>
                </table>
            </div>
        </div>

        <!-- TAB 3: REGIONAL TELEMETRY (SHEET 4) -->
        <div id="tab-market-summary" class="hidden p-6 overflow-y-auto space-y-6">
            <h2 class="text-base font-bold text-white">Market Telemetry & Regional Input Summary (Sheet 4)</h2>
            <div class="grid grid-cols-1 md:grid-cols-3 lg:grid-cols-5 gap-4">
                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-orange-400 text-sm">PANAY (08)</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">360.44 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">486.08 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">-125.64 MW</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-rose-400 text-sm">NEGROS (06)</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">646.11 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">411.58 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">+234.53 MW</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-cyan-400 text-sm">CEBU (05)</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">642.05 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">1,107.30 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">-465.25 MW</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-emerald-400 text-sm">LEYTE–SAMAR (04)</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">581.37 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">279.28 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">+302.09 MW</span></div>
                    </div>
                </div>

                <div class="bg-slate-900 border border-slate-800 p-4 rounded-xl space-y-2">
                    <div class="flex justify-between items-center"><span class="font-bold text-purple-400 text-sm">BOHOL (07)</span><span class="text-xs text-slate-400 font-mono">NORMAL</span></div>
                    <div class="text-xs space-y-1 text-slate-300">
                        <div class="flex justify-between"><span>Gen:</span><span class="mono text-emerald-400 font-bold">3.34 MW</span></div>
                        <div class="flex justify-between"><span>Demand:</span><span class="mono text-rose-400 font-bold">122.26 MW</span></div>
                        <div class="flex justify-between"><span>Net Balance:</span><span class="mono text-amber-400 font-bold">-118.92 MW</span></div>
                    </div>
                </div>
            </div>
        </div>

        <!-- TAB 4: DEFINITIONS (SHEET 2) -->
        <div id="tab-definitions" class="hidden p-6 overflow-y-auto space-y-4">
            <h2 class="text-base font-bold text-white">Market & EMS Definitions (Sheet 2)</h2>
            <div class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden shadow">
                <table class="w-full text-left text-xs border-collapse">
                    <thead>
                        <tr class="bg-slate-800 text-slate-400 uppercase border-b border-slate-700">
                            <th class="p-3">Item</th>
                            <th class="p-3">Category</th>
                            <th class="p-3">Definition</th>
                            <th class="p-3">Unit</th>
                        </tr>
                    </thead>
                    <tbody id="defs-tbody" class="divide-y divide-slate-800 text-slate-300"></tbody>
                </table>
            </div>
        </div>

        <!-- TAB 5: ELEMENT REGISTER (SHEET 3) -->
        <div id="tab-register" class="hidden p-6 overflow-y-auto space-y-4">
            <div class="bg-slate-900 border border-slate-800 rounded-xl overflow-hidden shadow">
                <table class="w-full text-left text-xs border-collapse">
                    <thead>
                        <tr class="bg-slate-800 text-slate-400 uppercase border-b border-slate-700">
                            <th class="p-3">Region</th>
                            <th class="p-3">Name</th>
                            <th class="p-3">Type</th>
                            <th class="p-3">Role</th>
                            <th class="p-3">Explanation</th>
                        </tr>
                    </thead>
                    <tbody id="reg-tbody" class="divide-y divide-slate-800 text-slate-300"></tbody>
                </table>
            </div>
        </div>

    </main>

    <!-- Footer -->
    <footer class="bg-slate-900 border-t border-slate-800 py-2.5 px-6 text-center text-xs text-slate-500 flex justify-between items-center">
        <span>INDEPENDENT ELECTRICITY MARKET OPERATOR OF THE PHILIPPINES (IEMOP) — VISAYAS GRID</span>
        <span class="mono text-cyan-400">BUS-ORIENTED MARKET NETWORK MODEL (MNM_SLD_VIS_20260817)</span>
    </footer>

    <!-- Application Script -->
    <script>
        // Parsed Circuits from PDF Model
        const MODEL_DATA = {
            "PANAY": [
                { id: "08BARTAC_L01", type: "Transmission Line 230kV", mw: 142.1, status: "Normal" },
                { id: "08NABASDPP_U01", type: "Generator Unit", mw: 112.0, status: "Dispatched" },
                { id: "08DINGL_T1L1", type: "Line Circuit 138kV", mw: 85.4, status: "Normal" },
                { id: "08PEDC_U01", type: "Thermal Plant", mw: 164.0, status: "Dispatched" }
            ],
            "NEGROS": [
                { id: "06AMLAN_T1L1", type: "Sub-transmission 138kV", mw: 92.0, status: "Normal" },
                { id: "06CADSOL_G01", type: "Solar Plant", mw: 132.5, status: "Dispatched" },
                { id: "06BACOL_C01", type: "Collector Line", mw: 64.2, status: "Normal" }
            ],
            "CEBU": [
                { id: "05MAGDUGO_L01", type: "230kV Backbone", mw: 310.0, status: "Normal" },
                { id: "05CEDC_U01", type: "Thermal Unit", mw: 246.0, status: "Dispatched" }
            ],
            "LEYTE-SAMAR": [
                { id: "04LGPP_G01", type: "Geothermal Unit", mw: 520.0, status: "Dispatched" },
                { id: "04TABAN_T1L1", type: "230kV Line", mw: 180.0, status: "Normal" }
            ],
            "BOHOL": [
                { id: "07UBAY_T1L1", type: "138kV Tie Line", mw: 77.5, status: "Normal" },
                { id: "07LOBOC_G01", type: "Hydro Unit", mw: 3.3, status: "Dispatched" }
            ]
        };

        // Canvas Zoom Logic
        let scale = 1;
        const canvas = document.getElementById('sld-canvas');

        function zoomCanvas(factor) {
            scale *= factor;
            scale = Math.min(Math.max(0.6, scale), 3);
            canvas.style.transform = `scale(${scale})`;
        }

        function resetZoom() {
            scale = 1;
            canvas.style.transform = `scale(1)`;
        }

        function inspectSubstation(name, region, role, gen, lmp) {
            document.getElementById('drawer-node-name').innerText = name;
            document.getElementById('drawer-node-region').innerText = region + " REGION";
            document.getElementById('drawer-node-role').innerText = role;
            document.getElementById('drawer-node-gen').innerText = gen + " MW";
            document.getElementById('drawer-node-lmp').innerText = "₱" + lmp.toFixed(2) + " / MWh";
            
            const circuitsList = document.getElementById('drawer-circuits-list');
            circuitsList.innerHTML = (MODEL_DATA[region] || []).map(c => `
                <li class="flex justify-between border-b border-slate-800 pb-1">
                    <span>${c.id}</span>
                    <span class="text-emerald-400 font-bold">${c.mw} MW</span>
                </li>
            `).join('');

            document.getElementById('inspector-drawer').classList.remove('translate-x-full');
        }

        function closeDrawer() {
            document.getElementById('inspector-drawer').classList.add('translate-x-full');
        }

        function switchTab(tab) {
            ['mnm-graphics', 'circuit-matrix', 'market-summary', 'definitions', 'register'].forEach(t => {
                document.getElementById(`tab-${t}`).classList.add('hidden');
                document.getElementById(`btn-${t}`).classList.remove('bg-cyan-500', 'text-slate-950', 'font-semibold');
                document.getElementById(`btn-${t}`).classList.add('text-slate-300');
            });
            document.getElementById(`tab-${tab}`).classList.remove('hidden');
            document.getElementById(`btn-${tab}`).classList.add('bg-cyan-500', 'text-slate-950', 'font-semibold');
        }

        function toggleDarkMode() {
            document.documentElement.classList.toggle('dark');
        }

        function renderCircuitMatrix() {
            let html = '';
            Object.keys(MODEL_DATA).forEach(reg => {
                MODEL_DATA[reg].forEach(c => {
                    html += `
                        <tr class="hover:bg-slate-800/50">
                            <td class="p-3 font-bold text-orange-400">${reg}</td>
                            <td class="p-3 font-bold text-white">${c.id}</td>
                            <td class="p-3 text-cyan-400">${c.type}</td>
                            <td class="p-3 text-emerald-400 font-bold">${c.mw} MW</td>
                            <td class="p-3"><span class="bg-emerald-500/10 text-emerald-400 px-2 py-0.5 rounded border border-emerald-500/20">${c.status}</span></td>
                        </tr>
                    `;
                });
            });
            document.getElementById('circuits-tbody').innerHTML = html;
        }

        renderCircuitMatrix();
    </script>
</body>
</html>
