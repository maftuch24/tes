[dddd.html](https://github.com/user-attachments/files/22756020/dddd.html)
<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8" />
  <title>Cek Backend Apps Script</title>
  <style>
    body { font-family: sans-serif; padding: 20px; background: #f9fafb; color: #333; }
    button { padding: 10px 16px; margin: 5px; border: none; border-radius: 6px; cursor: pointer; }
    button:hover { opacity: 0.8; }
    #result { margin-top: 20px; white-space: pre-wrap; background: #fff; border: 1px solid #ddd; padding: 10px; border-radius: 6px; }
  </style>
</head>
<body>
  <h2>🔍 Cek Backend Google Apps Script</h2>
  <p>Klik salah satu tombol di bawah untuk menguji respons backend.</p>

  <button style="background:#2563eb;color:white" onclick="testAction('LOGIN')">Tes LOGIN</button>
  <button style="background:#059669;color:white" onclick="testAction('GET_ALL_DATA')">Tes GET_ALL_DATA</button>
  <button style="background:#f59e0b;color:white" onclick="testAction('CONFIRM_PAYMENT')">Tes CONFIRM_PAYMENT</button>

  <div id="result">Hasil akan muncul di sini...</div>

  <script>
    const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxbpYBYuWnzKXplos-HTlEHBWhndXK6ksoB6EnfDKyA2eOgfodHIR404WkutAaDRu4UMA/exec";

    async function testAction(action) {
      const resultBox = document.getElementById('result');
      resultBox.textContent = `⏳ Menguji action: ${action}...`;

      try {
        const payload = { action };
        if (action === 'LOGIN') {
          payload.username = 'admin';
          payload.password = 'admin123';
        }

        const res = await fetch(SCRIPT_URL, {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify(payload)
        });

        const text = await res.text();
        try {
          const json = JSON.parse(text);
          resultBox.textContent = `✅ Respon JSON dari server:\n${JSON.stringify(json, null, 2)}`;
        } catch (err) {
          resultBox.textContent = `⚠️ Server tidak mengembalikan JSON valid:\n${text}`;
        }
      } catch (error) {
        resultBox.textContent = `❌ Gagal menghubungi server:\n${error}`;
      }
    }
  </script>
</body>
</html>
