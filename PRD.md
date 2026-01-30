一、專案概述
專案名稱：Snake Game Layout Optimization
一句話描述：調整遊戲介面佈局，將控制按鍵置於畫布右側，並將操作說明文字移至下方常態顯示。
版本：v1.3 (Layout Update)

二、問題與目標
問題陳述
介面空間利用不佳：原本的設計中，按鍵是懸浮於畫面上的，可能會隨著螢幕尺寸變動而遮擋遊戲關鍵區域。
資訊不持久：原本的操作說明僅在開始畫面出現，一旦遊戲開始，新手用戶若忘記操作方式（如暫停鍵）將無處查詢。
桌面體驗割裂：在寬螢幕上，按鍵懸浮在右側角落反而導致滑鼠/手指移動距離過長，不符合桌面軟體常見的「左圖右控」或「圖控並列」習慣。
目標
建立一個穩定的左右佈局結構，讓遊戲畫面與控制器互不干擾，並確保操作說明隨時可見，提升整體介面的專業度與易用性。

三、目標用戶
用戶特徵
桌面端瀏覽器用戶（使用滑鼠點擊按鍵）。
平板橫屏用戶。
需要隨時查看操作提示的記憶力較弱用戶。
用戶需求
清晰、不干擾視線的控制面板。
固定的操作指引文字。
佈局在不同裝置上（橫屏、直屏）皆不崩壞。

四、用戶故事
用戶故事1：作為一名桌面玩家，我想要按鍵整齊地排列在遊戲右邊，以便於我將遊戲當作一個完整的面板來看待，而不是浮動的元素。
用戶故事2：作為一名新玩家，我想要隨時看到按空白鍵可以暫停，以免我在遊戲中途想休息時亂按鍵導致操作失誤。
用戶故事3：作為一名手機用戶，我想要在螢幕變窄時按鍵能自動調整位置，以免按鍵擠壓到遊戲畫面導致無法遊玩。

五、功能需求與驗收標準
功能1：[右側並排控制器佈局]
描述：
使用 CSS Flexbox 創建一個容器 (.game-layout)，將 canvas 與 .controls 並排顯示。
移除原本的 position: fixed 或 absolute 定位，改為相對定位。
按鍵採用 3x3 Grid 佈局呈現十字形。
驗收標準：
Given 使用者在寬螢幕裝置開啟頁面，When 頁面載入完成，Then 左側應顯示遊戲畫布，右側應顯示方向按鍵區塊。
Given 按鍵區塊顯示在右側，When 視窗寬度縮小（例如小於 500px），Then 按鍵區塊應自動移至畫布下方或調整大小，不破壞佈局。
功能2：[常態化操作說明]
描述：
新增一個 div.game-instructions 區塊。
將文字內容設置為「使用方向鍵或右側按鈕控制，空白鍵暫停」。
將該區塊放置於 .game-layout 容器之後，確保在遊戲開始、進行中、結束時皆可見。
驗收標準：
Given 使用者點擊「開始遊戲」，When 開始畫面消失，Then 遊戲下方的操作說明文字依然存在且清晰可見。
Given 遊戲暫停中，When 畫面顯示暫停遮罩，Then 底部的操作說明文字不應被遮罩擋住（z-index 低於遮罩）。

