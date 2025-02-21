<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <title>テトリス</title>
    <link rel="stylesheet" href="styles.css">
  </head>
  <body onload="init()">
    <!-- ゲームタイトル -->
    <div id="gameTitle">
      <h1>テトリス</h1>
    </div>

    <!-- ゲームキャンバス -->
    <div id="container">
      <canvas id="cvs"></canvas>
    </div>

    <!-- スコア表示 -->
    <div id="scoreDisplay">
      <p>スコア: <span id="score">0</span></p>
    </div>

    <!-- 音楽再生ボタン -->
    <div class="audio_wrap">
      <button id="audioButton" onclick="toggleAudio()">音楽：再生/停止</button>
      <audio id="audio" src="bgm.mp3" loop>
        あなたのブラウザーは <code>audio</code> 要素をサポートしていません。
      </audio>
    </div>

    <!-- 中断/やり直しボタン -->
    <div id="buttons">
      <button id="pauseButton" onclick="pauseGame()">中断</button>
      <button id="restartButton" onclick="restartGame()">やり直し</button>
    </div>

    <!-- 中断メニュー -->
    <div id="pauseMenu" style="display: none;">
      <h2>ゲームが中断されました</h2>
      <button onclick="resumeGame()">ゲームを続ける</button>
      <button onclick="goHome()">ホームに戻る</button>
    </div>

    <script>
      // 音楽再生/停止処理
      let isAudioPlaying = false;

      const toggleAudio = () => {
        const bgm = document.getElementById("audio");
        if (bgm.paused) {
          bgm.play();
          isAudioPlaying = true;
        } else {
          bgm.pause();
          isAudioPlaying = false;
        }
      };

      // Ctrlキーで音楽の再生/停止を切り替える
      document.addEventListener('keydown', (e) => {
        if (e.ctrlKey && isAudioPlaying !== null) {
          toggleAudio();
        }
      });

      // ゲームの続行
      const resumeGame = () => {
        document.getElementById('pauseMenu').style.display = 'none';
        timerId = setInterval(gameLoop, speed);
      };

      // ホームに戻る
      const goHome = () => {
        window.location.href = 'index.html'; // ホームページに戻る
      };

      // ゲームオーバーやリスタート処理
      const pauseGame = () => {
        clearInterval(timerId);
        document.getElementById('pauseMenu').style.display = 'block';
      };

      const restartGame = () => {
        if (confirm("ゲームを最初から始めますか？")) {
          location.reload();
        }
      };

      // その他、テトリスゲームのコード
      const speed = 300; //落下サイクル
      const blockSize = 30; //1ブロックのサイズ
      const boardRow = 20;
      const boardCol = 10;
      const cvs = document.getElementById('cvs');
      const ctx = cvs.getContext('2d');
      const canvasW = blockSize * boardCol;
      const canvasH = blockSize * boardRow;
      cvs.width = canvasW;
      cvs.height = canvasH;

      const container = document.getElementById('container');
      container.style.width = canvasW + 'px';

      const tetSize = 4;
      const tetTypes = [
        [],
        [
          [0, 0, 0, 0],
          [0, 1, 1, 0],
          [0, 1, 1, 0],
          [0, 0, 0, 0],
        ],
        [
          [0, 0, 0, 0],
          [0, 1, 0, 0],
          [1, 1, 1, 0],
          [0, 0, 0, 0],
        ],
        [
          [0, 0, 0, 0],
          [1, 1, 0, 0],
          [0, 1, 1, 0],
          [0, 0, 0, 0],
        ],
        [
          [0, 0, 0, 0],
          [0, 0, 1, 1],
          [0, 1, 1, 0],
          [0, 0, 0, 0],
        ],
        [
          [0, 0, 0, 0],
          [1, 1, 1, 1],
          [0, 0, 0, 0],
          [0, 0, 0, 0],
        ],
        [
          [0, 0, 0, 0],
          [1, 1, 1, 0],
          [0, 0, 1, 0],
          [0, 0, 0, 0],
        ],
        [
          [0, 0, 0, 0],
          [0, 0, 1, 0],
          [1, 1, 1, 0],
          [0, 0, 0, 0],
        ],
      ];

      const tetColors = [
        '',
        '#f6fe85',
        '#07e0e7',
        '#7ced77',
        '#f78ff0',
        '#f94246',
        '#9693fe',
        '#f2b907',
      ];

      let tet_idx;
      let tet;
      let offsetX = 0;
      let offsetY = 0;
      const board = [];
      let timerId = NaN;
      let isGameOver = false;
      let score = 0; // スコアの初期化

      const draw = () => {
        ctx.fillStyle = '#000';
        ctx.fillRect(0, 0, canvasW, canvasH);

        for (let y = 0; y < boardRow; y++) {
          for (let x = 0; x < boardCol; x++) {
            if (board[y][x]) {
              drawBlock(x, y, board[y][x]);
            }
          }
        }

        for (let y = 0; y < tetSize; y++) {
          for (let x = 0; x < tetSize; x++) {
            if (tet[y][x]) {
              drawBlock(offsetX + x, offsetY + y, tet_idx);
            }
          }
        }

        if (isGameOver) {
          const s = 'GAME OVER';
          ctx.font = "40px 'MS ゴシック'";
          const w = ctx.measureText(s).width;
          const x = canvasW / 2 - w / 2;
          const y = canvasH / 2 - 20;
          ctx.fillStyle = 'white';
          ctx.fillText(s, x, y);
        }

        // スコア表示
        document.getElementById("score").innerText = score;
      };

      const drawBlock = (x, y, tet_idx) => {
        let px = x * blockSize;
        let py = y * blockSize;
        ctx.fillStyle = tetColors[tet_idx];
        ctx.fillRect(px, py, blockSize, blockSize);
        ctx.strokeStyle = 'black';
        ctx.strokeRect(px, py, blockSize, blockSize);
      };

      const canMove = (dx, dy, nowTet = tet) => {
        for (let y = 0; y < tetSize; y++) {
          for (let x = 0; x < tetSize; x++) {
            if (nowTet[y][x]) {
              let nx = offsetX + x + dx;
              let ny = offsetY + y + dy;
              if (
                ny < 0 ||
                nx < 0 ||
                ny >= boardRow ||
                nx >= boardCol ||
                board[ny][nx]
              ) {
                return false;
              }
            }
          }
        }
        return true;
      };

      const createRotateTet = () => {
        let newTet = [];
        for (let y = 0; y < tetSize; y++) {
          newTet[y] = [];
          for (let x = 0; x < tetSize; x++) {
            newTet[y][x] = tet[tetSize - 1 - x][y];
          }
        }
        return newTet;
      };

      // 行を消去する処理
      const clearFullLines = () => {
        let linesCleared = 0;
        outer: for (let y = boardRow - 1; y >= 0; y--) {
          for (let x = 0; x < boardCol; x++) {
            if (!board[y][x]) {
              continue outer;
            }
          }

          // 一列が埋まっている場合
          linesCleared++;

          // 行を下に移動
          for (let y2 = y; y2 >= 1; y2--) {
            for (let x = 0; x < boardCol; x++) {
              board[y2][x] = board[y2 - 1][x];
            }
          }

          // 最上段を空にする
          for (let x = 0; x < boardCol; x++) {
            board[0][x] = 0;
          }
          y++;
        }

        if (linesCleared > 0) {
          score += linesCleared * 100; // 行数に応じたスコア加算
        }
      };

      document.onkeydown = (e) => {
        if (isGameOver) return;
        switch (e.keyCode) {
          case 37:
            if (canMove(-1, 0)) offsetX--;
            break;
          case 38:
            if (canMove(0, -1)) offsetY--;
            break;
          case 39:
            if (canMove(1, 0)) offsetX++;
            break;
          case 40:
            if (canMove(0, 1)) offsetY++;
            break;
          case 32:
            let newTet = createRotateTet();
            if (canMove(0, 0, newTet)) {
              tet = newTet;
            }
        }
        draw();
      };

      const gameLoop = () => {
        if (isGameOver) {
          clearInterval(timerId);
          return;
        }
        if (!canMove(0, 1)) {
          for (let y = 0; y < tetSize; y++) {
            for (let x = 0; x < tetSize; x++) {
              if (tet[y][x]) {
                board[offsetY + y][offsetX + x] = tet_idx;
              }
            }
          }

          // 行の消去処理
          clearFullLines();

          tet_idx = Math.floor(Math.random() * (tetTypes.length - 1)) + 1;
          tet = JSON.parse(JSON.stringify(tetTypes[tet_idx]));
          offsetX = Math.floor(boardCol / 2) - Math.floor(tetSize / 2);
          offsetY = 0;
          if (!canMove(0, 0)) {
            isGameOver = true;
          }
        } else {
          offsetY++;
        }
        draw();
      };

      const init = () => {
        for (let y = 0; y < boardRow; y++) {
          board[y] = [];
          for (let x = 0; x < boardCol; x++) {
            board[y][x] = 0;
          }
        }
        tet_idx = Math.floor(Math.random() * (tetTypes.length - 1)) + 1;
        tet = JSON.parse(JSON.stringify(tetTypes[tet_idx]));
        offsetX = Math.floor(boardCol / 2) - Math.floor(tetSize / 2);
        offsetY = 0;

        timerId = setInterval(gameLoop, speed);
        draw();
      };
    </script>
  </body>
</html>
``
