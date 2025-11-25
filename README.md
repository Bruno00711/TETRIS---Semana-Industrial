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

2. Linguagem: C++
3. Software PC: Arduino IDE
4. Bibliotecas: LedControl.h
5. Diagrama:

    ![Diagrama Tetris](https://github.com/user-attachments/assets/7967926b-0e15-464e-877b-40d3f8927fbd)

  
Instalação e Montagem Passo a passo de como alguém pode replicar o projeto de vocês.
1. Montagem: Conexão da matriz de led MAX7219: GND ligado ao GND do Arduino; DIN ligado a porta digital 6; CLK ligado a porta digital 5; CS ligado ao 3 e; VCC ligado ao 5V.
     /Conexão dos botões push s/trava: todos devem estar conectados ao GND; Botão 1 (move para esquerda) conectado na porta 7; Botão 2 (move para direita) conectado na porta 8 e; Botão 3 (gira a peça) conectado na porta 2.
2. Bibliotecas: LedControl.h



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