六、技術約束
必須遵守
HTML 結構調整：必須引入 <div class="game-layout"> 包裹 main 和 div.controls。
CSS 響應式：必須加入 @media 查詢，處理小螢幕（手機直式）時的顯示問題（建議改為垂直排列）。
字串一致性：說明文字需與需求完全一致。
兼容性要求
需相容於不支援 Grid 的舊版瀏覽器（雖現代瀏覽器皆支援，但需預留降級處理或使用 Flexbox 模擬）。
確保按鍵點擊區域足夠大（至少 40x40 px），符合行動端觸控標準。
不要做
不要改變 JS 的遊戲邏輯。
不要讓說明文字在遊戲進行中閃爍或消失。
七、現有代碼
<!DOCTYPE html>
<html lang="zh-Hant">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>經典貪食蛇 - 含暫停功能</title>
    <style>
        :root {
            --bg-color: #000000;
            --text-color: #ffffff;
            --accent-color: #4CAF50;
            --apple-color: #ff3333;
            --grid-line: #1a1a1a;
            --ui-bg: rgba(20, 20, 20, 0.9);
            --pause-btn-color: #FFC107;
        }

        * {
            box-sizing: border-box;
            touch-action: manipulation;
        }

        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-color);
            color: var(--text-color);
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            min-height: 100vh;
            overflow: hidden;
        }

        header {
            text-align: center;
            margin-bottom: 10px;
            width: 100%;
            max-width: 800px;
            /* 寬度加大以容納併排佈局 */
            display: flex;
            flex-direction: column;
            align-items: center;
            position: relative;
            z-index: 5;
        }

        h1 {
            margin: 0;
            font-size: 1.5rem;
            letter-spacing: 2px;
            color: var(--accent-color);
            text-shadow: 0 0 10px rgba(76, 175, 80, 0.5);
        }

        .top-bar {
            display: flex;
            justify-content: space-between;
            width: 100%;
            max-width: 420px;
            /* 限制計分板寬度 */
            align-items: center;
            margin-top: 5px;
            padding: 0 10px;
        }

        .score-board {
            display: flex;
            gap: 15px;
            font-size: 1rem;
        }

        .score-item span {
            font-weight: bold;
            color: var(--accent-color);
        }

        button.btn-pause {
            background-color: #333;
            color: white;
            border: 1px solid #555;
            padding: 5px 12px;
            border-radius: 4px;
            cursor: pointer;
            font-size: 0.9rem;
            transition: background 0.2s;
        }

        button.btn-pause:hover {
            background-color: #555;
        }

        /* 遊戲主佈局容器：包含 Canvas 和 右側按鍵 */
        .game-layout {
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 20px;
            position: relative;
        }

        /* 遊戲容器 */
        #game-container {
            position: relative;
            box-shadow: 0 0 20px rgba(0, 0, 0, 0.5);
            border: 2px solid #333;
            z-index: 1;
        }

        canvas {
            display: block;
            background-color: var(--bg-color);
            background-image:
                linear-gradient(var(--grid-line) 1px, transparent 1px),
                linear-gradient(90deg, var(--grid-line) 1px, transparent 1px);
            background-size: 20px 20px;
        }

        /* 通用覆蓋層樣式 */
        .overlay {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: var(--ui-bg);
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 10;
            transition: opacity 0.3s;
        }

        .overlay.hidden {
            opacity: 0;
            pointer-events: none;
        }

        .overlay h2 {
            font-size: 2rem;
            margin-bottom: 1rem;
            color: var(--text-color);
        }

        .overlay p {
            margin-bottom: 2rem;
            color: #ccc;
        }

        button.btn-primary {
            background-color: var(--accent-color);
            color: white;
            border: none;
            padding: 12px 30px;
            font-size: 1.2rem;
            border-radius: 50px;
            cursor: pointer;
            transition: transform 0.1s, box-shadow 0.2s;
            box-shadow: 0 4px 15px rgba(76, 175, 80, 0.4);
        }

        button.btn-primary:hover {
            transform: scale(1.05);
            box-shadow: 0 6px 20px rgba(76, 175, 80, 0.6);
        }

        button.btn-primary:active {
            transform: scale(0.95);
        }

        /* 暫停專用樣式 */
        #pause-screen h2 {
            color: var(--pause-btn-color);
        }

        /* 
           右側控制器區域 (對應紅方格位置)
           改為相對定位，與 Canvas 並排 
        */
        .controls {
            /* 基礎設定 */
            display: grid;
            /* 3x3 網格，中間留空或佈局成十字 */
            grid-template-columns: 50px 50px 50px;
            grid-template-rows: 50px 50px 50px;
            gap: 10px;

            /* 視覺樣式：模擬一個方塊區域 */
            background-color: rgba(255, 255, 255, 0.05);
            /* 淡淡的背景表示區域 */
            padding: 15px;
            border-radius: 20px;
            border: 1px solid rgba(255, 255, 255, 0.1);
        }

        .d-pad-btn {
            background-color: rgba(255, 255, 255, 0.15);
            border: 1px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            /* 圓形按鈕 */
            color: white;
            font-size: 1.5rem;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: pointer;
            user-select: none;
            transition: all 0.1s ease;
            box-shadow: 0 2px 5px rgba(0, 0, 0, 0.3);
        }

        .d-pad-btn:active {
            background-color: rgba(255, 255, 255, 0.4);
            transform: scale(0.9);
        }

        /* Grid 定位形成十字 */
        .d-pad-up {
            grid-column: 2;
            grid-row: 1;
        }

        .d-pad-left {
            grid-column: 1;
            grid-row: 2;
        }

        .d-pad-down {
            grid-column: 2;
            grid-row: 2;
        }

        .d-pad-right {
            grid-column: 3;
            grid-row: 2;
        }

        /* 
           新增：下方的操作說明文字區域 
        */
        .game-instructions {
            margin-top: 15px;
            color: #888;
            font-size: 0.9rem;
            text-align: center;
            letter-spacing: 1px;
        }

        /* 響應式調整：小螢幕時改為垂直排列或佈局調整 */
        @media (max-width: 500px) {
            .game-layout {
                flex-direction: column;
            }

            .controls {
                /* 手機版可以稍微縮小一點，或者變成水平排列 */
                grid-template-columns: 40px 40px 40px;
                grid-template-rows: 40px 40px 40px;
                margin-top: 10px;
                gap: 5px;
            }

            .d-pad-btn {
                font-size: 1.2rem;
            }
        }
    </style>
