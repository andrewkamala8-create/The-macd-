# The-macd-
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no">
<title>MACD · FIB Terminal</title>
<link href="https://fonts.googleapis.com/css2?family=IBM+Plex+Mono:wght@300;400;500;600&family=IBM+Plex+Sans:wght@300;400;500;600&display=swap" rel="stylesheet">
<style>
/* ── RESET & TOKENS ── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
:root {
  --bg:      #080c12;
  --bg1:     #0c1118;
  --bg2:     #111822;
  --bg3:     #16202e;
  --line:    #1d2b3a;
  --line2:   #243344;
  --bull:    #26de81;
  --bear:    #ff4560;
  --amber:   #f5a623;
  --blue:    #4b91f7;
  --purple:  #a78bfa;
  --text:    #d8e3f0;
  --text2:   #6b8299;
  --text3:   #364a5e;
  --mono:    'IBM Plex Mono', monospace;
  --sans:    'IBM Plex Sans', sans-serif;
}
html, body { height: 100%; background: var(--bg); color: var(--text); font-family: var(--sans); overflow: hidden; }
::-webkit-scrollbar { width: 4px; }
::-webkit-scrollbar-track { background: var(--bg); }
::-webkit-scrollbar-thumb { background: var(--line2); border-radius: 2px; }

/* ── LAYOUT ── */
#app { display: flex; flex-direction: column; height: 100vh; }

/* ── TOPBAR ── */
#topbar {
  display: flex; align-items: center; gap: 0;
  height: 46px; background: var(--bg1);
  border-bottom: 1px solid var(--line); flex-shrink: 0;
  padding: 0 14px; position: relative; z-index: 20;
}
#logo {
  font-family: var(--mono); font-size: 13px; font-weight: 600;
  letter-spacing: 3px; color: var(--amber); margin-right: 18px;
  text-shadow: 0 0 16px rgba(245,166,35,.3);
}
/* Instrument tabs */
#instTabs { display: flex; gap: 0; overflow-x: auto; flex: 1; scrollbar-width: none; }
#instTabs::-webkit-scrollbar { display: none; }
.itab {
  padding: 0 16px; height: 46px; display: flex; align-items: center;
  font-family: var(--mono); font-size: 11px; font-weight: 500;
  letter-spacing: 1px; color: var(--text3); cursor: pointer;
  border-bottom: 2px solid transparent; transition: all .15s; white-space: nowrap;
  border-right: 1px solid var(--line);
}
.itab:hover { color: var(--text2); background: var(--bg2); }
.itab.on { color: var(--amber); border-bottom-color: var(--amber); background: rgba(245,166,35,.04); }
/* TF tabs */
#tfTabs { display: flex; gap: 2px; margin-left: 12px; }
.tftab {
  padding: 3px 9px; border-radius: 3px; font-family: var(--mono);
  font-size: 10px; font-weight: 600; letter-spacing: .5px;
  color: var(--text3); cursor: pointer; transition: all .15s;
}
.tftab:hover { color: var(--text2); }
.tftab.on { background: rgba(75,145,247,.15); color: var(--blue); }
/* Live dot */
#liveDot {
  width: 7px; height: 7px; border-radius: 50%;
  background: var(--text3); margin-left: 12px; flex-shrink: 0;
  transition: background .3s;
}
#liveDot.live { background: var(--bull); box-shadow: 0 0 6px var(--bull); animation: glow 2s infinite; }
@keyframes glow { 0%,100%{box-shadow:0 0 4px var(--bull)} 50%{box-shadow:0 0 10px var(--bull)} }
#liveLabel { font-family: var(--mono); font-size: 9px; color: var(--text3); margin-left: 5px; letter-spacing: .5px; }

/* ── PRICE BAR ── */
#pricebar {
  height: 36px; background: var(--bg2); border-bottom: 1px solid var(--line);
  display: flex; align-items: center; padding: 0 14px; gap: 20px;
  flex-shrink: 0; overflow-x: auto; scrollbar-width: none;
}
#pricebar::-webkit-scrollbar { display: none; }
.pb-price { font-family: var(--mono); font-size: 18px; font-weight: 600; letter-spacing: -.5px; }
.pb-chg { font-family: var(--mono); font-size: 11px; font-weight: 500; }
.pb-stat { display: flex; gap: 5px; font-family: var(--mono); font-size: 10px; }
.pb-key { color: var(--text3); }
.pb-val { color: var(--text2); }
.bull { color: var(--bull); } .bear { color: var(--bear); }

/* ── SIGNAL STRIP ── */
#signalStrip {
  height: 30px; background: var(--bg1); border-bottom: 1px solid var(--line);
  display: flex; align-items: center; padding: 0 14px; gap: 14px;
  flex-shrink: 0; overflow-x: auto; scrollbar-width: none;
}
#signalStrip::-webkit-scrollbar { display: none; }
.sig-chip {
  display: flex; align-items: center; gap: 5px;
  font-family: var(--mono); font-size: 9px; font-weight: 600;
  letter-spacing: 1px; padding: 2px 8px; border-radius: 3px;
  border: 1px solid; white-space: nowrap; flex-shrink: 0;
}
.chip-bull { background: rgba(38,222,129,.1); border-color: rgba(38,222,129,.3); color: var(--bull); }
.chip-bear { background: rgba(255,69,96,.1);  border-color: rgba(255,69,96,.3);  color: var(--bear); }
.chip-neu  { background: rgba(107,130,153,.08); border-color: rgba(107,130,153,.2); color: var(--text2); }
.chip-amber{ background: rgba(245,166,35,.1); border-color: rgba(245,166,35,.3); color: var(--amber); }
.sig-dot { width: 5px; height: 5px; border-radius: 50%; background: currentColor; }

/* ── BODY (charts) ── */
#body { flex: 1; display: flex; min-height: 0; }
#chartCol { flex: 1; display: flex; flex-direction: column; min-width: 0; position: relative; }

/* Chart panes */
#pricePane { flex: 1.8; position: relative; min-height: 0; border-bottom: 1px solid var(--line); }
#macdPane  { flex: 1;   position: relative; min-height: 0; }
canvas { display: block; width: 100% !important; height: 100% !important; }

/* Crosshair price label */
.crossLabel {
  position: absolute; right: 0; font-family: var(--mono); font-size: 9px;
  font-weight: 600; padding: 1px 6px; border-radius: 2px 0 0 2px;
  pointer-events: none; display: none; z-index: 10;
}
#crossPriceLbl { background: var(--text2); color: var(--bg); }
#crossMacdLbl  { background: var(--purple); color: var(--bg); }
#crossTimeLbl  {
  position: absolute; bottom: 22px; font-family: var(--mono); font-size: 9px;
  background: var(--bg3); color: var(--text2); padding: 1px 5px; border-radius: 2px;
  pointer-events: none; display: none; z-index: 10; transform: translateX(-50%);
}

/* MACD legend */
#macdLegend {
  position: absolute; top: 4px; left: 8px; z-index: 5;
  display: flex; gap: 10px; font-family: var(--mono); font-size: 9px;
  pointer-events: none;
}
.ml-item { display: flex; align-items: center; gap: 3px; }
.ml-swatch { width: 14px; height: 2px; border-radius: 1px; }

/* ── SIDEBAR ── */
#sidebar {
  width: 240px; flex-shrink: 0; background: var(--bg1);
  border-left: 1px solid var(--line); display: flex; flex-direction: column;
  overflow-y: auto;
}
.sb-section { border-bottom: 1px solid var(--line); padding: 12px; }
.sb-title { font-family: var(--mono); font-size: 9px; letter-spacing: 2px; text-transform: uppercase; color: var(--text3); margin-bottom: 10px; font-weight: 600; }
.sb-row { display: flex; justify-content: space-between; align-items: center; padding: 4px 0; border-bottom: 1px solid rgba(255,255,255,.03); }
.sb-row:last-child { border-bottom: none; }
.sb-key { font-family: var(--mono); font-size: 9px; color: var(--text3); letter-spacing: .3px; }
.sb-val { font-family: var(--mono); font-size: 10px; font-weight: 600; color: var(--text); }

