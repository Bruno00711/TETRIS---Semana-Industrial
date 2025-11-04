Tetris
Projeto desenvolvido para a Semana do Industrial 2025, como o tema principal era "Jogos Eletrônicos utilizando o Arduino" decidimos recriar o clássico jogo Tetris. Um jogo de quebra-cabeça em que os jogadores devem organizar os blocos em queda.  O objetivo é criar linhas horizontais completas com os blocos, que então desaparecem, permitindo que mais blocos caiam.
 Equipe (Colaboradores)
 https://github.com/Bruno00711 - Bruno Eiji Assakura Matsu/
 https://github.com/torrescarneiroraul-png - Raul Torres Carneiro de Lima/
 https://github.com/cad-u - Carlos Eduardo Amaral Maranhão/
 https://github.com/FelipeRodriguesBatista - Felipe Rodrigues Batista/


Descrição do Projeto Aqui
 O Desafio da Semana do Industrial 2025, tema geral do primeiro ano de mecatrônica, era a criação de um jogo eletrônico utilizando o Arduino. Sabendo disso decidimos recriar o Tetris.
 O nosso projeto funciona, principalmente, através do Arduino e o seu código programado em linguagem C++. Ele segue a mesma lógica do Tetris tradicional, mas adaptado para uma matriz de led 8x32.


 Hardware (Componentes Utilizados) Lista de todos os componentes físicos necessários para montar o projeto.
 Controlador: 1x Arduino Uno R3
 Sensores: 3x Botões Push s/trava
 Atuadores: Matriz de Led MAX7219 (8x32)
 Outros: Jumpers (Macho-Fêmea, Macho-Macho, Fêmea-Fêmea)


Software e Dependências O que é necessário para rodar o código?
1. Firmware/Código: 
#include <LedControl.h>

// ---------------- CONFIGURAÇÕES DE HARDWARE ----------------
#define DIN_PIN 6
#define CLK_PIN 5
#define CS_PIN  3
LedControl lc = LedControl(DIN_PIN, CLK_PIN, CS_PIN, 1); // 1 módulo MAX7219

// Botões
#define BTN_LEFT  7
#define BTN_RIGHT 8
#define BTN_ROT   2

// ---------------- VARIÁVEIS DO JOGO ----------------
const byte ROWS = 8;
const byte COLS = 32;
bool grid[ROWS][COLS]; // 0 = vazio, 1 = ocupado

// Peças (definidas em 4x4)
const byte NUM_PIECES = 3; // Exemplo: apenas 3 peças simples para caber na tela
const byte pieceShapes[NUM_PIECES][4][4] = {
  { // Quadrado
    {0,1,1,0},
    {0,1,1,0},
    {0,0,0,0},
    {0,0,0,0}
  },
  { // Linha
    {0,0,0,0},
    {1,1,1,1},
    {0,0,0,0},
    {0,0,0,0}
  },
  { // T
    {0,1,0,0},
    {1,1,1,0},
    {0,0,0,0},
    {0,0,0,0}
  }
};

// Estrutura da peça atual
int pieceX, pieceY;
byte currentPiece[4][4];
unsigned long lastFallTime = 0;
int fallDelay = 500; // milissegundos entre quedas

// ---------------- FUNÇÕES AUXILIARES ----------------

// Limpa o display e a grade
void clearGrid() {
  for (int r = 0; r < ROWS; r++) {
    for (int c = 0; c < COLS; c++) {
      grid[r][c] = 0;
    }
  }
}

// Atualiza o display LED
void drawGrid() {
  lc.clearDisplay(0);
  for (int r = 0; r < ROWS; r++) {
    for (int c = 0; c < COLS; c++) {
      lc.setLed(0, r, c, grid[r][c]);
    }
  }
}

