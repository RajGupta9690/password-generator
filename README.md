<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Password Generator</title>
  <style>
    :root{
      --bg:#071022; --card:#0f1724; --accent:#4f46e5;
      --muted:#9ca3af; --text:#e6eef8;
    }
    *{box-sizing:border-box}
    body{
      margin:0; min-height:100vh;
      font-family:Inter,system-ui,Roboto,Arial; background:linear-gradient(180deg,#071028,#07112a);
      color:var(--text); display:flex; align-items:center; justify-content:center; padding:24px;
    }
    .card{width:100%; max-width:720px; background:linear-gradient(180deg, rgba(255,255,255,0.03), rgba(255,255,255,0.02));
      padding:20px; border-radius:12px; box-shadow:0 8px 30px rgba(2,6,23,0.6); border:1px solid rgba(255,255,255,0.03);}
    h1{margin:0 0 12px;font-size:20px}
    .row{display:flex; gap:12px; align-items:center; margin-bottom:12px;}
    .col{flex:1}
    label{display:block; font-size:14px; color:var(--muted); margin-bottom:6px}
    input[type=range]{width:100%}
    .checks{display:flex; gap:12px; flex-wrap:wrap}
    .checks label{font-size:14px; color:var(--muted); display:flex; gap:8px; align-items:center}
    .actions{display:flex; gap:8px; flex-wrap:wrap}
    button{background:transparent;border:1px solid rgba(255,255,255,0.06);padding:8px 12px;border-radius:8px;color:var(--text);cursor:pointer}
    .btn-primary{background:var(--accent); border-color:transparent}
    .btn-icon{width:44px; display:flex; align-items:center; justify-content:center; font-size:18px}
    .output{min-height:56px;background:rgba(255,255,255,0.02);border-radius:8px;padding:10px;font-family:ui-monospace, SFMono-Regular, Menlo, Monaco, \"Roboto Mono\", monospace;word-break:break-all; display:flex; align-items:center; color:var(--text)}
    .muted{color:var(--muted)}
    .meter{height:10px;border-radius:6px;overflow:hidden;background:rgba(255,255,255,0.04);margin-top:8px}
    .meter > i{display:block;height:100%; width:0%; background:linear-gradient(90deg,#ef4444,#f59e0b,#10b981)}
    .hint{font-size:12px;color:var(--muted); margin-top:10px}
    @media (max-width:520px){ .row{flex-direction:column} .actions{flex-direction:column} }
  </style>
</head>
<body>
  <div class="card" role="application" aria-label="Password Generator">
    <h1>Password Generator</h1>

    <div class="row">
      <div class="col">
        <label for="length">Length: <span id="lenVal">12</span></label>
        <input id="length" type="range" min="4" max="64" value="12" />
      </div>
      <div style="width:160px; display:flex; flex-direction:column; gap:6px;">
        <label style="font-size:13px; color:var(--muted)">Entropy: <span id="entropyLabel">—</span></label>
        <div class="meter" aria-hidden="true"><i id="meterBar"></i></div>
      </div>
    </div>

    <div class="checks" style="margin-bottom:12px">
      <label><input id="upper" type="checkbox" checked /> Upper</label>
      <label><input id="lower" type="checkbox" checked /> Lower</label>
      <label><input id="numbers" type="checkbox" checked /> Numbers</label>
      <label><input id="symbols" type="checkbox" checked /> Symbols</label>
    </div>

    <div class="actions" style="margin-bottom:12px">
      <button id="generate" class="btn-primary">Generate</button>
      <button id="regen" class="btn-icon" title="Regenerate">🔄</button>
      <button id="copy">Copy</button>
      <button id="download">Download</button>
    </div>

    <div id="output" class="output" title="Double-click to copy" tabindex="0">
      <span class="muted">Your generated password will appear here</span>
    </div>

    <p class="hint">Double-click the password or press <strong>R</strong> to regenerate. Click Copy to copy to clipboard.</p>
  </div>

  <script>
    (function(){
      const lengthEl = document.getElementById('length');
      const lenVal = document.getElementById('lenVal');
      const upperEl = document.getElementById('upper');
      const lowerEl = document.getElementById('lower');
      const numbersEl = document.getElementById('numbers');
      const symbolsEl = document.getElementById('symbols');
      const outputEl = document.getElementById('output');
      const generateBtn = document.getElementById('generate');
      const regenBtn = document.getElementById('regen');
      const copyBtn = document.getElementById('copy');
      const downloadBtn = document.getElementById('download');
      const entropyLabel = document.getElementById('entropyLabel');
      const meterBar = document.getElementById('meterBar');

      const SYMBOLS = '!@#$%^&*()_+{}[]<>,.?/~`-=';

      function poolSize() {
        let size = 0;
        if (upperEl.checked) size += 26;
        if (lowerEl.checked) size += 26;
        if (numbersEl.checked) size += 10;
        if (symbolsEl.checked) size += SYMBOLS.length;
        return size;
      }

      function estimateEntropy(length, size) {
        if (!size) return 0;
        // entropy bits = length * log2(size)
        return length * Math.log2(size);
      }

      function strengthFromEntropy(bits) {
        if (bits < 28) return {label:'Very Weak', pct:20};
        if (bits < 36) return {label:'Weak', pct:40};
        if (bits < 60) return {label:'Medium', pct:65};
        if (bits < 90) return {label:'Strong', pct:85};
        return {label:'Very Strong', pct:100};
      }

      function updateEntropyUI(){
        const size = poolSize();
        const len = Number(lengthEl.value);
        const bits = estimateEntropy(len, size);
        const s = strengthFromEntropy(bits);
        entropyLabel.textContent = `${Math.round(bits)} bits — ${s.label}`;
        meterBar.style.width = s.pct + '%';
      }

      function shuffleString(str){
        const arr = str.split('');
        for (let i = arr.length -1; i > 0; i--){
          const j = Math.floor(Math.random()*(i+1));
          [arr[i], arr[j]] = [arr[j], arr[i]];
        }
        return arr.join('');
      }

      function generatePassword(){
        const len = Number(lengthEl.value);
        let chars = '';
        const pools = [];
        if (upperEl.checked) { chars += 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'; pools.push('ABCDEFGHIJKLMNOPQRSTUVWXYZ'); }
        if (lowerEl.checked) { chars += 'abcdefghijklmnopqrstuvwxyz'; pools.push('abcdefghijklmnopqrstuvwxyz'); }
        if (numbersEl.checked) { chars += '0123456789'; pools.push('0123456789'); }
        if (symbolsEl.checked) { chars += SYMBOLS; pools.push(SYMBOLS); }

        if (!chars) {
          outputEl.textContent = '';
          const notice = document.createElement('span');
          notice.className = 'muted';
          notice.textContent = 'Select at least one character type.';
          outputEl.appendChild(notice);
          return;
        }

        let pass = '';
        // ensure at least one from each selected pool (if length allows)
        for (let i = 0; i < pools.length && pass.length < len; i++){
          const pool = pools[i];
          pass += pool[Math.floor(Math.random()*pool.length)];
        }
        // fill remaining
        for (let i = pass.length; i < len; i++){
          pass += chars[Math.floor(Math.random()*chars.length)];
        }
        pass = shuffleString(pass);

        outputEl.textContent = pass;
      }

      function copyToClipboard(){
        const txt = outputEl.textContent.trim();
        if (!txt) return;
        navigator.clipboard.writeText(txt).then(() => {
          copyBtn.textContent = 'Copied';
          setTimeout(()=>copyBtn.textContent='Copy', 1500);
        }).catch(()=> {
          // fallback
          const ta = document.createElement('textarea');
          ta.value = txt; document.body.appendChild(ta); ta.select();
          try { document.execCommand('copy'); copyBtn.textContent='Copied'; setTimeout(()=>copyBtn.textContent='Copy',1500); } catch(e){}
          ta.remove();
        });
      }

      function downloadTxt(){
        const txt = outputEl.textContent.trim();
        if (!txt) return;
        const blob = new Blob([txt], {type:'text/plain'});
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url; a.download = 'password.txt';
        document.body.appendChild(a); a.click(); a.remove();
        URL.revokeObjectURL(url);
      }

      // events
      lengthEl.addEventListener('input', ()=> {
        lenVal.textContent = lengthEl.value;
        updateEntropyUI();
      });
      [upperEl, lowerEl, numbersEl, symbolsEl].forEach(el => el.addEventListener('change', updateEntropyUI));

      generateBtn.addEventListener('click', generatePassword);
      regenBtn.addEventListener('click', generatePassword);
      copyBtn.addEventListener('click', copyToClipboard);
      downloadBtn.addEventListener('click', downloadTxt);

      outputEl.addEventListener('dblclick', copyToClipboard);
      // keyboard shortcut R (regenerate)
      document.addEventListener('keydown', (e) => {
        if (e.key === 'r' || e.key === 'R') {
          // avoid interfering with input fields
          const tag = document.activeElement.tagName;
          if (tag === 'INPUT' || tag === 'TEXTAREA') return;
          generatePassword();
        }
      });

      // initial UI update
      lenVal.textContent = lengthEl.value;
      updateEntropyUI();
    })();
  </script>
</body>
</html>
