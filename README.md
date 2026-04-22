<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Quiz — La Sauvegarde</title>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&family=DM+Mono:wght@400;500&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #f5f5f0;
    --card: #ffffff;
    --green: #00a86b;
    --orange: #e07b00;
    --red: #d0002e;
    --blue: #0077c2;
    --purple: #7c3aed;
    --text: #1a1a1a;
    --muted: #666;
    --border: #ddd;
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { background: var(--bg); color: var(--text); font-family: 'DM Mono', monospace; min-height: 100vh; overflow-x: hidden; }

  #hud {
    position: fixed; top: 0; left: 0; right: 0; z-index: 100;
    background: var(--bg); border-bottom: 2px solid var(--border);
    padding: 10px 20px; display: flex; align-items: center; gap: 16px; flex-wrap: wrap;
  }
  #hud .hud-title { font-family: 'Press Start 2P', monospace; font-size: 8px; color: var(--blue); flex: 1; min-width: 120px; }
  .hud-stat { font-size: 11px; display: flex; align-items: center; gap: 6px; white-space: nowrap; }
  .hud-stat span { font-weight: 500; }
  .lives-display { font-size: 14px; letter-spacing: 2px; }
  .combo-badge {
    background: var(--orange); color: white; font-family: 'Press Start 2P', monospace;
    font-size: 7px; padding: 3px 7px; border-radius: 4px; display: none;
  }
  .combo-badge.active { display: inline-block; }
  #timer-bar-wrap { display: none; }
  #timer-bar { height: 100%; width: 100%; background: var(--green); border-radius: 3px; transition: width 0.25s linear, background 0.5s; }

  main { padding-top: 90px; max-width: 720px; margin: 0 auto; padding-bottom: 60px; padding-left: 16px; padding-right: 16px; }

  .cat-badge {
    display: inline-block; font-family: 'Press Start 2P', monospace;
    font-size: 7px; padding: 4px 10px; border-radius: 4px; margin-bottom: 18px;
    text-transform: uppercase; letter-spacing: 1px;
  }
  .cat-concepts { background: #e8f4ff; color: var(--blue); }
  .cat-methodes { background: #e8fff4; color: var(--green); }
  .cat-ad       { background: #f3e8ff; color: var(--purple); }
  .cat-outils   { background: #fff3e0; color: var(--orange); }
  .cat-piege    { background: #ffe8e8; color: var(--red); }

  #question-card {
    background: var(--card); border: 2px solid var(--border); border-radius: 12px;
    padding: 28px; margin-bottom: 20px;
    box-shadow: 0 2px 8px rgba(0,0,0,0.06);
  }
  #q-counter { font-size: 10px; color: var(--muted); margin-bottom: 10px; font-family: 'Press Start 2P', monospace; }
  #question-text { font-size: 15px; line-height: 1.7; font-weight: 500; }

  #options { display: flex; flex-direction: column; gap: 10px; margin-bottom: 16px; }
  .opt-btn {
    background: var(--card); border: 2px solid var(--border); border-radius: 8px;
    padding: 14px 18px; font-family: 'DM Mono', monospace; font-size: 13px;
    cursor: pointer; text-align: left; transition: all 0.15s;
    display: flex; align-items: flex-start; gap: 12px;
  }
  .opt-btn:hover:not(:disabled) { border-color: var(--blue); background: #f0f8ff; transform: translateX(3px); }
  .opt-key {
    font-family: 'Press Start 2P', monospace; font-size: 9px;
    background: var(--bg); border: 1.5px solid var(--border); border-radius: 4px;
    padding: 3px 7px; min-width: 28px; text-align: center; flex-shrink: 0; margin-top: 1px;
  }
  .opt-btn.correct { background: #e6fff4; border-color: var(--green); }
  .opt-btn.correct .opt-key { background: var(--green); color: white; border-color: var(--green); }
  .opt-btn.wrong { background: #ffe6ea; border-color: var(--red); }
  .opt-btn.wrong .opt-key { background: var(--red); color: white; border-color: var(--red); }
  .opt-btn:disabled { cursor: default; }

  #feedback {
    display: none; border-radius: 8px; padding: 14px 18px;
    font-size: 13px; line-height: 1.6; margin-bottom: 14px;
  }
  #feedback.correct { background: #e6fff4; border: 2px solid var(--green); }
  #feedback.wrong   { background: #ffe6ea; border: 2px solid var(--red); }
  #feedback .fb-title { font-family: 'Press Start 2P', monospace; font-size: 8px; margin-bottom: 8px; }
  #feedback.correct .fb-title { color: var(--green); }
  #feedback.wrong   .fb-title { color: var(--red); }

  #next-btn {
    display: none; background: var(--blue); color: white; border: none; border-radius: 8px;
    padding: 13px 28px; font-family: 'Press Start 2P', monospace; font-size: 9px;
    cursor: pointer; transition: background 0.15s;
  }
  #next-btn:hover { background: #005fa0; }

  .xp-float {
    position: fixed; font-family: 'Press Start 2P', monospace; font-size: 12px;
    pointer-events: none; z-index: 999; font-weight: bold;
    animation: floatUp 1.2s ease-out forwards;
  }
  @keyframes floatUp { 0%{opacity:1;transform:translateY(0)} 100%{opacity:0;transform:translateY(-80px)} }

  .flash-green { animation: flashG 0.3s; }
  .flash-red   { animation: flashR 0.3s; }
  @keyframes flashG { 0%,100%{background:var(--bg)} 50%{background:#b3ffe0} }
  @keyframes flashR { 0%,100%{background:var(--bg)} 50%{background:#ffd0d8} }

  #progress-wrap { background: var(--border); border-radius: 4px; height: 6px; margin-bottom: 16px; }
  #progress-bar  { height: 100%; background: var(--blue); border-radius: 4px; transition: width 0.3s; }

  #rank-screen {
    display: none; text-align: center; padding: 40px 20px;
    background: var(--card); border: 2px solid var(--border); border-radius: 16px; margin-top: 40px;
  }
  #rank-letter { font-family: 'Press Start 2P', monospace; font-size: 72px; margin: 20px 0; display: block; }
  .rank-S { color: var(--green); text-shadow: 0 0 30px rgba(0,168,107,0.5); }
  .rank-A { color: var(--blue); }
  .rank-B { color: var(--orange); }
  .rank-C { color: #888; }
  .rank-D { color: var(--red); }
  #rank-screen h2 { font-family: 'Press Start 2P', monospace; font-size: 10px; margin-bottom: 16px; }
  .stats-grid { display: grid; grid-template-columns: 1fr 1fr; gap: 12px; max-width: 360px; margin: 24px auto; }
  .stat-box      { background: var(--bg); border-radius: 8px; padding: 14px; }
  .stat-box .val { font-family: 'Press Start 2P', monospace; font-size: 18px; margin-bottom: 4px; }
  .stat-box .lbl { font-size: 10px; color: var(--muted); }
  #restart-btn {
    background: var(--green); color: white; border: none; border-radius: 8px;
    padding: 14px 32px; font-family: 'Press Start 2P', monospace; font-size: 9px;
    cursor: pointer; margin-top: 20px;
  }
  #restart-btn:hover { background: #008a58; }
  #timeout-msg { display: none; color: var(--orange); font-size: 11px; font-family: 'Press Start 2P', monospace; margin-bottom: 10px; }
  .kb-hint { text-align: center; font-size: 10px; color: var(--muted); margin-top: 10px; }
</style>
</head>
<body>

<div id="hud">
  <div class="hud-title">💾 LA SAUVEGARDE</div>
  <div class="hud-stat"><span class="lives-display" id="lives-disp">❤️❤️❤️❤️❤️</span></div>
  <div class="hud-stat">⭐ Score : <span id="score-disp">0</span></div>
  <div class="hud-stat">🔥 Combo : <span id="combo-disp">0</span></div>
  <div class="combo-badge" id="combo-badge">COMBO!</div>
  <div style="width:100%"><div id="timer-bar-wrap"><div id="timer-bar"></div></div></div>
</div>

<main>
  <div id="progress-wrap"><div id="progress-bar" style="width:0%"></div></div>
  <div id="game-area">
    <div id="question-card">
      <div id="q-counter"></div>
      <div class="cat-badge" id="cat-badge"></div>
      <div id="question-text"></div>
    </div>
    <div id="options"></div>
    <div id="timeout-msg">⏰ TEMPS ÉCOULÉ !</div>
    <div id="feedback">
      <div class="fb-title" id="fb-title"></div>
      <div id="fb-text"></div>
    </div>
    <button id="next-btn" onclick="nextQuestion()">SUIVANT →</button>
    <p class="kb-hint">Touches A B C D → répondre &nbsp;|&nbsp; Entrée / Espace → suivant</p>
  </div>
  <div id="rank-screen">
    <h2>RÉSULTAT FINAL</h2>
    <span id="rank-letter"></span>
    <div id="rank-title" style="font-size:14px;margin-bottom:8px;"></div>
    <div class="stats-grid">
      <div class="stat-box"><div class="val" id="final-score"   style="color:var(--blue)"></div><div class="lbl">Score</div></div>
      <div class="stat-box"><div class="val" id="final-correct" style="color:var(--green)"></div><div class="lbl">Bonnes rép.</div></div>
      <div class="stat-box"><div class="val" id="final-wrong"   style="color:var(--red)"></div><div class="lbl">Erreurs</div></div>
      <div class="stat-box"><div class="val" id="final-combo"   style="color:var(--orange)"></div><div class="lbl">Meilleur combo</div></div>
    </div>
    <button id="restart-btn" onclick="restartGame()">▶ RECOMMENCER</button>
  </div>
</main>

<script>
const QUESTIONS_RAW = [

  // ── Q1 ── toutes les options sont courtes et plausibles
  {
    cat: "concepts",
    q: "Qu'est-ce que le RPO ?",
    opts: [
      "Le temps maximal d'indisponibilité toléré",
      "Le moment le plus éloigné auquel on peut remonter",
      "Le coût maximal alloué aux sauvegardes",
      "Le nombre de copies conservées simultanément"
    ],
    correct: 1,
    expl: "RPO (Recovery Point Objective) = l'instant le plus ancien auquel on peut revenir sans gêner l'activité. Le RTO, lui, mesure le temps d'indisponibilité maximal toléré — attention à ne pas les confondre."
  },

  // ── Q2 ── 4 méthodes réelles, une seule bonne
  {
    cat: "methodes",
    q: "Quelle méthode offre la restauration la plus RAPIDE mais consomme le plus d'espace ?",
    opts: [
      "Incrémentielle",
      "Différentielle",
      "Complète",
      "Déduplication"
    ],
    correct: 2,
    expl: "La sauvegarde complète copie tout à chaque fois → restauration en une seule étape mais stockage maximal. L'incrémentielle économise l'espace mais oblige à réappliquer toutes les sauvegardes dans l'ordre pour restaurer."
  },

  // ── Q3 ── vrai/faux avec 4 propositions équilibrées
  {
    cat: "piege",
    q: "Un RAID 1 suffit à protéger les données d'une entreprise.",
    opts: [
      "Vrai, il maintient une copie miroir en permanence",
      "Vrai, mais uniquement pour les pannes matérielles",
      "Faux, il ne protège pas contre la suppression accidentelle",
      "Faux, le RAID 1 n'est plus supporté sous Windows Server"
    ],
    correct: 2,
    expl: "Le RAID protège contre la PANNE d'un disque, mais si un fichier est supprimé ou chiffré par un ransomware, la suppression est répliquée instantanément sur les deux disques. Le RAID n'est pas une sauvegarde."
  },

  // ── Q4 ── 4 acronymes courts, pièges réels
  {
    cat: "concepts",
    q: "PCA signifie :",
    opts: [
      "Plan de Contrôle d'Accès",
      "Plan de Continuité d'Activité",
      "Plan de Copie des Archives",
      "Plan de Correction Automatique"
    ],
    correct: 1,
    expl: "PCA = Plan de Continuité d'Activité → objectif : ne jamais s'interrompre. Le PRA (Plan de Reprise d'Activité) tolère une interruption et prévoit comment reprendre. Le PCA est plus ambitieux et plus coûteux."
  },

  // ── Q5 ── 4 durées plausibles
  {
    cat: "ad",
    q: "Par défaut, combien de jours un objet AD supprimé reste-t-il dans la Corbeille AD ?",
    opts: [
      "30 jours",
      "90 jours",
      "180 jours",
      "365 jours"
    ],
    correct: 2,
    expl: "La valeur par défaut est 180 jours. Passé ce délai, l'objet est définitivement supprimé. Cette durée est paramétrable via l'attribut msDS-deletedObjectLifetime."
  },

  // ── Q6 ── 4 comptes plausibles, tous du même gabarit
  {
    cat: "ad",
    q: "En mode DSRM (restauration AD), quel compte faut-il utiliser pour se connecter ?",
    opts: [
      "Un compte Domain Admin du domaine",
      "Le compte Administrateur local du DC",
      "Un compte Opérateur de sauvegarde du domaine",
      "Le compte SYSTEM intégré"
    ],
    correct: 1,
    expl: "En mode DSRM, les services AD sont arrêtés → aucun compte du domaine ne peut s'authentifier. Il faut le compte Administrateur LOCAL, avec le mot de passe défini lors de l'installation du contrôleur de domaine."
  },

  // ── Q7 ── 4 systèmes de fichiers courts
  {
    cat: "outils",
    q: "Les Clichés Instantanés (Shadow Copies) sont incompatibles avec quel système de fichiers ?",
    opts: [
      "NTFS",
      "FAT32",
      "ReFS",
      "exFAT"
    ],
    correct: 2,
    expl: "Les Shadow Copies ne fonctionnent que sur NTFS. ReFS (Resilient File System) offre une meilleure intégrité et résilience, mais ne supporte pas les Clichés Instantanés."
  },

  // ── Q8 ── règle 3-2-1
  {
    cat: "concepts",
    q: "Que signifie la règle 3-2-1 en matière de sauvegarde ?",
    opts: [
      "3 sauvegardes complètes, 2 incrémentielles, 1 différentielle",
      "3 copies des données, sur 2 supports différents, dont 1 hors site",
      "3 sites de secours, 2 fournisseurs cloud, 1 sauvegarde locale",
      "3 jours de rétention, 2 niveaux RAID, 1 sauvegarde quotidienne"
    ],
    correct: 1,
    expl: "La règle 3-2-1 : 3 copies des données (originale + 2 sauvegardes), sur 2 supports de types différents (ex : disque + bande), dont 1 copie stockée hors site (cloud, site secondaire). Cette règle protège contre la plupart des sinistres."
  },

  // ── Q9 ── 4 opérations du même gabarit, une seule bonne
  {
    cat: "methodes",
    q: "Pour restaurer au mercredi avec des sauvegardes incrémentielles (complète lundi, incréments mar/mer/jeu), que faut-il appliquer ?",
    opts: [
      "Complète lundi + incrément mercredi seulement",
      "Incrément mercredi seulement",
      "Complète lundi + incrément mardi + incrément mercredi",
      "Complète lundi + incrément jeudi"
    ],
    correct: 2,
    expl: "Chaque incrément ne contient que les changements depuis LE PRÉCÉDENT. Pour reconstruire l'état du mercredi : complète (lundi) + delta lun→mar + delta mar→mer. Sauter le mardi laisserait des données manquantes."
  },

  // ── Q10 ── 4 outils réels, pièges plausibles
  {
    cat: "ad",
    q: "Quel outil CLI permet de restaurer un objet AD sans passer par la Corbeille AD graphique ?",
    opts: [
      "ntdsutil.exe",
      "wbadmin.exe",
      "ldp.exe",
      "dsrm.exe"
    ],
    correct: 2,
    expl: "ldp.exe est l'outil CLI sous-jacent à la Corbeille AD — la corbeille graphique n'en est que l'interface. ntdsutil gère la maintenance de la base NTDS, wbadmin pilote les sauvegardes Windows, dsrm supprime des objets AD."
  }

];

// ─── Labels & classes ───────────────────────────────────────
const CAT_LABELS = {
  "concepts":"📚 Concepts","methodes":"⚙️ Méthodes",
  "ad":"🗄️ Active Directory","outils":"🛠️ Outils","piege":"⚠️ Piège"
};
const CAT_CLASS = {
  "concepts":"cat-concepts","methodes":"cat-methodes",
  "ad":"cat-ad","outils":"cat-outils","piege":"cat-piege"
};

function shuffle(arr){const a=[...arr];for(let i=a.length-1;i>0;i--){const j=Math.floor(Math.random()*(i+1));[a[i],a[j]]=[a[j],a[i]];}return a;}

let questions=[],current=0,score=0,lives=5,combo=0,bestCombo=0;
let correctCount=0,wrongCount=0,timerInterval=null,timeLeft=20,answered=false;

function startGame(){
  questions=shuffle(QUESTIONS_RAW);
  current=0;score=0;lives=5;combo=0;bestCombo=0;correctCount=0;wrongCount=0;
  document.getElementById('rank-screen').style.display='none';
  document.getElementById('game-area').style.display='block';
  loadQuestion();
}

function loadQuestion(){
  answered=false;
  const q=questions[current];
  const indices=shuffle([0,1,2,3]);
  const shuffled=indices.map(i=>q.opts[i]);
  const newCorrect=indices.indexOf(q.correct);

  document.getElementById('q-counter').textContent=`Q ${current+1} / ${questions.length}`;
  const badge=document.getElementById('cat-badge');
  badge.textContent=CAT_LABELS[q.cat]||q.cat;
  badge.className='cat-badge '+(CAT_CLASS[q.cat]||'cat-concepts');
  document.getElementById('question-text').textContent=q.q;

  const optDiv=document.getElementById('options');
  optDiv.innerHTML='';
  ['A','B','C','D'].forEach((key,i)=>{
    const btn=document.createElement('button');
    btn.className='opt-btn';
    btn.innerHTML=`<span class="opt-key">${key}</span><span>${shuffled[i]}</span>`;
    btn.dataset.correct=(i===newCorrect)?'1':'0';
    btn.onclick=()=>selectAnswer(btn,i===newCorrect,q);
    optDiv.appendChild(btn);
  });

  document.getElementById('feedback').style.display='none';
  document.getElementById('next-btn').style.display='none';
  document.getElementById('timeout-msg').style.display='none';
  updateProgress();updateHUD();startTimer(q,newCorrect);
}

function startTimer(q,correctIdx){
  clearInterval(timerInterval);
  // Pas de minuterie
}

function timeOut(q,correctIdx){
  answered=true;
  document.getElementById('timeout-msg').style.display='block';
  document.querySelectorAll('.opt-btn').forEach(btn=>{btn.disabled=true;if(btn.dataset.correct==='1')btn.classList.add('correct');});
  combo=0;lives--;updateHUD();showFeedback(false,q);
  document.getElementById('next-btn').style.display='block';
  if(lives<=0)setTimeout(endGame,1200);
}

function selectAnswer(btn,isCorrect,q){
  if(answered)return;
  answered=true;clearInterval(timerInterval);
  document.querySelectorAll('.opt-btn').forEach(b=>{b.disabled=true;if(b.dataset.correct==='1')b.classList.add('correct');});
  if(isCorrect){
    btn.classList.add('correct');combo++;if(combo>bestCombo)bestCombo=combo;
    const bonus=combo>=3?20:10;score+=bonus;correctCount++;
    spawnXP(btn,'+'+bonus+' XP','var(--green)');
    document.body.classList.remove('flash-red');void document.body.offsetWidth;document.body.classList.add('flash-green');
  } else {
    btn.classList.add('wrong');combo=0;lives--;wrongCount++;
    document.body.classList.remove('flash-green');void document.body.offsetWidth;document.body.classList.add('flash-red');
  }
  updateHUD();showFeedback(isCorrect,q);
  document.getElementById('next-btn').style.display='block';
  if(lives<=0)setTimeout(endGame,1200);
}

function showFeedback(isCorrect,q){
  const fb=document.getElementById('feedback');
  fb.style.display='block';fb.className=isCorrect?'correct':'wrong';
  document.getElementById('fb-title').textContent=isCorrect?'✅ CORRECT !':'❌ INCORRECT';
  document.getElementById('fb-text').textContent=q.expl;
}

function nextQuestion(){
  document.body.classList.remove('flash-green','flash-red');
  current++;
  if(current>=questions.length||lives<=0){endGame();return;}
  loadQuestion();
}

function endGame(){
  clearInterval(timerInterval);
  document.getElementById('game-area').style.display='none';
  const pct=(correctCount/questions.length)*100;
  let rank,cls,title;
  if(pct>=90){rank='S';cls='rank-S';title='Expert sauvegarde 🏆';}
  else if(pct>=70){rank='A';cls='rank-A';title='Excellent !';}
  else if(pct>=50){rank='B';cls='rank-B';title='Bonne base, continue !';}
  else if(pct>=30){rank='C';cls='rank-C';title='Relis les fiches Notion 📖';}
  else{rank='D';cls='rank-D';title='Révise et recommence !';}
  document.getElementById('rank-screen').style.display='block';
  const rl=document.getElementById('rank-letter');rl.textContent=rank;rl.className=cls;
  document.getElementById('rank-title').textContent=title;
  document.getElementById('final-score').textContent=score;
  document.getElementById('final-correct').textContent=correctCount+'/'+questions.length;
  document.getElementById('final-wrong').textContent=wrongCount;
  document.getElementById('final-combo').textContent='x'+bestCombo;
}

function restartGame(){startGame();}

function updateHUD(){
  document.getElementById('lives-disp').textContent='❤️'.repeat(Math.max(0,lives))+'🖤'.repeat(Math.max(0,5-lives));
  document.getElementById('score-disp').textContent=score;
  document.getElementById('combo-disp').textContent=combo;
  const badge=document.getElementById('combo-badge');
  if(combo>=3){badge.textContent='COMBO x'+combo+'!';badge.classList.add('active');}
  else badge.classList.remove('active');
}

function updateProgress(){
  document.getElementById('progress-bar').style.width=((current/questions.length)*100)+'%';
}

function spawnXP(el,text,color){
  const rect=el.getBoundingClientRect();
  const span=document.createElement('span');
  span.className='xp-float';span.textContent=text;span.style.color=color;
  span.style.left=(rect.left+rect.width/2)+'px';
  span.style.top=(rect.top+window.scrollY-10)+'px';
  document.body.appendChild(span);setTimeout(()=>span.remove(),1300);
}

document.addEventListener('keydown',e=>{
  const map={'a':0,'b':1,'c':2,'d':3};const k=e.key.toLowerCase();
  if(map[k]!==undefined&&!answered){const btns=document.querySelectorAll('.opt-btn');if(btns[map[k]])btns[map[k]].click();}
  if(e.key==='Enter'||e.key===' '){const nb=document.getElementById('next-btn');if(nb.style.display!=='none')nextQuestion();}
});

startGame();
</script>
</body>
</html>
