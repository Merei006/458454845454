<!DOCTYPE html>
<html lang="kk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Файлдар және қапшықтар — ойын</title>
<style>
  :root{
    --bg:#fdf1dd; --card:#ffffff; --text:#2b2a28; --muted:#7a756c;
    --purple:#7c5cff; --teal:#0d9488; --red:#e0473e; --good:#16a34a; --bad:#dc2626;
    --border:#f0d9b5; --shadow:0 6px 20px rgba(43,42,40,.08);
    box-sizing:border-box;
    padding-top:env(safe-area-inset-top,0px);
    padding-bottom:env(safe-area-inset-bottom,0px);
  }
  *{box-sizing:border-box;}
  body{margin:0;background:var(--bg);color:var(--text);font-family:Georgia, 'Times New Roman', serif;}
  .wrap{max-width:780px;margin:0 auto;padding:32px 18px 60px;}
  header{text-align:center;margin-bottom:20px;}
  header h1{font-size:clamp(24px,5.5vw,36px);margin:0 0 6px;}
  header p{color:var(--muted);margin:0;font-size:16px;}
  .score{
    text-align:center;background:var(--card);border:2px solid var(--border);border-radius:18px;
    padding:14px;margin-bottom:22px;font-size:18px;box-shadow:var(--shadow);
  }
  .score b{color:var(--purple);font-size:22px;}
  section.card{background:var(--card);border-radius:24px;padding:30px 24px;box-shadow:var(--shadow);margin-bottom:24px;}
  section.card h2{margin:0 0 4px;font-size:24px;text-align:center;}
  section.card .hint{color:var(--muted);font-size:15px;margin:0 0 20px;text-align:center;}

  /* Part 1: sort game */
  .item-box{text-align:center;margin-bottom:18px;}
  .item-box .e{font-size:84px;}
  .item-box p{font-size:22px;font-weight:700;margin:4px 0 0;}
  .choice-row{display:flex;gap:16px;}
  .choicebtn{
    flex:1;background:#ffffff;border:3px solid var(--border);border-radius:16px;padding:18px 10px;
    font-family:inherit;font-size:20px;font-weight:700;cursor:pointer;display:flex;flex-direction:column;
    align-items:center;gap:6px;transition:.15s;
  }
  .choicebtn .e{font-size:40px;}
  .choicebtn:hover{border-color:var(--purple);}
  .choicebtn.correct{border-color:var(--good);background:#eafbf1;}
  .choicebtn.wrong{border-color:var(--bad);background:#fdecea;}
  .choicebtn:disabled{cursor:default;}

  /* Part 2: matching */
  .match-grid{display:grid;grid-template-columns:1fr 1fr;gap:10px 16px;}
  @media (max-width:480px){.match-grid{grid-template-columns:1fr;}}
  .chip{
    display:flex;align-items:center;gap:10px;width:100%;padding:12px 14px;margin-bottom:8px;
    border:2px solid var(--border);border-radius:12px;background:var(--bg);
    color:var(--text);font-size:16px;cursor:pointer;transition:.15s;font-family:inherit;
  }
  .chip .e{font-size:26px;}
  .chip:hover{border-color:var(--purple);}
  .chip.selected{border-color:var(--purple);background:#f3ecff;}
  .chip.correct{border-color:var(--good);background:#eafbf1;cursor:default;opacity:.85;}
  .chip.wrong{border-color:var(--bad);animation:shake .3s;}
  @keyframes shake{0%,100%{transform:translateX(0);}25%{transform:translateX(-4px);}75%{transform:translateX(4px);}}

  .feedback{text-align:center;font-size:18px;font-weight:700;min-height:26px;margin-top:14px;}
  .feedback.ok{color:var(--good);}
  .feedback.no{color:var(--bad);}
  .next{
    display:block;margin:16px auto 0;padding:11px 26px;border:none;border-radius:999px;
    background:var(--purple);color:#fff;font-size:16px;font-weight:700;cursor:pointer;
  }
  .next:disabled{opacity:.4;cursor:default;}
  .reset{
    display:block;margin:14px auto 0;border:1px solid var(--border);background:transparent;color:var(--text);
    padding:7px 16px;border-radius:8px;font-size:13px;cursor:pointer;
  }
  .done{text-align:center;}
  .done .big{font-size:72px;}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>🗂️ Файлдар және қапшықтар</h1>
    <p>Екі бөлімнен тұратын ойын-жаттығу</p>
  </header>
  <div class="score">Жалпы ұпай: <b id="total-score">0</b> / <span id="total-max">12</span></div>

  <section class="card">
    <h2>1-бөлім. Файл ма, қапшық па?</h2>
    <p class="hint">Әр затты дұрыс топқа жатқыз</p>
    <div id="sortGame"></div>
  </section>

  <section class="card">
    <h2>2-бөлім. Әрекетті сәйкестендір</h2>
    <p class="hint">Сол жақтан әрекетті таңда, оң жақтан мағынасын тап</p>
    <div class="match-grid">
      <div id="col-left"></div>
      <div id="col-right"></div>
    </div>
    <div style="text-align:center;font-size:15px;color:var(--muted);margin-top:8px" id="match-status">Жұп табылды: 0 / 5</div>
    <button class="reset" id="match-reset">Қайта бастау</button>
  </section>
</div>

<script>
// ---------- Part 1: sort ----------
const sortItems = [
  { e:"📄", t:"Сурет.jpg", ans:"file" },
  { e:"📁", t:"«Ойындар» қапшығы", ans:"folder" },
  { e:"📝", t:"Дәптерім.docx", ans:"file" },
  { e:"📁", t:"«Мектеп» қапшығы", ans:"folder" },
  { e:"🎵", t:"Ән.mp3", ans:"file" },
  { e:"📁", t:"«Суреттер» қапшығы", ans:"folder" },
];
let sortIdx = 0, sortScore = 0;

function renderSort(){
  const box = document.getElementById('sortGame');
  if(sortIdx >= sortItems.length){
    box.innerHTML = `<div class="done"><div class="big">✅</div><p style="font-size:20px">1-бөлім бітті: ${sortScore} / ${sortItems.length}</p></div>`;
    updateTotal();
    return;
  }
  const it = sortItems[sortIdx];
  box.innerHTML = `
    <div class="item-box"><div class="e">${it.e}</div><p>${it.t}</p></div>
    <div class="choice-row">
      <button class="choicebtn" data-v="file"><span class="e">📄</span>Файл</button>
      <button class="choicebtn" data-v="folder"><span class="e">📁</span>Қапшық</button>
    </div>
    <div class="feedback" id="sort-fb"></div>
    <button class="next" id="sort-next" disabled>Келесі →</button>
  `;
  const btns = box.querySelectorAll('.choicebtn');
  btns.forEach(b=>{
    b.onclick = ()=>{
      btns.forEach(x=>x.disabled = true);
      const fb = document.getElementById('sort-fb');
      if(b.dataset.v === it.ans){
        b.classList.add('correct'); fb.textContent='Дұрыс! ✓'; fb.className='feedback ok'; sortScore++;
      } else {
        b.classList.add('wrong');
        btns.forEach(x=>{ if(x.dataset.v===it.ans) x.classList.add('correct'); });
        fb.textContent='Басқасы еді, бірақ үйрендік!'; fb.className='feedback no';
      }
      document.getElementById('sort-next').disabled = false;
      updateTotal();
    };
  });
  document.getElementById('sort-next').onclick = ()=>{ sortIdx++; renderSort(); };
}

// ---------- Part 2: matching ----------
const pairs = [
  { action:"✨ Жасау", def:"жаңа файл немесе қапшық құру" },
  { action:"📋 Көшіру", def:"файлдың көшірмесін алу" },
  { action:"📥 Қою", def:"көшірілген файлды орналастыру" },
  { action:"🔄 Ауыстыру", def:"файлдың атын немесе орнын өзгерту" },
  { action:"🗑️ Жою", def:"файл немесе қапшықты өшіру" },
];
function shuffle(a){ const r=a.slice(); for(let i=r.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1)); [r[i],r[j]]=[r[j],r[i]];} return r; }
let matched = 0, selectedAction = null;

function renderMatch(){
  matched = 0; selectedAction = null;
  const left = document.getElementById('col-left'), right = document.getElementById('col-right');
  left.innerHTML=''; right.innerHTML='';
  shuffle(pairs).forEach(p=>{
    const b=document.createElement('button'); b.className='chip'; b.dataset.k=p.action;
    b.innerHTML = p.action; b.onclick=()=>selA(b,p.action); left.appendChild(b);
  });
  shuffle(pairs).forEach(p=>{
    const b=document.createElement('button'); b.className='chip'; b.dataset.k=p.action;
    b.textContent = p.def; b.onclick=()=>selD(b,p.action); right.appendChild(b);
  });
  document.getElementById('match-status').textContent = `Жұп табылды: 0 / ${pairs.length}`;
  updateTotal();
}
function selA(btn,k){ if(btn.classList.contains('correct'))return; document.querySelectorAll('#col-left .chip').forEach(c=>c.classList.remove('selected')); btn.classList.add('selected'); selectedAction=k; }
function selD(btn,k){ if(btn.classList.contains('correct')||!selectedAction)return;
  if(k===selectedAction){
    btn.classList.add('correct');
    document.querySelectorAll('#col-left .chip').forEach(c=>{ if(c.dataset.k===k){c.classList.add('correct'); c.classList.remove('selected');} });
    matched++; selectedAction=null;
    document.getElementById('match-status').textContent = `Жұп табылды: ${matched} / ${pairs.length}`;
    updateTotal();
  } else {
    btn.classList.add('wrong'); setTimeout(()=>btn.classList.remove('wrong'),300);
  }
}
document.getElementById('match-reset').onclick = renderMatch;

function updateTotal(){
  document.getElementById('total-score').textContent = sortScore + matched;
}

renderSort();
renderMatch();
</script>
</body>
</html>
