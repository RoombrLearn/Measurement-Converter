<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Metric / Imperial Measurement Converter</title>
  <style>
    :root{ --bg:#f6fbff; --card:#ffffff; --accent:#0b7285; --muted:#6b7280; --radius:12px; }
    *{box-sizing:border-box;font-family:Inter, system-ui, -apple-system, 'Segoe UI', Roboto, 'Helvetica Neue', Arial}
    body{margin:0;background:linear-gradient(180deg,var(--bg),#f0f6fb);min-height:100vh;display:flex;align-items:center;justify-content:center;padding:28px}
    .card{width:100%;max-width:820px;background:var(--card);border-radius:var(--radius);box-shadow:0 10px 30px rgba(11,40,60,0.08);padding:22px}
    h1{margin:0 0 8px;font-size:20px}
    p.lead{margin:0 0 18px;color:var(--muted);font-size:13px}
    .grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
    .full{grid-column:1/-1}
    label{display:block;font-size:12px;color:var(--muted);margin-bottom:6px}
    input[type="number"], select{width:100%;padding:10px 12px;border-radius:8px;border:1px solid #e6eef6;font-size:15px}
    .results{display:grid;grid-template-columns:repeat(2,1fr);gap:10px;margin-top:12px}
    .result{background:#fbfdff;padding:12px;border-radius:10px;border:1px solid #eef6fb}
    .result .value{font-weight:700;font-size:16px}
    .result .unit{font-size:12px;color:var(--muted)}
    .controls{display:flex;gap:8px;align-items:center;margin-top:12px}
    button{background:var(--accent);color:white;border:0;padding:10px 12px;border-radius:10px;font-weight:600;cursor:pointer}
    button.secondary{background:transparent;color:var(--accent);border:1px solid #dff0f3}
    .footer{margin-top:14px;font-size:12px;color:var(--muted)}
    @media (max-width:640px){.grid{grid-template-columns:1fr}.results{grid-template-columns:1fr 1fr}}
  </style>
</head>
<body>
  <div class="card" role="application">
    <h1>Measurement Converter</h1>
    <p class="lead">Convert quickly between <strong>mm, cm, m, in (inches)</strong> and <strong>ft (feet)</strong>. Values update live. Default precision: 2 decimals.</p>

    <div class="grid">
      <div>
        <label for="value">Enter value</label>
        <input id="value" type="number" step="any" inputmode="decimal" placeholder="e.g. 2500" value="1">
      </div>

      <div>
        <label for="unit">From unit</label>
        <select id="unit">
          <option value="mm">Millimeter (mm)</option>
          <option value="cm">Centimeter (cm)</option>
          <option value="m">Meter (m)</option>
          <option value="in">Inch (in)</option>
          <option value="ft">Foot (ft)</option>
        </select>
      </div>

      <div class="full">
        <label for="precision">Decimal places</label>
        <select id="precision">
          <option value="0">0</option>
          <option value="1">1</option>
          <option value="2" selected>2</option>
          <option value="3">3</option>
          <option value="4">4</option>
        </select>
      </div>

      <div class="full controls">
        <button id="convert">Convert</button>
        <button id="swap" class="secondary">Swap unit example (not input)</button>
        <button id="clear" class="secondary">Clear</button>
      </div>

      <div class="full results" aria-live="polite">
        <div class="result">
          <div class="value" id="res-mm">—</div>
          <div class="unit">Millimetres (mm)</div>
        </div>

        <div class="result">
          <div class="value" id="res-cm">—</div>
          <div class="unit">Centimetres (cm)</div>
        </div>

        <div class="result">
          <div class="value" id="res-m">—</div>
          <div class="unit">Metres (m)</div>
        </div>

        <div class="result">
          <div class="value" id="res-in">—</div>
          <div class="unit">Inches (in)</div>
        </div>

        <div class="result">
          <div class="value" id="res-ft">—</div>
          <div class="unit">Feet (ft)</div>
        </div>
      </div>

    </div>

    <div class="footer">Tip: type a number and press Enter or click Convert. Use the precision dropdown to change decimal places.</div>
  </div>

  <script>
    // Conversion constants
    const MM_PER_M = 1000;
    const CM_PER_M = 100;
    const IN_PER_MM = 1 / 25.4; // inches per mm
    const FT_PER_IN = 1 / 12;

    const els = {
      value: document.getElementById('value'),
      unit: document.getElementById('unit'),
      precision: document.getElementById('precision'),
      res: {
        mm: document.getElementById('res-mm'),
        cm: document.getElementById('res-cm'),
        m: document.getElementById('res-m'),
        in: document.getElementById('res-in'),
        ft: document.getElementById('res-ft')
      },
      convertBtn: document.getElementById('convert'),
      clearBtn: document.getElementById('clear'),
      swapBtn: document.getElementById('swap')
    };

    function toMeters(value, unit){
      const v = Number(value) || 0;
      switch(unit){
        case 'mm': return v / MM_PER_M;
        case 'cm': return v / CM_PER_M;
        case 'm':  return v;
        case 'in': return (v * 25.4) / MM_PER_M; // inches -> mm -> m
        case 'ft': return (v * 12 * 25.4) / MM_PER_M; // ft -> in -> mm -> m
        default: return 0;
      }
    }

    function fromMeters(meters){
      return {
        mm: meters * MM_PER_M,
        cm: meters * CM_PER_M,
        m: meters,
        in: (meters * MM_PER_M) * IN_PER_MM,
        ft: (meters * MM_PER_M) * IN_PER_MM * FT_PER_IN
      };
    }

    function fmt(num, decimals){
      if (!isFinite(num)) return '—';
      return Number(num).toLocaleString(undefined, {minimumFractionDigits: decimals, maximumFractionDigits: decimals});
    }

    function convert(){
      const val = els.value.value.trim();
      if (val === ''){
        setResultsEmpty();
        return;
      }
      const unit = els.unit.value;
      const prec = Math.max(0, Math.min(10, Number(els.precision.value) || 2));

      const meters = toMeters(val, unit);
      const out = fromMeters(meters);

      els.res.mm.textContent = fmt(out.mm, prec) + ' mm';
      els.res.cm.textContent = fmt(out.cm, prec) + ' cm';
      els.res.m.textContent  = fmt(out.m, prec)  + ' m';
      els.res.in.textContent = fmt(out.in, prec) + ' in';
      els.res.ft.textContent = fmt(out.ft, prec) + ' ft';
    }

    function setResultsEmpty(){
      Object.values(els.res).forEach(el => el.textContent = '—');
    }

    // Event listeners
    els.convertBtn.addEventListener('click', convert);
    els.clearBtn.addEventListener('click', ()=>{ els.value.value=''; setResultsEmpty(); els.value.focus(); });
    els.swapBtn.addEventListener('click', ()=>{
      // quick example: cycle the "From unit" to next one so user can try differences quickly
      const options = Array.from(els.unit.options).map(o=>o.value);
      const currentIndex = options.indexOf(els.unit.value);
      const next = options[(currentIndex + 1) % options.length];
      els.unit.value = next;
      convert();
    });

    // allow pressing Enter to convert
    els.value.addEventListener('keydown', (e)=>{ if (e.key === 'Enter') convert(); });

    // live convert when precision or unit changes
    els.precision.addEventListener('change', convert);
    els.unit.addEventListener('change', convert);

    // initial conversion
    convert();
  </script>
</body>
</html>
