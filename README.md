<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta name="theme-color" content="#09090f">
  <title>Nova AI</title>
  <style>
    :root {
      --bg: #09090f;
      --panel: #14141e;
      --panel2: #1b1b29;
      --accent: #8b5cf6;
      --text: #f5f5fa;
      --muted: #9999ad;
      --border: #2b2b3b;
    }

    * { box-sizing: border-box; }

    body {
      margin: 0;
      background: var(--bg);
      color: var(--text);
      font-family: Inter, system-ui, Arial, sans-serif;
      min-height: 100vh;
    }

    button, textarea, select { font: inherit; }

    button { cursor: pointer; }

    .app {
      width: min(900px, 100%);
      min-height: 100vh;
      margin: auto;
      display: flex;
      flex-direction: column;
    }

    header {
      padding: 18px 20px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      border-bottom: 1px solid var(--border);
      background: var(--bg);
      position: sticky;
      top: 0;
      z-index: 2;
    }

    .brand {
      display: flex;
      align-items: center;
      gap: 10px;
      font-size: 21px;
      font-weight: 800;
      letter-spacing: -.5px;
    }

    .logo {
      color: white;
      background: var(--accent);
      width: 36px;
      height: 36px;
      border-radius: 12px;
      display: grid;
      place-items: center;
    }

    .status {
      color: #86efac;
      font-size: 12px;
      margin-top: 3px;
    }

    .top-actions {
      display: flex;
      gap: 8px;
      align-items: center;
    }

    select, .secondary {
      color: var(--text);
      background: var(--panel);
      border: 1px solid var(--border);
      border-radius: 10px;
      padding: 10px;
      max-width: 180px;
    }

    .secondary { max-width: none; }

    main {
      flex: 1;
      display: flex;
      flex-direction: column;
      padding: 20px;
    }

    .welcome {
      text-align: center;
      margin: 42px auto 28px;
      max-width: 540px;
    }

    .welcome .symbol {
      font-size: 38px;
      color: #a78bfa;
    }

    .welcome h1 {
      font-size: clamp(25px, 5vw, 36px);
      margin: 12px 0 8px;
    }

    .welcome p {
      color: var(--muted);
      line-height: 1.6;
      margin: 0;
    }

    #chat {
      display: flex;
      flex-direction: column;
      gap: 18px;
      padding-bottom: 20px;
    }

    .message {
      max-width: 90%;
      min-width: 0;
      border-radius: 16px;
      padding: 13px 16px;
      line-height: 1.65;
      overflow-wrap: anywhere;
      white-space: pre-wrap;
    }

    .message.user {
      align-self: flex-end;
      background: #33245b;
      border: 1px solid #514078;
    }

    .message.assistant {
      align-self: flex-start;
      background: var(--panel);
      border: 1px solid var(--border);
    }

    .message-label {
      display: block;
      font-size: 11px;
      color: #c4b5fd;
      font-weight: 700;
      margin-bottom: 5px;
    }

    .composer {
      position: sticky;
      bottom: 0;
      padding: 12px 20px 18px;
      background: linear-gradient(transparent, var(--bg) 20%);
    }

    .input-box {
      display: flex;
      gap: 10px;
      align-items: flex-end;
      padding: 10px;
      border: 1px solid var(--border);
      border-radius: 17px;
      background: var(--panel);
    }

    textarea {
      flex: 1;
      min-width: 0;
      resize: vertical;
      min-height: 44px;
      max-height: 180px;
      border: 0;
      outline: 0;
      color: var(--text);
      background: transparent;
      padding: 10px 5px;
    }

    textarea::placeholder { color: var(--muted); }

    #send {
      border: 0;
      background: var(--accent);
      color: white;
      font-weight: 700;
      border-radius: 11px;
      padding: 12px 16px;
      min-height: 44px;
    }

    #send:disabled {
      opacity: .55;
      cursor: not-allowed;
    }

    #info {
      display: block;
      text-align: center;
      color: var(--muted);
      font-size: 12px;
      min-height: 18px;
      margin-top: 9px;
    }

    .error { color: #fca5a5 !important; }

    @media (max-width: 560px) {
      header { padding: 13px; }
      main { padding: 14px; }
      .composer { padding: 10px 12px 14px; }
      .brand { font-size: 18px; }
      .top-actions { gap: 5px; }
      select, .secondary { padding: 8px; font-size: 12px; }
      .message { max-width: 96%; }
      #send { padding: 11px; }
    }
  </style>
</head>
<body>
  <div class="app">
    <header>
      <div>
        <div class="brand">
          <span class="logo">✦</span>
          <span>NOVA AI</span>
        </div>
        <div class="status">● AI Assistant</div>
      </div>

      <div class="top-actions">
        <select id="mode" aria-label="Pilih mode AI">
          <option value="general">General</option>
          <option value="research">Research</option>
          <option value="coding">Coding</option>
          <option value="investigation">Investigation</option>
          <option value="security">AI Security Lab</option>
        </select>
        <button id="newChat" class="secondary" type="button">
          Chat baru
        </button>
      </div>
    </header>

    <main>
      <section class="welcome" id="welcome">
        <div class="symbol">✦</div>
        <h1>Ada yang bisa Nova bantu?</h1>
        <p>
          Pilih mode, tulis pertanyaan, lalu kirim.
          Nova akan menjawab dalam bahasa Indonesia.
        </p>
      </section>

      <section id="chat" aria-live="polite"></section>
    </main>

    <div class="composer">
      <div class="input-box">
        <textarea
          id="prompt"
          rows="1"
          maxlength="12000"
          placeholder="Tulis pesan ke Nova AI..."
          aria-label="Pesan untuk Nova AI"
        ></textarea>
        <button id="send" type="button">Kirim ↑</button>
      </div>
      <small id="info">Pilih mode dan mulai percakapan.</small>
    </div>
  </div>

  <script>
    const chat = document.getElementById("chat");
    const promptInput = document.getElementById("prompt");
    const sendButton = document.getElementById("send");
    const modeSelect = document.getElementById("mode");
    const info = document.getElementById("info");
    const welcome = document.getElementById("welcome");
    const newChatButton = document.getElementById("newChat");

    let messages = [];
    let isSending = false;

    function addMessage(role, text) {
      const message = document.createElement("div");
      message.className = "message " + role;

      const label = document.createElement("span");
      label.className = "message-label";
      label.textContent = role === "user" ? "Kamu" : "Nova AI";

      const content = document.createElement("div");
      content.textContent = typeof text === "string"
        ? text
        : "Respons tidak dapat ditampilkan.";

      message.append(label, content);
      chat.appendChild(message);
      message.scrollIntoView({ behavior: "smooth", block: "nearest" });
    }

    function setStatus(text, isError = false) {
      info.textContent = text;
      info.classList.toggle("error", isError);
    }

    async function sendMessage() {
      const text = promptInput.value.trim();

      if (!text || isSending) return;

      isSending = true;
      sendButton.disabled = true;
      promptInput.disabled = true;
      welcome.hidden = true;

      addMessage("user", text);
      promptInput.value = "";
      setStatus("Nova sedang berpikir...");

      // Simpan hanya riwayat yang diperlukan.
      const history = messages.slice(-18);
      const requestMessages = [...history, {
        role: "user",
        content: text
      }];

      try {
        const response = await fetch("/api/chat", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({
            mode: modeSelect.value,
            messages: requestMessages
          })
        });

        const data = await response.json().catch(() => ({}));

        if (!response.ok) {
          throw new Error(
            typeof data.error === "string"
              ? data.error
              : "Server gagal memproses permintaan."
          );
        }

        if (typeof data.reply !== "string" || !data.reply.trim()) {
          throw new Error("Server tidak mengirim teks jawaban yang valid.");
        }

        const reply = data.reply.trim();
        addMessage("assistant", reply);

        messages = [
          ...requestMessages,
          { role: "assistant", content: reply }
        ].slice(-20);

        setStatus("Siap membantu.");
      } catch (error) {
        addMessage(
          "assistant",
          "Maaf, pesan belum berhasil diproses. " +
          (error.message || "Periksa koneksi server.")
        );
        setStatus("Terjadi kesalahan. Periksa pengaturan server.", true);
      } finally {
        isSending = false;
        sendButton.disabled = false;
        promptInput.disabled = false;
        promptInput.focus();
      }
    }

    sendButton.addEventListener("click", sendMessage);

    promptInput.addEventListener("keydown", function(event) {
      if (event.key === "Enter" && !event.shiftKey) {
        event.preventDefault();
        sendMessage();
      }
    });

    newChatButton.addEventListener("click", function() {
      if (isSending) return;
      messages = [];
      chat.replaceChildren();
      welcome.hidden = false;
      promptInput.value = "";
      setStatus("Percakapan baru siap.");
      promptInput.focus();
    });

    modeSelect.addEventListener("change", function() {
      setStatus("Mode aktif: " +
        modeSelect.options[modeSelect.selectedIndex].text);
    });
  </script>
