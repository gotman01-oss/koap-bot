<!DOCTYPE html>
<html lang="ru">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover" />
  <meta name="theme-color" content="#060d1a" />
  <meta name="apple-mobile-web-app-capable" content="yes" />
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent" />
  <meta name="apple-mobile-web-app-title" content="КоАП Бот" />
  <title>КоАП Защитник</title>
  <link rel="manifest" href="manifest.json" />
  <link rel="apple-touch-icon" href="icon-192.png" />
  <style>
    *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
    :root { --ring: #3d5afe; }
    html, body {
      height: 100%; background: #060d1a; color: #c8d8f0;
      font-family: Georgia, serif; overscroll-behavior: none;
      -webkit-tap-highlight-color: transparent;
    }
    #app {
      min-height: 100dvh; display: flex; flex-direction: column;
      align-items: center; justify-content: center;
      padding: env(safe-area-inset-top,24px) 20px env(safe-area-inset-bottom,24px);
      background: linear-gradient(160deg,#060d1a 0%,#0d1b2a 60%,#060d1a 100%);
    }
    .header { text-align: center; margin-bottom: 32px; }
    .header-sub { font-size:10px; letter-spacing:4px; color:#3a5a7a; margin-bottom:6px; font-family:monospace; }
    .header h1 { font-size:26px; font-weight:normal; letter-spacing:1px; }
    .header-line { width:48px; height:1px; background:var(--ring); margin:10px auto 0; opacity:0.6; transition:background .3s; }

    .btn-wrap { position:relative; margin-bottom:36px; display:flex; align-items:center; justify-content:center; width:230px; height:230px; }
    .ripple { position:absolute; border-radius:50%; border:1px solid var(--ring); pointer-events:none; opacity:0; }
    .ripple1 { inset:15px; }
    .ripple2 { inset:0; }
    .active .ripple1 { animation: ripple 2s infinite ease-out; }
    .active .ripple2 { animation: ripple 2s infinite ease-out 0.65s; }
    @keyframes ripple { 0%{transform:scale(1);opacity:.5} 100%{transform:scale(1.55);opacity:0} }

    #mic-btn {
      width:150px; height:150px; border-radius:50%;
      border:2px solid var(--ring);
      background:radial-gradient(circle at 38% 32%,#0d1e40,#060d1a);
      cursor:pointer; display:flex; flex-direction:column;
      align-items:center; justify-content:center; gap:6px;
      box-shadow:0 0 32px color-mix(in srgb,var(--ring) 27%,transparent);
      transition:border-color .3s,box-shadow .3s; outline:none; z-index:1;
    }
    #mic-btn:disabled { cursor:not-allowed; opacity:.7; }
    #btn-icon { font-size:38px; line-height:1; }
    #btn-label { font-size:10px; letter-spacing:1.5px; text-transform:uppercase; color:var(--ring); font-family:monospace; text-align:center; padding:0 10px; transition:color .3s; }

    #bubbles { width:100%; max-width:460px; display:flex; flex-direction:column; gap:10px; margin-bottom:14px; }
    .bubble-wrap { display:flex; flex-direction:column; }
    .bubble-wrap.left { align-items:flex-start; }
    .bubble-wrap.right { align-items:flex-end; }
    .bubble-label { font-size:10px; letter-spacing:2px; text-transform:uppercase; margin-bottom:4px; font-family:monospace; }
    .bubble-wrap.left .bubble-label { padding-left:6px; color:#ff6d00; }
    .bubble-wrap.right .bubble-label { padding-right:6px; color:#3d5afe; }
    .bubble-text { padding:10px 14px; font-size:14px; line-height:1.6; max-width:88%; color:#ccdaee; }
    .bubble-wrap.left .bubble-text { background:linear-gradient(135deg,#1a1a1a,#222); border:1px solid rgba(255,109,0,.18); border-radius:3px 14px 14px 14px; }
    .bubble-wrap.right .bubble-text { background:linear-gradient(135deg,#0a1e3a,#112840); border:1px solid rgba(61,90,254,.18); border-radius:14px 3px 14px 14px; }

    #error-box { display:none; background:#1a0505; border:1px solid #7a2020; border-radius:8px; padding:10px 16px; font-size:13px; color:#ff8a80; max-width:420px; text-align:center; margin-bottom:12px; }
    #error-box.show { display:block; }
    #nosupport { display:none; background:#1a1500; border:1px solid #7a6000; border-radius:8px; padding:12px 20px; color:#ffe082; font-size:13px; max-width:400px; text-align:center; }
    #nosupport.show { display:block; }

    #footer { display:none; align-items:center; gap:14px; margin-top:8px; }
    #footer.show { display:flex; }
    #exchange-count { font-size:11px; color:#2a4060; font-family:monospace; }
    #reset-btn { background:none; border:1px solid #1e3550; border-radius:6px; color:#3a6090; font-size:11px; padding:4px 12px; cursor:pointer; letter-spacing:1px; }
  </style>
</head>
<body>
<div id="app">
  <div class="header">
    <div class="header-sub">Юридический представитель</div>
    <h1>КоАП Защитник</h1>
    <div class="header-line"></div>
  </div>
  <div class="btn-wrap" id="btn-wrap">
    <div class="ripple ripple1"></div>
    <div class="ripple ripple2"></div>
    <button id="mic-btn" onclick="handleClick()">
      <span id="btn-icon">🎤</span>
      <span id="btn-label">Нажми — говори</span>
    </button>
  </div>
  <div id="bubbles"></div>
  <div id="error-box"></div>
  <div id="nosupport">Голосовой ввод не поддерживается. Используйте Яндекс Браузер или Chrome.</div>
  <div id="footer">
    <span id="exchange-count">0 обменов</span>
    <button id="reset-btn" onclick="resetAll()">Сбросить диалог</button>
  </div>
</div>
<script>
const SYSTEM_PROMPT = `Ты — юридический представитель водителя в Казахстане. Тебя слышит сотрудник дорожной полиции (ГАИ/ДПС). Ты говоришь ОТ ИМЕНИ водителя, защищая его права.

ТВОЯ РОЛЬ:
- Отвечай на вопросы и требования инспектора вместо водителя
- Говори уверенно, спокойно, юридически грамотно
- Ссылайся на конкретные статьи КоАП РК
- Отвечай КРАТКО — 2-4 предложения максимум

КЛЮЧЕВЫЕ ПРАВА ВОДИТЕЛЯ:
- Статья 623 КоАП РК: право не свидетельствовать против себя
- Статья 787 КоАП РК: право на юридическую помощь
- Инспектор ОБЯЗАН представиться: ФИО, звание, номер жетона
- Инспектор ОБЯЗАН объяснить причину остановки
- Водитель НЕ ОБЯЗАН выходить из машины без оснований
- Протокол: вправе не подписывать, вписать возражения
- Досмотр: только с понятыми или видеозаписью + протокол досмотра
- Эвакуация: только в присутствии водителя или с понятыми и видеофиксацией

ТАКТИКА:
- При давлении — спокойно напоминай о правах
- При намёке на взятку — "прошу оформить официально, составьте протокол"
- При несогласии — "не согласен, прошу внести в протокол"
- Всегда предлагай фиксировать на видео

ЯЗЫК: отвечай на языке инспектора (русский или казахский). По умолчанию — русский.
Говори прямо, без вводных слов.`;

let phase = "idle";
let history = [];
let recognition = null;

const COLORS = { idle:"#3d5afe", listening:"#ff6d00", thinking:"#9c27b0", speaking:"#00c853" };
const ICONS  = { idle:"🎤", listening:"⏹", thinking:"⏳", speaking:"⏹" };
const LABELS = { idle:"Нажми — говори", listening:"Слушаю... стоп", thinking:"Обрабатываю...", speaking:"Говорю... стоп" };

function setPhase(p) {
  phase = p;
  document.documentElement.style.setProperty("--ring", COLORS[p]);
  document.getElementById("btn-icon").textContent  = ICONS[p];
  document.getElementById("btn-label").textContent = LABELS[p];
  document.getElementById("btn-label").style.color = COLORS[p];
  document.getElementById("mic-btn").disabled = (p === "thinking");
  const active = p === "listening" || p === "speaking";
  document.getElementById("btn-wrap").classList.toggle("active", active);
}

function showError(msg) {
  const el = document.getElementById("error-box");
  el.textContent = msg; el.classList.add("show");
  setTimeout(() => el.classList.remove("show"), 7000);
}

function addBubble(side, label, text) {
  const bubbles = document.getElementById("bubbles");
  const old = bubbles.querySelector(".bubble-wrap." + side);
  if (old) old.remove();
  const wrap = document.createElement("div");
  wrap.className = "bubble-wrap " + side;
  wrap.innerHTML = `<div class="bubble-label">${label}</div><div class="bubble-text">${text}</div>`;
  bubbles.appendChild(wrap);
}

function updateFooter() {
  const count = Math.floor(history.length / 2);
  document.getElementById("footer").classList.toggle("show", count > 0);
  document.getElementById("exchange-count").textContent = count + " обмен(ов)";
}

function speak(text) {
  window.speechSynthesis.cancel();
  const utter = new SpeechSynthesisUtterance(text);
  utter.lang = "ru-RU"; utter.rate = 0.95;
  const voices = window.speechSynthesis.getVoices();
  const ruVoice = voices.find(v => v.lang.startsWith("ru"));
  if (ruVoice) utter.voice = ruVoice;
  utter.onend = () => setPhase("idle");
  utter.onerror = () => setPhase("idle");
  window.speechSynthesis.speak(utter);
}
window.speechSynthesis.onvoiceschanged = () => {};

async function askClaude(userText) {
  history.push({ role:"user", content:userText });
  try {
    const res = await fetch("https://api.anthropic.com/v1/messages", {
      method:"POST",
      headers:{ "Content-Type":"application/json" },
      body: JSON.stringify({ model:"claude-sonnet-4-20250514", max_tokens:1000, system:SYSTEM_PROMPT, messages:history })
    });
    const data = await res.json();
    if (data.error) throw new Error(data.error.message);
    const reply = data.content?.[0]?.text || "Не удалось получить ответ.";
    history.push({ role:"assistant", content:reply });
    updateFooter();
    addBubble("right", "⚖️ Представитель", reply);
    setPhase("speaking");
    speak(reply);
  } catch(err) {
    showError("Ошибка: " + err.message);
    setPhase("idle");
  }
}

function initRecognition() {
  const SR = window.SpeechRecognition || window.webkitSpeechRecognition;
  if (!SR) { document.getElementById("nosupport").classList.add("show"); document.getElementById("mic-btn").disabled=true; return; }
  recognition = new SR();
  recognition.lang = "ru-RU"; recognition.continuous = false; recognition.interimResults = false; recognition.maxAlternatives = 1;
  recognition.onresult = (e) => {
    const text = e.results[0][0].transcript.trim();
    if (!text) { setPhase("idle"); return; }
    addBubble("left","🚔 Инспектор", text);
    setPhase("thinking"); askClaude(text);
  };
  recognition.onerror = (e) => {
    if (e.error==="aborted"||e.error==="no-speech") { setPhase("idle"); return; }
    showError(e.error==="not-allowed" ? "Нет доступа к микрофону. Разрешите в настройках браузера." : "Ошибка: "+e.error);
    setPhase("idle");
  };
  recognition.onend = () => { if (phase==="listening") setPhase("idle"); };
}

function handleClick() {
  if (phase==="speaking") { window.speechSynthesis.cancel(); setPhase("idle"); return; }
  if (phase==="listening") { recognition?.stop(); return; }
  if (phase==="idle") {
    document.getElementById("error-box").classList.remove("show");
    setPhase("listening");
    try { recognition?.start(); }
    catch(e) { showError("Микрофон занят, попробуй ещё раз"); setPhase("idle"); }
  }
}

function resetAll() {
  window.speechSynthesis.cancel(); recognition?.abort();
  history = []; document.getElementById("bubbles").innerHTML = "";
  updateFooter(); setPhase("idle");
}

if ("serviceWorker" in navigator) navigator.serviceWorker.register("sw.js");
initRecognition();
</script>
</body>
</html>