/* Fib levels */
.fib-row {
  display: flex; align-items: center; gap: 8px; padding: 5px 0;
  border-bottom: 1px solid rgba(255,255,255,.03); font-family: var(--mono); font-size: 9px;
}
.fib-row:last-child { border-bottom: none; }
.fib-pct { width: 30px; color: var(--text3); flex-shrink: 0; }
.fib-bar-wrap { flex: 1; height: 3px; background: var(--line); border-radius: 1px; position: relative; overflow: visible; }
.fib-bar-fill { height: 100%; border-radius: 1px; }
.fib-price { width: 66px; text-align: right; color: var(--text2); flex-shrink: 0; font-size: 9px; }
.fib-cur-marker {
  position: absolute; top: -4px; width: 2px; height: 11px;
  background: var(--amber); border-radius: 1px; transform: translateX(-50%); z-index: 2;
}
/* Price proximity badge */
.fib-badge {
  position: absolute; top: -20px; left: 50%; transform: translateX(-50%);
  background: var(--amber); color: var(--bg); font-size: 8px; font-weight: 700;
  padding: 0px 4px; border-radius: 2px; white-space: nowrap; display: none;
}

/* Signal cards */
.sig-card {
  border-radius: 4px; padding: 8px 10px; margin-bottom: 6px;
  font-family: var(--mono); font-size: 10px;
}
.sig-card:last-child { margin-bottom: 0; }
.sig-card-title { font-size: 9px; font-weight: 700; letter-spacing: 1px; margin-bottom: 4px; text-transform: uppercase; }
.sig-card-body { font-size: 9px; color: inherit; opacity: .75; line-height: 1.5; }
.sc-bull { background: rgba(38,222,129,.08); border: 1px solid rgba(38,222,129,.2); color: var(--bull); }
.sc-bear { background: rgba(255,69,96,.08);  border: 1px solid rgba(255,69,96,.2);  color: var(--bear); }
.sc-neu  { background: rgba(75,145,247,.06); border: 1px solid rgba(75,145,247,.15); color: var(--blue); }
.sc-warn { background: rgba(245,166,35,.08); border: 1px solid rgba(245,166,35,.2); color: var(--amber); }

/* ── LOADING ── */
#loadScreen {
  position: absolute; inset: 0; background: rgba(8,12,18,.94);
  display: flex; flex-direction: column; align-items: center; justify-content: center;
  gap: 14px; z-index: 100;
}
#loadScreen .ld-logo { font-family: var(--mono); font-size: 18px; letter-spacing: 4px; color: var(--amber); }
#loadScreen .ld-msg  { font-family: var(--mono); font-size: 10px; letter-spacing: 1px; color: var(--text3); }
.spinner {
  width: 26px; height: 26px; border: 2px solid var(--line2);
  border-top-color: var(--amber); border-radius: 50%;
  animation: spin .7s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
.err-box {
  background: rgba(255,69,96,.08); border: 1px solid rgba(255,69,96,.25);
  border-radius: 5px; padding: 12px 16px; font-family: var(--mono); font-size: 10px;
  color: var(--bear); max-width: 280px; text-align: center; line-height: 1.6;
  display: none;
}
#retryBtn {
  background: rgba(245,166,35,.12); border: 1px solid rgba(245,166,35,.3);
  color: var(--amber); font-family: var(--mono); font-size: 10px; font-weight: 600;
  padding: 6px 18px; border-radius: 4px; cursor: pointer; display: none;
  letter-spacing: 1px; transition: all .15s;
}
#retryBtn:hover { background: rgba(245,166,35,.2); }

/* ── MOBILE ── */
@media (max-width: 640px) {
  #sidebar { display: none; }
  #pricePane { flex: 1.6; }
}
</style>
</head>
<body>
<div id="app">

  <!-- TOPBAR -->
  <div id="topbar">
    <div id="logo">MACD·FIB</div>
    <div id="instTabs">
      <div class="itab on"  data-sym="PAXGUSDT" data-dec="2" onclick="switchSym(this)">PAXG/USDT</div>
      <div class="itab"     data-sym="EURUSDT"  data-dec="5" onclick="switchSym(this)">EUR/USDT</div>
      <div class="itab"     data-sym="BTCUSDT"  data-dec="2" onclick="switchSym(this)">BTC/USDT</div>
      <div class="itab"     data-sym="ETHUSDT"  data-dec="2" onclick="switchSym(this)">ETH/USDT</div>
      <div class="itab"     data-sym="BNBUSDT"  data-dec="3" onclick="switchSym(this)">BNB/USDT</div>
      <div class="itab"     data-sym="SOLUSDT"  data-dec="3" onclick="switchSym(this)">SOL/USDT</div>
      <div class="itab"     data-sym="XRPUSDT"  data-dec="4" onclick="switchSym(this)">XRP/USDT</div>
    </div>
    <div id="tfTabs">
      <div class="tftab"    data-tf="15m"  onclick="switchTF(this)">15m</div>
      <div class="tftab on" data-tf="1h"   onclick="switchTF(this)">1H</div>
      <div class="tftab"    data-tf="4h"   onclick="switchTF(this)">4H</div>
      <div class="tftab"    data-tf="1d"   onclick="switchTF(this)">1D</div>
    </div>
    <div id="liveDot"></div>
    <div id="liveLabel">CONNECTING</div>
  </div>

  <!-- PRICE BAR -->
  <div id="pricebar">
    <span class="pb-price" id="pb-price">—</span>
    <span class="pb-chg"   id="pb-chg">—</span>
    <div class="pb-stat"><span class="pb-key">O</span><span class="pb-val" id="pb-o">—</span></div>
    <div class="pb-stat"><span class="pb-key">H</span><span class="pb-val bull" id="pb-h">—</span></div>
    <div class="pb-stat"><span class="pb-key">L</span><span class="pb-val bear" id="pb-l">—</span></div>
    <div class="pb-stat"><span class="pb-key">VOL</span><span class="pb-val" id="pb-v">—</span></div>
    <div class="pb-stat"><span class="pb-key">MACD</span><span class="pb-val" id="pb-macd">—</span></div>
    <div class="pb-stat"><span class="pb-key">SIG</span><span class="pb-val" id="pb-sig">—</span></div>
    <div class="pb-stat"><span class="pb-key">HIST</span><span class="pb-val" id="pb-hist">—</span></div>
  </div>

  <!-- SIGNAL STRIP -->
  <div id="signalStrip">
    <span style="font-family:var(--mono);font-size:9px;color:var(--text3);letter-spacing:1px;flex-shrink:0;">SIGNALS</span>
    <div id="sigChips" style="display:flex;gap:8px;overflow-x:auto;scrollbar-width:none;">
      <div class="sig-chip chip-neu"><span class="sig-dot"></span>LOADING…</div>
    </div>
  </div>

  <!-- BODY -->
  <div id="body">
    <div id="chartCol">

      <!-- PRICE PANE -->
      <div id="pricePane">
        <canvas id="priceCanvas"></canvas>
        <div class="crossLabel" id="crossPriceLbl"></div>
        <div id="crossTimeLbl" class="crossLabel" style="position:absolute;right:auto;border-radius:2px;"></div>
      </div>

      <!-- MACD PANE -->
      <div id="macdPane">
        <div id="macdLegend">
          <div class="ml-item"><div class="ml-swatch" style="background:var(--blue)"></div><span style="color:var(--text3)">MACD</span></div>
          <div class="ml-item"><div class="ml-swatch" style="background:var(--amber)"></div><span style="color:var(--text3)">SIGNAL</span></div>
          <div class="ml-item"><div class="ml-swatch" style="background:var(--bull);height:6px"></div><span style="color:var(--text3)">HIST</span></div>
        </div>
        <canvas id="macdCanvas"></canvas>
        <div class="crossLabel" id="crossMacdLbl"></div>
      </div>

      <!-- LOADING OVERLAY -->
      <div id="loadScreen">
        <div class="ld-logo">MACD · FIB</div>
        <div class="spinner"></div>
        <div class="ld-msg" id="ldMsg">FETCHING DATA…</div>
        <div class="err-box" id="errBox"></div>
        <button id="retryBtn" onclick="boot()">↺ RETRY</button>
      </div>
    </div>

    <!-- SIDEBAR -->
    <div id="sidebar">

      <!-- MACD Values -->
      <div class="sb-section">
        <div class="sb-title">MACD (12·26·9)</div>
        <div class="sb-row"><span class="sb-key">MACD Line</span><span class="sb-val" id="sb-macd">—</span></div>
        <div class="sb-row"><span class="sb-key">Signal (EMA9)</span><span class="sb-val" id="sb-signal">—</span></div>
        <div class="sb-row"><span class="sb-key">Histogram</span><span class="sb-val" id="sb-hist">—</span></div>
        <div class="sb-row"><span class="sb-key">Momentum</span><span class="sb-val" id="sb-momentum">—</span></div>
        <div class="sb-row"><span class="sb-key">Divergence</span><span class="sb-val" id="sb-div">—</span></div>
        <div class="sb-row"><span class="sb-key">Crossover</span><span class="sb-val" id="sb-cross">—</span></div>
        <div class="sb-row"><span class="sb-key">Zero Cross</span><span class="sb-val" id="sb-zero">—</span></div>
        <div class="sb-row"><span class="sb-key">Hist Strength</span><span class="sb-val" id="sb-hstrength">—</span></div>
      </div>

      <!-- Fibonacci Levels -->
      <div class="sb-section">
        <div class="sb-title">FIBONACCI RETRACEMENT</div>
        <div class="sb-row" style="margin-bottom:6px;">
          <span class="sb-key">Swing High</span><span class="sb-val bull" id="sb-swH">—</span>
        </div>
        <div class="sb-row" style="margin-bottom:8px;">
          <span class="sb-key">Swing Low</span><span class="sb-val bear" id="sb-swL">—</span>
        </div>
        <div id="fibLevels"></div>
      </div>

      <!-- Signal Cards -->
      <div class="sb-section" style="border-bottom:none;">
        <div class="sb-title">ANALYSIS</div>
        <div id="sigCards"></div>
      </div>
    </div>
  </div>
