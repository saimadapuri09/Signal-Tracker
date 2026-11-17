# Signal-Tracker
Real-time dashboard for 15-minute BTC, ETH, XRP, and SOL markets signal trading. Combines live exchange data with technical indicators to generate entry signals, show confidence scores, and track signal performance.

<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>15m Signal Desk</title>
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@400;500;600&family=IBM+Plex+Sans:wght@400;500;600;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #131311;
    --panel: #1a1a17;
    --panel-2: #22211d;
    --line: #2d2c27;
    --line-strong: #44423a;
    --text: #e6e1d5;
    --text-2: #bdb7aa;
    --muted: #8a8478;
    --up: #7aae84;
    --down: #d77a5d;
    --neutral: #cda753;
    --accent: #93adc0;
    --up-tint: rgba(122,174,132,0.14);
    --down-tint: rgba(215,122,93,0.14);
    --neutral-tint: rgba(205,167,83,0.14);
    --accent-tint: rgba(147,173,192,0.16);
    --phase-open: #2d2c27;
    --phase-build: #3a3f3c;
    --phase-confirm: rgba(147,173,192,0.45);
    --phase-lock: rgba(215,122,93,0.45);
    --sans: "IBM Plex Sans", ui-sans-serif, system-ui, -apple-system, sans-serif;
    --mono: "IBM Plex Mono", ui-monospace, "SF Mono", Menlo, monospace;
    color-scheme: dark;
  }
  :root[data-theme="light"] {
    --bg: #ebe6da;
    --panel: #f3efe5;
    --panel-2: #e3ddcf;
    --line: #d0c8b6;
    --line-strong: #a9a08c;
    --text: #1c1a16;
    --text-2: #3d3a32;
    --muted: #6b6559;
    --up: #2e7447;
    --down: #a8452b;
    --neutral: #86660f;
    --accent: #2c5875;
    --up-tint: rgba(46,116,71,0.12);
    --down-tint: rgba(168,69,43,0.12);
    --neutral-tint: rgba(134,102,15,0.12);
    --accent-tint: rgba(44,88,117,0.12);
    --phase-open: #d0c8b6;
    --phase-build: #b9b5a6;
    --phase-confirm: rgba(44,88,117,0.45);
    --phase-lock: rgba(168,69,43,0.45);
    color-scheme: light;
  }

  * { box-sizing: border-box; border-radius: 0; }
  html, body { margin: 0; padding: 0; background: var(--bg); color: var(--text); }
  body { font-family: var(--sans); font-size: 14px; line-height: 1.5; -webkit-font-smoothing: antialiased; }
  button, input, select { font: inherit; color: inherit; }
  a { color: var(--accent); text-decoration: none; }
  a:hover { text-decoration: underline; }
  .num, .mono { font-family: var(--mono); font-variant-numeric: tabular-nums; }
  .up { color: var(--up); } .down { color: var(--down); } .neutral { color: var(--neutral); } .accent { color: var(--accent); }
  .muted { color: var(--muted); }

  .page { max-width: 1180px; margin: 0 auto; padding: 0 24px 48px; }

  /* masthead */
  .masthead {
    display: flex; justify-content: space-between; align-items: flex-end; gap: 16px; flex-wrap: wrap;
    padding: 22px 0 16px; border-bottom: 1px solid var(--line-strong);
  }
  .brand { font-weight: 700; font-size: 19px; letter-spacing: -0.01em; }
  .brand .slash { color: var(--muted); font-weight: 400; margin: 0 6px; }
  .byline { font-size: 12px; color: var(--muted); margin-top: 2px; }
  .mast-right { display: flex; align-items: flex-end; gap: 22px; flex-wrap: wrap; }
  .meta { display: flex; flex-direction: column; }
  .meta-k { font-size: 10.5px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }
  .meta-v { font-family: var(--mono); font-size: 13px; }
  .btn-text {
    background: transparent; border: 1px solid var(--line-strong); padding: 5px 10px; cursor: pointer;
    font-size: 12px; font-weight: 600;
  }
  .btn-text:hover { border-color: var(--text-2); }

  /* window strip */
  .window-strip { padding: 14px 0 12px; border-bottom: 1px solid var(--line); }
  .ws-row { display: flex; justify-content: space-between; align-items: baseline; gap: 12px; flex-wrap: wrap; margin-bottom: 8px; font-size: 13px; color: var(--text-2); }
  .ws-row b { color: var(--text); font-family: var(--mono); font-weight: 500; }
  .phase-track { position: relative; height: 8px; background: var(--phase-open); }
  .phase-track.thin { height: 6px; }
  .phase-seg { position: absolute; top: 0; bottom: 0; }
  .phase-now { position: absolute; top: -3px; bottom: -3px; width: 2px; background: var(--text); }
  .phase-labels { position: relative; height: 16px; margin-top: 5px; font-size: 10.5px; color: var(--muted); letter-spacing: 0.04em; text-transform: uppercase; }
  .phase-labels span { position: absolute; top: 0; white-space: nowrap; }

  /* tabs */
  .tabs {
    display: flex; gap: 0; border-bottom: 1px solid var(--line-strong); margin-bottom: 28px;
    overflow-x: auto; scrollbar-width: none;
  }
  .tabs::-webkit-scrollbar { display: none; }
  .tab {
    background: transparent; border: 0; border-bottom: 2px solid transparent; margin-bottom: -1px;
    padding: 14px 18px 11px; cursor: pointer; font-size: 13.5px; font-weight: 600; color: var(--muted); white-space: nowrap;
  }
  .tab:hover { color: var(--text); }
  .tab.active { color: var(--text); border-bottom-color: var(--text); }
  .tab .count { font-family: var(--mono); font-weight: 400; font-size: 11.5px; color: var(--muted); margin-left: 6px; }

  .view { display: none; }
  .view.active { display: block; }

  /* section headings */
  .sec { margin-bottom: 36px; }
  .sec-head { display: flex; justify-content: space-between; align-items: baseline; gap: 12px; flex-wrap: wrap; border-bottom: 1px solid var(--line); padding-bottom: 8px; margin-bottom: 14px; }
  .sec-title { font-size: 12px; font-weight: 600; letter-spacing: 0.1em; text-transform: uppercase; color: var(--text-2); margin: 0; }
  .sec-note { font-size: 12px; color: var(--muted); }
  .lede { font-size: 15px; color: var(--text-2); max-width: 760px; margin: 0 0 26px; }

  /* top call */
  .call { display: grid; grid-template-columns: 1.25fr 1fr; border: 1px solid var(--line-strong); background: var(--panel); }
  .call-main { padding: 22px 24px; border-right: 1px solid var(--line); }
  .call-side { padding: 22px 24px; }
  .eyebrow { font-size: 11px; letter-spacing: 0.1em; text-transform: uppercase; color: var(--muted); margin-bottom: 6px; }
  .call-title { font-size: 34px; font-weight: 700; letter-spacing: -0.02em; line-height: 1.1; margin-bottom: 6px; }
  .call-sub { font-size: 13px; color: var(--text-2); margin-bottom: 16px; }
  .reasons { list-style: none; margin: 0; padding: 0; counter-reset: r; }
  .reasons li { counter-increment: r; display: grid; grid-template-columns: 30px 1fr; font-size: 13px; color: var(--text-2); padding: 6px 0; border-top: 1px solid var(--line); }
  .reasons li::before { content: counter(r, decimal-leading-zero); font-family: var(--mono); font-size: 11px; color: var(--muted); padding-top: 2px; }
  .kv { display: grid; grid-template-columns: 1fr auto; gap: 4px 16px; font-size: 13px; }
  .kv dt { color: var(--muted); padding: 7px 0; border-bottom: 1px solid var(--line); }
  .kv dd { margin: 0; padding: 7px 0; border-bottom: 1px solid var(--line); font-family: var(--mono); text-align: right; }
  .call-empty { padding: 22px 24px; font-size: 14px; color: var(--text-2); max-width: 760px; }
  .call-empty b { color: var(--text); }
  .fine { font-size: 11.5px; color: var(--muted); margin-top: 14px; }
  .confmeter { display: flex; align-items: center; gap: 10px; margin-bottom: 18px; }
  .confmeter .track { flex: 1; height: 6px; background: var(--panel-2); position: relative; }
  .confmeter .fill { position: absolute; top: 0; bottom: 0; left: 0; }
  .confmeter .tick { position: absolute; top: -3px; bottom: -3px; width: 1px; background: var(--text-2); }
  .confmeter .v { font-family: var(--mono); font-size: 13px; min-width: 64px; text-align: right; }

  /* tables */
  .tbl-wrap { overflow-x: auto; }
  table.tbl { width: 100%; border-collapse: collapse; font-size: 13px; }
  .tbl th { text-align: left; font-weight: 500; font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); padding: 8px 10px; border-bottom: 1px solid var(--line-strong); white-space: nowrap; }
  .tbl td { padding: 11px 10px; border-bottom: 1px solid var(--line); vertical-align: middle; }
  .tbl th.r, .tbl td.r { text-align: right; }
  .tbl td.num { font-family: var(--mono); white-space: nowrap; }
  .tbl tr.click { cursor: pointer; }
  .tbl tr.click:hover td { background: var(--panel); }
  .tbl tr.sel td { background: var(--panel); }
  .tbl tr.sel td:first-child { font-weight: 700; }
  .asset-cell b { display: block; font-weight: 700; }
  .asset-cell span { font-size: 11.5px; color: var(--muted); }
  .minibar { width: 80px; height: 5px; background: var(--panel-2); position: relative; display: inline-block; vertical-align: middle; margin-left: 8px; }
  .minibar i { position: absolute; top: 0; bottom: 0; left: 0; }
  .tag { display: inline-block; font-size: 11px; font-weight: 600; padding: 2px 7px; letter-spacing: 0.02em; white-space: nowrap; }
  .tag.up { background: var(--up-tint); } .tag.down { background: var(--down-tint); } .tag.neutral { background: var(--neutral-tint); } .tag.accent { background: var(--accent-tint); }
  .tag.plain { color: var(--muted); border: 1px solid var(--line); }
  .empty { font-size: 13px; color: var(--muted); padding: 14px 0; }

  /* timing list */
  .timing-row { display: grid; grid-template-columns: 70px 1fr 180px; gap: 18px; align-items: center; padding: 14px 0; border-bottom: 1px solid var(--line); }
  .timing-row:first-child { border-top: 1px solid var(--line); }
  .timing-asset { font-weight: 700; }
  .timing-asset span { display: block; font-family: var(--mono); font-weight: 400; font-size: 11.5px; color: var(--muted); }
  .timing-status { font-weight: 600; font-size: 13px; }
  .timing-detail { font-size: 12px; color: var(--muted); margin-top: 6px; }

  /* asset view */
  .switcher { display: inline-flex; border: 1px solid var(--line-strong); }
  .switcher button { background: transparent; border: 0; border-right: 1px solid var(--line-strong); padding: 7px 16px; cursor: pointer; font-weight: 600; font-size: 13px; color: var(--muted); }
  .switcher button:last-child { border-right: 0; }
  .switcher button:hover { color: var(--text); }
  .switcher button.on { background: var(--text); color: var(--bg); }
  .switcher button .sq { display: inline-block; width: 6px; height: 6px; margin-left: 7px; vertical-align: middle; }
  .view-head { display: flex; justify-content: space-between; align-items: center; gap: 14px; flex-wrap: wrap; margin-bottom: 22px; }

  .price-block { display: flex; align-items: baseline; gap: 18px; flex-wrap: wrap; margin-bottom: 4px; }
  .price { font-family: var(--mono); font-size: 44px; font-weight: 500; letter-spacing: -0.02em; line-height: 1.1; }
  .price.stale { color: var(--muted); }
  .chg { font-family: var(--mono); font-size: 15px; }
  .subline { font-size: 12.5px; color: var(--muted); margin-bottom: 14px; }
  .spark { width: 100%; height: 70px; display: block; margin-bottom: 26px; border-bottom: 1px solid var(--line); }

  .split { display: grid; grid-template-columns: minmax(0, 1.65fr) minmax(0, 1fr); gap: 28px; margin-bottom: 36px; }
  .tv-wrap { border: 1px solid var(--line); background: var(--panel); }
  .tv-frame { width: 100%; height: 470px; display: block; border: 0; }
  .rail-block { border-top: 1px solid var(--line-strong); padding-top: 12px; margin-bottom: 26px; }
  .rail-block:last-child { margin-bottom: 0; }
  .rail-k { font-size: 11px; letter-spacing: 0.1em; text-transform: uppercase; color: var(--muted); display: flex; justify-content: space-between; gap: 8px; }
  .verdict { font-size: 28px; font-weight: 700; letter-spacing: -0.01em; margin: 6px 0 2px; }
  .bias-scale { position: relative; height: 8px; background: var(--panel-2); margin: 12px 0 4px; }
  .bias-scale .mid { position: absolute; left: 50%; top: -3px; bottom: -3px; width: 1px; background: var(--line-strong); }
  .bias-scale .fill { position: absolute; top: 0; bottom: 0; }
  .bias-ends { display: flex; justify-content: space-between; font-family: var(--mono); font-size: 10.5px; color: var(--muted); }
  .strike-val { font-family: var(--mono); font-size: 24px; margin-top: 6px; }
  .strike-src { font-size: 12px; color: var(--muted); }
  .strike-diff { font-family: var(--mono); font-size: 13px; margin-top: 6px; }
  .linkbtn { background: none; border: 0; padding: 0; color: var(--accent); cursor: pointer; font-size: 12.5px; margin-top: 10px; }
  .linkbtn:hover { text-decoration: underline; }
  .inline-form { display: flex; gap: 8px; margin-top: 10px; }
  .field { background: var(--bg); border: 1px solid var(--line-strong); padding: 7px 10px; font-family: var(--mono); font-size: 13px; outline: none; width: 100%; min-width: 0; }
  .field:focus { border-color: var(--text-2); }
  .btn { background: var(--text); color: var(--bg); border: 1px solid var(--text); padding: 7px 14px; font-weight: 600; font-size: 12.5px; cursor: pointer; white-space: nowrap; }
  .btn.ghost { background: transparent; color: var(--text-2); border-color: var(--line-strong); }
  .btn.danger { background: transparent; color: var(--down); border-color: var(--down); }
  .ind { display: grid; grid-template-columns: 1fr auto; gap: 0 12px; padding: 9px 0; border-bottom: 1px solid var(--line); }
  .ind:last-child { border-bottom: 0; }
  .ind .k { font-size: 13px; }
  .ind .v { font-family: var(--mono); font-size: 13px; text-align: right; }
  .ind .d { grid-column: 1 / -1; font-size: 11.5px; color: var(--muted); }
  .hint { border-bottom: 1px dotted var(--muted); cursor: help; }

  /* chart view */
  .chart-legend { display: flex; flex-wrap: wrap; gap: 6px 18px; font-size: 12px; color: var(--muted); margin-bottom: 10px; }
  .chart-legend span { display: inline-flex; align-items: center; gap: 6px; }
  .chart-legend i { width: 10px; height: 10px; display: inline-block; }
  .chart-legend i.line { height: 0; border-top: 2px dashed var(--accent); }
  .chart-readout { font-family: var(--mono); font-size: 12px; color: var(--text-2); min-height: 20px; margin-bottom: 6px; }
  .chart-box { border: 1px solid var(--line); background: var(--panel); padding: 8px 0 0; }
  .candles-chart { width: 100%; height: 420px; display: block; touch-action: pan-y; }
  .ledger { display: grid; grid-template-columns: repeat(4, 1fr); border: 1px solid var(--line); border-top: 0; }
  .ledger > div { padding: 12px 16px; border-right: 1px solid var(--line); }
  .ledger > div:last-child { border-right: 0; }
  .ledger .k { font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }
  .ledger .v { font-family: var(--mono); font-size: 18px; margin-top: 2px; }
  .prose { font-size: 13.5px; color: var(--text-2); max-width: 780px; margin-top: 18px; }
  .prose b { color: var(--text); }

  /* log view */
  .stat-line { display: flex; flex-wrap: wrap; gap: 0; border: 1px solid var(--line); margin-bottom: 26px; }
  .stat-line > div { padding: 14px 20px; border-right: 1px solid var(--line); flex: 1 1 150px; }
  .stat-line > div:last-child { border-right: 0; }
  .stat-line .k { font-size: 11px; letter-spacing: 0.08em; text-transform: uppercase; color: var(--muted); }
  .stat-line .v { font-family: var(--mono); font-size: 22px; margin-top: 2px; }
  .actions { display: flex; gap: 10px; flex-wrap: wrap; }

  /* method view */
  .cols { display: grid; grid-template-columns: minmax(0, 1fr) minmax(0, 1fr); gap: 40px; }
  .pattern { display: grid; grid-template-columns: 44px 1fr; gap: 14px; padding: 12px 0; border-bottom: 1px solid var(--line); }
  .pattern:first-child { border-top: 1px solid var(--line); }
  .pattern .name { font-weight: 600; font-size: 13.5px; }
  .pattern .desc { font-size: 12.5px; color: var(--text-2); }
  dl.gloss { margin: 0; }
  dl.gloss dt { font-family: var(--mono); font-weight: 600; font-size: 13px; padding-top: 10px; border-top: 1px solid var(--line); }
  dl.gloss dd { margin: 2px 0 10px; font-size: 12.5px; color: var(--text-2); }
  .swatch { display: inline-block; width: 10px; height: 10px; margin-right: 8px; vertical-align: -1px; }

  /* settings view */
  .form-grid { display: grid; grid-template-columns: minmax(0, 1fr) minmax(0, 1fr); gap: 40px; }
  .setting { display: grid; grid-template-columns: 1fr 120px; gap: 6px 16px; align-items: center; padding: 14px 0; border-bottom: 1px solid var(--line); }
  .setting:first-of-type { border-top: 1px solid var(--line); }
  .setting label { font-weight: 600; font-size: 13.5px; }
  .setting p { grid-column: 1 / -1; margin: 0; font-size: 12px; color: var(--muted); }
  .setting.strike { grid-template-columns: 50px 1fr auto; }
  .setting.strike p { grid-column: 2 / -1; }

  /* footer */
  .statusbar { display: flex; justify-content: space-between; gap: 12px; flex-wrap: wrap; font-size: 12px; color: var(--muted); border-top: 1px solid var(--line-strong); padding-top: 12px; margin-top: 12px; }
  .statusbar .sq { display: inline-block; width: 7px; height: 7px; margin-right: 7px; vertical-align: 0; background: var(--muted); }
  .footer { font-size: 11.5px; color: var(--muted); max-width: 860px; margin-top: 12px; }
  .footer b { color: var(--text-2); }

  /* modal */
  .modal-overlay { position: fixed; inset: 0; background: rgba(10,10,8,0.72); display: none; align-items: center; justify-content: center; padding: 16px; z-index: 50; }
  .modal-overlay.show { display: flex; }
  .modal-card { width: 100%; max-width: 460px; background: var(--panel); border: 1px solid var(--line-strong); padding: 22px; position: relative; }
  .modal-close { position: absolute; top: 14px; right: 14px; background: transparent; border: 1px solid var(--line-strong); padding: 3px 9px; cursor: pointer; font-size: 12px; color: var(--muted); }
  .modal-title { font-size: 16px; font-weight: 600; margin: 4px 0 16px; padding-right: 60px; }
  .modal-ticker { font-family: var(--mono); font-size: 11.5px; color: var(--muted); margin: 12px 0 16px; word-break: break-all; }
  .modal-open { display: block; text-align: center; background: var(--text); color: var(--bg); font-weight: 600; font-size: 13px; padding: 10px; }
  .modal-open:hover { text-decoration: none; }

  @media (max-width: 900px) {
    .split, .cols, .form-grid { grid-template-columns: 1fr; }
    .call { grid-template-columns: 1fr; }
    .call-main { border-right: 0; border-bottom: 1px solid var(--line); }
    .tv-frame { height: 380px; }
  }
  @media (max-width: 640px) {
    .page { padding: 0 16px 40px; }
    .price { font-size: 34px; }
    .call-title { font-size: 28px; }
    .hide-sm { display: none; }
    .timing-row { grid-template-columns: 56px 1fr; }
    .timing-row .timing-right { grid-column: 1 / -1; }
    .ledger { grid-template-columns: repeat(2, 1fr); }
    .ledger > div:nth-child(2) { border-right: 0; }
    .ledger > div:nth-child(-n+2) { border-bottom: 1px solid var(--line); }
    .tv-frame { height: 320px; }
    .candles-chart { height: 340px; }
    .mast-right { gap: 14px; }
    .tab { padding: 12px 13px 10px; }
    .tbl td, .tbl th { padding-left: 6px; padding-right: 6px; }
  }
