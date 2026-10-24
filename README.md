
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

    select, button, textarea { font: inherit; }

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
      min-height: 0;
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
      resize: vertical;
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

    small { color: var(--muted); }

    @media (max-width: 600px) {
      .container {
        height: calc(100dvh - 65px);
        padding: 10px;
      }

      .user { margin-left: 7%; }
      .assistant { margin-right: 5%; }

      .input-area { gap: 7px; }
      .input-area button { padding: 10px 14px; }
    }
  </style>
</head>

<body>
  <header>
    <div class="logo">✦ NOVA AI</div>
    <div class="status">● V2.0</div>
  </header>

  <main class="container">
    <select id="mode" aria-label="Pilih mode AI">
      <option value="general">Nova General</option>
      <option value="research">Research Mode</option>
      <option value="coding">Coding Mode</option>
      <option value="investigation">Investigation Mode</option>
      <option value="security">AI Security Lab</option>
    </select>

    <button class="secondary" id="newChat">
      + Percakapan Baru
    </button>

    <div id="chat" aria-live="polite">
      <div class="message assistant">Halo! Saya Nova AI V2.

Silakan pilih mode dan ajukan pertanyaan.

Saya siap membantu riset, pemrograman, analisis, dan pembelajaran AI.</div>
    </div>

    <small id="info">Nova AI siap digunakan.</small>

    <div class="input-area">
      <textarea
        id="prompt"
        rows="2"
        maxlength="12000"
        placeholder="Tanyakan sesuatu kepada Nova..."
        aria-label="Pesan untuk Nova AI"
      ></textarea>

      <button id="send">Kirim</button>
    </div>
  </main>

  <script>
    const chat = document.getElementById("chat");
    const input = document.getElementById("prompt");
    const sendButton = document.getElementById("send");
    const modeSelect = document.getElementById("mode");
    const info = document.getElementById("info");
    const newChatButton = document.getElementById("newChat");

    let messages = [];
    let isSending = false;

    const instructions = {
      general: "Jawab dengan jelas, objektif, dan informatif.",
      research: "Jelaskan topik secara mendalam. Bedakan fakta dan dugaan.",
      coding: "Bantu menulis, menjelaskan, dan memperbaiki kode.",
      investigation: "Analisis informasi secara netral dan pisahkan fakta dari dugaan.",
      security: "Bantu pembelajaran keamanan AI dan pengujian defensif."
    };

    function addMessage(role, text) {
      const element = document.createElement("div");
      element.className = "message " + role;
      element.textContent = text;
      chat.appendChild(element);
      chat.scrollTop = chat.scrollHeight;
      return element;
    }

    async function sendMessage() {
      if (isSending) return;

      const text = input.value.trim();
      if (!text) return;

      isSending = true;
      sendButton.disabled = true;
      modeSelect.disabled = true;
      info.textContent = "Nova AI sedang berpikir...";

      addMessage("user", text);
      input.value = "";

      const loading = addMessage("assistant", "Memproses pesan...");

      const mode = modeSelect.value;
      const content =
        "Mode: " + mode + "\n" +
        instructions[mode] +
        "\n\nPesan pengguna:\n" + text;

      const requestMessages = [
        ...messages,
        { role: "user", content }
      ].slice(-20);

      try {
        const response = await fetch("/api/chat", {
          method: "POST",
          headers: {
            "Content-Type": "application/json"
          },
          body: JSON.stringify({
            messages: requestMessages
          })
        });

        const data = await response.json();

        if (!response.ok) {
          throw new Error(
            data.error || "Server gagal memproses permintaan."
          );
        }

        if (typeof data.reply !== "string" || !data.reply.trim()) {
          throw new Error("Jawaban AI kosong.");
        }

        loading.textContent = data.reply;

        messages = [
          ...requestMessages,
          { role: "assistant", content: data.reply }
        ].slice(-20);

        info.textContent = "Nova AI siap digunakan.";
      } catch (error) {
        loading.textContent = "Gagal: " + error.message;
        info.textContent = "Periksa koneksi atau backend API.";
      } finally {
        isSending = false;
        sendButton.disabled = false;
        modeSelect.disabled = false;
        input.focus();
      }
    }

    function clearChat() {
      if (isSending) return;

      messages = [];
      chat.replaceChildren();

      addMessage(
        "assistant",
        "Percakapan baru dimulai. Ada yang ingin kamu tanyakan?"
      );

      info.textContent = "Nova AI siap digunakan.";
    }

    sendButton.addEventListener("click", sendMessage);
    newChatButton.addEventListener("click", clearChat);

    input.addEventListener("keydown", function(event) {
      if (event.key === "Enter" && !event.shiftKey) {
        event.preventDefault();
        sendMessage();
      }
    });
  </script>
</body>
</html>
