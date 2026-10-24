# Nova-ai
Nova ai

<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Nova AI V2</title>

<style>
:root {
  --bg: #09090f;
  --panel: #14141e;
  --accent: #8b5cf6;
  --text: #f5f5fa;
  --muted: #9999ad;
}

* { box-sizing: border-box; }

body {
  margin: 0;
  background: var(--bg);
  color: var(--text);
  font-family: Arial, sans-serif;
}

header {
  padding: 18px;
  border-bottom: 1px solid #292938;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.logo {
  font-size: 23px;
  font-weight: bold;
  color: #a78bfa;
}

.status {
  font-size: 12px;
  color: #4ade80;
}

.container {
  max-width: 900px;
  height: calc(100dvh - 65px);
  margin: auto;
  display: flex;
  flex-direction: column;
  padding: 15px;
}

select, button, textarea {
  font: inherit;
}

select {
  background: var(--panel);
  color: white;
  padding: 12px;
  border: 1px solid #333344;
  border-radius: 10px;
  width: 100%;
}

#chat {
  flex: 1;
  overflow-y: auto;
  padding: 20px 0;
}

.message {
  padding: 15px;
  border-radius: 14px;
  margin-bottom: 15px;
  line-height: 1.6;
  white-space: pre-wrap;
  overflow-wrap: anywhere;
}

.user {
  background: #312450;
  margin-left: 15%;
}

.assistant {
  background: var(--panel);
  margin-right: 10%;
  border: 1px solid #292938;
}

.input-area {
  display: flex;
  gap: 10px;
  padding: 12px 0;
}

textarea {
  flex: 1;
  background: var(--panel);
  color: white;
  border: 1px solid #333344;
  border-radius: 12px;
  padding: 14px;
  resize: none;
  min-width: 0;
}

button {
  background: var(--accent);
  color: white;
  border: none;
  border-radius: 10px;
  padding: 12px 20px;
  cursor: pointer;
}

button:hover { background: #7c3aed; }
button:disabled { opacity: .5; cursor: wait; }

.secondary {
  background: #252535;
  margin-top: 10px;
}

small {
  color: var(--muted);
}
</style>
</head>

<body>

<header>
  <div class="logo">✦ NOVA AI</div>
  <div class="status">● V2.0</div>
</header>

<main class="container">

  <select id="mode">
    <option value="general">Nova General</option>
    <option value="research">Research Mode</option>
    <option value="coding">Coding Mode</option>
    <option value="investigation">Investigation Mode</option>
    <option value="security">AI Security Lab</option>
  </select>

  <button class="secondary" onclick="clearChat()">
    + Percakapan Baru
  </button>

  <div id="chat" aria-live="polite">
    <div class="message assistant">
Halo! Saya Nova AI V2.

Silakan pilih mode dan ajukan pertanyaan.

Saya dapat membantu riset, pemrograman, investigasi, dan pembelajaran keamanan AI.
    </div>
  </div>

  <small id="info">Nova AI siap digunakan.</small>

  <div class="input-area">
    <textarea
      id="prompt"
      rows="2"
      maxlength="12000"
      placeholder="Tanyakan sesuatu kepada Nova..."
    ></textarea>

    <button id="send" onclick="sendMessage()">
      Kirim
    </button>
  </div>

</main>

<script>
let messages = [];
let busy = false;

const chat = document.getElementById("chat");
const promptInput = document.getElementById("prompt");
const sendButton = document.getElementById("send");
const info = document.getElementById("info");
const modeSelect = document.getElementById("mode");

const modeInstructions = {
  general:
    "Jawab secara jelas, objektif, dan informatif.",

  research:
    "Berikan penjelasan mendalam. Bedakan fakta, teori, dan ketidakpastian.",

  coding:
    "Bantu memahami dan menulis kode. Jelaskan cara kerja serta kesalahannya.",

  investigation:
    "Analisis informasi secara netral. Pisahkan fakta, dugaan, dan bukti.",

  security:
    "Bantu pembelajaran keamanan AI, pengujian defensif, dan analisis prompt injection."
};

function addMessage(role, content) {
  const div = document.createElement("div");

  div.className = "message " + role;
  div.textContent = content;

  chat.appendChild(div);
  chat.scrollTop = chat.scrollHeight;

  return div;
}

function clearChat() {
  if (busy) return;

  messages = [];
  chat.replaceChildren();

  addMessage(
    "assistant",
    "Percakapan baru dimulai. Ada yang ingin ditanyakan?"
  );
}

async function sendMessage() {
  if (busy) return;

  const input = promptInput.value.trim();

  if (!input) return;

  busy = true;
  sendButton.disabled = true;
  modeSelect.disabled = true;
  info.textContent = "Nova sedang berpikir...";

  const mode = modeSelect.value;

  const content =
    "[" + mode.toUpperCase() + " MODE]\n" +
    modeInstructions[mode] +
    "\n\nPertanyaan pengguna:\n" + input;

  addMessage("user", input);
  promptInput.value = "";

  const loading = addMessage(
    "assistant",
    "Nova sedang memproses permintaan..."
  );

  const nextMessages = [
    ...messages,
    { role: "user", content }
  ];

  try {
    const response = await fetch("/api/chat", {
      method: "POST",
      headers: {
        "Content-Type": "application/json"
      },
      body: JSON.stringify({
        messages: nextMessages
      })
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(
        data.error || "Gagal menghubungi server."
      );
    }

    loading.textContent = data.reply;

    messages = [
      ...nextMessages,
      {
        role: "assistant",
        content: data.reply
      }
    ];

    // Sesuaikan dengan batas backend: maksimal 20 pesan.
    if (messages.length > 18) {
      messages = messages.slice(-18);
    }

    info.textContent = "Respons selesai.";

  } catch (error) {
    loading.textContent =
      "Terjadi kesalahan: " + error.message;

    info.textContent = "Permintaan gagal.";

  } finally {
    busy = false;
    sendButton.disabled = false;
    modeSelect.disabled = false;
    promptInput.focus();
  }
}

promptInput.addEventListener("keydown", function(e) {
  if (e.key === "Enter" && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
});
</script>


</body>
</html>