</div>

<script>
// ══════════════════════════════════════════════════════
//  CONSTANTS & STATE
// ══════════════════════════════════════════════════════
const MACD_FAST = 12, MACD_SLOW = 26, MACD_SIG = 9;
const SWING_LEN = 10;
const FIB_LEVELS = [
  { r: 0,     color: '#6b8299', label: '0.000' },
  { r: 0.236, color: '#a78bfa', label: '0.236' },
  { r: 0.382, color: '#4b91f7', label: '0.382' },
  { r: 0.5,   color: '#f5a623', label: '0.500' },
  { r: 0.618, color: '#26de81', label: '0.618' },
  { r: 0.786, color: '#ff4560', label: '0.786' },
  { r: 1.0,   color: '#6b8299', label: '1.000' },
];
const FIB_EXT = [
  { r: 1.272, color: '#a78bfa', label: '1.272' },
  { r: 1.618, color: '#4b91f7', label: '1.618' },
];

let sym    = 'PAXGUSDT';
let tf     = '1h';
let dec    = 2;
let bars   = [];          // closed OHLCV candles
let macdData = null;      // { macd[], signal[], hist[], startIdx }
let fibData  = null;      // { high, low, levels:[] }
let signals  = [];        // computed signal objects

let ws         = null;
let wsRetry    = 0;
let liveBar    = null;    // forming candle from WS

// Chart state
let scrollBars = 0;       // bars scrolled from right edge
let barW       = 10;
let hoverIdx   = -1;
let isDragging = false;
let dragX0     = 0, dragScroll0 = 0;
let pinch0     = 0;
let animId     = null;
const PAD = { T: 28, R: 68, B: 20, L: 2 };
const PAD_MACD = { T: 18, R: 68, B: 22, L: 2 };

// ══════════════════════════════════════════════════════
//  UTILS
// ══════════════════════════════════════════════════════
const fp  = (v, d) => (v == null || isNaN(+v)) ? '—' : (+v).toFixed(d ?? dec);
const fv  = v => !v ? '—' : v>=1e9?(v/1e9).toFixed(2)+'B':v>=1e6?(v/1e6).toFixed(2)+'M':v>=1e3?(v/1e3).toFixed(1)+'K':(+v).toFixed(0);
const clamp = (v, a, b) => Math.max(a, Math.min(b, v));

function fmtTime(tsMs, interval) {
  const d = new Date(tsMs);
  const hh = d.getUTCHours().toString().padStart(2,'0');
  const mm = d.getUTCMinutes().toString().padStart(2,'0');
  const mo = (d.getUTCMonth()+1).toString().padStart(2,'0');
  const dd = d.getUTCDate().toString().padStart(2,'0');
  if (interval === '1d') return `${d.getUTCFullYear()}-${mo}-${dd}`;
  if (interval === '4h') return `${mo}/${dd} ${hh}:${mm}`;
  return `${hh}:${mm}`;
}

// ══════════════════════════════════════════════════════
//  PURE MATH — no side effects
// ══════════════════════════════════════════════════════

// Correct EMA: seed with SMA of first `period` values
function calcEMA(values, period) {
  if (values.length < period) return new Array(values.length).fill(NaN);
  const k = 2 / (period + 1);
  const result = new Array(values.length).fill(NaN);
  // Seed: SMA of first `period` values
  let sum = 0;
  for (let i = 0; i < period; i++) sum += values[i];
  result[period - 1] = sum / period;
  // Recursive EMA forward
  for (let i = period; i < values.length; i++) {
    result[i] = values[i] * k + result[i-1] * (1 - k);
  }
  return result;
}

function calcMACD(closes) {
  const ema12 = calcEMA(closes, MACD_FAST);
  const ema26 = calcEMA(closes, MACD_SLOW);

  // MACD line: only valid where both EMAs exist
  const macdLine = closes.map((_, i) => {
    if (isNaN(ema12[i]) || isNaN(ema26[i])) return NaN;
    return ema12[i] - ema26[i];
  });

  // Signal: EMA(9) of macdLine — but only over valid (non-NaN) portion
  // Build contiguous valid slice, compute EMA9, re-align
  const firstValid = macdLine.findIndex(v => !isNaN(v));
  const validMacd  = macdLine.slice(firstValid);
  const signalRaw  = calcEMA(validMacd, MACD_SIG);

  const signalLine = new Array(closes.length).fill(NaN);
  for (let i = 0; i < signalRaw.length; i++) {
    signalLine[firstValid + i] = signalRaw[i];
  }

  const histogram = closes.map((_, i) => {
    if (isNaN(macdLine[i]) || isNaN(signalLine[i])) return NaN;
    return macdLine[i] - signalLine[i];
  });

  // startIdx = first index where all three values are valid
  const startIdx = histogram.findIndex(v => !isNaN(v));

  return { macd: macdLine, signal: signalLine, hist: histogram, startIdx };
}

// Swing high/low over a fixed lookback (not just global min/max)
function calcSwings(bars) {
  const N = bars.length;
  if (N < SWING_LEN * 2 + 1) return { swingHigh: null, swingLow: null, shIdx: -1, slIdx: -1 };

  // Scan from most recent backward to find last confirmed swing H and L
  let swingHigh = -Infinity, shIdx = -1;
  let swingLow  =  Infinity, slIdx = -1;

  for (let i = SWING_LEN; i < N - SWING_LEN; i++) {
    let isH = true, isL = true;
    for (let j = 1; j <= SWING_LEN; j++) {
      if (bars[i].high <= bars[i-j].high || bars[i].high <= bars[i+j].high) isH = false;
      if (bars[i].low  >= bars[i-j].low  || bars[i].low  >= bars[i+j].low)  isL = false;
    }
    if (isH && bars[i].high > swingHigh) { swingHigh = bars[i].high; shIdx = i; }
    if (isL && bars[i].low  < swingLow)  { swingLow  = bars[i].low;  slIdx = i; }
  }

  if (shIdx < 0 || slIdx < 0) return { swingHigh: null, swingLow: null, shIdx: -1, slIdx: -1 };
  return { swingHigh, swingLow, shIdx, slIdx };
}

