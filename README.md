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

        .workspace { display: flex; flex: 1; width: 100%; height: calc(100% - 48px); position: relative; overflow: hidden; }
        
        .sidebar-left { width: 260px; background: #111827; border-right: 1px solid #1f2937; display: flex; flex-direction: column; flex-shrink: 0; z-index: 5; }
        .panel-section-title { font-size: 10px; font-weight: 700; text-transform: uppercase; color: #9ca3af; letter-spacing: 0.8px; padding: 10px 12px 6px 12px; }
        .palette-grid { display: grid; grid-template-columns: repeat(2, 1fr); gap: 8px; padding: 8px 12px; }
        .palette-item { background: #1f2937; border: 1px solid #374151; border-radius: 6px; padding: 10px; display: flex; flex-direction: column; align-items: center; gap: 6px; cursor: pointer; transition: all 0.2s; touch-action: manipulation; }
        .palette-item:hover { background: #374151; border-color: #60a5fa; transform: translateY(-1px); }
        .palette-item span { font-size: 11px; color: #d1d5db; font-weight: 500; }
        .palette-icon { width: 24px; height: 24px; display: flex; align-items: center; justify-content: center; pointer-events: none; }

        .telemetry-box { background: #0f172a; border-top: 1px solid #1f2937; padding: 12px; margin-top: auto; font-size: 11px; display: flex; flex-direction: column; gap: 6px; }
        .telemetry-row { display: flex; justify-content: space
