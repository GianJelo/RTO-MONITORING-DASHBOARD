<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Visayas Single Line Diagram - Monitoring & Infographic</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&family=JetBrains+Mono:wght@400;600&display=swap" rel="stylesheet">
    <style>
        body { font-family: 'Inter', sans-serif; }
        .mono { font-family: 'JetBrains Mono', monospace; }
        .glow-red { text-shadow: 0 0 10px rgba(239, 68, 68, 0.5); }
        .glow-emerald { text-shadow: 0 0 10px rgba(16, 185, 129, 0.5); }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex flex-col">

    <!-- Top Navigation / Title Bar -->
    <header class="bg-slate-900 border-b border-slate-800 px-6 py-4 flex flex-wrap justify-between items-center sticky top-0 z-50">
        <div class="flex items-center space-x-3">
            <div class="w-3 h-3 rounded-full bg-emerald-500 animate-pulse"></div>
            <div>
                <h1 class="text-lg font-bold tracking-wider text-white">VISAYAS SYSTEM OPERATIONS</h1>
                <p class="text-xs text-slate-400">Energy Management System (EMS) / SCADA Training Infographic</p>
            </div>
        </div>
        <div class="flex items-center space-x-6 text-sm mt-2 sm:mt-0">
            <div class="bg-slate-800 px-3 py-1.5 rounded border border-slate-700">
                <span class="text-slate-400 text-xs block">SYSTEM FREQUENCY</span>
                <span class="mono font-bold text-emerald-400 text-base">60.345 Hz</span>
            </div>
            <div class="bg-slate-800 px-3 py-1.5 rounded border border-slate-700">
                <span class="text-slate-400 text-xs block">ALL-TIME PEAK</span>
                <span class="mono font-bold text-amber-400 text-base">2,779.37 MW</span>
            </div>
            <div class="bg-slate-800 px-3 py-1.5 rounded border border-slate-700">
                <span class="text-slate-400 text-xs block">MIN-VIS INTERCONNECTION</span>
                <span class="mono font-bold text-cyan-400 text-base">179.05 MW</span>
            </div>
        </div>
    </header>

    <!-- Main Container -->
    <main class="flex-1 p-6 max-w-7xl mx-auto w-full space-y-6">

        <!-- Inter-Area Flow Infographic Bar -->
        <section class="bg-slate-900/80 border border-slate-800 rounded-xl p-4 shadow-lg">
            <h2 class="text-xs font-semibold uppercase tracking-wider text-slate-400 mb-3">Inter-Area Power Transfers & Interconnections</h2>
            <div class="grid grid-cols-2 md:grid-cols-4 lg:grid-cols-6 gap-3 text-center">
                <div class="bg-slate-800/60 p-2.5 rounded border border-slate-700/50">
                    <div class="text-[11px] text-slate-400">PANAY ↔ NEGROS</div>
                    <div class="mono text-sm font-semibold text-emerald-400 mt-1">124.20 MW</div>
                </div>
                <div class="bg-slate-800/60 p-2.5 rounded border border-slate-700/50">
                    <div class="text-[11px] text-slate-400">NEGROS ↔ CEBU</div>
                    <div class="mono text-sm font-semibold text-rose-400 mt-1">-76.92 MW</div>
                </div>
                <div class="bg-slate-800/60 p-2.5 rounded border border-slate-700/50">
                    <div class="text-[11px] text-slate-400">CEBU ↔ LEYTE</div>
                    <div class="mono text-sm font-semibold text-emerald-400 mt-1">55.27 MW</div>
                </div>
                <div class="bg-slate-800/60 p-2.5 rounded border border-slate-700/50">
                    <div class="text-[11px] text-slate-400">CEBU ↔ BOHOL</div>
                    <div class="mono text-sm font-semibold text-cyan-400 mt-1">77.54 MW</div>
                </div>
                <div class="bg-slate-800/60 p-2.5 rounded border border-slate-700/50">
                    <div class="text-[11px] text-slate-400">MINDANAO ↔ VISAYAS</div>
                    <div class="mono text-sm font-semibold text-amber-400 mt-1">179.05 MW</div>
                </div>
                <div class="bg-slate-800/60 p-2.5 rounded border border-slate-700/50">
                    <div class="text-[11px] text-slate-400">LEYTE ↔ LUZON HVDC</div>
                    <div class="mono text-sm font-semibold text-slate-300 mt-1">0.00 MW</div>
                </div>
            </div>
        </section>

        <!-- Regional Grid Topology View -->
        <section class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">

            <!-- PANAY -->
            <div class="bg-slate-900 border border-slate-800 rounded-xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-800 pb-3 mb-4">
                        <h3 class="font-bold text-orange-400 tracking-wide text-base">PANAY REGION</h3>
                        <span class="text-xs bg-orange-500/10 text-orange-400 px-2 py-0.5 rounded border border-orange-500/25 font-mono">138 kV / 230 kV</span>
                    </div>
                    <ul class="space-y-3 text-sm text-slate-300">
                        <li class="flex justify-between items-center"><span class="text-slate-400">Nabas | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">PN3 / NPP / UET</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Panit-an | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Sub-transmission</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Barotac | 230 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">VAS / MNG / LCS</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Dingle | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">CSH / HPP 3</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Iloilo / PEDC</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">PEDC Generation</span></li>
                    </ul>
                </div>
                <div class="mt-6 pt-3 border-t border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">GEN</span><span class="mono font-bold text-emerald-400">360.44 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">DEMAND</span><span class="mono font-bold text-rose-400">486.08 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">FREQ</span><span class="mono font-bold text-cyan-400">60.338 Hz</span></div>
                </div>
            </div>

            <!-- NEGROS -->
            <div class="bg-slate-900 border border-slate-800 rounded-xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-800 pb-3 mb-4">
                        <h3 class="font-bold text-rose-400 tracking-wide text-base">NEGROS REGION</h3>
                        <span class="text-xs bg-rose-500/10 text-rose-400 px-2 py-0.5 rounded border border-rose-500/25 font-mono">138 kV / 230 kV</span>
                    </div>
                    <ul class="space-y-3 text-sm text-slate-300">
                        <li class="flex justify-between items-center"><span class="text-slate-400">Bacolod / Gahit</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">NMB / MSC / YMC</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Cadiz | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Solar / Thermal</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Calatrava | 230/69 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Collector Bus</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">San Carlos | 69 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">SCB / SCS / SCL</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Nasuji / Amlan</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Geothermal / Biomass</span></li>
                    </ul>
                </div>
                <div class="mt-6 pt-3 border-t border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">GEN</span><span class="mono font-bold text-emerald-400">646.11 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">DEMAND</span><span class="mono font-bold text-rose-400">411.58 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">FREQ</span><span class="mono font-bold text-cyan-400">60.350 Hz</span></div>
                </div>
            </div>

            <!-- CEBU -->
            <div class="bg-slate-900 border border-slate-800 rounded-xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-800 pb-3 mb-4">
                        <h3 class="font-bold text-cyan-400 tracking-wide text-base">CEBU REGION</h3>
                        <span class="text-xs bg-cyan-500/10 text-cyan-400 px-2 py-0.5 rounded border border-cyan-500/25 font-mono">230 kV Backbone</span>
                    </div>
                    <ul class="space-y-3 text-sm text-slate-300">
                        <li class="flex justify-between items-center"><span class="text-slate-400">Magdugo / Naga</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">230 kV Corridor</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Daanlungsod</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">CEDC / Generation</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Toledo | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">TBE / TSO / CER</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Colon / Mandaue</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Metro Load Center</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Cotcot | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Northern Node</span></li>
                    </ul>
                </div>
                <div class="mt-6 pt-3 border-t border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">GEN</span><span class="mono font-bold text-emerald-400">642.05 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">DEMAND</span><span class="mono font-bold text-rose-400">1,107.30 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">FREQ</span><span class="mono font-bold text-cyan-400">60.350 Hz</span></div>
                </div>
            </div>

            <!-- LEYTE - SAMAR -->
            <div class="bg-slate-900 border border-slate-800 rounded-xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-800 pb-3 mb-4">
                        <h3 class="font-bold text-emerald-400 tracking-wide text-base">LEYTE–SAMAR (LEYSAM)</h3>
                        <span class="text-xs bg-emerald-500/10 text-emerald-400 px-2 py-0.5 rounded border border-emerald-500/25 font-mono">Geothermal Hub</span>
                    </div>
                    <ul class="space-y-3 text-sm text-slate-300">
                        <li class="flex justify-between items-center"><span class="text-slate-400">Tabango | 230 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">TVT / Transmission</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Kananga | Geothermal</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">LTP / UPP / TFS</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Ormoc | Hub</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">SVM / AGC / CBE</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Maasin | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">TCN / TNB</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Isabel / PASAR</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Industrial Load</span></li>
                    </ul>
                </div>
                <div class="mt-6 pt-3 border-t border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">GEN</span><span class="mono font-bold text-emerald-400">581.37 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">DEMAND</span><span class="mono font-bold text-rose-400">279.28 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">FREQ</span><span class="mono font-bold text-cyan-400">60.354 Hz</span></div>
                </div>
            </div>

            <!-- BOHOL -->
            <div class="bg-slate-900 border border-slate-800 rounded-xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-800 pb-3 mb-4">
                        <h3 class="font-bold text-purple-400 tracking-wide text-base">BOHOL REGION</h3>
                        <span class="text-xs bg-purple-500/10 text-purple-400 px-2 py-0.5 rounded border border-purple-500/25 font-mono">69 kV / 138 kV</span>
                    </div>
                    <ul class="space-y-3 text-sm text-slate-300">
                        <li class="flex justify-between items-center"><span class="text-slate-400">Tagbilaran | 69 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Main Load Center</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Ubay | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">UBE / PB4 / JGP</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Corella | 138 kV</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">Substation Node</span></li>
                        <li class="flex justify-between items-center"><span class="text-slate-400">Loboc / Diesel</span> <span class="mono text-xs bg-slate-800 px-2 py-1 rounded">SPC Loboc / BOPP</span></li>
                    </ul>
                </div>
                <div class="mt-6 pt-3 border-t border-slate-800 grid grid-cols-3 gap-2 text-center text-xs">
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">GEN</span><span class="mono font-bold text-emerald-400">3.34 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">DEMAND</span><span class="mono font-bold text-rose-400">122.26 MW</span></div>
                    <div class="bg-slate-800/80 p-2 rounded"><span class="text-slate-400 block text-[10px]">FREQ</span><span class="mono font-bold text-cyan-400">60.354 Hz</span></div>
                </div>
            </div>

            <!-- SYSTEM TOTALS SUMMARY -->
            <div class="bg-gradient-to-br from-slate-900 to-slate-800 border border-slate-700 rounded-xl p-5 shadow-lg flex flex-col justify-between">
                <div>
                    <div class="flex justify-between items-center border-b border-slate-700 pb-3 mb-4">
                        <h3 class="font-bold text-white tracking-wide text-base">SYSTEM TOTALS</h3>
                        <span class="text-xs bg-emerald-500/20 text-emerald-300 px-2 py-0.5 rounded border border-emerald-500/40 font-mono">NORMAL</span>
                    </div>
                    <div class="space-y-4 text-sm">
                        <div class="flex justify-between items-center bg-slate-950/40 p-3 rounded border border-slate-800">
                            <span class="text-slate-400">Total Generation</span>
                            <span class="mono font-bold text-emerald-400 text-base">2,233.31 MW</span>
                        </div>
                        <div class="flex justify-between items-center bg-slate-950/40 p-3 rounded border border-slate-800">
                            <span class="text-slate-400">Total Demand</span>
                            <span class="mono font-bold text-rose-400 text-base">2,406.50 MW</span>
                        </div>
                        <div class="flex justify-between items-center bg-slate-950/40 p-3 rounded border border-slate-800">
                            <span class="text-slate-400">Net Balance</span>
                            <span class="mono font-bold text-amber-400 text-base">-173.19 MW</span>
                        </div>
                    </div>
                </div>
                <div class="mt-6 text-[11px] text-slate-400 bg-slate-950/60 p-2.5 rounded border border-slate-800">
                    <span class="text-amber-400 font-semibold">LEGEND NOTE:</span> RED = 230 kV Bus | ORANGE = 138 kV Bus | CYAN = 69 kV Bus | PURPLE = Generation.
                </div>
            </div>

        </section>

    </main>

    <!-- Footer -->
    <footer class="bg-slate-900 border-t border-slate-800 py-4 px-6 text-center text-xs text-slate-400">
        Visayas Single Line Diagram (SLD) Training & Infographic Version — Powered by EMS / SCADA Data Standards.
    </footer>

</body>
</html>
