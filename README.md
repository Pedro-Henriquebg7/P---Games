# P---Games
Codigo do P§ - Games

´´´
let currentScreen = 'menu';
let car;
let obstacles = [];
let collectedDots = [];
let score = 0;
let highScore = 0;
let numInitialDots = 10;
let numObstacles = 5;
let obstacleSpeed;
let obstacleFrequency;
let timeSinceLastObstacle = 0;
let timeSinceLastDot = 0;
let gameLevel = 1;
let obstacleColor;
let gameOver = false;
let buttons = {};
let carColor;
let dotSize = 15;
let dotFrequency;
let levelThreshold = 100; // Pontuação para mudar de fase
let specialButtonDisplayed = false;
let lives = 3; // Contador de vidas
let bgImage; // Variável para a imagem de fundo

function preload() {
  bgImage = loadImage('foto.png'); // Carrega a imagem de fundo
}

function setup() {
  createCanvas(windowWidth, windowHeight);
  carColor = color(255, 255, 255); // Cor inicial do carro (branco)
  car = { x: width / 2 - 20, y: height / 2 - 20, size: 40 }; // Carro

  // Carregar o recorde do localStorage
  loadHighScore();

  // Configurar nível e obstáculos
  setGameLevel(gameLevel);
  createLevel();

  // Definir botões para o menu principal
  createButtons();
}

function windowResized() {
  resizeCanvas(windowWidth, windowHeight);
  createButtons();
  car.x = width / 2 - car.size / 2;
  car.y = height / 2 - car.size / 2;
}

// Definição da classe Button
class Button {
  constructor(x, y, label, action, bgColor, textColor) {
    this.x = x;
    this.y = y;
    this.width = 200;
    this.height = 50;
    this.label = label;
    this.action = action;
    this.bgColor = bgColor;
    this.textColor = textColor;
  }

  display() {
    fill(this.bgColor);
    noStroke();
    rect(this.x, this.y, this.width, this.height, 10);

    fill(this.textColor);
    textAlign(CENTER, CENTER);
    textSize(24);
    textFont('Arial');
    text(this.label, this.x + this.width / 2, this.y + this.height / 2);
  }

  isMouseOver() {
    let mx = mouseX;
    let my = mouseY;
    return mx > this.x && mx < this.x + this.width && my > this.y && my < this.y + this.height;
  }
}

function createButtons() {
  buttons['menu'] = [
    new Button(width / 2 - 100, height / 2 + 60, 'Jogar', 'game', color(0, 102, 204), color(255)),
    new Button(width / 2 - 100, height / 2 + 120, 'Instruções', 'instructions', color(204, 153, 255), color(255)),
    new Button(width / 2 - 100, height / 2 + 180, 'Créditos', 'credits', color(255, 204, 153), color(0))
  ];

  buttons['instructions'] = [
    new Button(width / 2 - 100, height / 2 + 120, 'Voltar ao Menu', 'menu', color(204, 255, 204), color(0))
  ];

  buttons['credits'] = [
    new Button(width / 2 - 100, height / 2 + 120, 'Voltar ao Menu', 'menu', color(204, 204, 255), color(0))
  ];

  buttons['gameover'] = [
    new Button(width / 2 - 100, height / 2 + 60, 'Jogar Novamente', 'restart', color(255, 102, 102), color(255)),
    new Button(width / 2 - 100, height / 2 + 120, 'Voltar ao Menu', 'menu', color(255, 102, 102), color(255))
  ];

  buttons['congratulations'] = [
    new Button(width / 2 - 100, height / 2 + 60, 'Voltar ao Menu', 'menu', color(255, 255, 0), color(0))
  ];

  if (score >= 400 && !specialButtonDisplayed) {
    buttons['congratulations'].push(
      new Button(width / 2 - 100, height / 2 + 120, 'Desafie um Amigo!', 'challenge', color(0, 255, 0), color(0))
    );
    specialButtonDisplayed = true;
  }
}

function draw() {
  switch (currentScreen) {
    case 'menu':
      drawMenu();
      break;
    case 'game':
      drawGame();
      break;
    case 'instructions':
      drawInstructions();
      break;
    case 'credits':
      drawCredits();
      break;
    case 'gameover':
      drawGameOver();
      break;
    case 'congratulations':
      drawCongratulations();
      break;
  }
}

function drawMenu() {
  // Desenhar imagem de fundo
  image(bgImage, 0, 0, width, height);

  fill(255);
  textAlign(CENTER, CENTER);
  textSize(48);
  textFont('Georgia');
  text('P§ - Games', width / 2, height / 4);

  // Exibir botões do menu
  buttons['menu'].forEach(button => button.display());
}