</style>
</head>
<body>
<div class="page">

  <header class="masthead">
    <div>
      <div class="brand">Crypto<span class="slash">/</span>15m Signal Desk</div>
      <div class="byline">made by madapuri · BTC, ETH, XRP and SOL against Kalshi 15 minute contracts</div>
    </div>
    <div class="mast-right">
      <div class="meta"><span class="meta-k">UTC</span><span class="meta-v" id="clock">00:00:00</span></div>
      <div class="meta"><span class="meta-k">Price feed</span><span class="meta-v" id="feedMeta">connecting</span></div>
      <div class="meta"><span class="meta-k">Stream</span><span class="meta-v" id="wsBadge">connecting</span></div>
      <button class="btn-text" id="themeBtn" type="button">Light</button>
    </div>
  </header>

  <div class="window-strip">
    <div class="ws-row">
      <span>Current window <b id="winLabel">00:00 to 00:15 UTC</b></span>
      <span>Phase <b id="winPhase">Open</b></span>
      <span><b id="winLeft">15:00</b> left</span>
    </div>
    <div class="phase-track" id="globalTrack"></div>
    <div class="phase-labels">
      <span style="left:0">Open</span>
      <span style="left:20%">Build</span>
      <span style="left:66.67%">Confirm</span>
      <span style="left:86.67%">Lock</span>
    </div>
  </div>

  <nav class="tabs" id="tabs" role="tablist">
    <button class="tab" data-tab="overview" role="tab">Overview</button>
    <button class="tab" data-tab="asset" role="tab">Asset detail</button>
    <button class="tab" data-tab="chart" role="tab">15m chart</button>
    <button class="tab" data-tab="markets" role="tab">Kalshi markets<span class="count" id="mktCount">0</span></button>
    <button class="tab" data-tab="log" role="tab">Signal log<span class="count" id="logCount">0</span></button>
    <button class="tab" data-tab="method" role="tab">Method</button>
    <button class="tab" data-tab="settings" role="tab">Settings</button>
  </nav>

  <!-- Overview -->
  <section class="view" data-view="overview">
    <p class="lede">A live technical read on four coins, refreshed every few seconds and lined up with the fixed quarter hour windows Kalshi settles on. Use it as one input next to your own judgment.</p>

    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Best setup right now</h2><span class="sec-note" id="callNote">entry bar 40/100</span></div>
      <div id="topCall" class="call"><div class="call-empty">Loading the first read from the price feeds.</div></div>
    </div>

    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Board</h2><span class="sec-note">Select a row to open the asset</span></div>
      <div class="tbl-wrap">
        <table class="tbl" id="boardTbl">
          <thead><tr>
            <th>Asset</th><th class="r">Price</th><th class="r">This window</th><th>Bias</th><th class="hide-sm">Confidence</th><th class="hide-sm">Timing</th><th class="r hide-sm">Kalshi YES</th>
          </tr></thead>
          <tbody id="boardBody"><tr><td colspan="7" class="empty">Loading prices.</td></tr></tbody>
        </table>
      </div>
    </div>

    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Entry and exit timing</h2><span class="sec-note">Marker shows where each coin sits inside the current window</span></div>
      <div id="timingList"><div class="empty">Loading timing.</div></div>
    </div>
  </section>

  <!-- Asset detail -->
  <section class="view" data-view="asset">
    <div class="view-head">
      <div class="switcher" data-switcher></div>
      <span class="sec-note" id="assetSource">Loading</span>
    </div>

    <div class="price-block">
      <div class="price stale" id="price">$0.00</div>
      <div class="chg" id="chg15">0.00% this window</div>
    </div>
    <div class="subline" id="subline">Loading live price data.</div>
    <svg class="spark" id="spark" viewBox="0 0 700 70" preserveAspectRatio="none"></svg>

    <div class="split">
      <div>
        <div class="sec-head"><h2 class="sec-title">Candles</h2><span class="sec-note">TradingView, 1 minute</span></div>
        <div class="tv-wrap"><iframe id="tvFrame" class="tv-frame" src="about:blank" title="TradingView candlestick chart" scrolling="no"></iframe></div>
      </div>
      <div>
        <div class="rail-block">
          <div class="rail-k"><span class="hint" title="Combines EMA crossover, RSI, momentum since this window opened, volume and the last candle into one score from -100 to 100.">Model bias</span><span id="windowLabel">window</span></div>
          <div class="verdict neutral" id="verdict">Calculating</div>
          <div class="sec-note" id="conftext">confidence 0/100</div>
          <div class="bias-scale"><div class="mid"></div><div class="fill" id="biasFill"></div></div>
          <div class="bias-ends"><span>-100 down</span><span>0</span><span>up +100</span></div>
        </div>

        <div class="rail-block">
          <div class="rail-k"><span>Number to beat</span><span id="strikeSourceLabel">not set</span></div>
          <div id="strikeBody">
            <div class="strike-val" id="strikeValue">Not set</div>
            <div class="strike-src" id="strikeSourceDetail">Fetching the live strike from Kalshi every second.</div>
            <div class="strike-diff" id="strikeDiff"></div>
          </div>
          <button class="linkbtn" id="strikeEditLink" type="button">Set or override manually</button>
          <div class="inline-form" id="strikeInputRow" style="display:none;">
            <input class="field" type="number" step="any" id="strikeInput" placeholder="65250.00">
            <button class="btn" id="strikeSetBtn" type="button">Set</button>
            <button class="btn ghost" id="strikeClearBtn" type="button">Clear</button>
          </div>
        </div>

        <div class="rail-block">
          <div class="rail-k"><span>Indicators</span><span>1m candles</span></div>
          <div class="ind"><span class="k hint" title="9 period versus 21 period exponential moving average. The short one above the long one is usually read as short term bullish.">EMA 9 / 21</span><span class="v" id="emaVal">n/a</span><span class="d" id="emaDetail"></span></div>
          <div class="ind"><span class="k hint" title="Relative Strength Index, 0 to 100. Above about 70 is often called overbought, below about 30 oversold.">RSI 14</span><span class="v" id="rsiVal">n/a</span><span class="d" id="rsiDetail"></span></div>
          <div class="ind"><span class="k hint" title="How far price has moved since this specific 15 minute window opened.">Window momentum</span><span class="v" id="rocVal">n/a</span><span class="d" id="rocDetail"></span></div>
          <div class="ind"><span class="k hint" title="Volume over the last 15 minutes against the 15 minutes before. Rising volume with a move is generally read as more likely to hold.">Volume trend</span><span class="v" id="volVal">n/a</span><span class="d" id="volDetail"></span></div>
          <div class="ind"><span class="k hint" title="Reads body against wicks on the last fully closed 1 minute candle.">Last candle</span><span class="v" id="candleVal">n/a</span><span class="d" id="candleDetail"></span></div>
        </div>
      </div>
    </div>

    <div class="sec">
      <div class="sec-head"><h2 class="sec-title" id="kalshiSectionLabel">Kalshi contracts</h2><span class="sec-note" id="kalshiStatusText">connecting</span></div>
      <div class="tbl-wrap" id="assetMarkets"><div class="empty">Loading open contracts from Kalshi.</div></div>
    </div>
  </section>

  <!-- Chart -->
  <section class="view" data-view="chart">
    <div class="view-head">
      <div class="switcher" data-switcher></div>
      <span class="sec-note" id="chartStatusText">waiting</span>
    </div>
    <div class="chart-legend">
      <span><i style="background:var(--up)"></i>bullish candle</span>
      <span><i style="background:var(--down)"></i>bearish candle</span>
      <span><i class="line"></i>regression trend</span>
      <span><i style="background:var(--neutral-tint);outline:1px dashed var(--neutral)"></i>flag pattern</span>
      <span class="hint" title="Triangle under a candle: hammer. Triangle above: shooting star. Square on the body: strong body. Diamond: doji.">pattern markers</span>
    </div>
    <div class="chart-readout" id="chartReadout">Move across the chart to read a candle.</div>
    <div class="chart-box"><svg class="candles-chart" id="candlesChart"></svg></div>
    <div class="ledger">
      <div><div class="k">Trend</div><div class="v" id="lgTrend">n/a</div></div>
      <div><div class="k">Bullish shapes</div><div class="v up" id="lgBull">0</div></div>
      <div><div class="k">Bearish shapes</div><div class="v down" id="lgBear">0</div></div>
      <div><div class="k">Flags</div><div class="v" id="lgFlags">0</div></div>
    </div>
    <div class="prose" id="chartSummary">Loading 15 minute candles.</div>
  </section>

  <!-- Markets -->
  <section class="view" data-view="markets">
    <p class="lede">Every open 15 minute contract across the four series, pulled from Kalshi's public API once a second. The model column compares where the market leans with where the indicators lean.</p>
    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Open contracts</h2><span class="sec-note" id="seriesStatus">connecting</span></div>
      <div class="tbl-wrap" id="allMarkets"><div class="empty">Loading.</div></div>
    </div>
  </section>

  <!-- Log -->
  <section class="view" data-view="log">
    <p class="lede">Each window, the desk records its call once the window is a few minutes old, then grades it when the window closes. The log lives in this browser and only fills while the page is open.</p>
    <div class="stat-line">
      <div><div class="k">Calls logged</div><div class="v" id="lsTotal">0</div></div>
      <div><div class="k">Graded</div><div class="v" id="lsGraded">0</div></div>
      <div><div class="k">Hit rate</div><div class="v" id="lsRate">n/a</div></div>
      <div><div class="k">Above entry bar</div><div class="v" id="lsRateHigh">n/a</div></div>
    </div>
    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">By asset</h2></div>
      <div class="tbl-wrap"><table class="tbl"><thead><tr><th>Asset</th><th class="r">Calls</th><th class="r">Hits</th><th class="r">Misses</th><th class="r">Passed</th><th class="r">Hit rate</th></tr></thead><tbody id="logAssetBody"></tbody></table></div>
    </div>
    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Recent calls</h2>
        <div class="actions"><button class="btn ghost" id="exportLog" type="button">Export CSV</button><button class="btn danger" id="clearLog" type="button">Clear log</button></div>
      </div>
      <div class="tbl-wrap"><table class="tbl"><thead><tr><th>Window</th><th>Asset</th><th>Call</th><th class="r">Conf.</th><th class="r hide-sm">Reference</th><th class="r hide-sm">Close</th><th>Result</th></tr></thead><tbody id="logBody"></tbody></table></div>
    </div>
  </section>

  <!-- Method -->
  <section class="view" data-view="method">
    <p class="lede">How the score is built, what the timing phases mean, and how to read the candle markers. The weights below are the exact ones the page uses.</p>
    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Score components</h2><span class="sec-note">Summed, multiplied by 1.25, then clamped to -100..100</span></div>
      <div class="tbl-wrap"><table class="tbl">
        <thead><tr><th>Component</th><th class="r">Range</th><th>Rule</th></tr></thead>
        <tbody>
          <tr><td>EMA 9 against EMA 21</td><td class="num r">±32</td><td>+32 when the 9 sits above the 21, -32 when below.</td></tr>
          <tr><td>Window momentum</td><td class="num r">±32</td><td>Percent change since the window opened, times 11, capped.</td></tr>
          <tr><td>RSI 14</td><td class="num r">±28</td><td>Distance from 50, times 1.2, capped.</td></tr>
          <tr><td>Volume</td><td class="num r">±12</td><td>Added in the direction of the window move when volume is up more than 5% on the prior 15 minutes.</td></tr>
          <tr><td>Last closed candle</td><td class="num r">±12</td><td>Body and wick read on the last finished 1 minute candle, strength times 16, capped.</td></tr>
          <tr><td>Bias label</td><td class="num r" id="mNeutral">±15</td><td>Scores inside the neutral band read as Neutral. Change the band in Settings.</td></tr>
        </tbody>
      </table></div>
    </div>

    <div class="sec">
      <div class="sec-head"><h2 class="sec-title">Window phases</h2></div>
      <div class="tbl-wrap"><table class="tbl">
        <thead><tr><th>Phase</th><th class="r">Minutes</th><th>What usually happens</th></tr></thead>
        <tbody>
          <tr><td><span class="swatch" style="background:var(--phase-open)"></span>Open</td><td class="num r">0 to 3</td><td>Spreads are widest and the move has barely started.</td></tr>
          <tr><td><span class="swatch" style="background:var(--phase-build)"></span>Build</td><td class="num r">3 to 10</td><td>Trend is usually clearest relative to the time left. The preferred entry zone when confidence clears the bar.</td></tr>
          <tr><td><span class="swatch" style="background:var(--phase-confirm)"></span>Confirm</td><td class="num r">10 to 13</td><td>Momentum either confirms the read or fades. Some of the move is priced in.</td></tr>
          <tr><td><span class="swatch" style="background:var(--phase-lock)"></span>Lock</td><td class="num r">13 to 15</td><td>Kalshi settles on a short averaging window near the close, so a new entry has little edge.</td></tr>
        </tbody>
      </table></div>
      <p class="fine">These phases describe general market structure for short dated contracts. They are not a backtested best minute for any coin.</p>
    </div>

    <div class="cols">
      <div class="sec">
        <div class="sec-head"><h2 class="sec-title">Candle patterns</h2></div>
        <div class="pattern">
          <svg width="44" height="44" viewBox="0 0 44 44"><line x1="22" y1="8" x2="22" y2="40" stroke="var(--muted)" stroke-width="2"/><rect x="15" y="8" width="14" height="10" fill="var(--up)"/></svg>
          <div><div class="name">Hammer</div><div class="desc">Small body near the top with a long lower wick. Sellers pushed down and buyers pulled it back. Often bullish after a decline.</div></div>
        </div>
        <div class="pattern">
          <svg width="44" height="44" viewBox="0 0 44 44"><line x1="22" y1="4" x2="22" y2="36" stroke="var(--muted)" stroke-width="2"/><rect x="15" y="26" width="14" height="10" fill="var(--down)"/></svg>
          <div><div class="name">Shooting star</div><div class="desc">Small body near the bottom with a long upper wick. Buyers pushed up and sellers drove it back. Often bearish after a rally.</div></div>
        </div>
        <div class="pattern">
          <svg width="44" height="44" viewBox="0 0 44 44"><line x1="22" y1="4" x2="22" y2="40" stroke="var(--muted)" stroke-width="2"/><rect x="15" y="20" width="14" height="3" fill="var(--neutral)"/></svg>
          <div><div class="name">Doji</div><div class="desc">Open and close nearly equal. Indecision. The next candle usually shows which side breaks.</div></div>
        </div>
        <div class="pattern">
          <svg width="44" height="44" viewBox="0 0 44 44"><line x1="22" y1="4" x2="22" y2="40" stroke="var(--muted)" stroke-width="2"/><rect x="15" y="6" width="14" height="32" fill="var(--up)"/></svg>
          <div><div class="name">Strong body (marubozu)</div><div class="desc">Full body with little or no wick. One side controlled the whole candle and the move often continues.</div></div>
        </div>
        <div class="pattern">
          <svg width="44" height="44" viewBox="0 0 44 44"><rect x="8" y="18" width="10" height="10" fill="var(--down)"/><rect x="24" y="8" width="12" height="28" fill="var(--up)"/></svg>
          <div><div class="name">Engulfing</div><div class="desc">A body that fully covers the prior body in the opposite color. One of the stronger single reversal shapes.</div></div>
        </div>
        <div class="pattern">
          <svg width="44" height="44" viewBox="0 0 44 44"><line x1="22" y1="4" x2="22" y2="40" stroke="var(--muted)" stroke-width="2"/><rect x="15" y="16" width="14" height="12" fill="var(--neutral)"/></svg>
          <div><div class="name">Spinning top</div><div class="desc">Small body with wicks on both sides. Neither side won, which often marks a pause.</div></div>
        </div>
      </div>
      <div>
        <div class="sec">
          <div class="sec-head"><h2 class="sec-title">Glossary</h2></div>
          <dl class="gloss">
            <dt>EMA</dt><dd>Exponential moving average. Weights recent prices more heavily to show trend.</dd>
            <dt>RSI</dt><dd>Relative strength index from 0 to 100. Measures the balance of recent gains and losses.</dd>
            <dt>Momentum</dt><dd>Percent move since the current quarter hour opened, anchored to the window Kalshi prices.</dd>
            <dt>YES / NO</dt><dd>The two sides of a Kalshi contract. Prices read as implied probability.</dd>
            <dt>Conf.</dt><dd>The absolute model score from 0 to 100.</dd>
            <dt>Strike</dt><dd>The price the contract settles against, also shown as the number to beat.</dd>
          </dl>
        </div>
        <div class="sec">
          <div class="sec-head"><h2 class="sec-title">Colour key</h2></div>
          <dl class="gloss">
            <dt><span class="swatch" style="background:var(--up)"></span>Green</dt><dd>Bullish, up, or buy YES on an above contract.</dd>
            <dt><span class="swatch" style="background:var(--down)"></span>Rust</dt><dd>Bearish, down, or buy NO on an above contract.</dd>
            <dt><span class="swatch" style="background:var(--neutral)"></span>Ochre</dt><dd>Neutral, caution, or a manually entered value.</dd>
            <dt><span class="swatch" style="background:var(--accent)"></span>Slate</dt><dd>Live data, links and the trend line.</dd>
          </dl>
        </div>
        <div class="sec">
          <div class="sec-head"><h2 class="sec-title">Data sources</h2></div>
          <p class="prose" style="margin-top:0">Prices come from Coinbase, then Kraken, then Binance, whichever answers first, with a Coinbase websocket for trade by trade ticks. Contracts come from Kalshi's public trade API. TradingView supplies the embedded chart.</p>
        </div>
      </div>
    </div>
  </section>

  <!-- Settings -->
  <section class="view" data-view="settings">
    <p class="lede">Settings are saved in this browser.</p>
    <div class="form-grid">
      <div>
        <div class="sec-head"><h2 class="sec-title">Model</h2></div>
        <div class="setting">
          <label for="setEntry">Entry confidence bar</label>
          <input class="field" type="number" id="setEntry" min="10" max="95" step="1">
          <p>Minimum confidence before the desk names a trade or calls an entry good. Default 40.</p>
        </div>
        <div class="setting">
          <label for="setNeutral">Neutral band</label>
          <input class="field" type="number" id="setNeutral" min="0" max="50" step="1">
          <p>Scores between minus and plus this number read as Neutral. Default 15.</p>
        </div>
        <div class="setting">
          <label for="setSnap">Log snapshot minute</label>
          <input class="field" type="number" id="setSnap" min="1" max="12" step="1">
          <p>How many minutes into each window the call is recorded in the signal log. Default 5.</p>
        </div>
        <div class="setting">
          <label for="setTheme">Theme</label>
          <select class="field" id="setTheme"><option value="dark">Dark</option><option value="light">Light</option></select>
          <p>Also switchable from the header.</p>
        </div>
        <div style="margin-top:16px"><button class="btn ghost" id="resetSettings" type="button">Restore defaults</button></div>
      </div>
      <div>
        <div class="sec-head"><h2 class="sec-title">Manual strikes</h2></div>
        <div id="strikeSettings"></div>
        <p class="fine">A live Kalshi strike always wins. Manual values are a fallback while the API is unreachable.</p>
      </div>
    </div>
  </section>

  <div class="statusbar">
    <div><span class="sq" id="statusDot"></span><span id="statusText">connecting</span></div>
    <div id="nextRefresh">refresh in 15s</div>
  </div>
  <div class="footer">
    <b>Not financial advice.</b> This page reads recent spot price action through EMA crossover, RSI, short term momentum, volume and candle shape. It makes no prediction and offers no guarantee. Kalshi prices can move before you act, and the model column is a rough read of each contract title. Event contracts carry real risk of loss, and 15 minute windows are dominated by noise.
  </div>