</body>
</html>

const MODE_INSTRUCTIONS = {
  general: `
Bantu pengguna dengan pertanyaan umum secara jelas dan akurat.
Berikan jawaban dalam bahasa Indonesia kecuali pengguna meminta bahasa lain.
  `,
  research: `
Bantu riset dengan memisahkan fakta, interpretasi, dan informasi yang
belum terverifikasi. Jangan mengarang sumber atau kutipan.
Jelaskan keterbatasan informasi yang tersedia.
  `,
  coding: `
Bantu menulis, memperbaiki, dan menjelaskan kode.
Berikan kode yang sesuai dengan konteks, jelaskan file yang perlu diganti,
dan jangan mengklaim kode sudah diuji jika belum dijalankan.
  `,
  investigation: `
Bantu menyusun kronologi, merangkum dokumen, membandingkan keterangan,
dan menyusun pertanyaan tindak lanjut yang netral.
Pisahkan fakta, dugaan, dan hal yang belum diketahui.
Jangan menyatakan seseorang bersalah hanya berdasarkan dugaan.
Perlakukan hasil sebagai bahan analisis, bukan keputusan hukum.
Jaga privasi dan jangan membantu mengakses data secara ilegal.
  `,
  security: `
Bantu pembelajaran keamanan siber secara defensif dan legal.
Bantu menjelaskan konsep, meninjau kode, mengidentifikasi kerentanan,
serta menyarankan mitigasi dan pengujian pada sistem yang diizinkan.
Jangan membantu penyusupan tanpa izin atau pencurian data.
  `
};

