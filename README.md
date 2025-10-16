<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>婚禮跑步遊戲</title>
  <style>
    body { text-align: center; font-family: sans-serif; background: #fff0f5; }
    h1 { color: #d63384; }
    .track { margin: 20px auto; width: 90%; border-bottom: 2px dashed #aaa; position: relative; }
    .runner {
      position: absolute;
      bottom: 0;
      font-size: 24px;
      transition: left 0.2s;
      white-space: nowrap;
    }
    .label {
      position: absolute;
      bottom: 30px;
      font-size: 14px;
      background: rgba(255,255,255,0.8);
      padding: 2px 6px;
      border-radius: 6px;
    }
    #clickBtn, #resetBtn, #startBtn { margin-top: 20px; padding: 10px 20px; font-size: 18px; }
    #winner { font-size: 24px; color: green; margin-top: 20px; font-weight: bold; }
    #countdown { font-size: 36px; color: red; margin-top: 20px; font-weight: bold; }
  </style>
</head>
<body>
  <h1>🏃 婚禮點擊跑步比賽 🎉</h1>
  <input id="playerName" placeholder="輸入名字">
  <button id="joinBtn">加入遊戲</button>
  <br>
  <button id="clickBtn" disabled>點我跑！</button>
  <button id="startBtn">開始遊戲</button>
  <button id="resetBtn">重新開始</button>
  <div id="raceArea"></div>
  <div id="countdown"></div>
  <div id="winner"></div>

  <!-- Firebase CDN -->
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-app.js"></script>
  <script src="https://www.gstatic.com/firebasejs/8.10.0/firebase-database.js"></script>

  <script>
    // 🔑 換成你的專案 config
    var firebaseConfig = {
      apiKey: "AIzaSyAQzK5BX7q7uetIKxKyf86Xxxxxxxx",
      authDomain: "index-63c1d.firebaseapp.com",
      databaseURL: "https://index-63c1d-default-rtdb.firebaseio.com",
      projectId: "index-63c1d",
      storageBucket: "index-63c1d.appspot.com",
      messagingSenderId: "259240910482",
      appId: "1:259240910482:web:4e2d9a57fca6367848904b",
      measurementId: "G-YF4Y2S57DV"
    };
    firebase.initializeApp(firebaseConfig);
    var db = firebase.database();

    let playerName = "";
    let gameOver = false;
    let gameStarted = false;
    const raceArea = document.getElementById("raceArea");
    const winnerDiv = document.getElementById("winner");
    const countdownDiv = document.getElementById("countdown");
    const FINISH_LINE = 200; // ✅ 200 步獲勝

    // 加入遊戲
    document.getElementById("joinBtn").onclick = () => {
      playerName = document.getElementById("playerName").value.trim();
      if (!playerName) return alert("請輸入名字");
      db.ref("players/" + playerName).set({ steps: 0 });
      document.getElementById("clickBtn").disabled = true; // ❌ 加入時還不能跑
    };

    // 點擊前進
    document.getElementById("clickBtn").onclick = () => {
      if (gameOver || !gameStarted) return;
      db.ref("players/" + playerName + "/steps").transaction(s => (s||0)+1);
    };

    // 開始遊戲（主持人）
    document.getElementById("startBtn").onclick = () => {
      let count = 3;
      countdownDiv.innerText = count;
      let timer = setInterval(() => {
        count--;
        if (count > 0) {
          countdownDiv.innerText = count;
        } else {
          clearInterval(timer);
          countdownDiv.innerText = "開始！";
          db.ref("gameStarted").set(true);
          setTimeout(()=> countdownDiv.innerText="", 1000);
        }
      }, 1000);
    };

    // 重新開始
    document.getElementById("resetBtn").onclick = () => {
      db.ref("players").remove(); // 清空所有玩家
      db.ref("gameOver").set(false);
      db.ref("gameStarted").set(false);
      winnerDiv.innerText = "";
      countdownDiv.innerText = "";
      gameOver = false;
      gameStarted = false;
      document.getElementById("clickBtn").disabled = true;
    };

    // 即時監聽遊戲開始狀態
    db.ref("gameStarted").on("value", snapshot => {
      gameStarted = snapshot.val();
      document.getElementById("clickBtn").disabled = !gameStarted;
    });

    // 即時顯示所有玩家進度
    db.ref("players").on("value", snapshot => {
      raceArea.innerHTML = "";
      const players = snapshot.val();
      if (!players) return;
      for (let name in players) {
        let steps = players[name].steps;
        let percent = Math.min((steps / FINISH_LINE) * 100, 100);

        // 跑道容器
        let track = document.createElement("div");
        track.className = "track";
        track.style.height = "60px";

        // 玩家角色
        let runner = document.createElement("div");
        runner.className = "runner";
        runner.style.left = percent + "%";
        runner.innerText = "🏃";

        // 玩家名字
        let label = document.createElement("div");
        label.className = "label";
        label.style.left = percent + "%";
        label.innerText = name + " (" + steps + ")";

        track.appendChild(runner);
        track.appendChild(label);
        raceArea.appendChild(track);

        // 勝利判斷
        if (steps >= FINISH_LINE && !gameOver) {
          gameOver = true;
          winnerDiv.innerText = "🎉 " + name + " 獲勝啦！";
        }
      }
    });
  </script>
</body>
</html>
