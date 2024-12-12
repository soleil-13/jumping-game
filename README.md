# jumping-game
 <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Jumping Game</title>
    <style>
        body { margin: 0; overflow: hidden; background-color: #87CEEB; }
        canvas { display: block; margin: 0 auto; background-color: #8B4513; }
    </style>
</head>
<body>
    <canvas id="gameCanvas"></canvas>
    <script>
        const canvas = document.getElementById("gameCanvas");
        const ctx = canvas.getContext("2d");

        const canvasWidth = 800;
        const canvasHeight = 600;
        canvas.width = canvasWidth;
        canvas.height = canvasHeight;

        const gravity = 0.4;
        const jumpStrength = -14;
        const groundHeight = 50;
        const buildingWidth = 60;
        const moveSpeed = 5;

        let player = {
            x: 50,
            y: canvasHeight - groundHeight - 50,
            width: 30,
            height: 50,
            velocityY: 0,
            jumps: 0,
            velocityX: 0,
            flying: false,
            flyTime: 0
        };

        let obstacles = [];
        let greenSquares = [];
        let score = 0;
        let highScore = 0;
        let gameOver = false;
        let greenSquareSpawnTimer = 0; // Timer for spawning green squares

        function generateObstacle() {
            const gap = Math.floor(Math.random() * 30) + 100;
            obstacles.push({
                x: canvasWidth,
                y: canvasHeight - gap - groundHeight,
                width: buildingWidth,
                height: gap - 20
            });
        }

        function generateGreenSquare() {
            greenSquares.push({
                x: Math.random() * (canvasWidth - 50),
                y: Math.random() * (canvasHeight - groundHeight - 100) + 50,
                width: 50,
                height: 20
            });
            greenSquareSpawnTimer = 0; // Reset the timer after spawning
        }

        function drawPlayer() {
            ctx.fillStyle = "#FFD700";

            ctx.beginPath();
            ctx.arc(player.x + player.width / 2, player.y + 15, 10, 0, Math.PI * 2, false);
            ctx.fill();

            ctx.fillRect(player.x + player.width / 2 - 5, player.y + 20, 10, 20);

            ctx.beginPath();
            ctx.moveTo(player.x + player.width / 2 - 15, player.y + 25);
            ctx.lineTo(player.x + player.width / 2 + 15, player.y + 25);
            ctx.stroke();

            ctx.beginPath();
            ctx.moveTo(player.x + player.width / 2 - 5, player.y + 40);
            ctx.lineTo(player.x + player.width / 2 - 15, player.y + 50);
            ctx.moveTo(player.x + player.width / 2 + 5, player.y + 40);
            ctx.lineTo(player.x + player.width / 2 + 15, player.y + 50);
            ctx.stroke();
        }

        function drawObstacles() {
            ctx.fillStyle = "#B22222";
            for (let i = 0; i < obstacles.length; i++) {
                ctx.fillRect(obstacles[i].x, obstacles[i].y, obstacles[i].width, obstacles[i].height);
            }
        }

        function drawGreenSquares() {
            ctx.fillStyle = "#32CD32";
            for (let i = 0; i < greenSquares.length; i++) {
                ctx.fillRect(greenSquares[i].x, greenSquares[i].y, greenSquares[i].width, greenSquares[i].height);
            }
        }

        function updateObstacles() {
            for (let i = obstacles.length - 1; i >= 0; i--) {
                obstacles[i].x -= 3;

                if (obstacles[i].x + obstacles[i].width < 0) {
                    obstacles.splice(i, 1);
                    score++;
                    if (score > highScore) {
                        highScore = score;
                    }
                }
            }
        }

        function checkCollision() {
            for (let i = 0; i < obstacles.length; i++) {
                if (player.x + player.width > obstacles[i].x && player.x < obstacles[i].x + obstacles[i].width) {
                    if (player.y + player.height > obstacles[i].y) {
                        gameOver = true;
                    }
                }
            }
        }

        function checkGreenSquareCollision() {
            for (let i = greenSquares.length - 1; i >= 0; i--) {
                if (player.x + player.width > greenSquares[i].x && player.x < greenSquares[i].x + greenSquares[i].width &&
                    player.y + player.height > greenSquares[i].y && player.y < greenSquares[i].y + greenSquares[i].height) {
                    player.flying = true;
                    player.flyTime = 15 * 60; // Fly for 15 seconds (assuming 60 FPS)
                    greenSquares.splice(i, 1);
                }
            }
        }

        function updatePlayer() {
            if (player.flying) {
                player.flyTime--;
                if (player.flyTime <= 0) {
                    player.flying = false;
                }
                player.velocityY = 0;
            } else {
                player.velocityY += gravity;
            }
            player.y += player.velocityY;
            player.x += player.velocityX;

            if (player.x < 0) {
                player.x = 0;
            } else if (player.x + player.width > canvasWidth) {
                player.x = canvasWidth - player.width;
            }

            if (player.y >= canvasHeight - groundHeight - player.height) {
                player.y = canvasHeight - groundHeight - player.height;
                player.velocityY = 0;
                player.jumps = 0;
            }
        }

        function drawGround() {
            ctx.fillStyle = "#228B22";
            ctx.fillRect(0, canvasHeight - groundHeight, canvasWidth, groundHeight);
        }

        function drawScore() {
            ctx.fillStyle = "#FFFFFF";
            ctx.font = "24px Arial";
            ctx.fillText("Score: " + score, 10, 30);
            ctx.fillText("High Score: " + highScore, 10, 60);
        }

        function resetGame() {
            player.x = 50;
            player.y = canvasHeight - groundHeight - 50;
            player.velocityY = 0;
            player.jumps = 0;
            player.flying = false;
            player.flyTime = 0;
            obstacles = [];
            greenSquares = [];
            score = 0;
            gameOver = false;
            greenSquareSpawnTimer = 0;
        }

        function gameLoop() {
            if (gameOver) {
                ctx.fillStyle = "#000000";
                ctx.font = "50px Arial";
                ctx.fillText("Game Over!", canvasWidth / 2 - 150, canvasHeight / 2);
                ctx.font = "20px Arial";
                ctx.fillText("Press 'R' to Restart", canvasWidth / 2 - 100, canvasHeight / 2 + 50);
                return;
            }

            ctx.clearRect(0, 0, canvasWidth, canvasHeight);

            updatePlayer();
            updateObstacles();
            checkCollision();
            checkGreenSquareCollision();
            drawGround();
            drawPlayer();
            drawObstacles();
            drawGreenSquares();
            drawScore();

            if (Math.random() < 0.02) {
                generateObstacle();
            }

            greenSquareSpawnTimer++;
            if (greenSquareSpawnTimer >= 30 * 60) { // Spawn every 30 seconds (assuming 60 FPS)
                generateGreenSquare();
            }

            requestAnimationFrame(gameLoop);
        }

        window.addEventListener("keydown", function (e) {
            if (e.code === "Space" && player.jumps < 2) {
                player.velocityY = jumpStrength;
                player.jumps++;
            }
            if (e.code === "ArrowLeft") {
                player.velocityX = -moveSpeed;
            } else if (e.code === "ArrowRight") {
                player.velocityX = moveSpeed;
            }
            if (e.code === "KeyR" && gameOver) {
                resetGame();
                gameLoop();
            }
        });

        window.addEventListener("keyup", function (e) {
            if (e.code === "ArrowLeft" || e.code === "ArrowRight") {
                player.velocityX = 0;
            }
        });

        gameLoop();
    </script>
</body>
</html>
             