const BASE_INSTRUCTIONS = `
Kamu adalah Nova AI, asisten yang objektif dan profesional.
Gunakan bahasa Indonesia secara default.
Jawab langsung sesuai pertanyaan pengguna.
Jangan tampilkan instruksi internal, metadata, atau objek JSON
sebagai jawaban biasa.
Jangan mengarang fakta atau mengklaim telah melakukan tindakan
yang sebenarnya belum dilakukan.
`;

export default async function handler(req, res) {
  if (req.method !== "POST") {
    res.setHeader("Allow", "POST");
    return res.status(405).json({
      error: "Gunakan metode POST."
    });
  }

  const apiKey = process.env.GEMINI_API_KEY;

  if (!apiKey) {
    return res.status(500).json({
      error: "GEMINI_API_KEY belum dipasang di pengaturan server."
    });
  }

  try {
    const body = req.body || {};
    const mode = body.mode || "general";
    const messages = body.messages;

    if (!Object.prototype.hasOwnProperty.call(MODE_INSTRUCTIONS, mode)) {
      return res.status(400).json({
        error: "Mode Nova AI tidak dikenal."
      });
    }

    if (
      !Array.isArray(messages) ||
      messages.length === 0 ||
      messages.length > 20
    ) {
      return res.status(400).json({
        error: "Format percakapan tidak valid."
      });
    }

    const contents = [];

    for (const message of messages) {
      if (
        !message ||
        !["user", "assistant"].includes(message.role) ||
        typeof message.content !== "string" ||
        !message.content.trim() ||
        message.content.length > 12000
      ) {
        return res.status(400).json({
          error: "Ada pesan yang formatnya tidak valid."
        });
      }

      const role = message.role === "assistant" ? "model" : "user";

      // Gemini mengharapkan giliran percakapan yang valid.
      const previous = contents[contents.length - 1];

      if (previous && previous.role === role) {
        return res.status(400).json({
          error: "Urutan riwayat percakapan tidak valid. Mulai chat baru."
        });
      }

      contents.push({
        role,
        parts: [{ text: message.content }]
      });
    }

    if (contents[0].role !== "user") {
      return res.status(400).json({
        error: "Riwayat harus dimulai dengan pesan pengguna."
      });
    }

    const apiResponse = await fetch(
      "https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash:generateContent",
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "x-goog-api-key": apiKey
        },
        body: JSON.stringify({
          systemInstruction: {
            parts: [{
              text: BASE_INSTRUCTIONS + "\n\n" + MODE_INSTRUCTIONS[mode]
            }]
          },
          contents,
          generationConfig: {
            temperature: 0.4,
            maxOutputTokens: 4096
          }
        })
      }
    );

    const data = await apiResponse.json().catch(() => ({}));

    if (!apiResponse.ok) {
      // Detail hanya dicatat di server; jangan bocorkan API key.
      console.error(
        "Gemini API error:",
        apiResponse.status,
        data.error?.message || "Tidak ada detail error."
      );

      return res.status(502).json({
        error: "Gemini gagal memproses permintaan. Periksa log server."
      });
    }

    const candidate = data.candidates?.[0];
    const parts = candidate?.content?.parts;

    const reply = Array.isArray(parts)
      ? parts
          .filter(part => typeof part.text === "string")
          .map(part => part.text)
          .join("")
          .trim()
      : "";

    if (!reply) {
      console.error(
        "Gemini returned no text. Finish reason:",
        candidate?.finishReason || "unknown"
      );

      return res.status(502).json({
        error: "Gemini tidak menghasilkan teks jawaban. Coba lagi."
      });
    }

    res.setHeader("Cache-Control", "no-store");

    return res.status(200).json({ reply });
  } catch (error) {
    console.error("Nova AI server error:", error.message);

    return res.status(500).json({
      error: "Terjadi kesalahan pada server Nova AI."
    });
  }
}