</head>

<body>

    <header>
        <h1>貪食蛇大作戰</h1>
        <div class="top-bar">
            <div class="score-board">
                <div class="score-item">分數: <span id="score">0</span></div>
                <div class="score-item">最高: <span id="high-score">0</span></div>
            </div>
            <button id="pause-btn" class="btn-pause" onclick="togglePause()">暫停</button>
        </div>
    </header>

    <!-- 主佈局：遊戲 + 控制器 -->
    <div class="game-layout">
        <main id="game-container">
            <canvas id="gameCanvas" width="400" height="400"></canvas>

            <!-- 開始畫面 -->
            <div id="start-screen" class="overlay">
                <h2>準備好了嗎？</h2>
                <!-- 移除了原本的 p 標籤文字 -->
                <button class="btn-primary" onclick="startGame()">開始遊戲</button>
            </div>

            <!-- 暫停畫面 -->
            <div id="pause-screen" class="overlay hidden">
                <h2>已暫停</h2>
                <p>休息一下</p>
                <button class="btn-primary" onclick="togglePause()">繼續遊戲</button>
            </div>

            <!-- 遊戲結束畫面 -->
            <div id="game-over-screen" class="overlay hidden">
                <h2 style="color: #ff5252;">遊戲結束</h2>
                <p>最終得分: <span id="final-score">0</span></p>
                <button class="btn-primary" onclick="startGame()">再玩一次</button>
            </div>
        </main>

        <!-- 右側控制器區域 (紅方格位置) -->
        <div class="controls">
            <div class="d-pad-btn d-pad-up" onclick="handleInput('ArrowUp')">▲</div>
            <div class="d-pad-btn d-pad-left" onclick="handleInput('ArrowLeft')">◀</div>
            <div class="d-pad-btn d-pad-down" onclick="handleInput('ArrowDown')">▼</div>
            <div class="d-pad-btn d-pad-right" onclick="handleInput('ArrowRight')">▶</div>
        </div>
    </div>

    <!-- 放置在下方的說明文字 -->
    <div class="game-instructions">
        使用方向鍵或右側按鈕控制，空白鍵暫停
    </div>

    <script>
        // --- 遊戲設定變數 ---
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreEl = document.getElementById('score');
        const highScoreEl = document.getElementById('high-score');
        const finalScoreEl = document.getElementById('final-score');
        const startScreen = document.getElementById('start-screen');
        const pauseScreen = document.getElementById('pause-screen');
        const gameOverScreen = document.getElementById('game-over-screen');
        const pauseBtn = document.getElementById('pause-btn');

        const gridSize = 20;
        let tileCount = canvas.width / gridSize;

        // 遊戲狀態
        let score = 0;
        let highScore = localStorage.getItem('snakeHighScore') || 0;
        let gameRunning = false;
        let isPaused = false;
        let gameLoopInterval;
        let speed = 100;

        // 蛇與蘋果
        let snake = [];
        let apple = { x: 5, y: 5 };

        // 移動方向
        let velocity = { x: 0, y: 0 };
        let nextVelocity = { x: 0, y: 0 };

        highScoreEl.innerText = highScore;

        // --- 遊戲核心邏輯 ---

        function startGame() {
            // 重置狀態
            snake = [
                { x: 10, y: 10 },
                { x: 10, y: 11 },
                { x: 10, y: 12 }
            ];
            velocity = { x: 0, y: -1 };
            nextVelocity = { x: 0, y: -1 };
            score = 0;
            scoreEl.innerText = score;
            speed = 120;
            isPaused = false;
            pauseBtn.innerText = "暫停";

            // UI 更新
            startScreen.classList.add('hidden');
            gameOverScreen.classList.add('hidden');
            pauseScreen.classList.add('hidden');

            placeApple();

            gameRunning = true;
            if (gameLoopInterval) clearInterval(gameLoopInterval);
            gameLoopInterval = setInterval(gameLoop, speed);
        }

        function togglePause() {
            // 只有在遊戲進行中才能暫停
            if (!gameRunning || !startScreen.classList.contains('hidden') || !gameOverScreen.classList.contains('hidden')) {
                return;
            }

            if (isPaused) {
                // 繼續遊戲
                isPaused = false;
                gameLoopInterval = setInterval(gameLoop, speed);
                pauseScreen.classList.add('hidden');
                pauseBtn.innerText = "暫停";
            } else {
                // 暫停遊戲
                isPaused = true;
                clearInterval(gameLoopInterval);
                pauseScreen.classList.remove('hidden');
                pauseBtn.innerText = "繼續";
            }
        }

        function gameLoop() {
            if (!gameRunning || isPaused) return;

            update();
            draw();
        }

        function update() {
            velocity = { ...nextVelocity };
            const head = { x: snake[0].x + velocity.x, y: snake[0].y + velocity.y };

            // 撞牆檢測
            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                gameOver();
                return;
            }

            // 撞擊自身檢測
            for (let i = 0; i < snake.length; i++) {
                if (head.x === snake[i].x && head.y === snake[i].y) {
                    gameOver();
                    return;
                }
            }

            snake.unshift(head);

            // 吃蘋果
            if (head.x === apple.x && head.y === apple.y) {
                score++;
                scoreEl.innerText = score;

                // 加速機制
                if (score % 5 === 0 && speed > 50) {
                    clearInterval(gameLoopInterval);
                    speed -= 5;
                    if (!isPaused) {
                        gameLoopInterval = setInterval(gameLoop, speed);
                    }
                }
                placeApple();
            } else {
                snake.pop();
            }
        }

        function draw() {
            ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--bg-color');
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // 畫蘋果
            ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--apple-color');
            ctx.fillRect(apple.x * gridSize + 2, apple.y * gridSize + 2, gridSize - 4, gridSize - 4);

            // 畫蛇
            snake.forEach((segment, index) => {
                if (index === 0) {
                    ctx.fillStyle = '#66ff66';
                } else {
                    ctx.fillStyle = '#4CAF50';
                }

                ctx.fillRect(segment.x * gridSize + 1, segment.y * gridSize + 1, gridSize - 2, gridSize - 2);

                if (index === 0) {
                    ctx.fillStyle = 'black';
                    let eyeOffsetX = 0, eyeOffsetY = 0;
                    if (velocity.x === 1) eyeOffsetX = 4;
                    if (velocity.x === -1) eyeOffsetX = -4;
                    if (velocity.y === 1) eyeOffsetY = 4;
                    if (velocity.y === -1) eyeOffsetY = -4;

                    ctx.fillRect(segment.x * gridSize + 6 + eyeOffsetX, segment.y * gridSize + 6 + eyeOffsetY, 2, 2);
                    ctx.fillRect(segment.x * gridSize + 12 + eyeOffsetX, segment.y * gridSize + 6 + eyeOffsetY, 2, 2);
                }
            });
        }

        function placeApple() {
            let valid = false;
            while (!valid) {
                apple.x = Math.floor(Math.random() * tileCount);
                apple.y = Math.floor(Math.random() * tileCount);

                valid = true;
                for (let part of snake) {
                    if (part.x === apple.x && part.y === apple.y) {
                        valid = false;
                        break;
                    }
                }
            }
        }

        function gameOver() {
            gameRunning = false;
            isPaused = false;
            clearInterval(gameLoopInterval);

            if (score > highScore) {
                highScore = score;
                localStorage.setItem('snakeHighScore', highScore);
                highScoreEl.innerText = highScore;
            }

            finalScoreEl.innerText = score;
            gameOverScreen.classList.remove('hidden');
            pauseBtn.innerText = "暫停";
        }

        // --- 輸入控制 ---

        function handleInput(key) {
            if (isPaused || !gameRunning) return;

            switch (key) {
                case 'ArrowUp':
                    if (velocity.y === 0) nextVelocity = { x: 0, y: -1 };
                    break;
                case 'ArrowDown':
                    if (velocity.y === 0) nextVelocity = { x: 0, y: 1 };
                    break;
                case 'ArrowLeft':
                    if (velocity.x === 0) nextVelocity = { x: -1, y: 0 };
                    break;
                case 'ArrowRight':
                    if (velocity.x === 0) nextVelocity = { x: 1, y: 0 };
                    break;
            }
        }

        document.addEventListener('keydown', (e) => {
            if (e.code === 'Space') {
                e.preventDefault();
                togglePause();
                return;
            }

            if (["ArrowUp", "ArrowDown", "ArrowLeft", "ArrowRight"].indexOf(e.code) > -1) {
                e.preventDefault();
                handleInput(e.code);
            }
        });

        // 初始背景
        ctx.fillStyle = getComputedStyle(document.documentElement).getPropertyValue('--bg-color');
        ctx.fillRect(0, 0, canvas.width, canvas.height);

    </script>
</body>

</html>
