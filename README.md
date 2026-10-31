<!DOCTYPE html>
<html lang="kk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0">
<title>Менің ИИ-ім</title>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; -webkit-tap-highlight-color: transparent; }
  body {
    font-family: -apple-system, "Segoe UI", sans-serif;
    background: #0f0f1a;
    min-height: 100vh;
    min-height: 100dvh;
    display: flex;
    flex-direction: column;
    color: #fff;
  }
  .header {
    background: #1a1a2e;
    padding: 16px 18px;
    padding-top: max(16px, env(safe-area-inset-top));
    font-size: 17px;
    font-weight: 600;
    display: flex;
    align-items: center;
    gap: 8px;
    border-bottom: 1px solid #2a2a40;
  }
  .status {
    font-size: 12px;
    font-weight: 400;
    color: #8888aa;
    margin-left: auto;
  }
  .messages {
    flex: 1;
    overflow-y: auto;
    padding: 16px;
    display: flex;
    flex-direction: column;
    gap: 10px;
    -webkit-overflow-scrolling: touch;
  }
  .msg {
    max-width: 85%;
    padding: 11px 14px;
    border-radius: 16px;
    line-height: 1.45;
    white-space: pre-wrap;
    word-wrap: break-word;
    font-size: 15px;
    animation: fadeIn 0.2s ease;
  }
  @keyframes fadeIn { from { opacity: 0; transform: translateY(4px); } to { opacity: 1; transform: none; } }
  .user {
    align-self: flex-end;
    background: #667eea;
    border-bottom-right-radius: 4px;
  }
  .bot {
    align-self: flex-start;
    background: #1e1e32;
    border-bottom-left-radius: 4px;
  }
  .input-area {
    display: flex;
    padding: 12px;
    padding-bottom: max(12px, env(safe-area-inset-bottom));
    gap: 8px;
    background: #1a1a2e;
    border-top: 1px solid #2a2a40;
  }
  textarea {
    flex: 1;
    resize: none;
    border: 1px solid #2a2a40;
    background: #0f0f1a;
    color: #fff;
    border-radius: 20px;
    padding: 10px 14px;
    font-family: inherit;
    font-size: 16px;
    outline: none;
    max-height: 120px;
    line-height: 1.4;
  }
  textarea:focus { border-color: #667eea; }
  button {
    background: #667eea;
    color: #fff;
    border: none;
    width: 44px;
    height: 44px;
    border-radius: 50%;
    cursor: pointer;
    font-size: 18px;
    display: flex;
    align-items: center;
    justify-content: center;
    flex-shrink: 0;
  }
  button:disabled { opacity: 0.4; }
  .progress-box {
    padding: 10px 16px;
    background: #1a1a2e;
    font-size: 12px;
    color: #8888aa;
    display: none;
  }
  .progress-box.show { display: block; }
  .progress-bar {
    height: 4px;
    background: #2a2a40;
    border-radius: 2px;
    margin-top: 6px;
    overflow: hidden;
  }
  .progress-fill {
    height: 100%;
    background: #667eea;
    width: 0%;
    transition: width 0.3s;
  }
  .welcome {
    text-align: center;
    color: #6666aa;
    padding: 40px 20px;
    font-size: 14px;
    line-height: 1.6;
  }
</style>
</head>
<body>

<div class="header">
  🤖 Менің ИИ-ім
  <span class="status" id="status">жүктелуде...</span>
</div>

<div class="progress-box" id="progressBox">
  <div id="progressText">Модель жүктелуде... (бірінші рет 2-3 минут)</div>
  <div class="progress-bar"><div class="progress-fill" id="progressFill"></div></div>
</div>

<div class="messages" id="messages">
  <div class="welcome" id="welcome">
    Сәлем! Мен сенің жеке ИИ-іңмін.<br>
    Барлық жауаптар тек телефоныңда өтеді.<br>
    Ешкім көрмейді, ешбір шектеу жоқ.
  </div>
</div>

<div class="input-area">
  <textarea id="input" rows="1" placeholder="Хабарлама жаз..." disabled></textarea>
  <button id="send" disabled>➤</button>
</div>

<script type="module">
import * as webllm from "https://esm.run/@mlc-ai/web-llm";

const messagesEl = document.getElementById("messages");
const inputEl = document.getElementById("input");
const sendBtn = document.getElementById("send");
const statusEl = document.getElementById("status");
const progressBox = document.getElementById("progressBox");
const progressText = document.getElementById("progressText");
const progressFill = document.getElementById("progressFill");
const welcomeEl = document.getElementById("welcome");

const SYSTEM_PROMPT = "Сен — пайдаланушының жеке көмекшісісің. Қазақ тілінде, достық стильде, нақты жауап бер.";

let engine = null;
const history = [{ role: "system", content: SYSTEM_PROMPT }];

const MODEL = "Llama-3.2-1B-Instruct-q4f32_1-MLC";

async function initModel() {
  try {
    progressBox.classList.add("show");
    statusEl.textContent = "жүктелуде...";

    engine = await webllm.CreateMLCEngine(MODEL, {
      initProgressCallback: (report) => {
        progressText.textContent = report.text;
        if (report.progress !== undefined) {
          progressFill.style.width = (report.progress * 100).toFixed(0) + "%";
        }
      }
    });

    progressBox.classList.remove("show");
    statusEl.textContent = "дайын ✓";
    statusEl.style.color = "#4ade80";
    inputEl.disabled = false;
    sendBtn.disabled = false;
    welcomeEl.textContent = "Дайын! Сұрағыңды жаз 👇";

  } catch (e) {
    statusEl.textContent = "қате";
    progressText.textContent = "Қате: " + e.message;
    console.error(e);
  }
}

function addMessage(text, cls) {
  if (welcomeEl.parentNode) welcomeEl.remove();
  const div = document.createElement("div");
  div.className = "msg " + cls;
  div.textContent = text;
  messagesEl.appendChild(div);
  messagesEl.scrollTop = messagesEl.scrollHeight;
  return div;
}

async function send() {
  const text = inputEl.value.trim();
  if (!text || !engine) return;

  inputEl.value = "";
  inputEl.style.height = "auto";
  addMessage(text, "user");
  history.push({ role: "user", content: text });

  sendBtn.disabled = true;
  const botDiv = addMessage("...", "bot");

  try {
    const reply = await engine.chat.completions.create({
      messages: history,
      temperature: 0.7,
      max_tokens: 512,
      stream: true,
      stream_options: { include_usage: false }
    });

    let full = "";
    botDiv.textContent = "";
    for await (const chunk of reply) {
      const delta = chunk.choices[0]?.delta?.content || "";
      full += delta;
      botDiv.textContent = full;
      messagesEl.scrollTop = messagesEl.scrollHeight;
    }
    history.push({ role: "assistant", content: full });

  } catch (e) {
    botDiv.textContent = "Қате: " + e.message;
  } finally {
    sendBtn.disabled = false;
    inputEl.focus();
  }
}

sendBtn.addEventListener("click", send);
inputEl.addEventListener("input", () => {
  inputEl.style.height = "auto";
  inputEl.style.height = Math.min(inputEl.scrollHeight, 120) + "px";
});
inputEl.addEventListener("keydown", (e) => {
  if (e.key === "Enter" && !e.shiftKey) {
    e.preventDefault();
    send();
  }
});

initModel();
</script>

</body>
</html>