function drawGame() {
  // Desenhar imagem de fundo
  image(bgImage, 0, 0, width, height);

  // Borda do jogo
  noFill();
  stroke(0);
  strokeWeight(4);
  rect(2, 2, width - 4, height - 4);

  // Desenhar o carro (estrela)
  fill(carColor);
  stroke(0);
  strokeWeight(2);
  drawStar(car.x + car.size / 2, car.y + car.size / 2, 5, car.size / 2, car.size / 4);

  // Atualizar e desenhar obstáculos
  timeSinceLastObstacle++;
  if (timeSinceLastObstacle > obstacleFrequency) {
    obstacles.push(createObstacle());
    timeSinceLastObstacle = 0;
  }

  obstacles.forEach(obstacle => {
    fill(obstacleColor);
    stroke(0);
    strokeWeight(2);
    drawStar(obstacle.x + obstacle.width / 2, obstacle.y + obstacle.height / 2, 5, obstacle.width / 2, obstacle.width / 4);
    obstacle.y += obstacleSpeed;
    
    // Reposicionar o obstáculo quando fora da tela
    if (obstacle.y > height) {
      obstacles.splice(obstacles.indexOf(obstacle), 1);
    }
    
    // Verificar colisão
    if (collides(car.x, car.y, car.size, car.size, obstacle.x, obstacle.y, obstacle.width, obstacle.height)) {
      lives--;
      if (lives <= 0) {
        gameOver = true;
        if (score > highScore) {
          highScore = score;
          saveHighScore();
        }
        currentScreen = 'gameover';
      } else {
        // Resetar o jogo para a próxima vida
        resetGameForNextLife();
      }
    }
  });

  // Adicionar bolinhas novas dinamicamente
  timeSinceLastDot++;
  if (timeSinceLastDot > dotFrequency) {
    collectedDots.push(createDot());
    timeSinceLastDot = 0;
  }

  collectedDots.forEach(dot => {
    fill(0, 255, 0);
    noStroke();
    ellipse(dot.x, dot.y, dotSize);
    dot.y += 3;

    if (dot.y > height) {
      collectedDots.splice(collectedDots.indexOf(dot), 1);
    }

    if (dist(car.x + car.size / 2, car.y + car.size / 2, dot.x, dot.y) < (car.size / 2 + dotSize / 2)) {
      collectedDots.splice(collectedDots.indexOf(dot), 1);
      score += 10; // Adiciona 10 pontos para cada bolinha coletada

      if (score >= 300) {
        currentScreen = 'congratulations';
      } else if (score % levelThreshold === 0) {
        gameLevel++;
        setGameLevel(gameLevel);
        createLevel();
      }
    }
  });

  // Exibir pontuação e recorde
  fill(255);  // Cor do texto branco
  textAlign(LEFT, TOP);
  textSize(24);
  textFont('Arial');
  text('Pontuação: ' + score, 10, 10);
  text('Recorde: ' + highScore, 10, 40);
  text('Vidas: ' + lives, 10, 70); // Adicionando contagem de vidas

  // Movimentar carro
  if (keyIsDown(LEFT_ARROW)) {
    car.x -= 5;
  }
  if (keyIsDown(RIGHT_ARROW)) {
    car.x += 5;
  }
  if (keyIsDown(UP_ARROW)) {
    car.y -= 5;
  }
  if (keyIsDown(DOWN_ARROW)) {
    car.y += 5;
  }

  // Limitar carro aos limites da tela
  car.x = constrain(car.x, 0, width - car.size);
  car.y = constrain(car.y, 0, height - car.size);
}

function drawInstructions() {
  // Desenhar imagem de fundo
  image(bgImage, 0, 0, width, height);

  fill(255);  // Cor do texto branco
  textAlign(CENTER, CENTER);
  textSize(32);
  textFont('Arial');
  text('Instruções', width / 2, height / 4);

  textSize(18);
  text('Use as setas do teclado para mover a Estrela.', width / 2, height / 2.5);
  text('Colete as bolinhas verdes para ganhar pontos.', width / 2, height / 2.2);
  text('Desvie dos obstáculos para não perder vidas.', width / 2, height / 2);
  text('Alcance 300 pontos para zerar o jogo!', width / 2, height / 1.8);

  // Exibir botão de voltar ao menu
  buttons['instructions'].forEach(button => button.display());
}

function drawCredits() {
  // Desenhar imagem de fundo
  image(bgImage, 0, 0, width, height);

  fill(255);  // Cor do texto branco
  textAlign(CENTER, CENTER);
  textSize(32);
  textFont('Arial');
  text('Créditos', width / 2, height / 4);

  textSize(18);
  text('Desenvolvido por Pedro Henrique.', width / 2, height / 2.5);
  text('Bom Jogo!', width / 2, height / 2.2);

  // Exibir botão de voltar ao menu
  buttons['credits'].forEach(button => button.display());
}

