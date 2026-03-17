
<!DOCTYPE html>
<html lang="uk">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>ОСББ Космонавтів 52</title>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-app-compat.js"></script>
    <script src="https://www.gstatic.com/firebasejs/9.17.1/firebase-database-compat.js"></script>
    <style>
        body { font-family: sans-serif; background: #f0f4f8; padding: 15px; }
        .container { max-width: 500px; margin: auto; background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); }
        input, textarea, button { width: 100%; padding: 12px; margin: 5px 0; border-radius: 8px; border: 1px solid #ddd; box-sizing: border-box; }
        button { background: #0b2c5a; color: white; font-weight: bold; border: none; cursor: pointer; }
        .card { padding: 15px; margin-top: 10px; border-radius: 10px; border-left: 5px solid #0b2c5a; background: #fafafa; position: relative; }
        .admin-btn { display: none; background: #e74c3c; padding: 5px; margin-top: 5px; }
        .is-admin .admin-btn { display: block; }
        .chat { height: 150px; overflow-y: auto; background: #f9f9f9; border: 1px solid #ddd; padding: 10px; margin-top: 20px; font-size: 14px; }
    </style>
</head>
<body>
<div class="container">
    <h2>🚀 Космонавтів 52</h2>
    <input type="text" id="userName" placeholder="Прізвище та № квартири" oninput="checkAdmin(this.value)">
    <textarea id="userTask" placeholder="Що трапилося?"></textarea>
    <button onclick="addRequest()">ВІДПРАВИТИ</button>
    <div id="requestList"></div>
    <div class="chat" id="chatBox"></div>
    <div style="display:flex; gap:5px">
        <input type="text" id="chatMsg" placeholder="Повідомлення...">
        <button style="width:60px" onclick="sendChat()">OK</button>
    </div>
</div>

<script>
    const firebaseConfig = {
        apiKey: "AIzaSyCmQmxiQIRCP5psyfGyJFS5zh-wwez_wuQ",
        authDomain: "osbbkosmos52.firebaseapp.com",
        databaseURL: "https://osbbkosmos52-default-rtdb.firebaseio.com",
        projectId: "osbbkosmos52",
        appId: "1:608670954605:web:c546048e8de34158a47b64"
    };
    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    function checkAdmin(v) { if(v.toLowerCase() === 'адмін52') document.body.classList.add('is-admin'); }

    function addRequest() {
        const n = document.getElementById('userName').value;
        const t = document.getElementById('userTask').value;
        if(n && t) db.ref('requests/').push({ n, t });
        document.getElementById('userTask').value = '';
    }

    db.ref('requests/').on('value', s => {
        const l = document.getElementById('requestList'); l.innerHTML = "<b>Заявки:</b>";
        s.forEach(c => {
            const r = c.val();
            l.innerHTML += `<div class="card"><b>${r.n}</b><br>${r.t}<button class="admin-btn" onclick="db.ref('requests/${c.key}').remove()">Видалити 🗑</button></div>`;
        });
    });

    function sendChat() {
        const m = document.getElementById('chatMsg').value;
        const n = document.getElementById('userName').value || "Сусід";
        if(m) db.ref('messages/').push({ t: n + ": " + m });
        document.getElementById('chatMsg').value = "";
    }
    db.ref('messages/').limitToLast(15).on('value', s => {
        const b = document.getElementById('chatBox'); b.innerHTML = "";
        s.forEach(c => { b.innerHTML += `<div>${c.val().t}</div>`; });
        b.scrollTop = b.scrollHeight;
    });
</script>
</body>
</html>