</div>

<div class="modal-overlay" id="modalOverlay">
  <div class="modal-card" role="dialog" aria-modal="true">
    <button class="modal-close" id="modalClose" type="button">Close</button>
    <div class="eyebrow" id="modalEyebrow">Kalshi contract</div>
    <div class="modal-title" id="modalTitle"></div>
    <dl class="kv">
      <dt>Strike</dt><dd id="modalStrike">n/a</dd>
      <dt>Closes in</dt><dd id="modalTimer">n/a</dd>
      <dt>YES</dt><dd class="up" id="modalYes">n/a</dd>
      <dt>NO</dt><dd class="down" id="modalNo">n/a</dd>
      <dt>Model read</dt><dd id="modalRead">n/a</dd>
    </dl>
    <div class="modal-ticker" id="modalTicker"></div>
    <a class="modal-open" id="modalOpenLink" href="https://kalshi.com" target="_blank" rel="noopener">Open on Kalshi</a>
  </div>
</div>

<script>
const REFRESH_MS = 15000;
const FAST_MS = 3000;
const KALSHI_REFRESH_MS = 1000;
const WINDOW_MS = 15 * 60 * 1000;
let countdown = REFRESH_MS / 1000;

const ASSETS = {
  BTC: { label: 'BTC', name: 'Bitcoin',  coinbase: 'BTC-USD', kraken: 'XBTUSD', binance: 'BTCUSDT', series: 'KXBTC15M', urlSlug: 'bitcoin-price-up-down',  tvSymbol: 'COINBASE:BTCUSD' },
  ETH: { label: 'ETH', name: 'Ethereum', coinbase: 'ETH-USD', kraken: 'ETHUSD', binance: 'ETHUSDT', series: 'KXETH15M', urlSlug: 'ethereum-price-up-down', tvSymbol: 'COINBASE:ETHUSD' },
  XRP: { label: 'XRP', name: 'XRP',      coinbase: 'XRP-USD', kraken: 'XRPUSD', binance: 'XRPUSDT', series: 'KXXRP15M', urlSlug: 'xrp-price-up-down',      tvSymbol: 'COINBASE:XRPUSD' },
  SOL: { label: 'SOL', name: 'Solana',   coinbase: 'SOL-USD', kraken: 'SOLUSD', binance: 'SOLUSDT', series: 'KXSOL15M', urlSlug: 'solana-price-up-down',   tvSymbol: 'COINBASE:SOLUSD' },
};
const ASSET_KEYS = Object.keys(ASSETS);