function drawGameOver() {
  // Desenhar imagem de fundo
  image(bgImage, 0, 0, width, height);

  fill(255);  // Cor do texto branco
  textAlign(CENTER, CENTER);
  textSize(48);
  textFont('Arial');
  text('Fim de Jogo\n', width / 2, height / 3);

  textSize(24);
  text('Pontuação Final: ' + score, width / 2, height / 2.5);
  text('Recorde Atual: ' + highScore, width / 2, height / 2.2);

  // Exibir botões de jogar novamente e voltar ao menu
  buttons['gameover'].forEach(button => button.display());
}

function drawCongratulations() {
  // Desenhar imagem de fundo
  image(bgImage, 0, 0, width, height);

  fill(255);  // Cor do texto branco
  textAlign(CENTER, CENTER);
  textSize(48);
  textFont('Arial');
  text('Parabéns!\n', width / 2, height / 3);
  text('Você zerou o jogo!', width / 2, height / 2.5);

  // Exibir botão de voltar ao menu
  buttons['congratulations'].forEach(button => button.display());
}

// Função para criar o nível do jogo
function createLevel() {
  obstacles = [];
  collectedDots = [];
  for (let i = 0; i < numObstacles; i++) {
    obstacles.push(createObstacle());
  }
  for (let i = 0; i < numInitialDots; i++) {
    collectedDots.push(createDot());
  }
}

// Salvar e carregar recorde no localStorage
function saveHighScore() {
  localStorage.setItem('highScore', highScore);
}

function loadHighScore() {
  let storedHighScore = localStorage.getItem('highScore');
  if (storedHighScore !== null) {
    highScore = int(storedHighScore);
  }
}

// Função para redefinir o jogo após perder uma vida
function resetGameForNextLife() {
  car.x = width / 2 - car.size / 2;
  car.y = height / 2 - car.size / 2;
  obstacles = [];
  collectedDots = [];
  timeSinceLastObstacle = 0;
  timeSinceLastDot = 0;
  createLevel();
}

function mousePressed() {
  if (currentScreen === 'menu') {
    buttons['menu'].forEach(button => {
      if (button.isMouseOver()) {
        currentScreen = button.action;
      }
    });
  } else if (currentScreen === 'instructions') {
    buttons['instructions'].forEach(button => {
      if (button.isMouseOver()) {
        currentScreen = button.action;
      }
    });
  } else if (currentScreen === 'credits') {
    buttons['credits'].forEach(button => {
      if (button.isMouseOver()) {
        currentScreen = button.action;
      }
    });
  } else if (currentScreen === 'gameover') {
    buttons['gameover'].forEach(button => {
      if (button.isMouseOver()) {
        if (button.action === 'restart') {
          restartGame();
        } else {
          currentScreen = button.action;
        }
      }
    });
  } else if (currentScreen === 'congratulations') {
    buttons['congratulations'].forEach(button => {
      if (button.isMouseOver()) {
        currentScreen = button.action;
      }
    });
  }
}

function restartGame() {
  score = 0;
  lives = 3;
  gameLevel = 1;
  setGameLevel(gameLevel);
  createLevel();
  gameOver = false;
  currentScreen = 'game';
}

// Definir o nível do jogo e as propriedades
function setGameLevel(level) {
  switch (level) {
    case 1:
      obstacleColor = color(255, 0, 0);
      obstacleSpeed = 4;
      obstacleFrequency = 60;
      dotFrequency = 100;
      break;
    case 2:
      obstacleColor = color(0, 0, 255);
      obstacleSpeed = 6;
      obstacleFrequency = 45;
      dotFrequency = 80;
      break;
    case 3:
      obstacleColor = color(0, 255, 0);
      obstacleSpeed = 8;
      obstacleFrequency = 30;
      dotFrequency = 60;
      break;
  }
}

// Funções auxiliares para criar obstáculos e bolinhas
function createObstacle() {
  let obstacleWidth = random(30, 50);
  let obstacleHeight = obstacleWidth;
  return { x: random(0, width - obstacleWidth), y: -obstacleHeight, width: obstacleWidth, height: obstacleHeight };
}

function createDot() {
  return { x: random(dotSize, width - dotSize), y: -dotSize / 2 };
}

function drawStar(x, y, npoints, radius1, radius2) {
  let angle = TWO_PI / npoints;
  let halfAngle = angle / 2.0;
  beginShape();
  for (let a = 0; a < TWO_PI; a += angle) {
    let sx = x + cos(a) * radius2;
    let sy = y + sin(a) * radius2;
    vertex(sx, sy);
    sx = x + cos(a + halfAngle) * radius1;
    sy = y + sin(a + halfAngle) * radius1;
    vertex(sx, sy);
  }
  endShape(CLOSE);
}

// Função de detecção de colisão

´´´

function collides(x1, y1, w1, h1, x2, y2, w2, h2) {
  return !(x2 > x1 + w1 || x2 + w2 < x1 || y2 > y1 + h1 || y2 + h2 < y1);
}




