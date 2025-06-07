


# Flappy-Bird-
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>Flappy Bird</title>
    <style>
        body {
            margin: 0;
            padding: 0;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            font-family: Arial, sans-serif;
            touch-action: manipulation;
            overflow: hidden;
            background-color: #70c5ce;
        }
        
        #game-container {
            position: relative;
            width: 100%;
            max-width: 400px;
            height: 600px;
            overflow: hidden;
            border: 2px solid #000;
        }
        
        #background {
            position: absolute;
            width: 100%;
            height: 100%;
            background-image: url('https://iili.io/2mORBt9.png');
            background-size: cover;
            background-repeat: repeat-x;
            background-position: 0 0;
            animation: backgroundScroll 20s linear infinite;
        }
        
        #bird {
            position: absolute;
            width: 40px;
            height: 40px;
            background-image: url('https://iili.io/F266t72.png');
            background-size: contain;
            background-repeat: no-repeat;
            z-index: 10;
        }
        
        .pipe {
            position: absolute;
            width: 60px;
            background-image: url('https://i.postimg.cc/pdBBVgZs/Pipe.png');
            background-size: 100% 100%;
            background-repeat: no-repeat;
            z-index: 5;
        }
        
        .pipe-top {
            transform: rotate(180deg);
        }
        
        #score {
            position: absolute;
            top: 20px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 24px;
            font-weight: bold;
            color: white;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.5);
            z-index: 20;
        }
        
        #start-screen, #game-over {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            height: 100%;
            background-color: rgba(0, 0, 0, 0.7);
            color: white;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            z-index: 30;
        }
        
        #game-over {
            display: none;
        }
        
        .game-message {
            text-align: center;
            margin-bottom: 20px;
        }
        
        .btn {
            padding: 12px 24px;
            background-color: #5cb85c;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            margin: 5px;
            min-width: 150px;
        }
        
        .btn:hover {
            background-color: #4cae4c;
        }
        
        .controls {
            margin-top: 20px;
            text-align: center;
            font-size: 14px;
        }
        
        #sound-toggle {
            position: absolute;
            top: 20px;
            right: 20px;
            background: none;
            border: none;
            color: white;
            font-size: 24px;
            cursor: pointer;
            z-index: 40;
        }
        
        @keyframes backgroundScroll {
            from { background-position: 0 0; }
            to { background-position: -400px 0; }
        }
        
        @media (max-height: 650px) {
            #game-container {
                height: 100vh;
            }
        }
    </style>