function calcFibLevels(swingHigh, swingLow) {
  const range = swingHigh - swingLow;
  const levels = FIB_LEVELS.map(f => ({
    ...f,
    price: swingHigh - f.r * range, // retracement from high downward
  }));
  const extensions = FIB_EXT.map(f => ({
    ...f,
    price: swingLow - f.r * range,  // extension below low (bearish) OR above high (bullish)
  }));
  return { levels, extensions, range };
}

// ══════════════════════════════════════════════════════
//  SIGNAL ENGINE — reads macdData + fibData + bars
// ══════════════════════════════════════════════════════
function computeSignals() {
  if (!macdData || !fibData || bars.length < 2) return [];
  const sigs = [];
  const { macd, signal, hist, startIdx } = macdData;
  const N = bars.length;
  if (N < 3 || startIdx < 0) return [];

  const i = N - 1; // last confirmed closed bar
  const macdNow  = macd[i],   macdPrev  = macd[i-1];
  const sigNow   = signal[i], sigPrev   = signal[i-1];
  const histNow  = hist[i],   histPrev  = hist[i-1],   histPrev2 = hist[i-2];

  if ([macdNow, macdPrev, sigNow, sigPrev, histNow, histPrev].some(v => isNaN(v))) return [];

  const price = bars[i].close;

  // ── 1. MACD / Signal Line Crossover ──
  const crossedUp   = macdPrev < sigPrev && macdNow > sigNow;
  const crossedDown = macdPrev > sigPrev && macdNow < sigNow;
  if (crossedUp)   sigs.push({ type: 'bull', label: 'MACD CROSSOVER ▲', detail: `MACD crossed above Signal @ ${fp(macdNow, 6)}`, weight: 3 });
  if (crossedDown) sigs.push({ type: 'bear', label: 'MACD CROSSOVER ▼', detail: `MACD crossed below Signal @ ${fp(macdNow, 6)}`, weight: 3 });

  // ── 2. Zero-Line Cross ──
  const zeroCrossUp   = macdPrev < 0 && macdNow >= 0;
  const zeroCrossDown = macdPrev > 0 && macdNow <= 0;
  if (zeroCrossUp)   sigs.push({ type: 'bull', label: 'ZERO-LINE CROSS ▲', detail: 'MACD crossed above zero — trend confirmation', weight: 2 });
  if (zeroCrossDown) sigs.push({ type: 'bear', label: 'ZERO-LINE CROSS ▼', detail: 'MACD crossed below zero — trend confirmation', weight: 2 });

  // ── 3. Histogram Momentum Shift ──
  // Expanding histogram (acceleration): |hist| growing
  // Contracting (deceleration — potential reversal)
  const histExpBull = histNow > 0 && histNow > histPrev;
  const histExpBear = histNow < 0 && histNow < histPrev;
  const histContr   = Math.abs(histNow) < Math.abs(histPrev) && Math.abs(histPrev) < Math.abs(histPrev2);
  if (histExpBull) sigs.push({ type: 'bull', label: 'HIST ACCELERATION ▲', detail: `Bullish histogram expanding: ${fp(histPrev,6)} → ${fp(histNow,6)}`, weight: 1 });
  if (histExpBear) sigs.push({ type: 'bear', label: 'HIST ACCELERATION ▼', detail: `Bearish histogram expanding: ${fp(histPrev,6)} → ${fp(histNow,6)}`, weight: 1 });
  if (histContr)   sigs.push({ type: 'warn', label: 'HIST CONTRACTING', detail: 'Momentum decelerating — potential reversal zone', weight: 1 });

  // ── 4. MACD Divergence (last 20 bars) ──
  const lookback = 20;
  const scanStart = Math.max(startIdx, i - lookback);
  let priceHighIdx = i, priceLowIdx = i;
  let macdHighIdx  = i, macdLowIdx  = i;
  for (let j = scanStart; j <= i; j++) {
    if (bars[j].high > bars[priceHighIdx].high) priceHighIdx = j;
    if (bars[j].low  < bars[priceLowIdx].low)   priceLowIdx  = j;
    if (!isNaN(macd[j])) {
      if (macd[j] > (macd[macdHighIdx]??-Infinity)) macdHighIdx = j;
      if (macd[j] < (macd[macdLowIdx]?? Infinity))  macdLowIdx  = j;
    }
  }
  // Bearish divergence: price HH but MACD lower high
  if (priceHighIdx !== i && priceLowIdx !== i) { // at least one previous swing exists in scan
    const recentPriceHigh = Math.max(bars[i].high, bars[i-1]?.high ?? 0);
    const prevPriceHigh   = bars[priceHighIdx].high;
    const recentMacdVal   = macd[i];
    const prevMacdHigh    = macd[macdHighIdx];
    if (!isNaN(prevMacdHigh) && !isNaN(recentMacdVal) && priceHighIdx < i - 3) {
      if (recentPriceHigh > prevPriceHigh && recentMacdVal < prevMacdHigh)
        sigs.push({ type: 'bear', label: 'BEARISH DIVERGENCE', detail: 'Price: higher high · MACD: lower high — reversal risk', weight: 4 });
    }
    const recentPriceLow = Math.min(bars[i].low, bars[i-1]?.low ?? Infinity);
    const prevPriceLow   = bars[priceLowIdx].low;
    const prevMacdLow    = macd[macdLowIdx];
    if (!isNaN(prevMacdLow) && !isNaN(recentMacdVal) && priceLowIdx < i - 3) {
      if (recentPriceLow < prevPriceLow && recentMacdVal > prevMacdLow)
        sigs.push({ type: 'bull', label: 'BULLISH DIVERGENCE', detail: 'Price: lower low · MACD: higher low — reversal likely', weight: 4 });
    }
  }

  // ── 5. Fibonacci proximity (price within 0.5% of a Fib level) ──
  if (fibData) {
    for (const lvl of fibData.levels) {
      const dist = Math.abs(price - lvl.price) / price;
      if (dist < 0.005) {
        const isSup = price >= lvl.price;
        sigs.push({
          type: isSup ? 'bull' : 'bear',
          label: `FIB ${lvl.label} ${isSup ? 'SUPPORT' : 'RESISTANCE'}`,
          detail: `Price ${fp(price)} near Fib ${lvl.label} level ${fp(lvl.price)} (${(dist*100).toFixed(2)}% away)`,
          weight: 2
        });
      }
    }
  }

  // ── 6. MACD above/below zero + Fib zone ──
  if (fibData && macdNow > 0 && price < fibData.levels.find(l=>l.r===0.618).price)
    sigs.push({ type: 'bull', label: 'GOLDEN POCKET SETUP', detail: 'MACD bullish + price in 0.618 Fib zone — high-probability long zone', weight: 3 });
  if (fibData && macdNow < 0 && price > fibData.levels.find(l=>l.r===0.382).price)
    sigs.push({ type: 'bear', label: '0.382 RESISTANCE', detail: 'MACD bearish + price at 0.382 retracement — distribution zone', weight: 2 });

  // Sort by weight descending
  return sigs.sort((a, b) => b.weight - a.weight);
}

// ══════════════════════════════════════════════════════
//  DATA FETCH
// ══════════════════════════════════════════════════════
async function fetchKlines(symbol, interval, limit = 500) {
  const url = `https://api.binance.com/api/v3/klines?symbol=${symbol}&interval=${interval}&limit=${limit}`;
  const ctrl = new AbortController();
  const t = setTimeout(() => ctrl.abort(), 15000);
  try {
    const r = await fetch(url, { signal: ctrl.signal });
    clearTimeout(t);
    if (!r.ok) throw new Error(`Binance HTTP ${r.status}`);
    const raw = await r.json();
    if (!Array.isArray(raw) || !raw.length) throw new Error('Empty response');
    return raw.map(c => ({
      time: c[0], open: +c[1], high: +c[2], low: +c[3], close: +c[4], volume: +c[5]
    }));
  } catch(e) {
    clearTimeout(t);
    throw e.name === 'AbortError' ? new Error('Request timed out') : e;
  }
}

