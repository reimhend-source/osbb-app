
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ОСББ: Космонавтів 52</title>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-database-compat.js"></script>
    <style>
        :root { --space-blue: #0b2c5a; --space-gold: #ffb800; }
        body { font-family: 'Segoe UI', sans-serif; background-color: #f0f4f8; margin: 0; padding: 10px; color: #333; }
        .container { max-width: 500px; margin: auto; background: white; padding: 20px; border-radius: 20px; box-shadow: 0 8px 20px rgba(0,0,0,0.15); }
        .header { text-align: center; margin-bottom: 20px; }
        .header img { width: 100%; max-width: 280px; height: auto; border-radius: 10px; }
        .stats-bar { display: flex; gap: 10px; margin-bottom: 20px; }
        .stat-card { flex: 1; background: #f8faff; padding: 12px; border-radius: 12px; text-align: center; border: 1px solid #d1d9e6; }
        .stat-card span { display: block; font-size: 22px; font-weight: bold; color: var(--space-blue); }
        input, textarea, select { width: 100%; padding: 12px; margin: 6px 0; border-radius: 10px; border: 2px solid #eee; box-sizing: border-box; font-size: 16px; outline: none; }
        button { width: 100%; padding: 14px; margin-top: 10px; background: var(--space-blue); color: white; border: none; border-radius: 10px; font-weight: bold; cursor: pointer; }
        .card { padding: 15px; margin-bottom: 12px; border-radius: 12px; border-left: 8px solid #ccc; background: #fff; border: 1px solid #eee; }
        .card.done { opacity: 0.6; text-decoration: line-through; background: #f1f1f1; border-left-color: #2ecc71 !important; }
        .badge { font-size: 10px; padding: 4px 10px; border-radius: 20px; color: white; float: right; font-weight: bold; }
        .chat-box { height: 160px; overflow-y: auto; background: #f0f2f5; padding: 12px; border-radius: 12px; margin-bottom: 10px; }
        .msg { background: white; padding: 10px; border-radius: 12px; margin-bottom: 6px; font-size: 14px; border-bottom: 2px solid #ddd; }
    </style>
</head>
<body>
<div class="container">
    <div class="header">
        <img src="logo.jpg" alt="ОСББ Космонавтів 52">
    </div>
    <div class="stats-bar">
        <div class="stat-card"><span id="totalCount">0</span><small>В роботі</small></div>
        <div class="stat-card"><span id="doneCount" style="color:#27ae60">0</span><small>Виконано</small></div>
    </div>
    <div style="background: #f8f9fa; padding: 15px; border-radius: 15px; margin-bottom: 20px;">
        <input type="text" id="userName" placeholder="Прізвище та № квартири">
        <select id="category">
            <option value="general">Загальне питання</option>
            <option value="electric">⚡ Електрика</option>
            <option value="plumbing">💧 Сантехніка</option>
            <option value="construction">🏗 Ремонтні роботи</option>
        </select>
        <textarea id="userTask" rows="2" placeholder="Опишіть ситуацію..."></textarea>
        <button onclick="addRequest()">ВІДПРАВИТИ ЗАЯВКУ</button>
    </div>
    <div class="section">
        <h3 style="color: var(--space-blue);">📋 Активні звернення</h3>
        <div id="requestList"></div>
    </div>
    <div class="section">
        <h3 style="color: var(--space-blue);">💬 Чат мешканців</h3>
        <div id="chatBox" class="chat-box"></div>
        <div style="display: flex; gap: 8px;">
            <input type="text" id="chatMsg" placeholder="Повідомлення..." style="margin:0;">
            <button onclick="sendChat()" style="width: 80px; margin:0; padding:10px;">OK</button>
        </div>
    </div>
</div>
<script>
    const firebaseConfig = {
        apiKey: "AIzaSyCmQmxiQIRCP5psyfGyJFS5zh-wwez_wuQ",
        authDomain: "osbbkosmos52.firebaseapp.com",
        databaseURL: "https://osbbkosmos52-default-rtdb.firebaseio.com",
        projectId: "osbbkosmos52",
        storageBucket: "osbbkosmos52.firebasestorage.app",
        messagingSenderId: "608670954605",
        appId: "1:608670954605:web:c546048e8de34158a47b64"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();
    function addRequest() {
        const name = document.getElementById('userName').value;
        const task = document.getElementById('userTask').value;
        const category = document.getElementById('category').value;
        if (!name || !task) return alert("Заповніть поля!");
        db.ref('requests/').push({ name, task, category, done: false, time: Date.now() });
        document.getElementById('userTask').value = '';
    }
    db.ref('requests/').on('value', (snapshot) => {
        const list = document.getElementById('requestList'); list.innerHTML = "";
        const data = snapshot.val();
        let active = 0, completed = 0;
        if (data) {
            for (let id in data) {
                const req = data[id]; req.done ? completed++ : active++;
                const div = document.createElement('div');
                div.className = `card cat-${req.category} ${req.done ? 'done' : ''}`;
                div.innerHTML = `<span class="badge" style="background:var(--space-blue)">${req.category}</span><strong>${req.name}</strong><p>${req.task}</p><div style="display:flex; gap:8px;"><button style="width:auto; padding:6px 12px; font-size:12px; background:#2ecc71;" onclick="toggleDone('${id}', ${req.done})">Статус</button><button style="width:auto; padding:6px 12px; font-size:12px; background:#e74c3c;" onclick="deleteReq('${id}')">🗑</button></div>`;
                list.prepend(div);
            }
        }
        document.getElementById('totalCount').innerText = active;
        document.getElementById('doneCount').innerText = completed;
    });
    function toggleDone(id, current) { db.ref('requests/' + id).update({ done: !current }); }
    function deleteReq(id) { if (confirm("Видалити?")) db.ref('requests/' + id).remove(); }
    function sendChat() {
        const msg = document.getElementById('chatMsg').value;
        const name = document.getElementById('userName').value || "Сусід";
        if (!msg) return;
        db.ref('messages/').push({ text: `🚀 ${name}: ${msg}`, time: Date.now() });
        document.getElementById('chatMsg').value = "";
    }
    db.ref('messages/').limitToLast(15).on('value', (snapshot) => {
        const box = document.getElementById('chatBox'); box.innerHTML = "";
        snapshot.forEach(child => {
            const m = child.val(); const d = document.createElement('div');
            d.className = "msg"; d.innerText = m.text; box.appendChild(d);
        });
        box.scrollTop = box.scrollHeight;
    });
</script>
</body>
</html>