</head>
<body>
    <div id="game-container">
        <button id="sound-toggle">🔊</button>
        <div id="background"></div>
        <div id="bird"></div>
        <div id="score">0</div>
        
        <div id="start-screen">
            <div class="game-message">
                <h1>Flappy Bird</h1>
                <p>Press SPACE or tap to jump</p>
            </div>
            <button id="start-btn" class="btn">Start Game</button>
            <div class="controls">
                <p>Controls: Space/Up Arrow or Tap</p>
            </div>
        </div>
        
        <div id="game-over">
            <div class="game-message">
                <h2>Game Over!</h2>
                <p>Your score: <span id="final-score">0</span></p>
            </div>
            <button id="restart-btn" class="btn">Play Again</button>
        </div>
    </div>

    <!-- Audio elements -->
    <audio id="jump-sound" src="https://www.soundjay.com/buttons/sounds/button-09.mp3" preload="auto"></audio>
    <audio id="score-sound" src="https://www.soundjay.com/buttons/sounds/button-21.mp3" preload="auto"></audio>
    <audio id="hit-sound" src="https://www.soundjay.com/buttons/sounds/button-10.mp3" preload="auto"></audio>
    <audio id="background-music" loop src="https://www.soundjay.com/misc/sounds/magic-chime-01.mp3" preload="auto"></audio>

    <script>
        // Game variables
        const bird = document.getElementById('bird');
        const gameContainer = document.getElementById('game-container');
        const background = document.getElementById('background');
        const scoreElement = document.getElementById('score');
        const startScreen = document.getElementById('start-screen');
        const gameOverElement = document.getElementById('game-over');
        const finalScoreElement = document.getElementById('final-score');
        const startBtn = document.getElementById('start-btn');
        const restartBtn = document.getElementById('restart-btn');
        const soundToggle = document.getElementById('sound-toggle');
        
        // Audio elements
        const jumpSound = document.getElementById('jump-sound');
        const scoreSound = document.getElementById('score-sound');
        const hitSound = document.getElementById('hit-sound');
        const backgroundMusic = document.getElementById('background-music');
        
        // Game settings
        let birdPosition = 300;
        let birdVelocity = 0;
        let gravity = 0.5;
        let jumpForce = -10;
        let gameWidth = gameContainer.offsetWidth;
        let gameHeight = gameContainer.offsetHeight;
        let pipes = [];
        let score = 0;
        let gameRunning = false;
        let pipeGap = 150;
        let pipeFrequency = 1500; // milliseconds
        let lastPipeTime = 0;
        let animationFrameId;
        let backgroundPosition = 0;
        let soundEnabled = true;
        
        // Initialize game
        function initGame() {
            // Set bird position
            bird.style.top = birdPosition + 'px';
            bird.style.left = '100px';
            
            // Set game container dimensions
            gameWidth = gameContainer.offsetWidth;
            gameHeight = gameContainer.offsetHeight;
            
            // Event listeners
            document.addEventListener('keydown', handleKeyDown);
            gameContainer.addEventListener('touchstart', handleTouch);
            gameContainer.addEventListener('click', handleTouch);
            startBtn.addEventListener('click', startGame);
            restartBtn.addEventListener('click', startGame);
            soundToggle.addEventListener('click', toggleSound);
            
            // Handle window resize
            window.addEventListener('resize', function() {
                gameWidth = gameContainer.offsetWidth;
                gameHeight = gameContainer.offsetHeight;
            });
        }
        
        // Toggle sound
        function toggleSound() {
            soundEnabled = !soundEnabled;
            soundToggle.textContent = soundEnabled ? "🔊" : "🔇";
            
            if (soundEnabled && gameRunning) {
                backgroundMusic.play().catch(e => console.log("Audio play error:", e));
            } else {
                backgroundMusic.pause();
            }
        }
        
        // Play sound with check
        function playSound(sound) {
            if (soundEnabled) {
                sound.currentTime = 0;
                sound.play().catch(e => console.log("Audio play error:", e));
            }
        }
        
        // Handle keyboard input
        function handleKeyDown(e) {
            if ((e.code === 'Space' || e.key === 'ArrowUp') && gameRunning) {
                birdVelocity = jumpForce;
                playSound(jumpSound);
                e.preventDefault();
            }
        }
        
        // Handle touch input
        function handleTouch(e) {
            if (gameRunning) {
                birdVelocity = jumpForce;
                playSound(jumpSound);
                e.preventDefault();
            }
        }
        
        // Start game
        function startGame() {
            // Reset game state
            birdPosition = gameHeight / 2 - 20;
            birdVelocity = 0;
            bird.style.top = birdPosition + 'px';
            
            // Remove all pipes
            pipes.forEach(pipe => {
                gameContainer.removeChild(pipe.topElement);
                gameContainer.removeChild(pipe.bottomElement);
            });
            pipes = [];
            
            // Reset score
            score = 0;
            scoreElement.textContent = score;
            
            // Hide screens
            startScreen.style.display = 'none';
            gameOverElement.style.display = 'none';
            
            // Start background music
            if (soundEnabled) {
                backgroundMusic.currentTime = 0;
                backgroundMusic.play().catch(e => console.log("Audio play error:", e));
            }
            
            // Start game loop
            gameRunning = true;
            lastPipeTime = performance.now();
            backgroundPosition = 0;
            animationFrameId = requestAnimationFrame(gameLoop);
        }
        
        // Game over
        function gameOver() {
            gameRunning = false;
            finalScoreElement.textContent = score;
            gameOverElement.style.display = 'flex';
            playSound(hitSound);
            backgroundMusic.pause();
            cancelAnimationFrame(animationFrameId);
        }
        
        // Game loop
        function gameLoop(timestamp) {
            if (!gameRunning) return;
            
            updateBird();
            updatePipes(timestamp);
            updateBackground();
            checkCollisions();
            render();
            
            animationFrameId = requestAnimationFrame(gameLoop);
        }
        
        // Update bird position
        function updateBird() {
            birdVelocity += gravity;
            birdPosition += birdVelocity;
            
            // Keep bird within game bounds
            if (birdPosition < 0) {
                birdPosition = 0;
                birdVelocity = 0;
            }
            
            if (birdPosition > gameHeight - 40) {
                birdPosition = gameHeight - 40;
                gameOver();
            }
        }
        
        // Update pipes
        function updatePipes(timestamp) {
            // Add new pipes
            if (timestamp - lastPipeTime > pipeFrequency) {
                createPipe();
                lastPipeTime = timestamp;
                
                // Increase difficulty slightly
                if (score > 0 && score % 5 === 0) {
                    pipeFrequency = Math.max(1000, pipeFrequency - 50);
                    pipeGap = Math.max(120, pipeGap - 5);
                }
            }
            
            // Move pipes
            for (let i = pipes.length - 1; i >= 0; i--) {
                const pipe = pipes[i];
                pipe.x -= 2;
                
                // Remove pipes that are off screen
                if (pipe.x < -60) {
                    gameContainer.removeChild(pipe.topElement);
                    gameContainer.removeChild(pipe.bottomElement);
                    pipes.splice(i, 1);
                }
                
                // Increase score when bird passes a pipe
                if (!pipe.passed && pipe.x < 100 - 60) {
                    pipe.passed = true;
                    score++;
                    scoreElement.textContent = score;
                    playSound(scoreSound);
                }
            }
        }
        
        // Update background
        function updateBackground() {
            backgroundPosition = (backgroundPosition + 1) % 400;
            background.style.backgroundPosition = `-${backgroundPosition}px 0`;
        }
        
        // Create new pipe
        function createPipe() {
            const minHeight = 60;
            const maxHeight = gameHeight - pipeGap - minHeight;
            const topHeight = Math.floor(Math.random() * (maxHeight - minHeight + 1)) + minHeight;
            
            const topPipe = document.createElement('div');
            topPipe.className = 'pipe pipe-top';
            topPipe.style.height = topHeight + 'px';
            topPipe.style.left = gameWidth + 'px';
            topPipe.style.top = '0px';
            
            const bottomPipe = document.createElement('div');
            bottomPipe.className = 'pipe';
            bottomPipe.style.height = (gameHeight - topHeight - pipeGap) + 'px';
            bottomPipe.style.left = gameWidth + 'px';
            bottomPipe.style.bottom = '0px';
            
            gameContainer.appendChild(topPipe);
            gameContainer.appendChild(bottomPipe);
            
            pipes.push({
                x: gameWidth,
                topHeight: topHeight,
                topElement: topPipe,
                bottomElement: bottomPipe,
                passed: false
            });
        }
        
        // Check for collisions
        function checkCollisions() {
            const birdLeft = 100;
            const birdRight = birdLeft + 40;
            const birdTop = birdPosition;
            const birdBottom = birdPosition + 40;
            
            for (const pipe of pipes) {
                const pipeLeft = pipe.x;
                const pipeRight = pipe.x + 60;
                
                // Check if bird is horizontally aligned with pipe
                if (birdRight > pipeLeft && birdLeft < pipeRight) {
                    // Check top pipe collision
                    if (birdTop < pipe.topHeight) {
                        gameOver();
                        return;
                    }
                    
                    // Check bottom pipe collision
                    if (birdBottom > (gameHeight - (gameHeight - pipe.topHeight - pipeGap))) {
                        gameOver();
                        return;
                    }
                }
            }
        }
        
        // Render game state
        function render() {
            bird.style.top = birdPosition + 'px';
            
            for (const pipe of pipes) {
                pipe.topElement.style.left = pipe.x + 'px';
                pipe.bottomElement.style.left = pipe.x + 'px';
            }
        }
        
        // Initialize the game when the page loads
        window.onload = initGame;
    </script>
</body>
</html>