// ══════════════════════════════════════════════════════
//  WEBSOCKET
// ══════════════════════════════════════════════════════
function connectWS() {
  if (ws) { try { ws.close(); } catch(e) {} ws = null; }
  const stream = `${sym.toLowerCase()}@kline_${tf}`;
  try { ws = new WebSocket(`wss://stream.binance.com:443/ws/${stream}`); }
  catch(e) { scheduleReconnect(); return; }

  ws.onopen = () => {
    wsRetry = 0;
    setLiveStatus(true);
  };

  ws.onmessage = e => {
    const k = JSON.parse(e.data).k;
    if (!k) return;
    liveBar = { time: k.t, open: +k.o, high: +k.h, low: +k.l, close: +k.c, volume: +k.v };
    updatePriceBar(liveBar);
    requestDraw();

    if (k.x) { // candle closed — commit to bars, re-run math
      bars.push(liveBar);
      if (bars.length > 600) bars.shift();
      liveBar = null;
      runAnalysis();
      requestDraw();
    }
  };

  ws.onerror = () => {};
  ws.onclose = () => { setLiveStatus(false); scheduleReconnect(); };
}

function scheduleReconnect() {
  wsRetry = Math.min(wsRetry + 1, 6);
  setTimeout(connectWS, wsRetry * 3000);
}

function setLiveStatus(live) {
  const dot = document.getElementById('liveDot');
  const lbl = document.getElementById('liveLabel');
  dot.className = live ? 'live' : '';
  lbl.textContent = live ? 'LIVE' : 'RECONNECTING';
}

// ══════════════════════════════════════════════════════
//  ANALYSIS PIPELINE (only on closed bars)
// ══════════════════════════════════════════════════════
function runAnalysis() {
  if (!bars.length) return;
  const closes = bars.map(b => b.close);
  macdData = calcMACD(closes);
  const { swingHigh, swingLow, shIdx, slIdx } = calcSwings(bars);
  if (swingHigh !== null && swingLow !== null) {
    const { levels, extensions } = calcFibLevels(swingHigh, swingLow);
    fibData = { high: swingHigh, low: swingLow, levels, extensions, shIdx, slIdx };
  }
  signals = computeSignals();
  updateSidebar();
  updateSignalStrip();
}

// ══════════════════════════════════════════════════════
//  UI UPDATES
// ══════════════════════════════════════════════════════
function updatePriceBar(bar) {
  if (!bar) return;
  const up = bar.close >= bar.open;
  const pct = bar.open ? ((bar.close - bar.open) / bar.open * 100) : 0;
  const p = document.getElementById('pb-price');
  p.textContent = fp(bar.close);
  p.className   = 'pb-price ' + (up ? 'bull' : 'bear');
  const chg = document.getElementById('pb-chg');
  chg.textContent = `${pct >= 0 ? '+' : ''}${pct.toFixed(2)}%`;
  chg.className   = 'pb-chg ' + (up ? 'bull' : 'bear');
  document.getElementById('pb-o').textContent = fp(bar.open);
  document.getElementById('pb-h').textContent = fp(bar.high);
  document.getElementById('pb-l').textContent = fp(bar.low);
  document.getElementById('pb-v').textContent = fv(bar.volume);

  // MACD values for hovered/last bar
  if (macdData && bars.length) {
    const i = bars.length - 1;
    const m = macdData.macd[i], s = macdData.signal[i], h = macdData.hist[i];
    document.getElementById('pb-macd').textContent = isNaN(m) ? '—' : fp(m, 6);
    document.getElementById('pb-macd').style.color = !isNaN(m) && m >= 0 ? 'var(--bull)' : 'var(--bear)';
    document.getElementById('pb-sig').textContent  = isNaN(s) ? '—' : fp(s, 6);
    document.getElementById('pb-hist').textContent = isNaN(h) ? '—' : fp(h, 6);
    document.getElementById('pb-hist').style.color = !isNaN(h) && h >= 0 ? 'var(--bull)' : 'var(--bear)';
  }
}

function updateSidebar() {
  if (!macdData || !bars.length) return;
  const i = bars.length - 1;
  const { macd, signal, hist } = macdData;
  const m = macd[i], s = signal[i], h = hist[i];
  const mPrev = macd[i-1], hPrev = hist[i-1];

  const c = (v, pos, neg) => isNaN(v) ? '' : v >= 0 ? pos : neg;

  document.getElementById('sb-macd').textContent   = isNaN(m) ? '—' : fp(m, 6);
  document.getElementById('sb-macd').style.color    = c(m, 'var(--bull)', 'var(--bear)');
  document.getElementById('sb-signal').textContent  = isNaN(s) ? '—' : fp(s, 6);
  document.getElementById('sb-hist').textContent    = isNaN(h) ? '—' : fp(h, 6);
  document.getElementById('sb-hist').style.color    = c(h, 'var(--bull)', 'var(--bear)');

  // Momentum: MACD direction vs previous
  if (!isNaN(m) && !isNaN(mPrev)) {
    const mom = m - mPrev;
    document.getElementById('sb-momentum').textContent = (mom >= 0 ? '▲ ' : '▼ ') + fp(mom, 6);
    document.getElementById('sb-momentum').style.color = mom >= 0 ? 'var(--bull)' : 'var(--bear)';
  }

  // Divergence signal in sidebar
  const divSig = signals.find(s => s.label.includes('DIVERGENCE'));
  document.getElementById('sb-div').textContent   = divSig ? divSig.label : 'NONE';
  document.getElementById('sb-div').style.color   = divSig ? (divSig.type === 'bull' ? 'var(--bull)' : 'var(--bear)') : 'var(--text3)';

  // Crossover
  const crossSig = signals.find(s => s.label.includes('CROSSOVER'));
  document.getElementById('sb-cross').textContent = crossSig ? crossSig.label : 'NONE';
  document.getElementById('sb-cross').style.color = crossSig ? (crossSig.type === 'bull' ? 'var(--bull)' : 'var(--bear)') : 'var(--text3)';

  // Zero cross
  const zeroSig = signals.find(s => s.label.includes('ZERO'));
  document.getElementById('sb-zero').textContent  = zeroSig ? zeroSig.label : m >= 0 ? 'ABOVE ZERO' : 'BELOW ZERO';
  document.getElementById('sb-zero').style.color  = !isNaN(m) ? (m >= 0 ? 'var(--bull)' : 'var(--bear)') : 'var(--text3)';

  // Histogram strength
  if (!isNaN(h) && !isNaN(hPrev)) {
    const growing = Math.abs(h) > Math.abs(hPrev);
    document.getElementById('sb-hstrength').textContent = growing ? 'ACCELERATING' : 'DECELERATING';
    document.getElementById('sb-hstrength').style.color = growing ? 'var(--bull)' : 'var(--amber)';
  }

  // Fib sidebar
  if (fibData) {
    document.getElementById('sb-swH').textContent = fp(fibData.high);
    document.getElementById('sb-swL').textContent = fp(fibData.low);
    const price = bars[i].close;
    const range = fibData.high - fibData.low;
    const fibHTML = fibData.levels.map(lvl => {
      const dist = price - lvl.price;
      const pctFill = range > 0 ? clamp((lvl.price - fibData.low) / range * 100, 0, 100) : 50;
      const near    = Math.abs(dist / price) < 0.005;
      const curPct  = range > 0 ? clamp((price - fibData.low) / range * 100, 0, 100) : 50;
      return `<div class="fib-row">
        <span class="fib-pct" style="color:${lvl.color}">${lvl.label}</span>
        <div class="fib-bar-wrap">
          <div class="fib-bar-fill" style="width:${pctFill.toFixed(1)}%;background:${lvl.color};opacity:.5"></div>
          <div class="fib-cur-marker" style="left:${curPct.toFixed(1)}%">
            <div class="fib-badge" style="${near ? 'display:block' : ''}">PRICE</div>
          </div>
        </div>
        <span class="fib-price" style="color:${near ? lvl.color : 'var(--text2)'}">${fp(lvl.price)}</span>
      </div>`;
    }).join('');
    document.getElementById('fibLevels').innerHTML = fibHTML;
  }

  // Signal cards
  const top5 = signals.slice(0, 5);
  const clsMap = { bull: 'sc-bull', bear: 'sc-bear', warn: 'sc-warn', info: 'sc-neu' };
  document.getElementById('sigCards').innerHTML = top5.length
    ? top5.map(s => `<div class="sig-card ${clsMap[s.type]||'sc-neu'}">
        <div class="sig-card-title">${s.label}</div>
        <div class="sig-card-body">${s.detail}</div>
      </div>`).join('')
    : '<div class="sig-card sc-neu"><div class="sig-card-title">NO SIGNALS</div><div class="sig-card-body">Waiting for confirmations…</div></div>';
}