// Desenha a peça atual na matriz (sem fixar)
void drawPiece(bool state) {
  for (int r = 0; r < 4; r++) {
    for (int c = 0; c < 4; c++) {
      if (currentPiece[r][c]) {
        int gx = pieceX + c;
        int gy = pieceY + r;
        if (gx >= 0 && gx < COLS && gy >= 0 && gy < ROWS) {
          grid[gy][gx] = state;
        }
      }
    }
  }
}

// Testa colisão
bool checkCollision(int newX, int newY, byte newPiece[4][4]) {
  for (int r = 0; r < 4; r++) {
    for (int c = 0; c < 4; c++) {
      if (newPiece[r][c]) {
        int gx = newX + c;
        int gy = newY + r;
        if (gx < 0 || gx >= COLS || gy >= ROWS) return true;
        if (gy >= 0 && grid[gy][gx]) return true;
      }
    }
  }
  return false;
}

// Gira a peça 90 graus
void rotatePiece() {
  byte temp[4][4];
  for (int r = 0; r < 4; r++)
    for (int c = 0; c < 4; c++)
      temp[c][3 - r] = currentPiece[r][c];

  if (!checkCollision(pieceX, pieceY, temp)) {
    memcpy(currentPiece, temp, sizeof(temp));
  }
}

// Fixa a peça na grade
void fixPiece() {
  for (int r = 0; r < 4; r++) {
    for (int c = 0; c < 4; c++) {
      if (currentPiece[r][c]) {
        int gx = pieceX + c;
        int gy = pieceY + r;
        if (gx >= 0 && gx < COLS && gy >= 0 && gy < ROWS) {
          grid[gy][gx] = 1;
        }
      }
    }
  }
}

// Remove linhas completas (simples, de 8 colunas apenas)
void clearFullLines() {
  for (int r = ROWS - 1; r >= 0; r--) {
    bool full = true;
    for (int c = 0; c < COLS; c++) {
      if (!grid[r][c]) {
        full = false;
        break;
      }
    }
    if (full) {
      for (int y = r; y > 0; y--) {
        for (int c = 0; c < COLS; c++) {
          grid[y][c] = grid[y - 1][c];
        }
      }
      for (int c = 0; c < COLS; c++) grid[0][c] = 0;
      r++; // rechecagem
    }
  }
}

// Cria nova peça aleatória
void newPiece() {
  int type = random(NUM_PIECES);
  memcpy(currentPiece, pieceShapes[type], sizeof(currentPiece));
  pieceX = COLS / 2 - 2;
  pieceY = 0;
  if (checkCollision(pieceX, pieceY, currentPiece)) {
    // Game Over
    clearGrid();
  }
}

// ---------------- ARDUINO SETUP ----------------
void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 8);
  lc.clearDisplay(0);

  pinMode(BTN_LEFT, INPUT_PULLUP);
  pinMode(BTN_RIGHT, INPUT_PULLUP);
  pinMode(BTN_ROT, INPUT_PULLUP);

  randomSeed(analogRead(0));

  clearGrid();
  newPiece();
  drawGrid();
}

// ---------------- LOOP PRINCIPAL ----------------
void loop() {
  // Leitura de botões
  if (digitalRead(BTN_LEFT) == LOW) {
    drawPiece(false);
    if (!checkCollision(pieceX - 1, pieceY, currentPiece)) pieceX--;
    drawPiece(true);
    drawGrid();
    delay(100);
  }

  if (digitalRead(BTN_RIGHT) == LOW) {
    drawPiece(false);
    if (!checkCollision(pieceX + 1, pieceY, currentPiece)) pieceX++;
    drawPiece(true);
    drawGrid();
    delay(100);
  }

  if (digitalRead(BTN_ROT) == LOW) {
    drawPiece(false);
    rotatePiece();
    drawPiece(true);
    drawGrid();
    delay(150);
  }

  // Queda automática
  if (millis() - lastFallTime > fallDelay) {
    lastFallTime = millis();
    drawPiece(false);
    if (!checkCollision(pieceX, pieceY + 1, currentPiece)) {
      pieceY++;
    } else {
      fixPiece();
      clearFullLines();
      newPiece();
    }
    drawPiece(true);
    drawGrid();
  }
}


