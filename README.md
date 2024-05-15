# Doodle-Jump-Basic
My version of Doodle Jump, but basic downgraded version.
This is also my first github project!

Here the code:

<!DOCTYPE html>
<html>
<head>
  <title>Basic Doodle Jump HTML Game</title>
  <meta charset="UTF-8">
  <style>
    html, body {
      height: 100%;
      margin: 0;
    }

    body {
      display: flex;
      align-items: center;
      justify-content: center;
      flex-direction: column; /* Align content vertically */
    }

    canvas {
      border: 1px solid black;
    }

    #gameContainer {
      position: relative;
    }

    #startButton {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }

    #killScreen {
      display: none;
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      background-color: rgba(255, 255, 255, 0.8);
      padding: 20px;
      border: 2px solid black;
      text-align: center;
    }

    #resetButton {
      margin-top: 20px; /* Add some space between the Play Again button and other elements */
    }
  </style>
</head>
<body>
  <div id="gameContainer">
    <button id="startButton">Start Game</button> <!-- Start Game button -->
    <canvas width="375" height="667" id="game"></canvas>
  </div>
  <div id="killScreen">
    <h2>Game Over</h2>
    <p>Your Score: <span id="playerScore"></span></p>
    <p>High Score: <span id="highScore"></span></p>
    <p>Enemies Killed: <span id="enemiesKilled"></span></p>
    <button id="resetButton" onclick="resetGame()">Play Again</button>
  </div>
  <script>
    const canvas = document.getElementById('game');
    const context = canvas.getContext('2d');

    const platformWidth = 65;
    const platformHeight = 20;
    const platformStart = canvas.height - 50;

    const gravity = 0.33;
    const drag = 0.3;
    const bounceVelocity = -12.5;

    let minPlatformSpace = 15;
    let maxPlatformSpace = 20;

    let platforms = [{
      x: canvas.width / 2 - platformWidth / 2,
      y: platformStart
    }];

    const doodle = {
      width: 40,
      height: 60,
      x: canvas.width / 2 - 20,
      y: platformStart - 60,
      dx: 0,
      dy: 0
    };

    class Enemy {
      constructor(x, y) {
        this.width = 30;
        this.height = 30;
        this.x = x;
        this.y = y;
        this.dx = 2;
      }

      update() {
        this.x += this.dx;
        if (this.x + this.width > canvas.width || this.x < 0) {
          this.dx *= -1;
        }
      }

      draw() {
        context.fillStyle = 'red';
        context.fillRect(this.x, this.y, this.width, this.height);
      }
    }

    class BrokenPlatform {
      constructor(x, y) {
        this.x = x;
        this.y = y;
        this.width = platformWidth;
        this.height = platformHeight;
        this.jumpCount = 0;
        this.maxJumps = Math.floor(Math.random() * 3) + 1;
      }

      draw() {
        context.fillStyle = 'orange';
        context.fillRect(this.x, this.y, this.width, this.height);
        context.fillStyle = 'black';
        context.font = '16px Arial';
        context.fillText(this.maxJumps - this.jumpCount, this.x + this.width / 2 - 5, this.y + this.height / 2 + 5);
      }
    }

    class MovingPlatform {
      constructor(x, y) {
        this.x = x;
        this.y = y;
        this.width = platformWidth;
        this.height = platformHeight;
        this.dx = 2;
        this.direction = 1;
      }

      update() {
        this.x += this.dx * this.direction;
        if (this.x + this.width > canvas.width || this.x < 0) {
          this.direction *= -1;
        }
      }

      draw() {
        context.fillStyle = 'lightblue';
        context.fillRect(this.x, this.y, this.width, this.height);
      }
    }

    class Projectile {
      constructor(x, y, dx, dy) {
        this.width = 10;
        this.height = 10;
        this.x = x;
        this.y = y;
        this.dx = dx;
        this.dy = dy;
      }

      update() {
        this.x += this.dx;
        this.y += this.dy;
      }

      draw() {
        context.fillStyle = 'blue';
        context.fillRect(this.x, this.y, this.width, this.height);
      }
    }

    let enemies = [];
    let movingPlatforms = [];
    let projectiles = [];

    let playerDir = 0;
    let keydown = false;
    let prevDoodleY = doodle.y;

    let score = 0;
    let highScore = 0;
    let highestPoint = doodle.y;
    let enemiesKilled = 0; // Track the number of enemies killed

    let isGameOver = false;

    function loop() {
      if (isGameOver) return;

      requestAnimationFrame(loop);
      context.clearRect(0, 0, canvas.width, canvas.height);

      doodle.dy += gravity;

      if (doodle.y < canvas.height / 2 && doodle.dy < 0) {
        platforms.forEach(function(platform) {
          platform.y += -doodle.dy;
        });

        movingPlatforms.forEach(function(platform) {
          platform.update();
        });

        enemies.forEach(function(enemy) {
          enemy.y += -doodle.dy;
        });

        projectiles.forEach(function(projectile) {
          projectile.y += -doodle.dy;
        });

        while (platforms[platforms.length - 1].y > 0) {
          const isBroken = Math.random() < 0.2;
          const isNewMovingPlatform = Math.random() < 0.1;
          let newPlatform;

          if (isNewMovingPlatform) {
            newPlatform = new MovingPlatform(random(25, canvas.width - 25 - platformWidth), platforms[platforms.length - 1].y - (platformHeight + random(minPlatformSpace, maxPlatformSpace)));
            movingPlatforms.push(newPlatform);
          } else {
            newPlatform = isBroken ? new BrokenPlatform(random(25, canvas.width - 25 - platformWidth), platforms[platforms.length - 1].y - (platformHeight + random(minPlatformSpace, maxPlatformSpace)))
                                    : { x: random(25, canvas.width - 25 - platformWidth), y: platforms[platforms.length - 1].y - (platformHeight + random(minPlatformSpace, maxPlatformSpace)) };
          }

          platforms.push(newPlatform);

          if (!isBroken && !isNewMovingPlatform && Math.random() < 0.2) {
            enemies.push(new Enemy(newPlatform.x + platformWidth / 2 - 15, newPlatform.y - 30));
          }

          minPlatformSpace += 0.5;
          maxPlatformSpace += 0.5;
          maxPlatformSpace = Math.min(maxPlatformSpace, canvas.height / 2);
        }
      } else {
        doodle.y += doodle.dy;
      }

      if (!keydown) {
        if (playerDir < 0) {
          doodle.dx += drag;
          if (doodle.dx > 0) {
            doodle.dx = 0;
            playerDir = 0;
          }
        } else if (playerDir > 0) {
          doodle.dx  -= drag;
          if (doodle.dx < 0) {
            doodle.dx = 0;
            playerDir = 0;
          }
        }
      }

      doodle.x += doodle.dx;

      if (doodle.x + doodle.width < 0) {
        doodle.x = canvas.width;
      } else if (doodle.x > canvas.width) {
        doodle.x = -doodle.width;
      }

      platforms.forEach(function(platform, index) {
        if (platform instanceof BrokenPlatform) {
          platform.draw();
        } else {
          context.fillStyle = 'green';
          context.fillRect(platform.x, platform.y, platformWidth, platformHeight);
        }

        if (
          doodle.dy > 0 &&
          prevDoodleY + doodle.height <= platform.y &&
          doodle.x < platform.x + platformWidth &&
          doodle.x + doodle.width > platform.x &&
          doodle.y < platform.y + platformHeight &&
          doodle.y + doodle.height > platform.y
        ) {
          doodle.y = platform.y - doodle.height;
          doodle.dy = bounceVelocity;

          if (platform instanceof BrokenPlatform) {
            platform.jumpCount++;
            if (platform.jumpCount >= platform.maxJumps) {
              platforms.splice(index, 1);
            }
          }
        }
      });

      movingPlatforms.forEach(function(platform) {
        platform.draw();
        if (
          doodle.dy < 0 &&
          doodle.y + doodle.height >= platform.y &&
          doodle.x < platform.x + platformWidth &&
          doodle.x + doodle.width > platform.x &&
          doodle.y < platform.y + platformHeight &&
          doodle.y + doodle.height > platform.y
        ) {
          doodle.y = platform.y - doodle.height;
          doodle.dy = bounceVelocity;
        }
      });

      projectiles.forEach(function(projectile, index) {
        projectile.update();
        projectile.draw();

        enemies.forEach(function(enemy, enemyIndex) {
          if (
            projectile.x < enemy.x + enemy.width &&
            projectile.x + projectile.width > enemy.x &&
            projectile.y < enemy.y + enemy.height &&
            projectile.y + projectile.height > enemy.y
          ) {
            enemies.splice(enemyIndex, 1); // Remove the enemy
            projectiles.splice(index, 1); // Remove the projectile
            enemiesKilled++; // Increment the number of enemies killed
            score += 10; // Add 10 points for killing an enemy
          }
        });

        if (projectile.y < 0 || projectile.y > canvas.height || projectile.x < 0 || projectile.x > canvas.width) {
          projectiles.splice(index, 1); // Remove the projectile if it goes off-screen
        }
      });

      enemies.forEach(function(enemy) {
        enemy.update();
        enemy.draw();
        if (
          doodle.x < enemy.x + enemy.width &&
          doodle.x + doodle.width > enemy.x &&
          doodle.y < enemy.y + enemy.height &&
          doodle.y + doodle.height > enemy.y
        ) {
          endGame();
        }
      });

      context.fillStyle = 'yellow';
      context.fillRect(doodle.x, doodle.y, doodle.width, doodle.height);

      if (doodle.y < highestPoint) {
        highestPoint = doodle.y;
        score = Math.floor((canvas.height - highestPoint) / 10);
      }

      highScore = Math.max(highScore, score);

      context.fillStyle = 'black';
      context.font = '20px Arial';
      context.fillText('Score: ' + score, 10, 30);
      context.fillText('High Score: ' + highScore, 10, 60);
      context.fillText('Enemies Killed: ' + enemiesKilled, 10, 90); // Display enemies killed

      prevDoodleY = doodle.y;

      platforms = platforms.filter(function(platform) {
        return platform.y < canvas.height;
      });

      enemies = enemies.filter(function(enemy) {
        return enemy.y < canvas.height;
      });

      movingPlatforms = movingPlatforms.filter(function(platform) {
        return platform.y < canvas.height;
      });

      if (doodle.y > canvas.height) {
        endGame();
      }
    }

    document.addEventListener('keydown', function(e) {
      if (e.which === 37) {
        keydown = true;
        playerDir = -1;
        doodle.dx = -3;
      } else if (e.which === 39) {
        keydown = true;
        playerDir = 1;
        doodle.dx = 3;
      } else if (e.which === 32) { // Space key
        shootProjectile();
      }
    });

    document.addEventListener('keyup', function(e) {
      keydown = false;
    });

    document.getElementById('startButton').addEventListener('click', startGame);

    function startGame() {
      isGameOver = false;
      highestPoint = doodle.y; // Reset highest point at start
      requestAnimationFrame(loop);
      document.getElementById('startButton').style.display = 'none';
    }

    function random(min, max) {
      return Math.random() * (max - min) + min;
    }

    function shootProjectile() {
      const projectile = new Projectile(doodle.x + doodle.width / 2 - 5, doodle.y, 0, -5);
      projectiles.push(projectile);
    }

    function endGame() {
      isGameOver = true;
      document.getElementById('playerScore').textContent = score;
      document.getElementById('highScore').textContent = highScore;
      document.getElementById('enemiesKilled').textContent = enemiesKilled; // Display enemies killed
      document.getElementById('killScreen').style.display = 'block';
    }

    function resetGame() {
      platforms = [{
        x: canvas.width / 2 - platformWidth / 2,
        y: platformStart
      }];
      let spawnPlatformIndex = Math.floor(Math.random() * platforms.length);
      doodle.x = platforms[spawnPlatformIndex].x + platformWidth / 2 - doodle.width / 2;
      doodle.y = platforms[spawnPlatformIndex].y - doodle.height;
      doodle.dy = 0;
      score = 0;
      highestPoint = doodle.y;
      minPlatformSpace = 15;
      maxPlatformSpace = 20;
      enemies = [];
      movingPlatforms = [];
      projectiles = [];
      enemiesKilled = 0; // Reset enemies killed
      isGameOver = false;
      requestAnimationFrame(loop);
      document.getElementById('killScreen').style.display = 'none';
    }
  </script>
</body>
</html>