function updateSignalStrip() {
  const chips = signals.slice(0, 6).map(s => {
    const cls = s.type === 'bull' ? 'chip-bull' : s.type === 'bear' ? 'chip-bear' : s.type === 'warn' ? 'chip-amber' : 'chip-neu';
    return `<div class="sig-chip ${cls}"><span class="sig-dot"></span>${s.label}</div>`;
  }).join('');
  document.getElementById('sigChips').innerHTML = chips || '<div class="sig-chip chip-neu"><span class="sig-dot"></span>ANALYZING…</div>';
}

// ══════════════════════════════════════════════════════
//  CANVAS CHART ENGINE
// ══════════════════════════════════════════════════════
const priceCanvas = document.getElementById('priceCanvas');
const macdCanvas  = document.getElementById('macdCanvas');
const ctx  = priceCanvas.getContext('2d');
const mctx = macdCanvas.getContext('2d');

function sizeCanvases() {
  const dpr = window.devicePixelRatio || 1;
  const pp  = document.getElementById('pricePane');
  const mp  = document.getElementById('macdPane');
  [priceCanvas, ctx,  pp].forEach((_, i) => {
    if (i < 2) return;
    priceCanvas.width  = pp.clientWidth  * dpr;
    priceCanvas.height = pp.clientHeight * dpr;
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
  });
  macdCanvas.width  = mp.clientWidth  * dpr;
  macdCanvas.height = mp.clientHeight * dpr;
  mctx.setTransform(dpr, 0, 0, dpr, 0, 0);
}

function W_P() { return priceCanvas.width  / (window.devicePixelRatio||1); }
function H_P() { return priceCanvas.height / (window.devicePixelRatio||1); }
function W_M() { return macdCanvas.width   / (window.devicePixelRatio||1); }
function H_M() { return macdCanvas.height  / (window.devicePixelRatio||1); }
function cW_P(){ return W_P() - PAD.L      - PAD.R; }
function cH_P(){ return H_P() - PAD.T      - PAD.B; }
function cW_M(){ return W_M() - PAD_MACD.L - PAD_MACD.R; }
function cH_M(){ return H_M() - PAD_MACD.T - PAD_MACD.B; }

function allBars() { return liveBar ? [...bars, liveBar] : bars; }

function visibleRange() {
  const all    = allBars();
  const total  = all.length;
  const maxVis = Math.ceil(cW_P() / barW) + 2;
  const endIdx = clamp(total - 1 - Math.floor(scrollBars), 0, total - 1);
  const startIdx = Math.max(0, endIdx - maxVis + 1);
  return { all, startIdx, endIdx };
}

function priceExtent(all, s, e) {
  let lo = Infinity, hi = -Infinity;
  for (let i = s; i <= e; i++) {
    if (all[i].high > hi) hi = all[i].high;
    if (all[i].low  < lo) lo = all[i].low;
  }
  // include fib levels in visible range
  if (fibData) {
    for (const lvl of fibData.levels) { if (lvl.price > hi) hi = lvl.price; if (lvl.price < lo) lo = lvl.price; }
  }
  const pad = (hi - lo) * 0.06 || hi * 0.01 || 1;
  return { lo: lo - pad, hi: hi + pad };
}

function macdExtent(s, e) {
  if (!macdData) return { lo: -1, hi: 1 };
  let lo = 0, hi = 0;
  for (let i = s; i <= e; i++) {
    const m = macdData.macd[i], sg = macdData.signal[i], h = macdData.hist[i];
    if (!isNaN(m))  { if (m  > hi) hi = m;  if (m  < lo) lo = m; }
    if (!isNaN(sg)) { if (sg > hi) hi = sg; if (sg < lo) lo = sg; }
    if (!isNaN(h))  { if (h  > hi) hi = h;  if (h  < lo) lo = h; }
  }
  const pad = Math.max(Math.abs(hi), Math.abs(lo)) * 0.1 || 0.001;
  return { lo: lo - pad, hi: hi + pad };
}

function pxY(p, lo, hi, padT, height) { return padT + (1 - (p - lo) / (hi - lo)) * height; }
function bxP(i, endIdx) { return PAD.L + cW_P() - (endIdx - i) * barW; }
function bxM(i, endIdx) { return PAD_MACD.L + cW_M() - (endIdx - i) * barW; }

