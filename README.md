<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Snake Game</title>
    <style>
        body {
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            font-family: 'Courier New', monospace;
        }
        
        .game-container {
            text-align: center;
            background: rgba(255, 255, 255, 0.1);
            padding: 30px;
            border-radius: 20px;
            backdrop-filter: blur(10px);
            box-shadow: 0 8px 32px rgba(31, 38, 135, 0.37);
            border: 1px solid rgba(255, 255, 255, 0.18);
        }
        
        h1 {
            color: white;
            margin: 0 0 20px 0;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0,0,0,0.3);
        }
        
        .score {
            color: white;
            font-size: 1.5em;
            margin-bottom: 20px;
            text-shadow: 1px 1px 2px rgba(0,0,0,0.3);
        }
        
        #gameCanvas {
            border: 3px solid #fff;
            border-radius: 10px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.2);
            background: #1a1a2e;
        }
        
        .controls {
            margin-top: 20px;
            color: white;
            font-size: 0.9em;
            opacity: 0.8;
        }
        
        .game-over {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0,0,0,0.8);
            color: white;
            padding: 30px;
            border-radius: 15px;
            text-align: center;
            display: none;
        }
        
        button {
            background: linear-gradient(45deg, #667eea, #764ba2);
            border: none;
            color: white;
            padding: 12px 24px;
            font-size: 16px;
            border-radius: 25px;
            cursor: pointer;
            margin-top: 15px;
            transition: transform 0.2s;
        }
        
        button:hover {
            transform: scale(1.05);
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1>🐍 Snake Game</h1>
        <div class="score">Score: <span id="score">0</span></div>
        <canvas id="gameCanvas" width="400" height="400"></canvas>
        <div class="controls">
            Use WASD or Arrow Keys to move<br>
            Space to restart when game over
        </div>
        
        <div class="game-over" id="gameOver">
            <h2>Game Over!</h2>
            <p>Final Score: <span id="finalScore">0</span></p>
            <button onclick="resetGame()">Play Again</button>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('gameCanvas');
        const ctx = canvas.getContext('2d');
        const scoreElement = document.getElementById('score');
        const gameOverElement = document.getElementById('gameOver');
        const finalScoreElement = document.getElementById('finalScore');

        // Game variables
        const gridSize = 20;
        const tileCount = canvas.width / gridSize;

        let snake = [
            {x: 10, y: 10}
        ];
        let food = {};
        let dx = 0;
        let dy = 0;
        let score = 0;
        let gameRunning = true;

        // Generate random food position
        function randomFood() {
            food = {
                x: Math.floor(Math.random() * tileCount),
                y: Math.floor(Math.random() * tileCount)
            };
        }

        // Draw everything
        function drawGame() {
            // Clear canvas with gradient
            const gradient = ctx.createLinearGradient(0, 0, canvas.width, canvas.height);
            gradient.addColorStop(0, '#0f0f23');
            gradient.addColorStop(1, '#1a1a2e');
            ctx.fillStyle = gradient;
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            // Draw snake
            ctx.fillStyle = '#4ecdc4';
            snake.forEach((segment, index) => {
                if (index === 0) {
                    // Snake head - make it slightly different
                    ctx.fillStyle = '#45b7aa';
                    ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize - 2, gridSize - 2);
                    
                    // Add eyes
                    ctx.fillStyle = '#fff';
                    ctx.fillRect(segment.x * gridSize + 5, segment.y * gridSize + 5, 3, 3);
                    ctx.fillRect(segment.x * gridSize + 12, segment.y * gridSize + 5, 3, 3);
                } else {
                    ctx.fillStyle = '#4ecdc4';
                    ctx.fillRect(segment.x * gridSize, segment.y * gridSize, gridSize - 2, gridSize - 2);
                }
            });

            // Draw food with pulsing effect
            const pulseSize = Math.sin(Date.now() * 0.01) * 2;
            ctx.fillStyle = '#ff6b6b';
            ctx.beginPath();
            ctx.arc(
                food.x * gridSize + gridSize / 2,
                food.y * gridSize + gridSize / 2,
                (gridSize / 2 - 1) + pulseSize,
                0,
                2 * Math.PI
            );
            ctx.fill();
        }

        // Move snake
        function moveSnake() {
            if (!gameRunning) return;

            const head = {x: snake[0].x + dx, y: snake[0].y + dy};

            // Check wall collision
            if (head.x < 0 || head.x >= tileCount || head.y < 0 || head.y >= tileCount) {
                gameOver();
                return;
            }

            // Check self collision
            for (let segment of snake) {
                if (head.x === segment.x && head.y === segment.y) {
                    gameOver();
                    return;
                }
            }

            snake.unshift(head);

            // Check food collision
            if (head.x === food.x && head.y === food.y) {
                score += 10;
                scoreElement.textContent = score;
                randomFood();
            } else {
                snake.pop();
            }
        }

        // Game over
        function gameOver() {
            gameRunning = false;
            finalScoreElement.textContent = score;
            gameOverElement.style.display = 'block';
        }

        // Reset game
        function resetGame() {
            snake = [{x: 10, y: 10}];
            dx = 0;
            dy = 0;
            score = 0;
            scoreElement.textContent = score;
            gameRunning = true;
            gameOverElement.style.display = 'none';
            randomFood();
        }

        // Handle keyboard input
        document.addEventListener('keydown', (e) => {
            if (!gameRunning && e.code === 'Space') {
                resetGame();
                return;
            }

            if (!gameRunning) return;

            // Prevent reverse direction
            switch(e.code) {
                case 'ArrowUp':
                case 'KeyW':
                    if (dy !== 1) { dx = 0; dy = -1; }
                    break;
                case 'ArrowDown':
                case 'KeyS':
                    if (dy !== -1) { dx = 0; dy = 1; }
                    break;
                case 'ArrowLeft':
                case 'KeyA':
                    if (dx !== 1) { dx = -1; dy = 0; }
                    break;
                case 'ArrowRight':
                case 'KeyD':
                    if (dx !== -1) { dx = 1; dy = 0; }
                    break;
            }
        });

        // Game loop
        function gameLoop() {
            moveSnake();
            drawGame();
        }

        // Initialize game
        randomFood();
        setInterval(gameLoop, 150);
        
        // Initial draw
        drawGame();
    </script>
</body>
</html>