2. Linguagem: C++
3. Software PC: Arduino IDE
4. Bibliotecas: LedControl.h
5. Diagrama:

    ![Diagrama Tetris](https://github.com/user-attachments/assets/7967926b-0e15-464e-877b-40d3f8927fbd)

  
Instalação e Montagem Passo a passo de como alguém pode replicar o projeto de vocês.
1. Montagem: Conexão da matriz de led MAX7219: GND ligado ao GND do Arduino; DIN ligado a porta digital 6; CLK ligado a porta digital 5; CS ligado ao 3 e; VCC ligado ao 5V.
     /Conexão dos botões push s/trava: todos devem estar conectados ao GND; Botão 1 (move para esquerda) conectado na porta 7; Botão 2 (move para direita) conectado na porta 8 e; Botão 3 (gira a peça) conectado na porta 2.
2. Bibliotecas: LedControl.h
3. Upload Do Código:
#include <LedControl.h>

// ---------------- CONFIGURAÇÕES DE HARDWARE ----------------
#define DIN_PIN 6
#define CLK_PIN 5
#define CS_PIN  3
LedControl lc = LedControl(DIN_PIN, CLK_PIN, CS_PIN, 1); // 1 módulo MAX7219

// Botões
#define BTN_LEFT  7
#define BTN_RIGHT 8
#define BTN_ROT   2

// ---------------- VARIÁVEIS DO JOGO ----------------
const byte ROWS = 8;
const byte COLS = 32;
bool grid[ROWS][COLS]; // 0 = vazio, 1 = ocupado

// Peças (definidas em 4x4)
const byte NUM_PIECES = 3; // Exemplo: apenas 3 peças simples para caber na tela
const byte pieceShapes[NUM_PIECES][4][4] = {
  { // Quadrado
    {0,1,1,0},
    {0,1,1,0},
    {0,0,0,0},
    {0,0,0,0}
  },
  { // Linha
    {0,0,0,0},
    {1,1,1,1},
    {0,0,0,0},
    {0,0,0,0}
  },
  { // T
    {0,1,0,0},
    {1,1,1,0},
    {0,0,0,0},
    {0,0,0,0}
  }
};

// Estrutura da peça atual
int pieceX, pieceY;
byte currentPiece[4][4];
unsigned long lastFallTime = 0;
int fallDelay = 500; // milissegundos entre quedas

// ---------------- FUNÇÕES AUXILIARES ----------------

// Limpa o display e a grade
void clearGrid() {
  for (int r = 0; r < ROWS; r++) {
    for (int c = 0; c < COLS; c++) {
      grid[r][c] = 0;
    }
  }
}

// Atualiza o display LED
void drawGrid() {
  lc.clearDisplay(0);
  for (int r = 0; r < ROWS; r++) {
    for (int c = 0; c < COLS; c++) {
      lc.setLed(0, r, c, grid[r][c]);
    }
  }
}

// Desenha a peça atual na matriz (sem fixar)
void drawPiece(bool state) {
  for (int r = 0; r < 4; r++) {
    for (int c = 0; c < 4; c++) {
      if (currentPiece[r][c]) {
        int gx = pieceX + c;
        int gy = pieceY + r;
        if (gx >= 0 && gx < COLS && gy >= 0 && gy < ROWS) {
          grid[gy][gx] = state;
        }
      }
    }
  }
}

// Testa colisão
bool checkCollision(int newX, int newY, byte newPiece[4][4]) {
  for (int r = 0; r < 4; r++) {
    for (int c = 0; c < 4; c++) {
      if (newPiece[r][c]) {
        int gx = newX + c;
        int gy = newY + r;
        if (gx < 0 || gx >= COLS || gy >= ROWS) return true;
        if (gy >= 0 && grid[gy][gx]) return true;
      }
    }
  }
  return false;
}

// Gira a peça 90 graus
void rotatePiece() {
  byte temp[4][4];
  for (int r = 0; r < 4; r++)
    for (int c = 0; c < 4; c++)
      temp[c][3 - r] = currentPiece[r][c];

  if (!checkCollision(pieceX, pieceY, temp)) {
    memcpy(currentPiece, temp, sizeof(temp));
  }
}

// Fixa a peça na grade
void fixPiece() {
  for (int r = 0; r < 4; r++) {
    for (int c = 0; c < 4; c++) {
      if (currentPiece[r][c]) {
        int gx = pieceX + c;
        int gy = pieceY + r;
        if (gx >= 0 && gx < COLS && gy >= 0 && gy < ROWS) {
          grid[gy][gx] = 1;
        }
      }
    }
  }
}

// Remove linhas completas (simples, de 8 colunas apenas)
void clearFullLines() {
  for (int r = ROWS - 1; r >= 0; r--) {
    bool full = true;
    for (int c = 0; c < COLS; c++) {
      if (!grid[r][c]) {
        full = false;
        break;
      }
    }
    if (full) {
      for (int y = r; y > 0; y--) {
        for (int c = 0; c < COLS; c++) {
          grid[y][c] = grid[y - 1][c];
        }
      }
      for (int c = 0; c < COLS; c++) grid[0][c] = 0;
      r++; // rechecagem
    }
  }
}

// Cria nova peça aleatória
void newPiece() {
  int type = random(NUM_PIECES);
  memcpy(currentPiece, pieceShapes[type], sizeof(currentPiece));
  pieceX = COLS / 2 - 2;
  pieceY = 0;
  if (checkCollision(pieceX, pieceY, currentPiece)) {
    // Game Over
    clearGrid();
  }
}

// ---------------- ARDUINO SETUP ----------------
void setup() {
  lc.shutdown(0, false);
  lc.setIntensity(0, 8);
  lc.clearDisplay(0);

  pinMode(BTN_LEFT, INPUT_PULLUP);
  pinMode(BTN_RIGHT, INPUT_PULLUP);
  pinMode(BTN_ROT, INPUT_PULLUP);

  randomSeed(analogRead(0));

  clearGrid();
  newPiece();
  drawGrid();
}

// ---------------- LOOP PRINCIPAL ----------------
void loop() {
  // Leitura de botões
  if (digitalRead(BTN_LEFT) == LOW) {
    drawPiece(false);
    if (!checkCollision(pieceX - 1, pieceY, currentPiece)) pieceX--;
    drawPiece(true);
    drawGrid();
    delay(100);
  }

  if (digitalRead(BTN_RIGHT) == LOW) {
    drawPiece(false);
    if (!checkCollision(pieceX + 1, pieceY, currentPiece)) pieceX++;
    drawPiece(true);
    drawGrid();
    delay(100);
  }

  if (digitalRead(BTN_ROT) == LOW) {
    drawPiece(false);
    rotatePiece();
    drawPiece(true);
    drawGrid();
    delay(150);
  }

  // Queda automática
  if (millis() - lastFallTime > fallDelay) {
    lastFallTime = millis();
    drawPiece(false);
    if (!checkCollision(pieceX, pieceY + 1, currentPiece)) {
      pieceY++;
    } else {
      fixPiece();
      clearFullLines();
      newPiece();
    }
    drawPiece(true);
    drawGrid();
  }
}




Como Usar Depois de montado e programado, como o projeto funciona?
 1. Ligue a fonte de alimentação
 2. O jogo inicia automaticamente com uma peça caindo.
Botão 1 (pino 7): move para a esquerda
Botão 2 (pino 8): move para a direita
Botão 3 (pino 2): gira a peça
As peças descem automaticamente a cada meio segundo (fallDelay = 500 ms).
Quando uma peça atinge o fundo, ela se fixa e surge uma nova.


 Vídeo/GIF do Projeto em Ação:

https://github.com/user-attachments/assets/0ab5b232-1b32-433e-b47f-18b4ba3a787f