// ─── DRAW PRICE PANE ───
function drawPricePane(all, s, e) {
  const W = W_P(), H = H_P(), cw = cW_P(), ch = cH_P();
  ctx.clearRect(0, 0, W, H);
  ctx.fillStyle = '#080c12'; ctx.fillRect(0, 0, W, H);

  const { lo, hi } = priceExtent(all, s, e);
  const py = (p) => pxY(p, lo, hi, PAD.T, ch);
  const bx = (i) => bxP(i, e);

  // Grid
  const gSteps = 6;
  ctx.strokeStyle = '#111822'; ctx.lineWidth = 1; ctx.setLineDash([]);
  for (let g = 0; g <= gSteps; g++) {
    const p = lo + (hi - lo) * g / gSteps;
    const y = py(p);
    ctx.beginPath(); ctx.moveTo(PAD.L, y); ctx.lineTo(W - PAD.R, y); ctx.stroke();
    ctx.fillStyle = '#364a5e'; ctx.font = '9px IBM Plex Mono,monospace';
    ctx.textAlign = 'left';
    ctx.fillText(fp(p), W - PAD.R + 3, y + 3);
  }

  // ── FIB LEVELS ──
  if (fibData) {
    for (const lvl of fibData.levels) {
      const y = py(lvl.price);
      if (y < PAD.T || y > PAD.T + ch) continue;
      // Shaded zone between consecutive levels (only 0.382–0.618 for OTE)
      ctx.strokeStyle = lvl.color + '55'; ctx.lineWidth = 0.8; ctx.setLineDash([4, 4]);
      ctx.beginPath(); ctx.moveTo(PAD.L, y); ctx.lineTo(W - PAD.R, y); ctx.stroke();
      ctx.setLineDash([]);
      ctx.fillStyle = lvl.color + 'aa';
      ctx.font = 'bold 8px IBM Plex Mono,monospace'; ctx.textAlign = 'right';
      ctx.fillText(`${lvl.label}  ${fp(lvl.price)}`, W - PAD.R - 2, y - 2);
    }
    // Highlight golden pocket (0.618 – 0.5 zone)
    const y618 = py(fibData.levels.find(l => l.r === 0.618).price);
    const y500 = py(fibData.levels.find(l => l.r === 0.5).price);
    ctx.fillStyle = 'rgba(38,222,129,0.05)';
    ctx.fillRect(PAD.L, Math.min(y618, y500), cw, Math.abs(y618 - y500));

    // Swing high/low markers
    const shX = bx(fibData.shIdx), slX = bx(fibData.slIdx);
    const shY = py(fibData.high),  slY = py(fibData.low);
    if (shX >= PAD.L && shX <= W - PAD.R) {
      ctx.fillStyle = 'rgba(38,222,129,0.7)';
      ctx.beginPath(); ctx.arc(shX, shY, 3, 0, Math.PI*2); ctx.fill();
      ctx.fillStyle = 'rgba(38,222,129,0.6)'; ctx.font = 'bold 8px IBM Plex Mono,monospace'; ctx.textAlign = 'center';
      ctx.fillText('SH', shX, shY - 7);
    }
    if (slX >= PAD.L && slX <= W - PAD.R) {
      ctx.fillStyle = 'rgba(255,69,96,0.7)';
      ctx.beginPath(); ctx.arc(slX, slY, 3, 0, Math.PI*2); ctx.fill();
      ctx.fillStyle = 'rgba(255,69,96,0.6)'; ctx.font = 'bold 8px IBM Plex Mono,monospace'; ctx.textAlign = 'center';
      ctx.fillText('SL', slX, slY + 13);
    }
  }

  // ── CANDLES ──
  const bw = Math.max(1.5, barW - 2);
  const hw = Math.max(0.75, bw / 2);
  for (let i = s; i <= e; i++) {
    const bar = all[i];
    const x   = bx(i);
    if (x < PAD.L - barW || x > W - PAD.R + barW) continue;
    const oY = py(bar.open), cY = py(bar.close);
    const hY = py(bar.high), lY = py(bar.low);
    const up = bar.close >= bar.open;
    const col = up ? '#26de81' : '#ff4560';
    const bodyT = Math.min(oY, cY), bodyH = Math.max(1, Math.abs(cY - oY));

    ctx.strokeStyle = col + '88'; ctx.lineWidth = 1; ctx.setLineDash([]);
    ctx.beginPath(); ctx.moveTo(x, hY); ctx.lineTo(x, bodyT); ctx.stroke();
    ctx.beginPath(); ctx.moveTo(x, bodyT + bodyH); ctx.lineTo(x, lY); ctx.stroke();

    if (up) {
      if (bw > 3) { ctx.fillStyle = col + '1a'; ctx.fillRect(x - hw, bodyT, bw, bodyH); }
      ctx.strokeStyle = col; ctx.lineWidth = 1;
      ctx.strokeRect(x - hw, bodyT, bw, bodyH);
    } else {
      ctx.fillStyle = col + 'cc'; ctx.fillRect(x - hw, bodyT, bw, bodyH);
    }

    // Hover crosshair vertical
    if (i === hoverIdx) {
      ctx.strokeStyle = 'rgba(255,255,255,0.1)'; ctx.setLineDash([3, 3]);
      ctx.beginPath(); ctx.moveTo(x, PAD.T); ctx.lineTo(x, PAD.T + ch); ctx.stroke();
      ctx.setLineDash([]);
    }
  }

  // ── CURRENT PRICE LINE ──
  const lastBar = all[all.length - 1];
  if (lastBar) {
    const lp  = lastBar.close;
    const lpY = py(lp);
    const up  = lastBar.close >= lastBar.open;
    ctx.strokeStyle = (up ? '#26de81' : '#ff4560') + '55'; ctx.lineWidth = 1; ctx.setLineDash([3, 3]);
    ctx.beginPath(); ctx.moveTo(PAD.L, lpY); ctx.lineTo(W - PAD.R, lpY); ctx.stroke();
    ctx.setLineDash([]);
    ctx.fillStyle = up ? '#26de81' : '#ff4560';
    ctx.fillRect(W - PAD.R, lpY - 8, PAD.R - 1, 16);
    ctx.fillStyle = '#080c12'; ctx.font = 'bold 9px IBM Plex Mono,monospace'; ctx.textAlign = 'center';
    ctx.fillText(fp(lp), W - PAD.R + (PAD.R - 1) / 2, lpY + 3);
  }

  // ── TIME AXIS ──
  ctx.fillStyle = '#364a5e'; ctx.font = '8px IBM Plex Mono,monospace'; ctx.textAlign = 'center';
  const tStep = Math.max(1, Math.round(80 / barW));
  for (let i = s; i <= e; i += tStep) {
    const x = bx(i);
    if (x < PAD.L + 20 || x > W - PAD.R - 20) continue;
    ctx.fillText(fmtTime(all[i].time, tf), x, H - PAD.B + 12);
  }

  // Cross price label
  if (hoverIdx >= 0 && hoverIdx < all.length) {
    const hBar = all[hoverIdx];
    const hY   = py(hBar.close);
    const lbl  = document.getElementById('crossPriceLbl');
    lbl.style.display = 'block'; lbl.style.top = (hY - 8) + 'px';
    lbl.textContent   = fp(hBar.close);
    ctx.strokeStyle = 'rgba(255,255,255,0.08)'; ctx.lineWidth = 1; ctx.setLineDash([3, 3]);
    ctx.beginPath(); ctx.moveTo(PAD.L, hY); ctx.lineTo(W - PAD.R, hY); ctx.stroke();
    ctx.setLineDash([]);
    // update pb on hover
    const t = document.getElementById('crossTimeLbl');
    t.style.display = 'block'; t.style.left = bx(hoverIdx) + 'px';
    t.textContent   = fmtTime(hBar.time, tf);
  } else {
    document.getElementById('crossPriceLbl').style.display = 'none';
    document.getElementById('crossTimeLbl').style.display  = 'none';
  }
}

// ─── DRAW MACD PANE ───
function drawMacdPane(all, s, e) {
  if (!macdData) return;
  const W = W_M(), H = H_M(), cw = cW_M(), ch = cH_M();
  mctx.clearRect(0, 0, W, H);
  mctx.fillStyle = '#080c12'; mctx.fillRect(0, 0, W, H);

  const { lo, hi } = macdExtent(s, e);
  const py = (v) => pxY(v, lo, hi, PAD_MACD.T, ch);
  const bx = (i) => bxM(i, e);

  // Grid
  mctx.strokeStyle = '#111822'; mctx.lineWidth = 1; mctx.setLineDash([]);
  [-1, 0, 1].forEach(g => {
    const v = lo + (hi - lo) * (g + 1) / 2;
    const y = py(v);
    if (y < PAD_MACD.T || y > PAD_MACD.T + ch) return;
    mctx.beginPath(); mctx.moveTo(PAD_MACD.L, y); mctx.lineTo(W - PAD_MACD.R, y); mctx.stroke();
    mctx.fillStyle = '#364a5e'; mctx.font = '8px IBM Plex Mono,monospace'; mctx.textAlign = 'left';
    mctx.fillText(fp(v, 5), W - PAD_MACD.R + 2, y + 3);
  });

  // Zero line
  const zeroY = py(0);
  mctx.strokeStyle = '#243344'; mctx.lineWidth = 1.5;
  mctx.beginPath(); mctx.moveTo(PAD_MACD.L, zeroY); mctx.lineTo(W - PAD_MACD.R, zeroY); mctx.stroke();

  // ── HISTOGRAM ──
  const bw = Math.max(1.5, barW - 2);
  const hw = Math.max(0.75, bw / 2);
  for (let i = s; i <= e; i++) {
    const h = macdData.hist[i];
    if (isNaN(h)) continue;
    const x    = bx(i);
    const y0   = zeroY;
    const y1   = py(h);
    const hPrev = macdData.hist[i - 1];
    // Color: bull expanding=bright, bull contracting=dim; same for bear
    const expanding = !isNaN(hPrev) ? Math.abs(h) > Math.abs(hPrev) : true;
    const col = h >= 0
      ? (expanding ? '#26de81dd' : '#26de8166')
      : (expanding ? '#ff4560dd' : '#ff456066');
    mctx.fillStyle = col;
    mctx.fillRect(x - hw, Math.min(y0, y1), bw, Math.abs(y1 - y0) || 1);
  }

  // ── MACD LINE ──
  mctx.beginPath(); mctx.strokeStyle = '#4b91f7'; mctx.lineWidth = 1.5; mctx.setLineDash([]);
  let firstMacd = true;
  for (let i = s; i <= e; i++) {
    const m = macdData.macd[i];
    if (isNaN(m)) { firstMacd = true; continue; }
    const x = bx(i), y = py(m);
    firstMacd ? mctx.moveTo(x, y) : mctx.lineTo(x, y);
    firstMacd = false;
  }
  mctx.stroke();

  // ── SIGNAL LINE ──
  mctx.beginPath(); mctx.strokeStyle = '#f5a623'; mctx.lineWidth = 1.2; mctx.setLineDash([]);
  let firstSig = true;
  for (let i = s; i <= e; i++) {
    const sg = macdData.signal[i];
    if (isNaN(sg)) { firstSig = true; continue; }
    const x = bx(i), y = py(sg);
    firstSig ? mctx.moveTo(x, y) : mctx.lineTo(x, y);
    firstSig = false;
  }
  mctx.stroke();

  // ── CROSSOVER ARROWS ──
  for (let i = s + 1; i <= e; i++) {
    const m = macdData.macd[i], mP = macdData.macd[i-1];
    const sg = macdData.signal[i], sgP = macdData.signal[i-1];
    if (isNaN(m)||isNaN(mP)||isNaN(sg)||isNaN(sgP)) continue;
    const crossedUp   = mP < sgP && m >= sg;
    const crossedDown = mP > sgP && m <= sg;
    if (!crossedUp && !crossedDown) continue;
    const x  = bx(i);
    const yS = py(sg);
    const col = crossedUp ? '#26de81' : '#ff4560';
    const ad  = crossedUp ? -1 : 1;
    mctx.fillStyle = col;
    mctx.beginPath();
    mctx.moveTo(x,     yS + ad * 8);
    mctx.lineTo(x - 4, yS + ad * 15);
    mctx.lineTo(x + 4, yS + ad * 15);
    mctx.closePath(); mctx.fill();
  }

  // MACD hover label
  if (hoverIdx >= 0 && hoverIdx < all.length) {
    const m = macdData.macd[hoverIdx];
    if (!isNaN(m)) {
      const lbl = document.getElementById('crossMacdLbl');
      lbl.style.display = 'block'; lbl.style.top = (py(m) - 8) + 'px';
      lbl.textContent = fp(m, 6);
    }
    // vertical crosshair in MACD pane
    const x = bx(hoverIdx);
    mctx.strokeStyle = 'rgba(255,255,255,0.08)'; mctx.setLineDash([3,3]);
    mctx.beginPath(); mctx.moveTo(x, PAD_MACD.T); mctx.lineTo(x, PAD_MACD.T+ch); mctx.stroke();
    mctx.setLineDash([]);
  } else {
    document.getElementById('crossMacdLbl').style.display = 'none';
  }

  // MACD value labels in pane (top right)
  if (all.length && macdData) {
    const li = all.length - 1;
    const m = macdData.macd[li], sg = macdData.signal[li], h = macdData.hist[li];
    mctx.font = '9px IBM Plex Mono,monospace';
    mctx.textAlign = 'right';
    if (!isNaN(m))  { mctx.fillStyle = '#4b91f7'; mctx.fillText('M:'+fp(m,5), W-PAD_MACD.R-2, PAD_MACD.T+10); }
    if (!isNaN(sg)) { mctx.fillStyle = '#f5a623'; mctx.fillText('S:'+fp(sg,5), W-PAD_MACD.R-2, PAD_MACD.T+20); }
    if (!isNaN(h))  { mctx.fillStyle = h>=0?'#26de81':'#ff4560'; mctx.fillText('H:'+fp(h,5), W-PAD_MACD.R-2, PAD_MACD.T+30); }
  }
}