// Storage (best effort)
const LS = {
  get(k, def) { try { const v = localStorage.getItem(k); return v == null ? def : JSON.parse(v); } catch (e) { return def; } },
  set(k, v) { try { localStorage.setItem(k, JSON.stringify(v)); } catch (e) {} },
  del(k) { try { localStorage.removeItem(k); } catch (e) {} },
};

const DEFAULT_SETTINGS = { entryConf: 40, neutralBand: 15, snapMin: 5 };
let settings = Object.assign({}, DEFAULT_SETTINGS, LS.get('sd15:settings', {}));
let theme = LS.get('sd15:theme', null);
if (!theme) { try { theme = matchMedia('(prefers-color-scheme: light)').matches ? 'light' : 'dark'; } catch (e) { theme = 'dark'; } }

let selectedAsset = LS.get('sd15:asset', 'BTC');
if (!ASSETS[selectedAsset]) selectedAsset = 'BTC';
let activeTab = 'overview';

const assetData = {};
const kalshiData = {};
const marketsByTicker = {};
const manualStrikes = {};
ASSET_KEYS.forEach(k => { const v = LS.get('sd15:strike:' + k, null); if (typeof v === 'number') manualStrikes[k] = v; });
let signalLog = LS.get('sd15:log', []);
if (!Array.isArray(signalLog)) signalLog = [];

