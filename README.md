<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Test Koneksi Google Apps Script</title>
<style>
body { font-family: sans-serif; max-width: 500px; margin: 50px auto; background: #f8f8f8; padding: 20px; border-radius: 12px; }
input, button { padding: 8px; margin-top: 10px; width: 100%; border-radius: 6px; border: 1px solid #ccc; }
button { background: #4caf50; color: white; cursor: pointer; }
button:hover { background: #45a049; }
#output { margin-top: 20px; white-space: pre-wrap; background: #fff; padding: 10px; border-radius: 8px; }
</style>
</head>
<body>
<h2>🔍 Tes Koneksi Backend</h2>
<input id="username" placeholder="Username" value="admin">
<input id="password" placeholder="Password" type="password" value="admin123">
<button onclick="login()">Coba Login</button>
<pre id="output"></pre>

<script>
const SCRIPT_URL = "https://script.google.com/macros/s/AKfycbxbpYBYuWnzKXplos-HTlEHBWhndXK6ksoB6EnfDKyA2eOgfodHIR404WkutAaDRu4UMA/exec"; // ganti dengan URL-mu

async function login() {
  const username = document.getElementById('username').value;
  const password = document.getElementById('password').value;
  const output = document.getElementById('output');
  output.textContent = "🔄 Mengirim...";
  try {
    const res = await fetch(SCRIPT_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ action: 'LOGIN', username, password })
    });
    const data = await res.json();
    output.textContent = JSON.stringify(data, null, 2);
  } catch (err) {
    output.textContent = "❌ Error: " + err.message;
  }
}
</script>
</body>
</html>