// Master draw
function draw() {
  if (!bars.length) return;
  const { all, startIdx, endIdx } = visibleRange();
  drawPricePane(all, startIdx, endIdx);
  drawMacdPane(all, startIdx, endIdx);
}

function requestDraw() {
  if (animId) cancelAnimationFrame(animId);
  animId = requestAnimationFrame(draw);
}

// ══════════════════════════════════════════════════════
//  INTERACTION
// ══════════════════════════════════════════════════════
function xToIdx(clientX, canvas) {
  const rect  = canvas.getBoundingClientRect();
  const x     = clientX - rect.left;
  const { endIdx } = visibleRange();
  return Math.round(endIdx - (rect.width - PAD.R - x) / barW);
}

// Mouse
priceCanvas.addEventListener('mousemove', e => {
  const idx = xToIdx(e.clientX, priceCanvas);
  hoverIdx = idx;
  if (isDragging) {
    const dx = e.clientX - dragX0;
    scrollBars = clamp(dragScroll0 - dx / barW, 0, allBars().length - 5);
  }
  requestDraw();
});
priceCanvas.addEventListener('mousedown', e => { isDragging = true; dragX0 = e.clientX; dragScroll0 = scrollBars; priceCanvas.style.cursor = 'grabbing'; });
priceCanvas.addEventListener('mouseup',   () => { isDragging = false; priceCanvas.style.cursor = 'crosshair'; });
priceCanvas.addEventListener('mouseleave',() => { isDragging = false; hoverIdx = -1; requestDraw(); });
priceCanvas.addEventListener('wheel', e => {
  e.preventDefault();
  if (e.ctrlKey || e.metaKey) {
    barW = clamp(barW * (e.deltaY < 0 ? 1.12 : 0.9), 2, 28);
  } else {
    scrollBars = clamp(scrollBars + e.deltaY * 0.06, 0, allBars().length - 5);
  }
  requestDraw();
}, { passive: false });

// Touch
let _tx0 = 0, _ts0 = 0, _pin0 = 0;
priceCanvas.addEventListener('touchstart', e => {
  e.preventDefault();
  if (e.touches.length === 1) { _tx0 = e.touches[0].clientX; _ts0 = scrollBars; hoverIdx = xToIdx(e.touches[0].clientX, priceCanvas); }
  else if (e.touches.length === 2) { _pin0 = Math.hypot(e.touches[0].clientX - e.touches[1].clientX, e.touches[0].clientY - e.touches[1].clientY); }
  requestDraw();
}, { passive: false });
priceCanvas.addEventListener('touchmove', e => {
  e.preventDefault();
  if (e.touches.length === 1) {
    const dx = e.touches[0].clientX - _tx0;
    scrollBars = clamp(_ts0 - dx / barW, 0, allBars().length - 5);
    hoverIdx   = xToIdx(e.touches[0].clientX, priceCanvas);
  } else if (e.touches.length === 2) {
    const d  = Math.hypot(e.touches[0].clientX - e.touches[1].clientX, e.touches[0].clientY - e.touches[1].clientY);
    barW = clamp(barW * (d / _pin0), 2, 28); _pin0 = d;
  }
  requestDraw();
}, { passive: false });
priceCanvas.addEventListener('touchend', () => { hoverIdx = -1; requestDraw(); }, { passive: false });

// ══════════════════════════════════════════════════════
//  SWITCHING
// ══════════════════════════════════════════════════════
function switchSym(el) {
  if (el.dataset.sym === sym) return;
  document.querySelectorAll('.itab').forEach(t => t.classList.remove('on'));
  el.classList.add('on');
  sym = el.dataset.sym;
  dec = +el.dataset.dec;
  boot();
}

function switchTF(el) {
  if (el.dataset.tf === tf) return;
  document.querySelectorAll('.tftab').forEach(t => t.classList.remove('on'));
  el.classList.add('on');
  tf = el.dataset.tf;
  boot();
}

// ══════════════════════════════════════════════════════
//  BOOT
// ══════════════════════════════════════════════════════
function showLoad(msg) {
  document.getElementById('ldMsg').textContent  = msg;
  document.getElementById('loadScreen').style.display = 'flex';
  document.getElementById('errBox').style.display     = 'none';
  document.getElementById('retryBtn').style.display   = 'none';
}

function showErr(msg) {
  document.getElementById('ldMsg').style.display    = 'none';
  document.getElementById('errBox').style.display   = 'block';
  document.getElementById('errBox').textContent     = msg;
  document.getElementById('retryBtn').style.display = 'block';
  document.querySelector('.spinner').style.display  = 'none';
}

async function boot() {
  bars = []; macdData = null; fibData = null; signals = []; liveBar = null;
  scrollBars = 0; hoverIdx = -1;
  if (ws) { try { ws.close(); } catch(e) {} ws = null; }
  setLiveStatus(false);
  document.querySelector('.spinner').style.display = 'block';
  document.getElementById('ldMsg').style.display   = 'block';
  showLoad(`FETCHING ${sym} ${tf.toUpperCase()}…`);

  try {
    const raw = await fetchKlines(sym, tf, 500);
    if (!raw.length) throw new Error('No candles returned');
    bars = raw.slice(0, -1); // confirmed closed only — exclude forming
    runAnalysis();
    updatePriceBar(bars[bars.length - 1]);

    // Set bar width to show ~80 bars initially
    barW = clamp(Math.floor(cW_P() / 80), 3, 14);
    requestDraw();
    document.getElementById('loadScreen').style.display = 'none';
    connectWS();
  } catch(e) {
    showErr(`Connection failed: ${e.message}\n\nCheck your network or try again.`);
  }
}

// Resize
let rTimer;
window.addEventListener('resize', () => {
  clearTimeout(rTimer);
  rTimer = setTimeout(() => { sizeCanvases(); requestDraw(); }, 80);
});

// Init
sizeCanvases();
boot();
</script>
</body>
</html>