// Helpers
const $ = id => document.getElementById(id);
function esc(s) { return String(s == null ? '' : s).replace(/[&<>"']/g, c => ({ '&': '&amp;', '<': '&lt;', '>': '&gt;', '"': '&quot;', "'": '&#39;' }[c])); }
function fmtUsd(n) {
  if (n == null || isNaN(n)) return 'n/a';
  const decimals = n >= 100 ? 2 : (n >= 1 ? 4 : 6);
  return '$' + n.toLocaleString('en-US', { minimumFractionDigits: decimals, maximumFractionDigits: decimals });
}
function fmtPct(n, dp = 2) { return (n >= 0 ? '+' : '') + n.toFixed(dp) + '%'; }
function hhmm(ms) { const d = new Date(ms); return d.getUTCHours().toString().padStart(2, '0') + ':' + d.getUTCMinutes().toString().padStart(2, '0'); }
function fmtWindowLabel(start, end) { return `${hhmm(start)} to ${hhmm(end)} UTC`; }
function fmtMsRemaining(ms) {
  if (ms <= 0) return '0:00';
  const s = Math.floor(ms / 1000);
  return `${Math.floor(s / 60)}:${(s % 60).toString().padStart(2, '0')}`;
}
function fmtCountdown(closeIso) {
  const ms = new Date(closeIso).getTime() - Date.now();
  if (isNaN(ms)) return 'n/a';
  if (ms <= 0) return 'closed';
  return fmtMsRemaining(ms);
}
function currentWindowBounds(nowMs) { const start = Math.floor(nowMs / WINDOW_MS) * WINDOW_MS; return { start, end: start + WINDOW_MS }; }
function phaseOf(elapsedMin) {
  if (elapsedMin < 3) return 'Open';
  if (elapsedMin < 10) return 'Build';
  if (elapsedMin < 13) return 'Confirm';
  return 'Lock';
}
function biasOf(score) {
  const nb = settings.neutralBand;
  if (score > nb) return { label: 'Lean up', short: 'Up', cls: 'up', dir: 'up' };
  if (score < -nb) return { label: 'Lean down', short: 'Down', cls: 'down', dir: 'down' };
  return { label: 'Neutral', short: 'Neutral', cls: 'neutral', dir: null };
}
function colorVar(cls) { return cls === 'up' ? 'var(--up)' : cls === 'down' ? 'var(--down)' : 'var(--neutral)'; }

const PHASES = [
  { end: 3, color: 'var(--phase-open)' },
  { end: 10, color: 'var(--phase-build)' },
  { end: 13, color: 'var(--phase-confirm)' },
  { end: 15, color: 'var(--phase-lock)' },
];
function phaseTrackHtml(elapsedMin) {
  const segs = PHASES.map((p, i) => {
    const start = i === 0 ? 0 : PHASES[i - 1].end;
    return `<div class="phase-seg" style="left:${start / 15 * 100}%;width:${(p.end - start) / 15 * 100}%;background:${p.color}"></div>`;
  }).join('');
  return segs + `<div class="phase-now" style="left:calc(${Math.min(100, elapsedMin / 15 * 100)}% - 1px)"></div>`;
}

// Indicators
function ema(values, period) {
  const k = 2 / (period + 1);
  const out = [values[0]];
  for (let i = 1; i < values.length; i++) out.push(values[i] * k + out[i - 1] * (1 - k));
  return out;
}
function rsi(values, period) {
  let gains = 0, losses = 0;
  for (let i = values.length - period; i < values.length; i++) {
    const diff = values[i] - values[i - 1];
    if (diff >= 0) gains += diff; else losses -= diff;
  }
  const avgGain = gains / period, avgLoss = losses / period;
  if (avgLoss === 0) return 100;
  return 100 - (100 / (1 + avgGain / avgLoss));
}
function readCandle(open, high, low, close) {
  const range = Math.max(high - low, 1e-9);
  const body = Math.abs(close - open);
  const upperPct = (high - Math.max(open, close)) / range;
  const lowerPct = (Math.min(open, close) - low) / range;
  const bodyPct = body / range;
  const bullish = close >= open;
  let label, kind, dir, strength;
  if (bodyPct > 0.65) {
    kind = 'strong'; label = bullish ? 'Strong bullish body' : 'Strong bearish body'; dir = bullish ? 'up' : 'down'; strength = 0.75;
  } else if (lowerPct > 0.55 && bodyPct < 0.35) {
    kind = 'hammer'; label = 'Hammer, buyers stepped in'; dir = 'up'; strength = lowerPct;
  } else if (upperPct > 0.55 && bodyPct < 0.35) {
    kind = 'star'; label = 'Shooting star, sellers stepped in'; dir = 'down'; strength = upperPct;
  } else if (bodyPct < 0.15) {
    kind = 'doji'; label = 'Doji, indecision'; dir = null; strength = 0;
  } else {
    kind = 'mild'; label = bullish ? 'Mild bullish candle' : 'Mild bearish candle'; dir = bullish ? 'up' : 'down'; strength = 0.3;
  }
  return { label, kind, dir, strength, bodyPct, upperPct, lowerPct, bullish };
}

// Price sources
function normCoinbase(raw, label) {
  const s = raw.slice().sort((a, b) => a[0] - b[0]);
  return { times: s.map(c => c[0] * 1000), opens: s.map(c => c[3]), highs: s.map(c => c[2]), lows: s.map(c => c[1]), closes: s.map(c => c[4]), volumes: s.map(c => c[5]), source: label };
}
function normKraken(rows, label) {
  return { times: rows.map(r => r[0] * 1000), opens: rows.map(r => +r[1]), highs: rows.map(r => +r[2]), lows: rows.map(r => +r[3]), closes: rows.map(r => +r[4]), volumes: rows.map(r => +r[6]), source: label };
}
function normBinance(raw, label) {
  return { times: raw.map(k => k[0]), opens: raw.map(k => +k[1]), highs: raw.map(k => +k[2]), lows: raw.map(k => +k[3]), closes: raw.map(k => +k[4]), volumes: raw.map(k => +k[5]), source: label };
}
async function fetchCoinbase(productId, gran) {
  const res = await fetch(`https://api.exchange.coinbase.com/products/${productId}/candles?granularity=${gran}`);
  if (!res.ok) throw new Error('coinbase http ' + res.status);
  const raw = await res.json();
  if (!Array.isArray(raw) || raw.length < 5) throw new Error('coinbase: too few candles');
  return normCoinbase(raw, `Coinbase ${productId}`);
}
async function fetchKraken(pair, interval) {
  const res = await fetch(`https://api.kraken.com/0/public/OHLC?pair=${pair}&interval=${interval}`);
  if (!res.ok) throw new Error('kraken http ' + res.status);
  const data = await res.json();
  if (data.error && data.error.length) throw new Error('kraken: ' + data.error.join(', '));
  const key = Object.keys(data.result || {}).find(k => k !== 'last');
  const rows = key ? data.result[key] : null;
  if (!rows || rows.length < 5) throw new Error('kraken: too few candles');
  return normKraken(rows, `Kraken ${pair}`);
}
async function fetchBinance(symbol, interval, limit) {
  const res = await fetch(`https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=${interval}&limit=${limit}`);
  if (!res.ok) throw new Error('binance http ' + res.status);
  const raw = await res.json();
  if (!Array.isArray(raw) || raw.length < 5) throw new Error('binance: too few candles');
  return normBinance(raw, `Binance ${symbol}`);
}
async function firstOf(fns) {
  let lastErr = null;
  for (const fn of fns) { try { return await fn(); } catch (e) { lastErr = e; } }
  throw lastErr || new Error('all sources failed');
}
function getCandles(cfg) {
  return firstOf([() => fetchCoinbase(cfg.coinbase, 60), () => fetchKraken(cfg.kraken, 1), () => fetchBinance(cfg.binance, '1m', 60)])
    .then(d => { if (d.closes.length < 20) throw new Error('too few'); return d; });
}
function get15mCandles(cfg) {
  return firstOf([() => fetchCoinbase(cfg.coinbase, 900), () => fetchKraken(cfg.kraken, 15), () => fetchBinance(cfg.binance, '15m', 100)]);
}
function windowOpenPrice(times, opens, windowStart) {
  let idx = -1;
  for (let i = 0; i < times.length; i++) { if (times[i] <= windowStart) idx = i; else break; }
  return idx >= 0 ? opens[idx] : opens[0];
}

async function fetchLivePrice(cfg) {
  try { const r = await fetch(`https://api.exchange.coinbase.com/products/${cfg.coinbase}/ticker`); if (r.ok) { const p = parseFloat((await r.json()).price); if (!isNaN(p)) return p; } } catch (e) {}
  try { const r = await fetch(`https://api.kraken.com/0/public/Ticker?pair=${cfg.kraken}`); if (r.ok) { const j = await r.json(); const k = Object.keys(j.result || {})[0]; const p = k ? parseFloat(j.result[k].c[0]) : NaN; if (!isNaN(p)) return p; } } catch (e) {}
  try { const r = await fetch(`https://api.binance.com/api/v3/ticker/price?symbol=${cfg.binance}`); if (r.ok) { const p = parseFloat((await r.json()).price); if (!isNaN(p)) return p; } } catch (e) {}
  return null;
}

// WebSocket
const COINBASE_WS_URL = 'wss://ws-feed.exchange.coinbase.com';
const PRODUCT_TO_ASSET = {};
ASSET_KEYS.forEach(k => { PRODUCT_TO_ASSET[ASSETS[k].coinbase] = k; });
let coinbaseSocket = null, wsConnected = false, wsReconnectAttempts = 0, wsGaveUp = false;
const WS_MAX_ATTEMPTS = 6;

function setWsBadge(state) {
  const el = $('wsBadge');
  if (state === 'live') { el.textContent = 'live'; el.className = 'meta-v up'; }
  else if (state === 'connecting') { el.textContent = 'connecting'; el.className = 'meta-v muted'; }
  else { el.textContent = 'polling'; el.className = 'meta-v neutral'; }
}
function connectCoinbaseWS() {
  if (wsGaveUp) return;
  setWsBadge('connecting');
  try { coinbaseSocket = new WebSocket(COINBASE_WS_URL); } catch (e) { scheduleWsReconnect(); return; }
  coinbaseSocket.addEventListener('open', () => {
    wsConnected = true; wsReconnectAttempts = 0; setWsBadge('live');
    coinbaseSocket.send(JSON.stringify({ type: 'subscribe', product_ids: ASSET_KEYS.map(k => ASSETS[k].coinbase), channels: ['ticker'] }));
  });
  coinbaseSocket.addEventListener('message', evt => {
    let msg; try { msg = JSON.parse(evt.data); } catch (e) { return; }
    if (msg.type !== 'ticker' || !msg.product_id || msg.price == null) return;
    const key = PRODUCT_TO_ASSET[msg.product_id];
    if (key) applyLiveTick(key, parseFloat(msg.price));
  });
  coinbaseSocket.addEventListener('close', () => { wsConnected = false; setWsBadge('fallback'); scheduleWsReconnect(); });
  coinbaseSocket.addEventListener('error', () => { wsConnected = false; try { coinbaseSocket.close(); } catch (e) {} });
}
function scheduleWsReconnect() {
  wsReconnectAttempts += 1;
  if (wsReconnectAttempts > WS_MAX_ATTEMPTS) { wsGaveUp = true; setWsBadge('fallback'); return; }
  setTimeout(connectCoinbaseWS, Math.min(15000, 1000 * Math.pow(1.6, wsReconnectAttempts)));
}

let lastLiveRender = 0;
function applyLiveTick(key, price) {
  const d = assetData[key];
  if (!d || !d.closes || d.openPrice == null || price == null || isNaN(price)) return;
  const closes = d.closes.slice();
  closes[closes.length - 1] = price;
  d.lastPrice = price;
  if (Date.now() < d.windowEnd) d.chg15 = ((price - d.openPrice) / d.openPrice) * 100;
  d.liveCloses = closes;
  if (key === selectedAsset) quickRenderPrice(key);
  const now = Date.now();
  if (now - lastLiveRender > 1000) { lastLiveRender = now; renderBoard(); }
}
async function fastTickAsset(key) {
  if (wsConnected) return;
  const d = assetData[key];
  if (!d || !d.closes || d.openPrice == null) return;
  applyLiveTick(key, await fetchLivePrice(ASSETS[key]));
}
function fastTickAll() { return Promise.all(ASSET_KEYS.map(fastTickAsset)); }

// Core update
async function updateAsset(key) {
  const cfg = ASSETS[key];
  try {
    const c = await getCandles(cfg);
    const closes = c.closes.slice(-60), highs = c.highs.slice(-60), lows = c.lows.slice(-60), opens = c.opens.slice(-60), volumes = c.volumes.slice(-60);
    const lastPrice = closes[closes.length - 1];
    const { start: windowStart, end: windowEnd } = currentWindowBounds(Date.now());
    const openPrice = windowOpenPrice(c.times, c.opens, windowStart);
    const chg15 = ((lastPrice - openPrice) / openPrice) * 100;

    const e9 = ema(closes, 9), e21 = ema(closes, 21);
    const ema9 = e9[e9.length - 1], ema21 = e21[e21.length - 1];
    const emaBull = ema9 > ema21;
    const emaGapPct = ((ema9 - ema21) / ema21) * 100;
    const rsiVal = rsi(closes, 14);
    const last15vol = volumes.slice(-15).reduce((a, b) => a + b, 0);
    const prev15vol = volumes.slice(-30, -15).reduce((a, b) => a + b, 0) || last15vol;
    const volChangePct = ((last15vol - prev15vol) / prev15vol) * 100;
    const wi = closes.length - 2;
    const candle = wi >= 0 ? readCandle(opens[wi], highs[wi], lows[wi], closes[wi]) : null;

    const GAIN = 1.25;
    let score = 0;
    score += emaBull ? 32 : -32;
    score += Math.max(-32, Math.min(32, chg15 * 11));
    if (rsiVal > 50) score += Math.min(28, (rsiVal - 50) * 1.2); else score -= Math.min(28, (50 - rsiVal) * 1.2);
    if (volChangePct > 5) score += chg15 >= 0 ? 12 : -12;
    if (candle && candle.dir === 'up') score += Math.min(12, candle.strength * 16);
    if (candle && candle.dir === 'down') score -= Math.min(12, candle.strength * 16);
    score = Math.max(-100, Math.min(100, score * GAIN));

    assetData[key] = { closes, lastPrice, chg15, ema9, ema21, emaBull, emaGapPct, rsiVal, volChangePct, score, source: c.source, ok: true, windowStart, windowEnd, openPrice, candle };
    maybeLog(key, assetData[key]);
  } catch (err) {
    if (assetData[key]) assetData[key].ok = false; else assetData[key] = { ok: false };
  }
  if (key === selectedAsset) renderAsset(key);
  renderSwitchers();
  renderBoard();
  renderTopCall();
  renderTiming();
  renderStatus();
}
function updateAllAssets() { return Promise.all(ASSET_KEYS.map(updateAsset)); }

function buildReasons(d) {
  const r = [];
  r.push({ dir: d.emaBull ? 'up' : 'down', text: `9 EMA ${d.emaBull ? 'above' : 'below'} the 21 EMA by ${Math.abs(d.emaGapPct).toFixed(3)}%` });
  r.push({ dir: d.chg15 >= 0 ? 'up' : 'down', text: `${fmtPct(d.chg15)} since this window opened` });
  r.push({ dir: d.rsiVal > 50 ? 'up' : 'down', text: `RSI at ${d.rsiVal.toFixed(0)}, ${d.rsiVal > 70 ? 'overbought and may cool off' : d.rsiVal < 30 ? 'oversold and may bounce' : d.rsiVal > 50 ? 'leaning bullish' : 'leaning bearish'}` });
  if (Math.abs(d.volChangePct) > 5) r.push({ dir: d.chg15 >= 0 ? 'up' : 'down', text: `Volume ${d.volChangePct >= 0 ? 'up' : 'down'} ${Math.abs(d.volChangePct).toFixed(0)}% on the prior 15 minutes` });
  else r.push({ dir: 'neutral', text: 'Volume roughly flat on the prior 15 minutes' });
  if (d.candle) r.push({ dir: d.candle.dir || 'neutral', text: `Last candle: ${d.candle.label.toLowerCase()}` });
  return r;
}

// Timing
function timingFor(d) {
  const elapsedMin = Math.max(0, Math.min(15, (Date.now() - d.windowStart) / 60000));
  const remaining = d.windowEnd - Date.now();
  const conf = Math.abs(d.score);
  const bar = settings.entryConf;
  let status, cls, detail;
  if (remaining <= 60000) {
    status = 'Too late'; cls = 'down';
    detail = 'The settlement averaging window is live. A new entry here is a coin flip on the average. Wait for the next window.';
  } else if (elapsedMin > 13) {
    status = 'Last call'; cls = 'neutral';
    detail = 'Under two minutes left and spreads usually widen. Only worth it with high confidence and a plan to hold to settlement.';
  } else if (conf >= bar && elapsedMin <= 10) {
    status = 'Good entry'; cls = 'up';
    detail = `Confidence ${Math.round(conf)}/100 inside the build phase, when the least of the move is usually priced in.`;
  } else if (conf >= bar) {
    status = 'Still workable'; cls = 'neutral';
    detail = `Confidence ${Math.round(conf)}/100 past the build phase. Some of the move may already be priced in.`;
  } else {
    status = 'Wait'; cls = 'neutral';
    detail = `Confidence ${Math.round(conf)}/100 sits below the ${bar} bar. Let it develop or sit this window out.`;
  }
  return { status, cls, detail, elapsedMin, remaining };
}

function renderTiming() {
  const html = ASSET_KEYS.map(key => {
    const d = assetData[key];
    if (!d || !d.ok || d.windowStart == null) return `<div class="timing-row"><div class="timing-asset">${key}</div><div class="muted">Loading</div></div>`;
    const t = timingFor(d);
    return `<div class="timing-row">
      <div class="timing-asset">${key}<span>${t.elapsedMin.toFixed(1)}m in</span></div>
      <div><div class="phase-track thin">${phaseTrackHtml(t.elapsedMin)}</div><div class="timing-detail">${esc(t.detail)}</div></div>
      <div class="timing-right"><div class="timing-status ${t.cls}">${t.status}</div><div class="timing-detail num" style="margin-top:2px">${fmtMsRemaining(t.remaining)} left</div></div>
    </div>`;
  }).join('');
  $('timingList').innerHTML = html;
}

// Top call
function findMatchingKalshiMarket(key) {
  const kd = kalshiData[key];
  if (!kd || !kd.markets || !kd.markets.length) return null;
  const now = Date.now();
  const open = kd.markets.filter(m => new Date(m.close_time).getTime() > now);
  return open.length ? open[0] : null;
}
function guessDirectionFromTitle(title) {
  const t = (title || '').toLowerCase();
  if (/above|higher|over(?!all)|greater|\bup\b/.test(t)) return 'up';
  if (/below|lower|under|less than|\bdown\b/.test(t)) return 'down';
  return null;
}
function marketUrl(key, ticker) {
  const cfg = ASSETS[key];
  return `https://kalshi.com/markets/${cfg.series.toLowerCase()}/${cfg.urlSlug}/${ticker.toLowerCase()}`;
}

function renderTopCall() {
  const panel = $('topCall');
  $('callNote').textContent = `entry bar ${settings.entryConf}/100`;
  let best = null;
  ASSET_KEYS.forEach(key => {
    const d = assetData[key];
    if (!d || !d.ok) return;
    if (!best || Math.abs(d.score) > Math.abs(best.d.score)) best = { key, d };
  });
  if (!best) { panel.innerHTML = `<div class="call-empty">Waiting on the first read from the price feeds.</div>`; return; }

  const conf = Math.round(Math.abs(best.d.score));
  if (conf < settings.entryConf || !biasOf(best.d.score).dir) {
    const { end } = currentWindowBounds(Date.now());
    panel.innerHTML = `<div class="call-empty">
      <div class="eyebrow">No trade</div>
      <div class="call-title neutral">Sit this window out</div>
      <p>No coin clears the <b>${settings.entryConf}/100</b> bar. The strongest read is <b>${best.key} at ${conf}/100</b>, leaning ${best.d.score >= 0 ? 'up' : 'down'}. Forcing a trade on a weak read is how fees and noise eat the edge.</p>
      <p class="fine">Next window opens at ${hhmm(end)} UTC, in ${fmtMsRemaining(end - Date.now())}.</p>
    </div>`;
    return;
  }

  const { key, d } = best;
  const b = biasOf(d.score);
  const reasons = buildReasons(d).filter(r => r.dir === b.dir).slice(0, 4);
  const t = timingFor(d);
  const match = findMatchingKalshiMarket(key);
  const strike = getStrikeInfo(key);

  let side = 'Check market', price = null, ticker = 'none matched';
  if (match) {
    ticker = `<a href="${marketUrl(key, match.ticker)}" target="_blank" rel="noopener">${esc(match.ticker)}</a>`;
    const cdir = guessDirectionFromTitle(match.title);
    if (cdir) {
      side = cdir === b.dir ? 'YES' : 'NO';
      price = side === 'YES' ? (match.yes_ask ?? match.yes_bid) : (match.no_ask ?? match.no_bid);
    }
  }
  const diff = strike && d.lastPrice != null ? d.lastPrice - strike.value : null;

  panel.innerHTML = `
    <div class="call-main">
      <div class="eyebrow">${fmtWindowLabel(d.windowStart, d.windowEnd)}</div>
      <div class="call-title ${b.cls}">${ASSETS[key].name} ${b.dir === 'up' ? 'up' : 'down'}</div>
      <div class="call-sub">${fmtMsRemaining(t.remaining)} left in the window · timing reads <span class="${t.cls}">${t.status.toLowerCase()}</span></div>
      <div class="confmeter">
        <div class="track"><div class="fill" style="width:${conf}%;background:${colorVar(b.cls)}"></div><div class="tick" style="left:${settings.entryConf}%"></div></div>
        <div class="v">${conf}/100</div>
      </div>
      <ol class="reasons">${reasons.map(r => `<li>${esc(r.text)}</li>`).join('')}</ol>
    </div>
    <div class="call-side">
      <div class="eyebrow">Kalshi</div>
      <dl class="kv">
        <dt>Contract</dt><dd>${ticker}</dd>
        <dt>Side to buy</dt><dd class="${side === 'YES' ? 'up' : side === 'NO' ? 'down' : 'neutral'}">${side}${price != null ? ' at $' + price.toFixed(2) : ''}</dd>
        <dt>Strike</dt><dd>${strike ? fmtUsd(strike.value) : 'not set'}</dd>
        <dt>Spot</dt><dd>${fmtUsd(d.lastPrice)}</dd>
        <dt>Distance</dt><dd class="${diff == null ? '' : diff >= 0 ? 'up' : 'down'}">${diff == null ? 'n/a' : (diff >= 0 ? '+' : '-') + fmtUsd(Math.abs(diff))}</dd>
      </dl>
      <p class="fine">Size any position as if the model could be wrong, because it can be.</p>
    </div>`;
}

// Board
function renderBoard() {
  const rows = ASSET_KEYS.map(key => {
    const d = assetData[key];
    const cfg = ASSETS[key];
    if (!d || (!d.ok && d.lastPrice === undefined)) {
      return `<tr><td class="asset-cell"><b>${key}</b><span>${cfg.name}</span></td><td colspan="6" class="muted">${d ? 'Feeds unreachable, retrying' : 'Loading'}</td></tr>`;
    }
    const b = biasOf(d.score);
    const conf = Math.round(Math.abs(d.score));
    const t = timingFor(d);
    const m = findMatchingKalshiMarket(key);
    const yp = m ? yesProb(m) : null;
    return `<tr class="click ${key === selectedAsset ? 'sel' : ''}" data-asset="${key}">
      <td class="asset-cell"><b>${key}</b><span>${cfg.name}</span></td>
      <td class="num r">${fmtUsd(d.lastPrice)}</td>
      <td class="num r ${d.chg15 >= 0 ? 'up' : 'down'}">${fmtPct(d.chg15)}</td>
      <td><span class="tag ${b.cls}">${b.label}</span></td>
      <td class="num hide-sm">${conf}<span class="minibar"><i style="width:${conf}%;background:${colorVar(b.cls)}"></i></span></td>
      <td class="hide-sm"><span class="${t.cls}">${t.status}</span></td>
      <td class="num r hide-sm">${yp != null ? Math.round(yp * 100) + '%' : 'n/a'}</td>
    </tr>`;
  }).join('');
  $('boardBody').innerHTML = rows;
}
$('boardBody').addEventListener('click', e => {
  const tr = e.target.closest('tr[data-asset]');
  if (tr) { selectAsset(tr.dataset.asset); setTab('asset'); }
});

// Asset detail
function drawSpark(closes) {
  const svg = $('spark');
  const w = 700, h = 70;
  const min = Math.min(...closes), max = Math.max(...closes);
  const range = (max - min) || 1;
  const pts = closes.map((c, i) => `${(i / (closes.length - 1) * w).toFixed(1)},${(h - ((c - min) / range) * (h - 10) - 5).toFixed(1)}`).join(' ');
  const up = closes[closes.length - 1] >= closes[0];
  svg.innerHTML = `<polyline points="${pts}" fill="none" stroke="${up ? 'var(--up)' : 'var(--down)'}" stroke-width="1.6" vector-effect="non-scaling-stroke"/>`;
}
function quickRenderPrice(key) {
  const d = assetData[key];
  if (!d || d.lastPrice === undefined) return;
  $('price').textContent = fmtUsd(d.lastPrice);
  $('price').className = 'price';
  $('chg15').textContent = (d.chg15 >= 0 ? '▲ ' : '▼ ') + Math.abs(d.chg15).toFixed(2) + '% this window';
  $('chg15').className = 'chg ' + (d.chg15 >= 0 ? 'up' : 'down');
  drawSpark(d.liveCloses || d.closes);
  renderStrikePanel(key);
}
function renderAsset(key) {
  const d = assetData[key];
  const cfg = ASSETS[key];
  if (!d) { $('price').textContent = '$0.00'; $('price').className = 'price stale'; $('subline').textContent = 'Loading live price data.'; return; }
  if (!d.ok && d.lastPrice === undefined) { $('subline').textContent = `All price feeds unreachable for ${cfg.name}. Retrying.`; return; }

  quickRenderPrice(key);
  $('price').className = 'price' + (d.ok ? '' : ' stale');
  $('subline').textContent = `${cfg.name} spot · 1 minute candles, last 60 minutes sampled`;
  $('assetSource').textContent = d.source;
  $('windowLabel').textContent = `${fmtWindowLabel(d.windowStart, d.windowEnd)}`;

  const b = biasOf(d.score);
  const abs = Math.abs(d.score);
  $('verdict').textContent = b.label;
  $('verdict').className = 'verdict ' + b.cls;
  $('conftext').textContent = `confidence ${Math.round(abs)}/100` + (b.dir ? '' : ', no clear edge');
  const fill = $('biasFill');
  const pos = (d.score + 100) / 2;
  fill.style.left = Math.min(50, pos) + '%';
  fill.style.width = Math.abs(pos - 50) + '%';
  fill.style.background = colorVar(b.cls);

  const set = (id, text, cls) => { $(id).textContent = text; $(id).className = 'v ' + (cls || ''); };
  set('emaVal', d.emaBull ? 'Bullish' : 'Bearish', d.emaBull ? 'up' : 'down');
  $('emaDetail').textContent = `9 EMA ${d.emaBull ? 'above' : 'below'} the 21 EMA by ${Math.abs(d.emaGapPct).toFixed(3)}%`;
  set('rsiVal', d.rsiVal.toFixed(1), d.rsiVal > 55 ? 'up' : d.rsiVal < 45 ? 'down' : 'neutral');
  $('rsiDetail').textContent = d.rsiVal > 70 ? 'Overbought territory' : d.rsiVal < 30 ? 'Oversold territory' : d.rsiVal > 50 ? 'Leaning bullish' : 'Leaning bearish';
  set('rocVal', fmtPct(d.chg15, 3), d.chg15 >= 0 ? 'up' : 'down');
  $('rocDetail').textContent = `From the window open at ${fmtUsd(d.openPrice)}`;
  set('volVal', fmtPct(d.volChangePct, 1), Math.abs(d.volChangePct) < 5 ? 'neutral' : d.volChangePct >= 0 ? 'up' : 'down');
  $('volDetail').textContent = 'Last 15 minutes against the 15 before';
  if (d.candle) {
    set('candleVal', d.candle.kind === 'doji' ? 'Doji' : d.candle.kind === 'hammer' ? 'Hammer' : d.candle.kind === 'star' ? 'Star' : d.candle.bullish ? 'Bullish' : 'Bearish', d.candle.dir || 'neutral');
    $('candleDetail').textContent = `${d.candle.label} · body ${Math.round(d.candle.bodyPct * 100)}%, upper wick ${Math.round(d.candle.upperPct * 100)}%, lower wick ${Math.round(d.candle.lowerPct * 100)}%`;
  } else { set('candleVal', 'n/a', 'neutral'); $('candleDetail').textContent = 'Waiting on a closed candle'; }
  renderStrikePanel(key);
}

// Strike
function getStrikeInfo(key) {
  const m = findMatchingKalshiMarket(key);
  if (m && m.floor_strike) return { value: m.floor_strike, source: 'live', label: `Kalshi ${m.ticker}` };
  if (manualStrikes[key] != null) return { value: manualStrikes[key], source: 'manual', label: 'Manual entry' };
  return null;
}
function renderStrikePanel(key) {
  const info = getStrikeInfo(key);
  const src = $('strikeSourceLabel');
  const d = assetData[key];
  if (!info) {
    src.textContent = 'not set'; src.className = 'muted';
    $('strikeValue').textContent = 'Not set';
    $('strikeSourceDetail').textContent = 'Fetching the live strike from Kalshi every second. Set one manually as a backup.';
    $('strikeDiff').textContent = '';
    return;
  }
  src.textContent = info.source === 'live' ? 'live' : 'manual';
  src.className = info.source === 'live' ? 'up' : 'neutral';
  $('strikeValue').textContent = fmtUsd(info.value);
  $('strikeSourceDetail').textContent = info.label;
  if (d && d.lastPrice != null) {
    const diff = d.lastPrice - info.value;
    const pct = diff / info.value * 100;
    $('strikeDiff').textContent = `${diff >= 0 ? '▲' : '▼'} ${fmtUsd(Math.abs(diff))} (${Math.abs(pct).toFixed(3)}%) ${diff >= 0 ? 'above' : 'below'}`;
    $('strikeDiff').className = 'strike-diff ' + (diff >= 0 ? 'up' : 'down');
  } else $('strikeDiff').textContent = '';
}
function saveManualStrike(key, v) { manualStrikes[key] = v; LS.set('sd15:strike:' + key, v); }
function clearManualStrike(key) { delete manualStrikes[key]; LS.del('sd15:strike:' + key); }
function toggleStrikeInput(show) {
  $('strikeInputRow').style.display = show ? 'flex' : 'none';
  if (show) { const info = getStrikeInfo(selectedAsset); $('strikeInput').value = info ? info.value : ''; $('strikeInput').focus(); }
}
$('strikeEditLink').addEventListener('click', () => toggleStrikeInput($('strikeInputRow').style.display !== 'flex'));
$('strikeSetBtn').addEventListener('click', () => {
  const v = parseFloat($('strikeInput').value);
  if (isNaN(v) || v <= 0) return;
  saveManualStrike(selectedAsset, v); toggleStrikeInput(false); renderStrikePanel(selectedAsset); renderStrikeSettings();
});
$('strikeInput').addEventListener('keydown', e => { if (e.key === 'Enter') $('strikeSetBtn').click(); });
$('strikeClearBtn').addEventListener('click', () => { clearManualStrike(selectedAsset); toggleStrikeInput(false); renderStrikePanel(selectedAsset); renderStrikeSettings(); });

// Kalshi
const KALSHI_BASE = 'https://api.elections.kalshi.com/trade-api/v2';
const kalshiBackoffUntil = {};
function yesProb(m) {
  if (m.yes_bid != null && m.yes_ask != null) return (m.yes_bid + m.yes_ask) / 2;
  if (m.yes_bid != null) return m.yes_bid;
  if (m.yes_ask != null) return m.yes_ask;
  return null;
}
async function updateKalshiForAsset(key) {
  if (kalshiBackoffUntil[key] && Date.now() < kalshiBackoffUntil[key]) return;
  const cfg = ASSETS[key];
  try {
    const res = await fetch(`${KALSHI_BASE}/events?series_ticker=${cfg.series}&status=open&with_nested_markets=true&limit=6`);
    if (res.status === 429) { kalshiBackoffUntil[key] = Date.now() + 10000; throw new Error('rate limited'); }
    if (!res.ok) throw new Error('http ' + res.status);
    const data = await res.json();
    let markets = [];
    (data.events || []).forEach(ev => (ev.markets || []).forEach(m => {
      const num = v => v != null ? parseFloat(v) : null;
      const obj = {
        ticker: m.ticker, title: ev.title || m.yes_sub_title || m.title, close_time: m.close_time, floor_strike: m.floor_strike,
        yes_bid: num(m.yes_bid_dollars), yes_ask: num(m.yes_ask_dollars), no_bid: num(m.no_bid_dollars), no_ask: num(m.no_ask_dollars), asset: key,
      };
      markets.push(obj); marketsByTicker[obj.ticker] = obj;
    }));
    markets = markets.filter(m => m.close_time).sort((a, b) => new Date(a.close_time) - new Date(b.close_time));
    if (!markets.length) throw new Error('no open markets');
    kalshiData[key] = { ok: true, markets: markets.slice(0, 4) };
  } catch (err) {
    kalshiData[key] = { ok: false, markets: (kalshiData[key] && kalshiData[key].markets) || [] };
  }
}
let kalshiTickRunning = false;
async function kalshiRefreshCycle() {
  if (kalshiTickRunning) return;
  kalshiTickRunning = true;
  try { await Promise.all(ASSET_KEYS.map(updateKalshiForAsset)); } catch (e) {}
  kalshiTickRunning = false;
  renderMarkets();
  renderStrikePanel(selectedAsset);
  renderTopCall();
}

function alignmentFor(m) {
  const d = assetData[m.asset];
  const dir = guessDirectionFromTitle(m.title);
  const yp = yesProb(m);
  if (!dir || yp == null || !d || !d.ok) return { text: 'no read', cls: 'plain' };
  const b = biasOf(d.score);
  let marketDir = null;
  if (yp > 0.52) marketDir = dir; else if (yp < 0.48) marketDir = dir === 'up' ? 'down' : 'up';
  if (!marketDir || !b.dir) return { text: 'no clear read', cls: 'neutral' };
  return marketDir === b.dir ? { text: 'model agrees', cls: 'up' } : { text: 'model diverges', cls: 'down' };
}
function marketRowsHtml(list, withAsset) {
  if (!list.length) return '';
  return `<table class="tbl"><thead><tr>
    ${withAsset ? '<th>Asset</th>' : ''}<th>Contract</th><th class="r">Strike</th><th class="r">Closes in</th><th class="r">YES</th><th class="r">NO</th><th class="hide-sm">Model</th>
  </tr></thead><tbody>${list.map(m => {
    const yp = yesProb(m);
    const a = alignmentFor(m);
    return `<tr class="click" data-ticker="${esc(m.ticker)}">
      ${withAsset ? `<td><b>${m.asset}</b></td>` : ''}
      <td>${esc(m.title || m.ticker)}<div class="muted mono" style="font-size:11px">${esc(m.ticker)}</div></td>
      <td class="num r">${m.floor_strike ? fmtUsd(m.floor_strike) : 'n/a'}</td>
      <td class="num r accent">${fmtCountdown(m.close_time)}</td>
      <td class="num r up">${yp != null ? Math.round(yp * 100) + '%' : 'n/a'}</td>
      <td class="num r down">${yp != null ? Math.round((1 - yp) * 100) + '%' : 'n/a'}</td>
      <td class="hide-sm"><span class="tag ${a.cls}">${a.text}</span></td>
    </tr>`;
  }).join('')}</tbody></table>`;
}
function renderMarkets() {
  // asset view
  const key = selectedAsset;
  const kd = kalshiData[key];
  $('kalshiSectionLabel').textContent = `Kalshi contracts · ${ASSETS[key].series}`;
  const st = $('kalshiStatusText');
  if (!kd || (!kd.ok && !kd.markets.length)) {
    st.textContent = kd ? 'unavailable' : 'connecting'; st.className = 'sec-note ' + (kd ? 'down' : '');
    $('assetMarkets').innerHTML = `<div class="empty">Kalshi's public API did not answer for ${ASSETS[key].series}, or no contracts are open. Check <a href="https://kalshi.com/category/crypto/frequency/fifteen_min" target="_blank" rel="noopener">Kalshi crypto 15 min</a> directly. The signal above works on its own.</div>`;
  } else {
    st.textContent = kd.ok ? 'live' : 'stale, last good data shown'; st.className = 'sec-note ' + (kd.ok ? 'up' : 'neutral');
    $('assetMarkets').innerHTML = marketRowsHtml(kd.markets, false);
  }
  // all markets view
  const all = [];
  const parts = [];
  ASSET_KEYS.forEach(k => {
    const x = kalshiData[k];
    parts.push(`${ASSETS[k].series} <span class="${!x ? 'muted' : x.ok ? 'up' : 'down'}">${!x ? 'connecting' : x.ok ? 'live' : 'down'}</span>`);
    if (x) all.push(...x.markets.filter(m => new Date(m.close_time).getTime() > Date.now()));
  });
  all.sort((a, b) => new Date(a.close_time) - new Date(b.close_time) || ASSET_KEYS.indexOf(a.asset) - ASSET_KEYS.indexOf(b.asset));
  $('seriesStatus').innerHTML = parts.join(' · ');
  $('allMarkets').innerHTML = all.length ? marketRowsHtml(all, true) : `<div class="empty">No open contracts returned yet.</div>`;
  $('mktCount').textContent = all.length;
}
function onMarketClick(e) { const tr = e.target.closest('tr[data-ticker]'); if (tr) openMarketModal(tr.dataset.ticker); }
$('assetMarkets').addEventListener('click', onMarketClick);
$('allMarkets').addEventListener('click', onMarketClick);

// Modal
function openMarketModal(ticker) {
  const m = marketsByTicker[ticker];
  if (!m) return;
  $('modalEyebrow').textContent = `Kalshi · ${ASSETS[m.asset].series}`;
  $('modalTitle').textContent = m.title || m.ticker;
  $('modalTicker').textContent = m.ticker;
  $('modalStrike').textContent = m.floor_strike ? fmtUsd(m.floor_strike) : 'n/a';
  $('modalTimer').textContent = fmtCountdown(m.close_time);
  const yp = yesProb(m);
  $('modalYes').textContent = yp != null ? Math.round(yp * 100) + '%' : 'n/a';
  $('modalNo').textContent = yp != null ? Math.round((1 - yp) * 100) + '%' : 'n/a';
  const a = alignmentFor(m);
  $('modalRead').textContent = a.text; $('modalRead').className = a.cls === 'plain' ? 'muted' : a.cls;
  $('modalOpenLink').href = marketUrl(m.asset, m.ticker);
  $('modalOverlay').dataset.ticker = ticker;
  $('modalOverlay').classList.add('show');
}
function closeMarketModal() { $('modalOverlay').classList.remove('show'); }
$('modalClose').addEventListener('click', closeMarketModal);
$('modalOverlay').addEventListener('click', e => { if (e.target.id === 'modalOverlay') closeMarketModal(); });
document.addEventListener('keydown', e => { if (e.key === 'Escape') closeMarketModal(); });

// 15m chart
let chartState = null;
function linearTrend(values) {
  const n = values.length;
  let sx = 0, sy = 0, sxy = 0, sx2 = 0;
  for (let i = 0; i < n; i++) { sx += i; sy += values[i]; sxy += i * values[i]; sx2 += i * i; }
  const slope = (n * sxy - sx * sy) / ((n * sx2 - sx * sx) || 1);
  return { slope, intercept: (sy - slope * sx) / n };
}
function detectFlags(opens, highs, lows, closes) {
  const n = closes.length;
  const ranges = highs.map((h, i) => Math.max(h - lows[i], 1e-9));
  const avg = ranges.reduce((a, b) => a + b, 0) / n;
  const flags = [];
  for (let i = 0; i < n - 3; i++) {
    const bodyPct = Math.abs(closes[i] - opens[i]) / ranges[i];
    if (!(bodyPct > 0.6 && ranges[i] > avg * 1.4)) continue;
    let len = 0;
    for (let j = i + 1; j < Math.min(i + 5, n); j++) { if (ranges[j] < avg * 0.75) len++; else break; }
    if (len >= 2) { flags.push({ start: i, end: i + len, dir: closes[i] >= opens[i] ? 'up' : 'down' }); i += len; }
  }
  return flags;
}
function renderCandlesChart(data) {
  const svg = $('candlesChart');
  const W = Math.max(320, Math.round(svg.clientWidth || 900));
  const H = Math.max(300, Math.round(svg.clientHeight || 420));
  svg.setAttribute('viewBox', `0 0 ${W} ${H}`);
  const narrow = W < 560;
  const N = narrow ? Math.min(48, data.closes.length) : Math.min(96, data.closes.length);
  const sl = a => a.slice(-N);
  const opens = sl(data.opens), highs = sl(data.highs), lows = sl(data.lows), closes = sl(data.closes), vols = sl(data.volumes), times = sl(data.times);
  const n = closes.length;
  const padL = 8, padR = narrow ? 64 : 84, padT = 12;
  const priceBottom = H - 92, volTop = H - 80, volBottom = H - 24;
  const plotH = priceBottom - padT;
  const min = Math.min(...lows), max = Math.max(...highs);
  const range = (max - min) || 1;
  const y = p => padT + (1 - (p - min) / range) * plotH;
  const plotW = W - padL - padR;
  const slot = plotW / n;
  const cw = Math.max(1.5, slot * 0.64);
  const vmax = Math.max(...vols) || 1;
  const parts = [];

  for (let g = 0; g <= 4; g++) {
    const gp = min + range * g / 4, gy = y(gp);
    parts.push(`<line x1="${padL}" y1="${gy.toFixed(1)}" x2="${W - padR}" y2="${gy.toFixed(1)}" stroke="var(--line)" stroke-width="1"/>`);
    parts.push(`<text x="${W - padR + 8}" y="${(gy + 4).toFixed(1)}" fill="var(--muted)" font-size="11" font-family="IBM Plex Mono, ui-monospace, monospace">${fmtUsd(gp)}</text>`);
  }
  parts.push(`<line x1="${padL}" y1="${volTop - 4}" x2="${W - padR}" y2="${volTop - 4}" stroke="var(--line)"/>`);
  parts.push(`<text x="${W - padR + 8}" y="${volTop + 8}" fill="var(--muted)" font-size="10.5" font-family="IBM Plex Sans, sans-serif">volume</text>`);

  const flags = detectFlags(opens, highs, lows, closes);
  flags.forEach(f => {
    const xs = padL + f.start * slot, xe = padL + (f.end + 1) * slot;
    parts.push(`<rect x="${xs.toFixed(1)}" y="${padT}" width="${(xe - xs).toFixed(1)}" height="${plotH}" fill="var(--neutral-tint)" stroke="var(--neutral)" stroke-width="1" stroke-dasharray="3,3"/>`);
    parts.push(`<text x="${((xs + xe) / 2).toFixed(1)}" y="${padT + 13}" fill="var(--neutral)" font-size="10" text-anchor="middle" font-family="IBM Plex Sans, sans-serif" font-weight="600">${f.dir === 'up' ? 'BULL FLAG' : 'BEAR FLAG'}</text>`);
  });

  const patterns = [];
  const labelEvery = Math.max(1, Math.round(n / (narrow ? 4 : 8)));
  for (let i = 0; i < n; i++) {
    const x = padL + i * slot + slot / 2;
    const o = opens[i], h = highs[i], l = lows[i], c = closes[i];
    const up = c >= o, col = up ? 'var(--up)' : 'var(--down)';
    const yH = y(h), yL = y(l), yO = y(o), yC = y(c);
    const top = Math.min(yO, yC), bh = Math.max(Math.abs(yC - yO), 1);
    parts.push(`<line x1="${x.toFixed(1)}" y1="${yH.toFixed(1)}" x2="${x.toFixed(1)}" y2="${yL.toFixed(1)}" stroke="${col}" stroke-width="1"/>`);
    parts.push(`<rect x="${(x - cw / 2).toFixed(1)}" y="${top.toFixed(1)}" width="${cw.toFixed(1)}" height="${bh.toFixed(1)}" fill="${col}"/>`);
    const vh = (vols[i] / vmax) * (volBottom - volTop);
    parts.push(`<rect x="${(x - cw / 2).toFixed(1)}" y="${(volBottom - vh).toFixed(1)}" width="${cw.toFixed(1)}" height="${vh.toFixed(1)}" fill="${col}" opacity="0.45"/>`);
    const r = readCandle(o, h, l, c);
    patterns.push(r);
    if (r.kind === 'hammer') parts.push(`<polygon points="${x - 4},${yL + 10} ${x + 4},${yL + 10} ${x},${yL + 3}" fill="var(--up)"/>`);
    else if (r.kind === 'star') parts.push(`<polygon points="${x - 4},${yH - 10} ${x + 4},${yH - 10} ${x},${yH - 3}" fill="var(--down)"/>`);
    else if (r.kind === 'doji') { const my = (yO + yC) / 2; parts.push(`<rect x="${x - 3}" y="${my - 3}" width="6" height="6" fill="var(--neutral)" transform="rotate(45 ${x} ${my})"/>`); }
    if (i % labelEvery === 0) parts.push(`<text x="${x.toFixed(1)}" y="${H - 6}" fill="var(--muted)" font-size="10.5" text-anchor="middle" font-family="IBM Plex Mono, ui-monospace, monospace">${hhmm(times[i])}</text>`);
  }
  const { slope, intercept } = linearTrend(closes);
  parts.push(`<line x1="${(padL + slot / 2).toFixed(1)}" y1="${y(intercept).toFixed(1)}" x2="${(padL + (n - 1) * slot + slot / 2).toFixed(1)}" y2="${y(intercept + slope * (n - 1)).toFixed(1)}" stroke="var(--accent)" stroke-width="2" stroke-dasharray="7,5"/>`);
  const last = closes[n - 1];
  parts.push(`<line x1="${padL}" y1="${y(last).toFixed(1)}" x2="${W - padR}" y2="${y(last).toFixed(1)}" stroke="var(--text-2)" stroke-width="1" stroke-dasharray="2,3"/>`);
  parts.push(`<rect x="${W - padR + 2}" y="${(y(last) - 9).toFixed(1)}" width="${padR - 4}" height="18" fill="var(--text)"/>`);
  parts.push(`<text x="${W - padR + 8}" y="${(y(last) + 4).toFixed(1)}" fill="var(--bg)" font-size="11" font-family="IBM Plex Mono, ui-monospace, monospace">${fmtUsd(last)}</text>`);
  parts.push(`<line id="chHover" x1="0" y1="${padT}" x2="0" y2="${volBottom}" stroke="var(--text-2)" stroke-width="1" visibility="hidden"/>`);
  svg.innerHTML = parts.join('');

  chartState = { data, geom: { padL, slot, n, W }, slice: { opens, highs, lows, closes, vols, times }, patterns, flags, slope };
  return chartState;
}
function renderChartSummary(cs, key) {
  const { slope, patterns, flags, slice } = cs;
  const last = slice.closes[slice.closes.length - 1];
  const sp = slope / last * 100;
  const trend = sp > 0.02 ? ['Uptrend', 'up'] : sp < -0.02 ? ['Downtrend', 'down'] : ['Sideways', 'neutral'];
  const bull = patterns.filter(p => p.dir === 'up').length, bear = patterns.filter(p => p.dir === 'down').length;
  $('lgTrend').textContent = trend[0]; $('lgTrend').className = 'v ' + trend[1];
  $('lgBull').textContent = bull; $('lgBear').textContent = bear;
  $('lgFlags').textContent = flags.length;
  const span = slice.closes.length * 15 / 60;
  $('chartSummary').innerHTML = `<b>${slice.closes.length} candles</b> of ${ASSETS[key].name} at 15 minutes, about ${span.toFixed(0)} hours, from ${esc(cs.data.source)}. The dashed slate line is a least squares fit through the closes and slopes <b class="${trend[1]}">${(sp >= 0 ? '+' : '') + sp.toFixed(3)}%</b> per candle. `
    + (flags.length ? `${flags.length} flag shape${flags.length > 1 ? 's' : ''} in view: a strong pole candle followed by tight consolidation, a common continuation setup. ` : 'No pole and consolidation shape is in view. ')
    + `Pattern recognition on price shape carries no backtested edge, so weigh it alongside the indicators.`;
}
let chartLoading = false;
function loadChart() {
  if (chartLoading) return;
  chartLoading = true;
  const key = selectedAsset;
  const st = $('chartStatusText');
  if (!chartState || chartState.key !== key) { st.textContent = 'loading'; st.className = 'sec-note'; }
  get15mCandles(ASSETS[key]).then(data => {
    if (key !== selectedAsset) return;
    const cs = renderCandlesChart(data);
    cs.key = key;
    renderChartSummary(cs, key);
    st.textContent = 'live · ' + data.source + ' · 15m'; st.className = 'sec-note up';
  }).catch(() => {
    st.textContent = 'unavailable, retrying next cycle'; st.className = 'sec-note down';
    $('chartSummary').textContent = 'Could not load 15 minute candles right now. The page retries automatically.';
  }).finally(() => { chartLoading = false; });
}
$('candlesChart').addEventListener('pointermove', e => {
  if (!chartState) return;
  const svg = $('candlesChart');
  const rect = svg.getBoundingClientRect();
  const x = (e.clientX - rect.left) * (chartState.geom.W / rect.width);
  const { padL, slot, n } = chartState.geom;
  const i = Math.floor((x - padL) / slot);
  const hl = $('chHover');
  if (i < 0 || i >= n) { if (hl) hl.setAttribute('visibility', 'hidden'); return; }
  const s = chartState.slice;
  const cx = padL + i * slot + slot / 2;
  if (hl) { hl.setAttribute('x1', cx); hl.setAttribute('x2', cx); hl.setAttribute('visibility', 'visible'); }
  const chg = (s.closes[i] - s.opens[i]) / s.opens[i] * 100;
  const p = chartState.patterns[i];
  $('chartReadout').innerHTML = `${hhmm(s.times[i])} UTC &nbsp; O ${fmtUsd(s.opens[i])} &nbsp; H ${fmtUsd(s.highs[i])} &nbsp; L ${fmtUsd(s.lows[i])} &nbsp; C ${fmtUsd(s.closes[i])} &nbsp; <span class="${chg >= 0 ? 'up' : 'down'}">${fmtPct(chg)}</span> &nbsp; <span class="muted">${esc(p.label)}</span>`;
});
$('candlesChart').addEventListener('pointerleave', () => {
  const hl = $('chHover'); if (hl) hl.setAttribute('visibility', 'hidden');
  $('chartReadout').textContent = 'Move across the chart to read a candle.';
});
let resizeTimer = null;
window.addEventListener('resize', () => {
  clearTimeout(resizeTimer);
  resizeTimer = setTimeout(() => { if (activeTab === 'chart' && chartState) { const cs = renderCandlesChart(chartState.data); cs.key = selectedAsset; renderChartSummary(cs, selectedAsset); } }, 150);
});

// Signal log
function saveLog() { if (signalLog.length > 400) signalLog = signalLog.slice(-400); LS.set('sd15:log', signalLog); }
function maybeLog(key, d) {
  let changed = false;
  const now = Date.now();
  signalLog.forEach(e => {
    if (e.key !== key || e.result) return;
    if (d.windowStart === e.windowEnd && now - d.windowStart > 60000) {
      e.closePrice = d.openPrice;
      const ref = e.strike != null ? e.strike : e.openPrice;
      if (!e.dir) e.result = 'pass';
      else e.result = ((e.dir === 'up') === (e.closePrice > ref)) ? 'hit' : 'miss';
      changed = true;
    } else if (d.windowStart > e.windowEnd) { e.result = 'nodata'; changed = true; }
  });
  const elapsedMin = (now - d.windowStart) / 60000;
  const id = key + ':' + d.windowStart;
  if (elapsedMin >= settings.snapMin && elapsedMin < 14 && !signalLog.some(e => e.id === id)) {
    const b = biasOf(d.score);
    const strike = getStrikeInfo(key);
    signalLog.push({ id, key, windowStart: d.windowStart, windowEnd: d.windowEnd, openPrice: d.openPrice, callPrice: d.lastPrice,
      strike: strike ? strike.value : null, score: Math.round(d.score), conf: Math.round(Math.abs(d.score)), dir: b.dir, result: null, closePrice: null });
    changed = true;
  }
  if (changed) { saveLog(); renderLog(); }
}
function rate(h, m) { return h + m ? Math.round(h / (h + m) * 100) + '%' : 'n/a'; }
function renderLog() {
  const graded = signalLog.filter(e => e.result === 'hit' || e.result === 'miss');
  const hits = graded.filter(e => e.result === 'hit').length;
  const high = graded.filter(e => e.conf >= settings.entryConf);
  const hHits = high.filter(e => e.result === 'hit').length;
  $('logCount').textContent = signalLog.length;
  $('lsTotal').textContent = signalLog.length;
  $('lsGraded').textContent = graded.length;
  $('lsRate').textContent = rate(hits, graded.length - hits);
  $('lsRateHigh').textContent = rate(hHits, high.length - hHits);
  $('logAssetBody').innerHTML = ASSET_KEYS.map(k => {
    const es = signalLog.filter(e => e.key === k);
    const h = es.filter(e => e.result === 'hit').length, m = es.filter(e => e.result === 'miss').length, p = es.filter(e => e.result === 'pass').length;
    return `<tr><td><b>${k}</b></td><td class="num r">${es.length}</td><td class="num r up">${h}</td><td class="num r down">${m}</td><td class="num r">${p}</td><td class="num r">${rate(h, m)}</td></tr>`;
  }).join('');
  const recent = signalLog.slice().reverse().slice(0, 60);
  $('logBody').innerHTML = recent.length ? recent.map(e => {
    const res = !e.result ? ['open', 'accent'] : e.result === 'hit' ? ['hit', 'up'] : e.result === 'miss' ? ['miss', 'down'] : e.result === 'pass' ? ['passed', 'neutral'] : ['no close', 'plain'];
    const day = new Date(e.windowStart).toISOString().slice(5, 10);
    return `<tr>
      <td class="num">${day} ${hhmm(e.windowStart)}</td><td><b>${e.key}</b></td>
      <td class="${e.dir || 'neutral'}">${e.dir ? (e.dir === 'up' ? 'Up' : 'Down') : 'Neutral'}</td>
      <td class="num r">${e.conf}</td>
      <td class="num r hide-sm">${fmtUsd(e.strike != null ? e.strike : e.openPrice)}</td>
      <td class="num r hide-sm">${e.closePrice != null ? fmtUsd(e.closePrice) : 'n/a'}</td>
      <td><span class="tag ${res[1]}">${res[0]}</span></td>
    </tr>`;
  }).join('') : `<tr><td colspan="7" class="empty">Nothing logged yet. The first call is recorded ${settings.snapMin} minutes into a window.</td></tr>`;
}
$('clearLog').addEventListener('click', () => {
  if (!signalLog.length) return;
  if (!confirm('Clear all ' + signalLog.length + ' logged calls from this browser?')) return;
  signalLog = []; saveLog(); renderLog();
});
$('exportLog').addEventListener('click', () => {
  const head = 'window_start_utc,asset,call,score,confidence,open_price,strike,call_price,close_price,result';
  const rows = signalLog.map(e => [new Date(e.windowStart).toISOString(), e.key, e.dir || 'neutral', e.score, e.conf, e.openPrice, e.strike ?? '', e.callPrice, e.closePrice ?? '', e.result || 'open'].join(','));
  const blob = new Blob([head + '\n' + rows.join('\n')], { type: 'text/csv' });
  const a = document.createElement('a');
  a.href = URL.createObjectURL(blob); a.download = 'signal-log.csv';
  document.body.appendChild(a); a.click(); a.remove();
  setTimeout(() => URL.revokeObjectURL(a.href), 1000);
});

// Settings
function renderSettingsForm() {
  $('setEntry').value = settings.entryConf;
  $('setNeutral').value = settings.neutralBand;
  $('setSnap').value = settings.snapMin;
  $('setTheme').value = theme;
  $('mNeutral').textContent = '±' + settings.neutralBand;
}
function renderStrikeSettings() {
  $('strikeSettings').innerHTML = ASSET_KEYS.map(k => `
    <div class="setting strike">
      <label for="ms-${k}">${k}</label>
      <input class="field" type="number" step="any" id="ms-${k}" data-k="${k}" value="${manualStrikes[k] != null ? manualStrikes[k] : ''}" placeholder="none">
      <button class="btn ghost" type="button" data-clear="${k}">Clear</button>
    </div>`).join('');
}
function commitSettings() {
  const clamp = (v, lo, hi, def) => { v = parseFloat(v); return isNaN(v) ? def : Math.max(lo, Math.min(hi, Math.round(v))); };
  settings.entryConf = clamp($('setEntry').value, 10, 95, DEFAULT_SETTINGS.entryConf);
  settings.neutralBand = clamp($('setNeutral').value, 0, 50, DEFAULT_SETTINGS.neutralBand);
  settings.snapMin = clamp($('setSnap').value, 1, 12, DEFAULT_SETTINGS.snapMin);
  LS.set('sd15:settings', settings);
  renderSettingsForm();
  renderEverything();
}
['setEntry', 'setNeutral', 'setSnap'].forEach(id => $(id).addEventListener('change', commitSettings));
$('setTheme').addEventListener('change', e => applyTheme(e.target.value));
$('resetSettings').addEventListener('click', () => { settings = Object.assign({}, DEFAULT_SETTINGS); LS.set('sd15:settings', settings); renderSettingsForm(); renderEverything(); });
$('strikeSettings').addEventListener('change', e => {
  const k = e.target.dataset.k; if (!k) return;
  const v = parseFloat(e.target.value);
  if (isNaN(v) || v <= 0) clearManualStrike(k); else saveManualStrike(k, v);
  if (k === selectedAsset) renderStrikePanel(k);
});
$('strikeSettings').addEventListener('click', e => {
  const k = e.target.dataset.clear; if (!k) return;
  clearManualStrike(k); renderStrikeSettings(); if (k === selectedAsset) renderStrikePanel(k);
});

// Theme
function applyTheme(t) {
  theme = t === 'light' ? 'light' : 'dark';
  document.documentElement.setAttribute('data-theme', theme);
  $('themeBtn').textContent = theme === 'dark' ? 'Light' : 'Dark';
  $('setTheme').value = theme;
  LS.set('sd15:theme', theme);
  tvLoadedFor = null;
  if (activeTab === 'asset') updateTvChart();
  if (activeTab === 'chart' && chartState) { const cs = renderCandlesChart(chartState.data); cs.key = selectedAsset; renderChartSummary(cs, selectedAsset); }
}
$('themeBtn').addEventListener('click', () => applyTheme(theme === 'dark' ? 'light' : 'dark'));

// TradingView
let tvLoadedFor = null;
function updateTvChart() {
  const sig = selectedAsset + ':' + theme;
  if (tvLoadedFor === sig) return;
  tvLoadedFor = sig;
  const params = new URLSearchParams({
    frameElementId: 'tvFrame', symbol: ASSETS[selectedAsset].tvSymbol, interval: '1', hidesidetoolbar: '1', symboledit: '0', saveimage: '0',
    toolbarbg: theme === 'dark' ? '1a1a17' : 'f3efe5', studies: '[]', theme: theme, style: '1', timezone: 'Etc/UTC', withdateranges: '1', hideideas: '1',
  });
  $('tvFrame').src = `https://s.tradingview.com/widgetembed/?${params.toString()}`;
}

// Switchers, tabs
function renderSwitchers() {
  const html = ASSET_KEYS.map(k => {
    const d = assetData[k];
    const cls = d && d.ok ? biasOf(d.score).cls : 'neutral';
    return `<button type="button" data-asset="${k}" class="${k === selectedAsset ? 'on' : ''}">${k}<span class="sq" style="background:${colorVar(cls)}"></span></button>`;
  }).join('');
  document.querySelectorAll('[data-switcher]').forEach(el => { el.innerHTML = html; });
}
document.querySelectorAll('[data-switcher]').forEach(el => el.addEventListener('click', e => {
  const b = e.target.closest('button[data-asset]'); if (b) selectAsset(b.dataset.asset);
}));

function selectAsset(key) {
  if (!ASSETS[key]) return;
  selectedAsset = key;
  LS.set('sd15:asset', key);
  toggleStrikeInput(false);
  renderSwitchers();
  renderAsset(key);
  renderMarkets();
  renderBoard();
  fastTickAsset(key);
  if (activeTab === 'asset') updateTvChart();
  if (activeTab === 'chart') { chartState = null; $('candlesChart').innerHTML = ''; loadChart(); }
}

function setTab(tab) {
  if (!document.querySelector(`.view[data-view="${tab}"]`)) tab = 'overview';
  activeTab = tab;
  document.querySelectorAll('.tab').forEach(b => { const on = b.dataset.tab === tab; b.classList.toggle('active', on); b.setAttribute('aria-selected', on); });
  document.querySelectorAll('.view').forEach(v => v.classList.toggle('active', v.dataset.view === tab));
  if (location.hash !== '#' + tab) history.replaceState(null, '', '#' + tab);
  if (tab === 'asset') updateTvChart();
  if (tab === 'chart') loadChart();
  if (tab === 'log') renderLog();
  if (tab === 'markets') renderMarkets();
}
$('tabs').addEventListener('click', e => { const b = e.target.closest('.tab'); if (b) setTab(b.dataset.tab); });
window.addEventListener('hashchange', () => setTab(location.hash.slice(1)));

// Global strip + status
function renderWindowStrip() {
  const now = Date.now();
  const { start, end } = currentWindowBounds(now);
  const el = (now - start) / 60000;
  $('winLabel').textContent = fmtWindowLabel(start, end);
  $('winPhase').textContent = phaseOf(el);
  $('winLeft').textContent = fmtMsRemaining(end - now);
  $('globalTrack').innerHTML = phaseTrackHtml(el);
}
function renderStatus() {
  const d = assetData[selectedAsset];
  const okCount = ASSET_KEYS.filter(k => assetData[k] && assetData[k].ok).length;
  const dot = $('statusDot');
  dot.style.background = okCount === ASSET_KEYS.length ? 'var(--up)' : okCount ? 'var(--neutral)' : 'var(--down)';
  $('statusText').textContent = d && d.ok ? `${okCount}/4 feeds live · ${d.source}` : 'feed unreachable, retrying';
  const fm = $('feedMeta');
  fm.textContent = `${okCount}/4 live`;
  fm.className = 'meta-v ' + (okCount === 4 ? 'up' : okCount ? 'neutral' : 'down');
}
function renderEverything() {
  renderSwitchers(); renderBoard(); renderTopCall(); renderTiming(); renderAsset(selectedAsset); renderMarkets(); renderLog(); renderStatus();
}

function tickSecond() {
  $('clock').textContent = new Date().toISOString().slice(11, 19);
  countdown -= 1; if (countdown < 0) countdown = REFRESH_MS / 1000;
  $('nextRefresh').textContent = `indicators refresh in ${countdown}s`;
  renderWindowStrip();
  if (activeTab === 'overview') { renderTopCall(); renderTiming(); }
  const m = $('modalOverlay');
  if (m.classList.contains('show')) { const t = m.dataset.ticker; if (t && marketsByTicker[t]) $('modalTimer').textContent = fmtCountdown(marketsByTicker[t].close_time); }
}

async function refreshCycle() {
  countdown = REFRESH_MS / 1000;
  try { await updateAllAssets(); } catch (e) {}
  if (activeTab === 'chart') loadChart();
}

// Boot
applyTheme(theme);
renderSettingsForm();
renderStrikeSettings();
renderSwitchers();
renderWindowStrip();
renderLog();
setTab(location.hash.slice(1) || 'overview');
tickSecond();
setInterval(tickSecond, 1000);
refreshCycle();
kalshiRefreshCycle();
connectCoinbaseWS();
setInterval(refreshCycle, REFRESH_MS);
setInterval(kalshiRefreshCycle, KALSHI_REFRESH_MS);
setInterval(fastTickAll, FAST_MS);
document.addEventListener('visibilitychange', () => {
  if (!document.hidden) { refreshCycle(); kalshiRefreshCycle(); fastTickAll(); if (!wsConnected && !wsGaveUp) connectCoinbaseWS(); }
});
</script>
</body>
</html>
