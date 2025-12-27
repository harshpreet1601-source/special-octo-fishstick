# special-octo-fishstick
Learn and practice french
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>French Practice – Verbs (A1)</title>
  <style>
    body { font-family: system-ui, Arial, sans-serif; margin: 22px; line-height: 1.45; }
    .card { max-width: 860px; border: 1px solid #ddd; border-radius: 14px; padding: 16px; }
    h1 { margin: 0 0 6px; font-size: 20px; }
    .sub { color:#555; margin: 0 0 12px; }
    .row { display:flex; gap:10px; flex-wrap: wrap; align-items:center; }
    .q { font-size: 18px; margin: 12px 0 8px; }
    input { width: 100%; padding: 10px; font-size: 16px; margin: 6px 0; }
    button, select { padding: 9px 12px; font-size: 15px; cursor: pointer; }
    .ok { color: #0a7a0a; font-weight: 700; }
    .bad { color: #b00020; font-weight: 700; }
    .meta { color:#666; font-size: 14px; }
    .box { background:#fafafa; border:1px solid #eee; border-radius: 12px; padding: 10px; margin-top: 10px; }
    .pill { display:inline-block; padding: 2px 8px; border-radius: 999px; background:#f2f2f2; }
    .grid { display:grid; grid-template-columns: 1fr 1fr; gap:10px; }
    @media (max-width: 720px){ .grid { grid-template-columns: 1fr; } }
    ul { margin: 6px 0 0 18px; }
    .small { font-size: 13px; color:#666; }
  </style>
</head>
<body>
  <div class="card">
    <h1>French Practice <span class="pill">one-by-one</span></h1>
    <p class="sub">All 16 verbs • random mode • score + mistakes • levels • daily 10-minute mode • shows English meaning after correct.</p>

    <div class="grid">
      <div class="box">
        <div class="row">
          <b>Mode</b>
          <select id="mode">
            <option value="normal">Normal</option>
            <option value="random">Random</option>
            <option value="daily">Daily (10 min)</option>
          </select>

          <b>Level</b>
          <select id="level">
            <option value="easy">Easy</option>
            <option value="medium">Medium</option>
            <option value="hard">Hard</option>
          </select>

          <button id="restartBtn" type="button">Restart</button>
        </div>
        <div class="row" style="margin-top:8px;">
          <span class="meta">Progress: <span id="progress"></span></span>
          <span class="meta">Score: <span id="score"></span></span>
          <span class="meta" id="timerLine" style="display:none;">Time left: <span id="timer"></span></span>
        </div>
        <div class="small" style="margin-top:6px;">
          Tip: Type the missing word (verb form) or the full sentence if asked. Press <b>Enter</b> to check.
        </div>
      </div>

      <div class="box">
        <b>Mistakes</b> <span class="meta">(shows what you got wrong)</span>
        <ul id="mistakes"></ul>
      </div>
    </div>

    <div class="q" id="question"></div>
    <input id="answer" placeholder="Type your answer…" autocomplete="off" />

    <div class="row">
      <button id="checkBtn">Check</button>
      <button id="hintBtn" type="button">Hint</button>
      <button id="skipBtn" type="button">Skip</button>
      <button id="revealBtn" type="button">Reveal</button>
    </div>

    <div id="result" style="margin-top:12px;"></div>
    <div class="meta" id="hint" style="display:none; margin-top:6px;"></div>
  </div>

<script>
  // ---------- DATA (16 verbs, 9 pronouns) ----------
  const pronouns = ["je", "tu", "il", "elle", "on", "nous", "vous", "ils", "elles"];

  // For each verb: label, meaning, and the present tense forms for each pronoun.
  // Then we generate questions automatically.
  const verbs = [
    {v:"être", meaning:"to be", forms:{je:"suis", tu:"es", il:"est", elle:"est", on:"est", nous:"sommes", vous:"êtes", ils:"sont", elles:"sont"}},
    {v:"avoir", meaning:"to have", forms:{je:"ai", tu:"as", il:"a", elle:"a", on:"a", nous:"avons", vous:"avez", ils:"ont", elles:"ont"}},
    {v:"aller", meaning:"to go", forms:{je:"vais", tu:"vas", il:"va", elle:"va", on:"va", nous:"allons", vous:"allez", ils:"vont", elles:"vont"}},
    {v:"venir", meaning:"to come", forms:{je:"viens", tu:"viens", il:"vient", elle:"vient", on:"vient", nous:"venons", vous:"venez", ils:"viennent", elles:"viennent"}},
    {v:"faire", meaning:"to do/make", forms:{je:"fais", tu:"fais", il:"fait", elle:"fait", on:"fait", nous:"faisons", vous:"faites", ils:"font", elles:"font"}},

    {v:"parler", meaning:"to speak", forms:{je:"parle", tu:"parles", il:"parle", elle:"parle", on:"parle", nous:"parlons", vous:"parlez", ils:"parlent", elles:"parlent"}},
    {v:"habiter", meaning:"to live", forms:{je:"habite", tu:"habites", il:"habite", elle:"habite", on:"habite", nous:"habitons", vous:"habitez", ils:"habitent", elles:"habitent"}},
    {v:"aimer", meaning:"to like/love", forms:{je:"aime", tu:"aimes", il:"aime", elle:"aime", on:"aime", nous:"aimons", vous:"aimez", ils:"aiment", elles:"aiment"}},
    {v:"jouer", meaning:"to play", forms:{je:"joue", tu:"joues", il:"joue", elle:"joue", on:"joue", nous:"jouons", vous:"jouez", ils:"jouent", elles:"jouent"}},
    {v:"regarder", meaning:"to watch/look at", forms:{je:"regarde", tu:"regardes", il:"regarde", elle:"regarde", on:"regarde", nous:"regardons", vous:"regardez", ils:"regardent", elles:"regardent"}},

    {v:"écouter", meaning:"to listen", forms:{je:"écoute", tu:"écoutes", il:"écoute", elle:"écoute", on:"écoute", nous:"écoutons", vous:"écoutez", ils:"écoutent", elles:"écoutent"}},
    {v:"chanter", meaning:"to sing", forms:{je:"chante", tu:"chantes", il:"chante", elle:"chante", on:"chante", nous:"chantons", vous:"chantez", ils:"chantent", elles:"chantent"}},
    {v:"danser", meaning:"to dance", forms:{je:"danse", tu:"danses", il:"danse", elle:"danse", on:"danse", nous:"dansons", vous:"dansez", ils:"dansent", elles:"dansent"}},
    {v:"manger", meaning:"to eat", forms:{je:"mange", tu:"manges", il:"mange", elle:"mange", on:"mange", nous:"mangeons", vous:"mangez", ils:"mangent", elles:"mangent"}},
    {v:"boire", meaning:"to drink", forms:{je:"bois", tu:"bois", il:"boit", elle:"boit", on:"boit", nous:"buvons", vous:"buvez", ils:"boivent", elles:"boivent"}},
    {v:"travailler", meaning:"to work", forms:{je:"travaille", tu:"travailles", il:"travaille", elle:"travaille", on:"travaille", nous:"travaillons", vous:"travaillez", ils:"travaillent", elles:"travaillent"}}
  ];

  // Example templates to keep it simple and consistent.
  // We will generate a French sentence + English meaning. (Not perfect for every verb, but good practice.)
  function makeSentence(pronoun, verb, form){
    // French examples
    const ex = {
      "être":      {fr:`${cap(pronoun)} ${form} prêt.`, en:`${engPron(pronoun)} am ready.`},
      "avoir":     {fr:`${cap(pronoun)} ${form} un livre.`, en:`${engPron(pronoun)} have a book.`},
      "aller":     {fr:`${cap(pronoun)} ${form} au travail.`, en:`${engPron(pronoun)} go to work.`},
      "venir":     {fr:`${cap(pronoun)} ${form} ici.`, en:`${engPron(pronoun)} come here.`},
      "faire":     {fr:`${cap(pronoun)} ${form} une pause.`, en:`${engPron(pronoun)} take a break.`},
      "parler":    {fr:`${cap(pronoun)} ${form} français.`, en:`${engPron(pronoun)} speak French.`},
      "habiter":   {fr:`${cap(pronoun)} ${form} ici.`, en:`${engPron(pronoun)} live here.`},
      "aimer":     {fr:`${cap(pronoun)} ${form} la musique.`, en:`${engPron(pronoun)} like music.`},
      "jouer":     {fr:`${cap(pronoun)} ${form} au football.`, en:`${engPron(pronoun)} play football.`},
      "regarder":  {fr:`${cap(pronoun)} ${form} la télé.`, en:`${engPron(pronoun)} watch TV.`},
      "écouter":   {fr:`${cap(pronoun)} ${form} la radio.`, en:`${engPron(pronoun)} listen to the radio.`},
      "chanter":   {fr:`${cap(pronoun)} ${form} une chanson.`, en:`${engPron(pronoun)} sing a song.`},
      "danser":    {fr:`${cap(pronoun)} ${form} ici.`, en:`${engPron(pronoun)} dance here.`},
      "manger":    {fr:`${cap(pronoun)} ${form} du pain.`, en:`${engPron(pronoun)} eat bread.`},
      "boire":     {fr:`${cap(pronoun)} ${form} de l’eau.`, en:`${engPron(pronoun)} drink water.`},
      "travailler":{fr:`${cap(pronoun)} ${form} maintenant.`, en:`${engPron(pronoun)} work now.`}
    };
    return ex[verb];
  }

  // ---------- QUESTION GENERATOR ----------
  function buildPool(level){
    // EASY: just "pronoun + ___" (fill the verb form)
    // MEDIUM: choose verb + fill form
    // HARD: sometimes ask full line (type the whole answer)
    const pool = [];

    for(const vb of verbs){
      for(const p of pronouns){
        const form = vb.forms[p];
        const sent = makeSentence(p, vb.v, form);
        if(!sent) continue;

        if(level === "easy"){
          pool.push({
            prompt: `Fill the missing verb form:\n${cap(p)} ___ (${vb.v})`,
            accepted: [form],
            correctFR: `${cap(p)} ${form}.`,
            english: `${sent.en}`,
            hint: `${vb.v} = ${vb.meaning} • pronoun: ${p}`
          });
        } else if(level === "medium"){
          pool.push({
            prompt: `Fill the blank:\n${sent.fr.replace(form, "___")}  (${vb.v})`,
            accepted: [form],
            correctFR: sent.fr,
            english: sent.en,
            hint: `${vb.v} (${vb.meaning}) with "${p}"`
          });
        } else { // hard
          // Mix: either fill blank or type full sentence
          const askFull = Math.random() < 0.35;
          if(askFull){
            pool.push({
              prompt: `Type the FULL French sentence:\nEnglish: ${sent.en}\n(verb: ${vb.v})`,
              accepted: [sent.fr.toLowerCase().replaceAll("’","'")],
              correctFR: sent.fr,
              english: sent.en,
              hint: `Start with "${cap(p)}" and use ${vb.v}`
            });
          } else {
            pool.push({
              prompt: `Fill the blank:\n${sent.fr.replace(form, "___")}  (${vb.v})`,
              accepted: [form],
              correctFR: sent.fr,
              english: sent.en,
              hint: `${vb.v} (${vb.meaning}) with "${p}"`
            });
          }
        }
      }
    }
    return pool;
  }

  // ---------- APP STATE ----------
  let pool = [];
  let index = 0;
  let score = {correct:0, total:0};
  let mistakes = [];
  let timer = null;
  let timeLeft = 600; // 10 minutes

  const qEl = document.getElementById("question");
  const aEl = document.getElementById("answer");
  const rEl = document.getElementById("result");
  const hEl = document.getElementById("hint");
  const pEl = document.getElementById("progress");
  const sEl = document.getElementById("score");
  const mEl = document.getElementById("mistakes");
  const modeEl = document.getElementById("mode");
  const levelEl = document.getElementById("level");
  const timerLine = document.getElementById("timerLine");
  const timerEl = document.getElementById("timer");

  function cap(s){ return s.charAt(0).toUpperCase() + s.slice(1); }

  function engPron(p){
    // quick mapping (simple)
    const map = {je:"I", tu:"You", il:"He", elle:"She", on:"We", nous:"We", vous:"You", ils:"They", elles:"They"};
    return map[p] || "They";
  }

  function normalize(s){
    return (s || "")
      .trim()
      .toLowerCase()
      .replaceAll("’","'")
      .replace(/\s+/g, " ");
  }

  function shuffle(arr){
    for(let i=arr.length-1;i>0;i--){
      const j = Math.floor(Math.random()*(i+1));
      [arr[i],arr[j]] = [arr[j],arr[i]];
    }
    return arr;
  }

  function setMistakes(){
    mEl.innerHTML = "";
    for(const m of mistakes.slice(-10).reverse()){
      const li = document.createElement("li");
      li.textContent = `${m.prompt}  | your: "${m.your}"  → correct: "${m.correct}"`;
      mEl.appendChild(li);
    }
    if(mistakes.length === 0){
      const li = document.createElement("li");
      li.textContent = "No mistakes yet ✅";
      mEl.appendChild(li);
    }
  }

  function setHUD(){
    pEl.textContent = `${index+1} / ${pool.length}`;
    sEl.textContent = `${score.correct} correct / ${score.total} tried`;
    setMistakes();
  }

  function startTimer(){
    stopTimer();
    timeLeft = 600;
    timerLine.style.display = "inline";
    timerEl.textContent = formatTime(timeLeft);
    timer = setInterval(()=>{
      timeLeft--;
      timerEl.textContent = formatTime(timeLeft);
      if(timeLeft <= 0){
        stopTimer();
        rEl.innerHTML = `<div class="ok">⏰ Time! Daily practice finished.</div>
          <div class="meta">Score: ${score.correct}/${score.total}. Click Restart to play again.</div>`;
        aEl.disabled = true;
        document.getElementById("checkBtn").disabled = true;
      }
    }, 1000);
  }

  function stopTimer(){
    if(timer) clearInterval(timer);
    timer = null;
    timerLine.style.display = "none";
    aEl.disabled = false;
    document.getElementById("checkBtn").disabled = false;
  }

  function formatTime(s){
    const m = Math.floor(s/60);
    const r = s%60;
    return `${m}:${String(r).padStart(2,"0")}`;
  }

  function show(){
    const q = pool[index];
    qEl.textContent = q.prompt;
    aEl.value = "";
    rEl.innerHTML = "";
    hEl.style.display = "none";
    hEl.textContent = "";
    setHUD();
    aEl.focus();
  }

  function next(){
    const mode = modeEl.value;
    if(mode === "random"){
      index = Math.floor(Math.random() * pool.length);
      show();
      return;
    }
    if(index < pool.length - 1){
      index++;
      show();
    } else {
      rEl.innerHTML += `<div class="ok" style="margin-top:10px;">🎉 Finished! Click Restart to practice again.</div>`;
    }
  }

  function check(){
    const q = pool[index];
    const ans = normalize(aEl.value);
    if(!ans) return;

    const accepted = q.accepted.map(normalize);
    const ok = accepted.includes(ans);

    score.total++;

    if(ok){
      score.correct++;
      rEl.innerHTML = `
        <div class="ok">✅ Correct</div>
        <div><b>French:</b> ${q.correctFR}</div>
        <div><b>English:</b> ${q.english}</div>
        <div class="meta">Next question is loading…</div>
      `;
      setHUD();
      setTimeout(next, 650);
    } else {
      rEl.innerHTML = `<div class="bad">❌ Not correct</div><div class="meta">Try again or press Hint/Reveal.</div>`;
      mistakes.push({prompt:q.prompt.replace(/\n/g," "), your: aEl.value, correct: q.accepted[0]});
      setHUD();
    }
  }

  function hint(){
    const q = pool[index];
    hEl.style.display = "block";
    hEl.textContent = "Hint: " + q.hint;
  }

  function reveal(){
    const q = pool[index];
    rEl.innerHTML = `
      <div class="bad">Revealed</div>
      <div><b>French:</b> ${q.correctFR}</div>
      <div><b>English:</b> ${q.english}</div>
    `;
  }

  function restart(){
    const level = levelEl.value;
    pool = buildPool(level);

    // Randomize order in normal + daily, so you don’t memorize the sequence.
    shuffle(pool);

    index = 0;
    score = {correct:0, total:0};
    mistakes = [];
    setMistakes();
    stopTimer();

    if(modeEl.value === "daily"){
      startTimer();
    }
    show();
  }

  // ---------- EVENTS ----------
  document.getElementById("checkBtn").addEventListener("click", check);
  document.getElementById("hintBtn").addEventListener("click", hint);
  document.getElementById("revealBtn").addEventListener("click", reveal);
  document.getElementById("skipBtn").addEventListener("click", next);
  document.getElementById("restartBtn").addEventListener("click", restart);

  aEl.addEventListener("keydown", (e)=> { if(e.key === "Enter") check(); });

  modeEl.addEventListener("change", ()=> restart());
  levelEl.addEventListener("change", ()=> restart());

  // Start
  restart();
</script>
</body>
</html>
