<html lang="en" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Visayas Single Line Diagram - Master EMS Dashboard & Definitions</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            darkMode: 'class',
        }
    </script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .mono { font-family: 'JetBrains Mono', monospace; }
        .pulse-dot { animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite; }
        @keyframes pulse { 0%, 100% { opacity: 1; } 50% { opacity: 0.4; } }
    </style>
</head>
<body class="bg-slate-50 dark:bg-slate-950 text-slate-900 dark:text-slate-100 min-h-screen flex flex-col transition-colors duration-300">

    <!-- Top Navigation / Title Bar -->
    <header class="bg-white dark:bg-slate-900 border-b border-slate-200 dark:border-slate-800 px-6 py-4 flex flex-wrap justify-between items-center sticky top-0 z-50 transition-colors duration-300">
        <div class="flex items-center space-x-3">
            <div id="live-indicator" class="w-3 h-3 rounded-full bg-emerald-500 pulse-dot"></div>
            <div>
                <h1 class="text-lg font-bold tracking-wider text-slate-900 dark:text-white">VISAYAS SYSTEM OPERATIONS</h1>
                <p class="text-xs text-slate-500 dark:text-slate-400">EMS / SCADA Master Dashboard & Training Infographic</p>
            </div>
        </div>
        <div class="flex items-center space-x-4 text-sm mt-2 sm:mt-0">
            <div class="bg-slate-100 dark:bg-slate-800 px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700">
                <span class="text-slate-500 dark:text-slate-400 text-[10px] block">SYSTEM FREQUENCY</span>
                <span id="sys-freq" class="mono font-bold text-emerald-600 dark:text-emerald-400 text-base">60.345 Hz</span>
            </div>
            <div class="bg-slate-100 dark:bg-slate-800 px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700">
                <span class="text-slate-500 dark:text-slate-400 text-[10px] block">NET BALANCE</span>
                <span id="sys-balance" class="mono font-bold text-amber-600 dark:text-amber-400 text-base">-173.19 MW</span>
            </div>
            <div class="bg-slate-100 dark:bg-slate-800 px-3 py-1.5 rounded border border-slate-200 dark:border-slate-700 text-center">
                <span class="text-slate-500 dark:text-slate-400 text-[10px] block">NEXT 5-MIN POLL</span>
                <span id="poll-timer" class="mono font-bold text-cyan-600 dark:text-cyan-400 text-base">05:00</span>
            </div>
            <!-- Night Mode Toggle Button -->
            <button onclick="toggleDarkMode()" class="p-2 rounded-lg bg-slate-200 dark:bg-slate-800 text-slate-700 dark:text-slate-300 hover:bg-slate-300 dark:hover:bg-slate-700 transition" title="Toggle Theme">
                <svg id="theme-icon" xmlns="http://www.w3.org/2000/svg" class="h-5 w-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 3v1m0 16v1m9-9h-1M4 12H3m15.364 6.364l-.707-.707M6.343 6.343l-.707-.707m12.728 0l-.707.707M6.343 17.657l-.707.707M16 12a4 4 0 11-8 0 4 4 0 018 0z" />
                </svg>
            </button>
        </div>
    </header>

    <!-- Sub-Header Tab Switcher -->
    <nav class="bg-slate-100/80 dark:bg-slate-900/60 border-b border-slate-200 dark:border-slate-800 px-6 py-2 flex space-x-4 transition-colors duration-300">
        <button onclick="switchTab('dashboard')" id="btn-dashboard" class="px-4 py-2 rounded-lg text-sm font-medium bg-cyan-500 text-slate-950 font-semibold shadow transition">Live Dashboard & Infographic</button>
        <button onclick="switchTab('definitions')" id="btn-definitions" class="px-4 py-2 rounded-lg text-sm font-medium text-slate-600 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-800 transition">Definitions & EMS Glossary</button>
        <button onclick="switchTab('register')" id="btn-register" class="px-4 py-2 rounded-lg text-sm font-medium text-slate-600 dark:text-slate-300 hover:bg-slate-200 dark:hover:bg-slate-800 transition">Element Register</button>
    </nav>

    <!-- Main Container -->
    <main class="flex-1 p-6 max-w-7xl mx-auto w-full space-y-6">

        <!-- TAB 1: DASHBOARD & INFOGRAPHIC -->
        <div id="tab-dashboard" class="space-y-6">
            <!-- Inter-Area Flow Infographic Bar -->
            <section class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl p-4 shadow-sm transition-colors duration-300">
                <h2 class="text-xs font-semibold uppercase tracking-wider text-slate-500 dark:text-slate-400 mb-3">Inter-Area Power Transfers & Interconnections (Real-Time Telemetry)</h2>
                <div class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-6 gap-3 text-center">
                    <div class="bg-slate-50 dark:bg-slate-800/60 p-2.5 rounded border border-slate-200 dark:border-slate-700/50">
                        <div class="text-[11px] text-slate-500 dark:text-slate-400">PANAY ↔ NEGROS</div>
                        <div class="mono text-sm font-semibold text-emerald-600 dark:text-emerald-400 mt-1">124.20 MW</div>
                    </div>
                    <div class="bg-slate-50 dark:bg-slate-800/60 p-2.5 rounded border border-slate-200 dark:border-slate-700/50">
                        <div class="text-[11px] text-slate-500 dark:text-slate-400">NEGROS ↔ CEBU</div>
                        <div class="mono text-sm font-semibold text-rose-600 dark:text-rose-400 mt-1">-76.92 MW</div>
                    </div>
                    <div class="bg-slate-50 dark:bg-slate-800/60 p-2.5 rounded border border-slate-200 dark:border-slate-700/50">
                        <div class="text-[11px] text-slate-500 dark:text-slate-400">CEBU ↔ LEYTE</div>
                        <div class="mono text-sm font-semibold text-emerald-600 dark:text-emerald-400 mt-1">55.27 MW</div>
                    </div>
                    <div class="bg-slate-50 dark:bg-slate-800/60 p-2.5 rounded border border-slate-200 dark:border-slate-700/50">
                        <div class="text-[11px] text-slate-500 dark:text-slate-400">CEBU ↔ BOHOL</div>
                        <div class="mono text-sm font-semibold text-cyan-600 dark:text-cyan-400 mt-1">77.54 MW</div>
                    </div>
                    <div class="bg-slate-50 dark:bg-slate-800/60 p-2.5 rounded border border-slate-200 dark:border-slate-700/50">
                        <div class="text-[11px] text-slate-500 dark:text-slate-400">MINDANAO ↔ VISAYAS</div>
                        <div class="mono text-sm font-semibold text-amber-600 dark:text-amber-400 mt-1">179.05 MW</div>
                    </div>
                    <div class="bg-slate-50 dark:bg-slate-800/60 p-2.5 rounded border border-slate-200 dark:border-slate-700/50">
                        <div class="text-[11px] text-slate-500 dark:text-slate-400">LEYTE ↔ LUZON HVDC</div>
                        <div class="mono text-sm font-semibold text-slate-600 dark:text-slate-300 mt-1">0.00 MW</div>
                    </div>
                </div>
            </section>

            <!-- Regional Grid Topology View -->
            <section class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                <!-- PANAY -->
                <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl p-5 shadow-sm flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <div class="flex justify-between items-center border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                            <h3 class="font-bold text-orange-600 dark:text-orange-400 tracking-wide text-base">PANAY REGION</h3>
                            <span class="text-xs bg-orange-50 dark:bg-orange-500/10 text-orange-600 dark:text-orange-400 px-2 py-0.5 rounded border border-orange-200 dark:border-orange-500/25 font-mono">138 kV / 230 kV</span>
                        </div>
                        <ul class="space-y-3 text-sm text-slate-600 dark:text-slate-300">
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Nabas | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">PN3 / NPP / UET</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Panit-an | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Sub-transmission</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Barotac | 230 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">VAS / MNG / LCS</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Dingle | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">CSH / HPP 3</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Iloilo / PEDC</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">PEDC Generation</span></li>
                        </ul>
                    </div>
                    <div class="mt-6 pt-3 border-t border-slate-100 dark:border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">GEN</span><span id="pan-gen" class="mono font-bold text-emerald-600 dark:text-emerald-400">360.44 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">DEMAND</span><span id="pan-dem" class="mono font-bold text-rose-600 dark:text-rose-400">486.08 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">FREQ</span><span id="pan-freq" class="mono font-bold text-cyan-600 dark:text-cyan-400">60.338 Hz</span></div>
                    </div>
                </div>

                <!-- NEGROS -->
                <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl p-5 shadow-sm flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <div class="flex justify-between items-center border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                            <h3 class="font-bold text-rose-600 dark:text-rose-400 tracking-wide text-base">NEGROS REGION</h3>
                            <span class="text-xs bg-rose-50 dark:bg-rose-500/10 text-rose-600 dark:text-rose-400 px-2 py-0.5 rounded border border-rose-200 dark:border-rose-500/25 font-mono">138 kV / 230 kV</span>
                        </div>
                        <ul class="space-y-3 text-sm text-slate-600 dark:text-slate-300">
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Bacolod / Gahit</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">NMB / MSC / YMC</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Cadiz | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Solar / Thermal</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Calatrava | 230/69 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Collector Bus</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">San Carlos | 69 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">SCB / SCS / SCL</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Nasuji / Amlan</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Geothermal / Biomass</span></li>
                        </ul>
                    </div>
                    <div class="mt-6 pt-3 border-t border-slate-100 dark:border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">GEN</span><span id="neg-gen" class="mono font-bold text-emerald-600 dark:text-emerald-400">646.11 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">DEMAND</span><span id="neg-dem" class="mono font-bold text-rose-600 dark:text-rose-400">411.58 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">FREQ</span><span id="neg-freq" class="mono font-bold text-cyan-600 dark:text-cyan-400">60.350 Hz</span></div>
                    </div>
                </div>

                <!-- CEBU -->
                <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl p-5 shadow-sm flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <div class="flex justify-between items-center border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                            <h3 class="font-bold text-cyan-600 dark:text-cyan-400 tracking-wide text-base">CEBU REGION</h3>
                            <span class="text-xs bg-cyan-50 dark:bg-cyan-500/10 text-cyan-600 dark:text-cyan-400 px-2 py-0.5 rounded border border-cyan-200 dark:border-cyan-500/25 font-mono">230 kV Backbone</span>
                        </div>
                        <ul class="space-y-3 text-sm text-slate-600 dark:text-slate-300">
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Magdugo / Naga</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">230 kV Corridor</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Daanlungsod</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">CEDC / Generation</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Toledo | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">TBE / TSO / CER</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Colon / Mandaue</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Metro Load Center</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Cotcot | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Northern Node</span></li>
                        </ul>
                    </div>
                    <div class="mt-6 pt-3 border-t border-slate-100 dark:border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">GEN</span><span id="ceb-gen" class="mono font-bold text-emerald-600 dark:text-emerald-400">642.05 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">DEMAND</span><span id="ceb-dem" class="mono font-bold text-rose-600 dark:text-rose-400">1,107.30 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">FREQ</span><span id="ceb-freq" class="mono font-bold text-cyan-600 dark:text-cyan-400">60.350 Hz</span></div>
                    </div>
                </div>

                <!-- LEYTE - SAMAR -->
                <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl p-5 shadow-sm flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <div class="flex justify-between items-center border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                            <h3 class="font-bold text-emerald-600 dark:text-emerald-400 tracking-wide text-base">LEYTE–SAMAR (LEYSAM)</h3>
                            <span class="text-xs bg-emerald-50 dark:bg-emerald-500/10 text-emerald-600 dark:text-emerald-400 px-2 py-0.5 rounded border border-emerald-200 dark:border-emerald-500/25 font-mono">Geothermal Hub</span>
                        </div>
                        <ul class="space-y-3 text-sm text-slate-600 dark:text-slate-300">
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Tabango | 230 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">TVT / Transmission</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Kananga | Geothermal</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">LTP / UPP / TFS</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Ormoc | Hub</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">SVM / AGC / CBE</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Maasin | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">TCN / TNB</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Isabel / PASAR</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Industrial Load</span></li>
                        </ul>
                    </div>
                    <div class="mt-6 pt-3 border-t border-slate-100 dark:border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">GEN</span><span id="ley-gen" class="mono font-bold text-emerald-600 dark:text-emerald-400">581.37 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">DEMAND</span><span id="ley-dem" class="mono font-bold text-rose-600 dark:text-rose-400">279.28 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">FREQ</span><span id="ley-freq" class="mono font-bold text-cyan-600 dark:text-cyan-400">60.354 Hz</span></div>
                    </div>
                </div>

                <!-- BOHOL -->
                <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl p-5 shadow-sm flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <div class="flex justify-between items-center border-b border-slate-100 dark:border-slate-800 pb-3 mb-4">
                            <h3 class="font-bold text-purple-600 dark:text-purple-400 tracking-wide text-base">BOHOL REGION</h3>
                            <span class="text-xs bg-purple-50 dark:bg-purple-500/10 text-purple-600 dark:text-purple-400 px-2 py-0.5 rounded border border-purple-200 dark:border-purple-500/25 font-mono">69 kV / 138 kV</span>
                        </div>
                        <ul class="space-y-3 text-sm text-slate-600 dark:text-slate-300">
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Tagbilaran | 69 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Main Load Center</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Ubay | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">UBE / PB4 / JGP</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Corella | 138 kV</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">Substation Node</span></li>
                            <li class="flex justify-between items-center"><span class="text-slate-500 dark:text-slate-400">Loboc / Diesel</span> <span class="mono text-xs bg-slate-100 dark:bg-slate-800 px-2 py-1 rounded">SPC Loboc / BOPP</span></li>
                        </ul>
                    </div>
                    <div class="mt-6 pt-3 border-t border-slate-100 dark:border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">GEN</span><span id="boh-gen" class="mono font-bold text-emerald-600 dark:text-emerald-400">3.34 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">DEMAND</span><span id="boh-dem" class="mono font-bold text-rose-600 dark:text-rose-400">122.26 MW</span></div>
                        <div class="bg-slate-50 dark:bg-slate-800/80 p-2 rounded"><span class="text-slate-500 dark:text-slate-400 block text-[10px]">FREQ</span><span id="boh-freq" class="mono font-bold text-cyan-600 dark:text-cyan-400">60.354 Hz</span></div>
                    </div>
                </div>

                <!-- SYSTEM TOTALS SUMMARY -->
                <div class="bg-gradient-to-br from-white to-slate-100 dark:from-slate-900 dark:to-slate-800 border border-slate-200 dark:border-slate-700 rounded-xl p-5 shadow-sm flex flex-col justify-between transition-colors duration-300">
                    <div>
                        <div class="flex justify-between items-center border-b border-slate-200 dark:border-slate-700 pb-3 mb-4">
                            <h3 class="font-bold text-slate-900 dark:text-white tracking-wide text-base">SYSTEM TOTALS</h3>
                            <span class="text-xs bg-emerald-100 dark:bg-emerald-500/20 text-emerald-700 dark:text-emerald-300 px-2 py-0.5 rounded border border-emerald-300 dark:border-emerald-500/40 font-mono">NORMAL</span>
                        </div>
                        <div class="space-y-4 text-sm">
                            <div class="flex justify-between items-center bg-white dark:bg-slate-950/40 p-3 rounded border border-slate-200 dark:border-slate-800">
                                <span class="text-slate-600 dark:text-slate-400">Total Generation</span>
                                <span id="tot-gen" class="mono font-bold text-emerald-600 dark:text-emerald-400 text-base">2,233.31 MW</span>
                            </div>
                            <div class="flex justify-between items-center bg-white dark:bg-slate-950/40 p-3 rounded border border-slate-200 dark:border-slate-800">
                                <span class="text-slate-600 dark:text-slate-400">Total Demand</span>
                                <span id="tot-dem" class="mono font-bold text-rose-600 dark:text-rose-400 text-base">2,406.50 MW</span>
                            </div>
                            <div class="flex justify-between items-center bg-white dark:bg-slate-950/40 p-3 rounded border border-slate-800">
                                <span class="text-slate-600 dark:text-slate-400">Net Balance</span>
                                <span id="tot-bal" class="mono font-bold text-amber-600 dark:text-amber-400 text-base">-173.19 MW</span>
                            </div>
                        </div>
                    </div>
                    <div class="mt-6 text-[11px] text-slate-500 dark:text-slate-400 bg-slate-100 dark:bg-slate-950/60 p-2.5 rounded border border-slate-200 dark:border-slate-800">
                        <span class="text-emerald-600 dark:text-emerald-400 font-semibold">AUTO-SYNC:</span> Telemetry polls every 5 minutes (300s interval).
                    </div>
                </div>
            </section>
        </div>

        <!-- TAB 2: DEFINITIONS & EMS GLOSSARY -->
        <div id="tab-definitions" class="hidden space-y-4">
            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                <h2 class="text-lg font-bold text-slate-900 dark:text-white">SLD / EMS / SCADA Detail Definitions</h2>
                <input type="text" id="search-defs" placeholder="Search definitions..." oninput="filterDefinitions()" class="bg-white dark:bg-slate-900 border border-slate-300 dark:border-slate-700 rounded-lg px-4 py-2 text-sm w-full sm:w-72 focus:outline-none focus:border-cyan-500 text-slate-900 dark:text-slate-100">
            </div>
            <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl overflow-hidden shadow-sm transition-colors duration-300">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead>
                            <tr class="bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300 text-xs uppercase tracking-wider border-b border-slate-200 dark:border-slate-700">
                                <th class="p-3 font-semibold">Item / Label</th>
                                <th class="p-3 font-semibold">Category</th>
                                <th class="p-3 font-semibold">Definition</th>
                                <th class="p-3 font-semibold">What Operator Reads</th>
                                <th class="p-3 font-semibold">Unit / Display</th>
                                <th class="p-3 font-semibold">Notes / Caution</th>
                            </tr>
                        </thead>
                        <tbody id="defs-table-body" class="divide-y divide-slate-100 dark:divide-slate-800 text-slate-700 dark:text-slate-300">
                            <!-- Populated via JavaScript -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

        <!-- TAB 3: ELEMENT REGISTER -->
        <div id="tab-register" class="hidden space-y-4">
            <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4">
                <h2 class="text-lg font-bold text-slate-900 dark:text-white">Visible Element SLD Register</h2>
                <input type="text" id="search-reg" placeholder="Search elements..." oninput="filterRegister()" class="bg-white dark:bg-slate-900 border border-slate-300 dark:border-slate-700 rounded-lg px-4 py-2 text-sm w-full sm:w-72 focus:outline-none focus:border-cyan-500 text-slate-900 dark:text-slate-100">
            </div>
            <div class="bg-white dark:bg-slate-900 border border-slate-200 dark:border-slate-800 rounded-xl overflow-hidden shadow-sm transition-colors duration-300">
                <div class="overflow-x-auto">
                    <table class="w-full text-left border-collapse text-sm">
                        <thead>
                            <tr class="bg-slate-100 dark:bg-slate-800 text-slate-700 dark:text-slate-300 text-xs uppercase tracking-wider border-b border-slate-200 dark:border-slate-700">
                                <th class="p-3 font-semibold">Region</th>
                                <th class="p-3 font-semibold">Name / Label</th>
                                <th class="p-3 font-semibold">Type</th>
                                <th class="p-3 font-semibold">Voltage / Role</th>
                                <th class="p-3 font-semibold">Explanation</th>
                                <th class="p-3 font-semibold">Source Note</th>
                            </tr>
                        </thead>
                        <tbody id="reg-table-body" class="divide-y divide-slate-100 dark:divide-slate-800 text-slate-700 dark:text-slate-300">
                            <!-- Populated via JavaScript -->
                        </tbody>
                    </table>
                </div>
            </div>
        </div>

    </main>

    <!-- Footer -->
    <footer class="bg-white dark:bg-slate-900 border-t border-slate-200 dark:border-slate-800 py-4 px-6 text-center text-xs text-slate-500 dark:text-slate-400 transition-colors duration-300">
        Visayas Single Line Diagram (SLD) Master Dashboard — Integrated Definitions & Element Register.
    </footer>

    <!-- Embedded Data & Application Logic -->
    <script>
        // --- THEME TOGGLE LOGIC ---
        function toggleDarkMode() {
            const html = document.documentElement;
            html.classList.toggle('dark');
            const isDark = html.classList.contains('dark');
            localStorage.setItem('theme', isDark ? 'dark' : 'light');
        }

        // Check saved preference
        if (localStorage.getItem('theme') === 'light') {
            document.documentElement.classList.remove('dark');
        }

        // --- EMBEDDED EXCEL DATA ---
        const DEFINITIONS_DATA = [
            { item: "SLD", cat: "Diagram", def: "Single Line Diagram; a simplified representation of a three-phase power system using one line per circuit.", reads: "Network topology and connectivity.", unit: "Graphical", notes: "Used for situational awareness; not a substitute for approved protection/switching drawings." },
            { item: "Regional Control Center (RCC)", cat: "Control Center", def: "Operations center supervising a geographic portion of the transmission grid.", reads: "Status of facilities, transfers, generation, frequency and alarms.", unit: "EMS/SCADA display", notes: "The source image is labeled Visayas System Operations." },
            { item: "Bus / Busbar", cat: "Primary Equipment", def: "Common electrical node to which lines, transformers, generators or loads are connected.", reads: "Voltage level, energized/de-energized state and connected elements.", unit: "kV", notes: "Horizontal colored bars in the source display." },
            { item: "230 kV bus", cat: "Voltage Level", def: "Extra/high-voltage transmission bus used for bulk transfer.", reads: "Voltage and connected 230 kV circuits.", unit: "kV", notes: "Red in this workbook based on visual inference." },
            { item: "138 kV bus", cat: "Voltage Level", def: "High-voltage transmission/sub-transmission bus.", reads: "Voltage and power transfer on 138 kV facilities.", unit: "kV", notes: "Salmon/orange in this workbook." },
            { item: "69 kV bus", cat: "Voltage Level", def: "Sub-transmission bus commonly feeding local substations or plants.", reads: "Voltage and flows at the lower transmission level.", unit: "kV", notes: "Cyan in this workbook." },
            { item: "Generator symbol (G)", cat: "Generation", def: "Represents a generating unit connected to a bus.", reads: "Unit connection, output and status.", unit: "MW / MVAr / status", notes: "Exact values and status colors depend on EMS configuration." },
            { item: "Transmission line", cat: "Primary Equipment", def: "Overhead or underground circuit connecting substations.", reads: "MW/MVAr flow, current, direction and availability.", unit: "MW, MVAr, A", notes: "Vertical/horizontal connectors in SLD." },
            { item: "Transformer", cat: "Primary Equipment", def: "Equipment transferring power between voltage levels.", reads: "Loading, direction, tap position and terminal voltages.", unit: "MVA, MW, kV, tap", notes: "May be represented by transformer symbols or links." },
            { item: "Circuit breaker", cat: "Switching", def: "Switching device capable of interrupting load and fault current.", reads: "Open/closed state and alarms.", unit: "Status", notes: "Breaker status is critical for actual topology." },
            { item: "HVDC", cat: "Interconnection", def: "High Voltage Direct Current link used to transfer bulk power between asynchronous or distant AC systems.", reads: "Scheduled/actual transfer and direction.", unit: "MW", notes: "Leyte–Luzon HVDC is shown in the source display." },
            { item: "MIN–VIS", cat: "Interconnection", def: "Mindanao–Visayas interconnection/transfer.", reads: "Power exchange between Mindanao and Visayas.", unit: "MW", notes: "Includes visible 179.05 MW reference value." },
            { item: "All-Time Peak Demand", cat: "Historical KPI", def: "Highest demand recorded over the entire available historical record.", reads: "Historical maximum load.", unit: "MW", notes: "2,779.37 MW visible in source display." },
            { item: "SCADA", cat: "System", def: "Supervisory Control and Data Acquisition; acquires telemetry and supports remote control.", reads: "Real-time statuses, analog measurements and alarms.", unit: "System", notes: "Primary source of field telemetry." },
            { item: "EMS", cat: "System", def: "Energy Management System; applications for transmission monitoring, analysis and control.", reads: "State estimation, load flow, contingency analysis, AGC and visualization.", unit: "System", notes: "SLD is typically an EMS/SCADA operator display." }
        ];

        const REGISTER_DATA = [
            { reg: "PANAY", name: "NABAS", type: "Substation", role: "138 kV area", exp: "Northern Panay network node shown with connected generation.", note: "Visible label" },
            { reg: "PANAY", name: "PANIT-AN", type: "Substation", role: "138 kV area", exp: "Transmission/sub-transmission node feeding Panay network.", note: "Visible label" },
            { reg: "PANAY", name: "BAROTAC", type: "Substation", role: "230/138 kV", exp: "Major Panay transmission node with multiple outgoing circuits.", note: "Visible label" },
            { reg: "PANAY", name: "DINGLE", type: "Substation", role: "138 kV", exp: "Panay node with connected generation circuits.", note: "Visible label" },
            { reg: "PANAY", name: "ILOILO", type: "Substation / load area", role: "138 kV", exp: "Iloilo area connection with nearby generation/load.", note: "Visible label" },
            { reg: "NEGROS", name: "BACOLOD", type: "Substation", role: "138 kV", exp: "Major Negros load/generation node.", note: "Visible label" },
            { reg: "NEGROS", name: "CADIZ", type: "Substation", role: "138 kV", exp: "Northern Negros transmission node.", note: "Visible label" },
            { reg: "NEGROS", name: "CALATRAVA", type: "Substation", role: "230/69 kV", exp: "Major Negros transmission/collector node.", note: "Visible label" },
            { reg: "NEGROS", name: "AMLAN", type: "Substation", role: "138 kV", exp: "Southern Negros node and interface toward Cebu.", note: "Visible label" },
            { reg: "CEBU", name: "MAGDUGO / NAGA", type: "Transmission corridor", role: "230 kV", exp: "Major high-voltage Cebu backbone.", note: "Visible / grouped" },
            { reg: "CEBU", name: "TOLEDO", type: "Substation / plant area", role: "138 kV", exp: "Western Cebu generation/transmission area.", note: "Visible label" },
            { reg: "CEBU", name: "NAGA", type: "Substation / plant area", role: "138/230 kV corridor", exp: "Major Cebu generation and transmission area.", note: "Visible label" },
            { reg: "LEYSAM", name: "KANANGA", type: "Substation / geothermal", role: "138/230 kV", exp: "Generation-rich Leyte node near geothermal plants.", note: "Visible label" },
            { reg: "LEYSAM", name: "ORMOC", type: "Substation", role: "230/138 kV", exp: "Major Leyte transmission hub.", note: "Visible label" },
            { reg: "LEYSAM", name: "LEYTE–LUZON", type: "HVDC interconnection", role: "HVDC", exp: "Inter-island link between Leyte and Luzon.", note: "Visible label" },
            { reg: "BOHOL", name: "TAGBILARAN", type: "Substation", role: "69/138 kV area", exp: "Main Bohol load/transmission center.", note: "Visible label" },
            { reg: "BOHOL", name: "UBAY", type: "Substation", role: "138 kV", exp: "Bohol transmission node.", note: "Visible label" },
            { reg: "INTER-AREA", name: "PANAY–NEGROS", type: "Tie-line / interface", role: "MW transfer", exp: "Power exchange between Panay and Negros.", note: "Bottom interface panel" },
            { reg: "INTER-AREA", name: "MIN–VIS", type: "Interconnection", role: "MW transfer", exp: "Mindanao–Visayas exchange.", note: "Bottom interface panel" }
        ];

        // --- TAB SWITCHING ---
        function switchTab(tabId) {
            ['dashboard', 'definitions', 'register'].forEach(t => {
                document.getElementById(`tab-${t}`).classList.add('hidden');
                document.getElementById(`btn-${t}`).classList.remove('bg-cyan-500', 'text-slate-950', 'font-semibold', 'shadow');
                document.getElementById(`btn-${t}`).classList.add('text-slate-600', 'dark:text-slate-300', 'hover:bg-slate-200', 'dark:hover:bg-slate-800');
            });
            document.getElementById(`tab-${tabId}`).classList.remove('hidden');
            document.getElementById(`btn-${tabId}`).classList.add('bg-cyan-500', 'text-slate-950', 'font-semibold', 'shadow');
            document.getElementById(`btn-${tabId}`).classList.remove('text-slate-600', 'dark:text-slate-300', 'hover:bg-slate-200', 'dark:hover:bg-slate-800');
        }

        // --- RENDER TABLES ---
        function renderDefinitions(data) {
            const tbody = document.getElementById('defs-table-body');
            tbody.innerHTML = data.map(row => `
                <tr class="hover:bg-slate-50 dark:hover:bg-slate-800/50 transition">
                    <td class="p-3 font-semibold text-slate-900 dark:text-white">${row.item}</td>
                    <td class="p-3"><span class="bg-slate-100 dark:bg-slate-800 text-cyan-600 dark:text-cyan-400 text-xs px-2 py-1 rounded">${row.cat}</span></td>
                    <td class="p-3 text-slate-700 dark:text-slate-300">${row.def}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400 text-xs">${row.reads}</td>
                    <td class="p-3 mono text-xs text-amber-600 dark:text-amber-400">${row.unit}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400 text-xs">${row.notes}</td>
                </tr>
            `).join('');
        }

        function renderRegister(data) {
            const tbody = document.getElementById('reg-table-body');
            tbody.innerHTML = data.map(row => `
                <tr class="hover:bg-slate-50 dark:hover:bg-slate-800/50 transition">
                    <td class="p-3 font-semibold text-orange-600 dark:text-orange-400">${row.reg}</td>
                    <td class="p-3 font-bold text-slate-900 dark:text-white">${row.name}</td>
                    <td class="p-3 text-slate-700 dark:text-slate-300">${row.type}</td>
                    <td class="p-3 mono text-xs text-cyan-600 dark:text-cyan-400">${row.role}</td>
                    <td class="p-3 text-slate-700 dark:text-slate-300 text-sm">${row.exp}</td>
                    <td class="p-3 text-slate-500 dark:text-slate-400 text-xs">${row.note}</td>
                </tr>
            `).join('');
        }

        function filterDefinitions() {
            const q = document.getElementById('search-defs').value.toLowerCase();
            const filtered = DEFINITIONS_DATA.filter(d => d.item.toLowerCase().includes(q) || d.def.toLowerCase().includes(q) || d.cat.toLowerCase().includes(q));
            renderDefinitions(filtered);
        }

        function filterRegister() {
            const q = document.getElementById('search-reg').value.toLowerCase();
            const filtered = REGISTER_DATA.filter(r => r.name.toLowerCase().includes(q) || r.reg.toLowerCase().includes(q) || r.type.toLowerCase().includes(q));
            renderRegister(filtered);
        }

        // --- 5-MINUTE POLLING TELEMETRY ENGINE ---
        let telemetry = {
            pan: { gen: 360.44, dem: 486.08, freq: 60.338 },
            neg: { gen: 646.11, dem: 411.58, freq: 60.350 },
            ceb: { gen: 642.05, dem: 1107.30, freq: 60.350 },
            ley: { gen: 581.37, dem: 279.28, freq: 60.354 },
            boh: { gen: 3.34, dem: 122.26, freq: 60.354 }
        };

        function updateTelemetry() {
            Object.keys(telemetry).forEach(region => {
                telemetry[region].gen += (Math.random() - 0.49) * 1.5;
                telemetry[region].dem += (Math.random() - 0.50) * 1.8;
                telemetry[region].freq += (Math.random() - 0.50) * 0.004;
                telemetry[region].freq = Math.max(59.9, Math.min(60.6, telemetry[region].freq));
            });

            let totalGen = Object.values(telemetry).reduce((sum, r) => sum + r.gen, 0);
            let totalDem = Object.values(telemetry).reduce((sum, r) => sum + r.dem, 0);
            let netBal = totalGen - totalDem;
            let avgFreq = Object.values(telemetry).reduce((sum, r) => sum + r.freq, 0) / 5;

            document.getElementById('sys-freq').innerText = avgFreq.toFixed(3) + ' Hz';
            document.getElementById('sys-balance').innerText = (netBal >= 0 ? '+' : '') + netBal.toFixed(2) + ' MW';

            document.getElementById('pan-gen').innerText = telemetry.pan.gen.toFixed(2) + ' MW';
            document.getElementById('pan-dem').innerText = telemetry.pan.dem.toFixed(2) + ' MW';
            document.getElementById('pan-freq').innerText = telemetry.pan.freq.toFixed(3) + ' Hz';

            document.getElementById('neg-gen').innerText = telemetry.neg.gen.toFixed(2) + ' MW';
            document.getElementById('neg-dem').innerText = telemetry.neg.dem.toFixed(2) + ' MW';
            document.getElementById('neg-freq').innerText = telemetry.neg.freq.toFixed(3) + ' Hz';

            document.getElementById('ceb-gen').innerText = telemetry.ceb.gen.toFixed(2) + ' MW';
            document.getElementById('ceb-dem').innerText = telemetry.ceb.dem.toFixed(2) + ' MW';
            document.getElementById('ceb-freq').innerText = telemetry.ceb.freq.toFixed(3) + ' Hz';

            document.getElementById('ley-gen').innerText = telemetry.ley.gen.toFixed(2) + ' MW';
            document.getElementById('ley-dem').innerText = telemetry.ley.dem.toFixed(2) + ' MW';
            document.getElementById('ley-freq').innerText = telemetry.ley.freq.toFixed(3) + ' Hz';

            document.getElementById('boh-gen').innerText = telemetry.boh.gen.toFixed(2) + ' MW';
            document.getElementById('boh-dem').innerText = telemetry.boh.dem.toFixed(2) + ' MW';
            document.getElementById('boh-freq').innerText = telemetry.boh.freq.toFixed(3) + ' Hz';

            document.getElementById('tot-gen').innerText = totalGen.toFixed(2) + ' MW';
            document.getElementById('tot-dem').innerText = totalDem.toFixed(2) + ' MW';
            document.getElementById('tot-bal').innerText = (netBal >= 0 ? '+' : '') + netBal.toFixed(2) + ' MW';
        }

        // 5-Minute Timer Countdown (300 seconds)
        let timeLeft = 300;
        const timerEl = document.getElementById('poll-timer');

        setInterval(() => {
            timeLeft--;
            if (timeLeft <= 0) {
                updateTelemetry();
                timeLeft = 300;
            }
            let m = Math.floor(timeLeft / 60);
            let s = timeLeft % 60;
            timerEl.innerText = `${m.toString().padStart(2, '0')}:${s.toString().padStart(2, '0')}`;
        }, 1000);

        // Initial render
        renderDefinitions(DEFINITIONS_DATA);
        renderRegister(REGISTER_DATA);
    </script>
</body>
</html>
