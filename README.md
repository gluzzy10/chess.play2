# chess.play2<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Chess Arena — Современные шахматы</title>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.css">
    
    <style>
        :root {
            --bg-color: #1a1a1a;
            --panel-bg: #252526;
            --accent-color: #26ab63;
            --accent-hover: #219653;
            --text-main: #ffffff;
            --text-muted: #b3b3b3;
            --border-color: #3a3a3a;
            --light-square: #f0d9b5;
            --dark-square: #b58863;
            --highlight-color: #baca44;
        }

        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
        }

        html, body {
            width: 100%;
            height: 100%;
            font-family: 'Inter', sans-serif;
            background: linear-gradient(135deg, #1a1a1a 0%, #252526 100%);
            color: var(--text-main);
            overflow-x: hidden;
        }

        body {
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            padding: 20px;
            min-height: 100vh;
        }

        .header {
            width: 100%;
            max-width: 1200px;
            padding: 20px 0;
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-bottom: 20px;
            border-bottom: 1px solid var(--border-color);
        }

        .logo {
            font-size: 28px;
            font-weight: 800;
            color: var(--accent-color);
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .header-right {
            display: flex;
            gap: 20px;
            align-items: center;
        }

        .timer-display {
            font-size: 18px;
            font-weight: 600;
            color: var(--text-muted);
            display: flex;
            gap: 30px;
        }

        .timer-group {
            text-align: center;
        }

        .timer-player {
            font-size: 12px;
            color: var(--text-muted);
            margin-bottom: 4px;
        }

        .timer-value {
            font-size: 20px;
            font-weight: 700;
            color: var(--text-main);
            font-family: 'Courier New', monospace;
        }

        .sound-toggle {
            display: flex;
            align-items: center;
            gap: 8px;
            padding: 8px 14px;
            background: rgba(255, 255, 255, 0.08);
            border-radius: 8px;
            cursor: pointer;
            user-select: none;
            border: 1px solid var(--border-color);
            transition: all 0.2s;
        }

        .sound-toggle:hover {
            background: rgba(255, 255, 255, 0.12);
        }

        .sound-toggle input {
            cursor: pointer;
            width: 18px;
            height: 18px;
        }

        .main-container {
            display: flex;
            gap: 25px;
            max-width: 1200px;
            width: 100%;
            background: var(--panel-bg);
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.8);
            border: 1px solid var(--border-color);
        }

        .board-section {
            display: flex;
            flex-direction: column;
            align-items: center;
            gap: 15px;
        }

        .player-info-card {
            width: 420px;
            padding: 14px;
            background: rgba(255, 255, 255, 0.04);
            border-radius: 10px;
            border: 1px solid var(--border-color);
            display: flex;
            align-items: center;
            justify-content: space-between;
            font-size: 14px;
            font-weight: 500;
            color: var(--text-muted);
            transition: all 0.2s;
        }

        .player-info-card.active {
            border: 2px solid var(--accent-color);
            background: rgba(38, 171, 99, 0.1);
            color: var(--text-main);
            box-shadow: 0 0 15px rgba(38, 171, 99, 0.2);
        }

        .player-content {
            display: flex;
            align-items: center;
            gap: 12px;
        }

        .player-avatar {
            width: 36px;
            height: 36px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 18px;
            color: white;
            font-weight: bold;
            border: 2px solid rgba(255, 255, 255, 0.1);
        }

        .player-details {
            display: flex;
            flex-direction: column;
            gap: 2px;
        }

        .player-name {
            font-weight: 600;
            color: var(--text-main);
        }

        .player-rating {
            font-size: 12px;
            color: var(--text-muted);
        }

        .player-status {
            font-size: 11px;
            padding: 2px 8px;
            background: rgba(38, 171, 99, 0.2);
            border-radius: 4px;
            color: var(--accent-color);
        }

        #board {
            width: 420px;
            height: 420px;
            background: linear-gradient(135deg, var(--light-square) 0%, var(--dark-square) 100%);
            border-radius: 12px;
            box-shadow: 0 10px 40px rgba(0, 0, 0, 0.6);
            overflow: hidden;
            border: 3px solid var(--border-color);
        }

        .chess-board {
            width: 100%;
            height: 100%;
            background: linear-gradient(45deg, var(--light-square) 25%, transparent 25%, transparent 75%, var(--light-square) 75%, var(--light-square)),
                        linear-gradient(45deg, var(--light-square) 25%, transparent 25%, transparent 75%, var(--light-square) 75%, var(--light-square));
            background-position: 0 0, 52.5px 52.5px;
            background-size: 105px 105px;
            background-color: var(--dark-square);
        }

        .sidebar {
            flex: 1;
            min-width: 300px;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .game-status-box {
            padding: 18px;
            background: rgba(255, 255, 255, 0.06);
            border-radius: 10px;
            border: 1px solid var(--border-color);
        }

        .status-title {
            font-size: 12px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: var(--text-muted);
            margin-bottom: 8px;
            font-weight: 600;
        }

        .status-content {
            font-size: 20px;
            font-weight: 700;
            color: var(--accent-color);
            display: flex;
            align-items: center;
            gap: 8px;
        }

        .status-icon {
            font-size: 24px;
        }

        .moves-section {
            display: flex;
            flex-direction: column;
            gap: 10px;
            flex: 1;
        }

        .moves-title {
            font-size: 13px;
            text-transform: uppercase;
            letter-spacing: 1px;
            color: var(--text-muted);
            font-weight: 600;
        }

        .moves-box {
            flex: 1;
            background: rgba(0, 0, 0, 0.3);
            border: 1px solid var(--border-color);
            border-radius: 10px;
            padding: 14px;
            overflow-y: auto;
            min-height: 200px;
            max-height: 300px;
            font-family: 'Courier New', monospace;
            font-size: 14px;
            color: var(--text-main);
        }

        .moves-box::-webkit-scrollbar {
            width: 6px;
        }

        .moves-box::-webkit-scrollbar-track {
            background: transparent;
        }

        .moves-box::-webkit-scrollbar-thumb {
            background: var(--border-color);
            border-radius: 3px;
        }

        .moves-box::-webkit-scrollbar-thumb:hover {
            background: var(--text-muted);
        }

        .moves-table {
            width: 100%;
            border-collapse: collapse;
        }

        .moves-table tr {
            transition: background 0.2s;
        }

        .moves-table tr:hover {
            background: rgba(38, 171, 99, 0.1);
        }

        .moves-table td {
            padding: 6px 8px;
        }

        .move-number {
            color: var(--text-muted);
            font-weight: 600;
            width: 35px;
        }

        .move-white, .move-black {
            color: var(--text-main);
            min-width: 60px;
        }

        .controls {
            display: flex;
            gap: 10px;
        }

        .btn {
            flex: 1;
            padding: 14px;
            font-size: 14px;
            font-weight: 600;
            border: none;
            border-radius: 10px;
            cursor: pointer;
            transition: all 0.3s;
            text-transform: uppercase;
            letter-spacing: 0.5px;
        }

        .btn-primary {
            background: linear-gradient(135deg, var(--accent-color) 0%, #1fa14f 100%);
            color: white;
            box-shadow: 0 4px 15px rgba(38, 171, 99, 0.3);
        }

        .btn-primary:hover {
            background: linear-gradient(135deg, var(--accent-hover) 0%, #188541 100%);
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(38, 171, 99, 0.4);
        }

        .btn-secondary {
            background: rgba(255, 255, 255, 0.08);
            color: var(--text-main);
            border: 1px solid var(--border-color);
        }

        .btn-secondary:hover {
            background: rgba(255, 255, 255, 0.12);
            border-color: var(--text-muted);
        }

        .game-info {
            display: flex;
            gap: 12px;
            padding: 12px;
            background: rgba(38, 171, 99, 0.1);
            border-radius: 8px;
            border: 1px solid rgba(38, 171, 99, 0.2);
            font-size: 12px;
            color: var(--text-muted);
        }

        .game-info-item {
            flex: 1;
        }

        .game-info-label {
            color: var(--text-muted);
            font-size: 10px;
            text-transform: uppercase;
            margin-bottom: 2px;
        }

        .game-info-value {
            color: var(--text-main);
            font-weight: 600;
        }

        @media (max-width: 1024px) {
            .main-container {
                flex-direction: column;
                align-items: center;
            }

            #board, .player-info-card {
                width: 100%;
                max-width: 420px;
            }

            .sidebar {
                width: 100%;
                max-width: 420px;
            }
        }

        @media (max-width: 768px) {
            .header {
                flex-direction: column;
                gap: 15px;
            }

            .timer-display {
                flex-direction: column;
                gap: 15px;
            }

            .main-container {
                padding: 20px;
            }

            #board, .player-info-card {
                width: 100%;
                max-width: 320px;
            }
        }
    </style>
</head>
<body>
    <div class="header">
        <div class="logo">
            ♟️ Chess Arena
        </div>
        <div class="header-right">
            <div class="timer-display" id="timerDisplay">
                <div class="timer-group">
                    <div class="timer-player">Соперник</div>
                    <div class="timer-value" id="blackTimer">10:00</div>
                </div>
                <div class="timer-group">
                    <div class="timer-player">Вы</div>
                    <div class="timer-value" id="whiteTimer">10:00</div>
                </div>
            </div>
            <label class="sound-toggle">
                <input type="checkbox" id="soundToggle" checked>
                <span id="soundIcon">🔊</span>
            </label>
        </div>
    </div>

    <div class="main-container">
        <div class="board-section">
            <div class="player-info-card" id="blackPlayerCard">
                <div class="player-content">
                    <div class="player-avatar">🤖</div>
                    <div class="player-details">
                        <div class="player-name">Stockfish</div>
                        <div class="player-rating">Рейтинг: 2500</div>
                    </div>
                </div>
                <div class="player-status" id="blackStatus">Ходит</div>
            </div>

            <div id="board"></div>

            <div class="player-info-card active" id="whitePlayerCard">
                <div class="player-content">
                    <div class="player-avatar">👤</div>
                    <div class="player-details">
                        <div class="player-name">Вы</div>
                        <div class="player-rating">Рейтинг: 1800</div>
                    </div>
                </div>
                <div class="player-status" id="whiteStatus">Ходит</div>
            </div>
        </div>

        <div class="sidebar">
            <div class="game-status-box">
                <div class="status-title">Статус игры</div>
                <div class="status-content">
                    <span class="status-icon">🎮</span>
                    <span id="gameStatus">Ход белых</span>
                </div>
            </div>

            <div class="moves-section">
                <div class="moves-title">История ходов</div>
                <div class="moves-box">
                    <table class="moves-table" id="movesTable">
                    </table>
                </div>
            </div>

            <div class="game-info">
                <div class="game-info-item">
                    <div class="game-info-label">Темп</div>
                    <div class="game-info-value">Блиц 10+0</div>
                </div>
                <div class="game-info-item">
                    <div class="game-info-label">Вариант</div>
                    <div class="game-info-value">Классический</div>
                </div>
                <div class="game-info-item">
                    <div class="game-info-label">Ходы</div>
                    <div class="game-info-value" id="moveCount">0</div>
                </div>
            </div>

            <div class="controls">
                <button class="btn btn-primary" id="resetBtn">↻ Новая игра</button>
                <button class="btn btn-secondary" id="undoBtn">↶ Назад</button>
            </div>
        </div>
    </div>

    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chess.js/0.10.3/chess.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/chessboard-1.0.0.min.js"></script>

    <script>
        var audioContext;
        var soundEnabled = true;
        var whiteTime = 600;
        var blackTime = 600;
        var timerInterval = null;
        var gameStarted = false;

        function initAudio() {
            if (!audioContext) {
                var AudioCtor = window.AudioContext || window.webkitAudioContext;
                if (AudioCtor) {
                    audioContext = new AudioCtor();
                }
            }
        }

        function playTone(freq, duration, gainLevel, delay) {
            if (!soundEnabled || !audioContext) return;
            if (delay === undefined) delay = 0;

            try {
                var oscillator = audioContext.createOscillator();
                var gainNode = audioContext.createGain();

                oscillator.type = 'sine';
                oscillator.frequency.setValueAtTime(freq, audioContext.currentTime + delay);
                gainNode.gain.setValueAtTime(0.0001, audioContext.currentTime + delay);
                gainNode.gain.exponentialRampToValueAtTime(gainLevel, audioContext.currentTime + delay + 0.01);
                gainNode.gain.exponentialRampToValueAtTime(0.0001, audioContext.currentTime + delay + duration);

                oscillator.connect(gainNode);
                gainNode.connect(audioContext.destination);

                oscillator.start(audioContext.currentTime + delay);
                oscillator.stop(audioContext.currentTime + delay + duration);
            } catch (e) {
                console.log('Audio error:', e);
            }
        }

        function playMoveSound() {
            playTone(420, 0.09, 0.08);
        }

        function playCaptureSound() {
            playTone(540, 0.12, 0.08);
        }

        function playCheckSound() {
            playTone(620, 0.15, 0.08);
            playTone(740, 0.15, 0.06, 0.08);
        }

        function playCheckmateSound() {
            playTone(520, 0.18, 0.08);
            playTone(660, 0.18, 0.08, 0.1);
            playTone(820, 0.22, 0.08, 0.2);
        }

        function formatTime(seconds) {
            var mins = Math.floor(seconds / 60);
            var secs = seconds % 60;
            return (mins < 10 ? '0' : '') + mins + ':' + (secs < 10 ? '0' : '') + secs;
        }

        function updateTimerDisplay() {
            $('#whiteTimer').text(formatTime(whiteTime));
            $('#blackTimer').text(formatTime(blackTime));
        }

        function startTimer() {
            if (timerInterval) clearInterval(timerInterval);
            
            timerInterval = setInterval(function() {
                if (game.turn() === 'w') {
                    whiteTime--;
                    if (whiteTime < 0) {
                        clearInterval(timerInterval);
                        gameOver('Черные победили (время белых истекло)');
                    }
                } else {
                    blackTime--;
                    if (blackTime < 0) {
                        clearInterval(timerInterval);
                        gameOver('Белые победили (время черных истекло)');
                    }
                }
                updateTimerDisplay();
            }, 1000);
        }

        var board = null;
        var game = new Chess();

        function updateUI() {
            var moveColor = game.turn() === 'w' ? 'Белые' : 'Черные';
            var moveCount = game.history().length;

            if (game.turn() === 'w') {
                $('#whitePlayerCard').addClass('active');
                $('#blackPlayerCard').removeClass('active');
                $('#whiteStatus').text('Ходит');
                $('#blackStatus').text('Ждет');
            } else {
                $('#blackPlayerCard').addClass('active');
                $('#whitePlayerCard').removeClass('active');
                $('#blackStatus').text('Ходит');
                $('#whiteStatus').text('Ждет');
            }

            let statusText = moveColor;
            let statusIcon = '🎮';

            if (game.in_checkmate()) {
                statusText = 'Мат! Победа ' + (game.turn() === 'w' ? 'Черных' : 'Белых');
                statusIcon = '♕';
                playCheckmateSound();
                if (timerInterval) clearInterval(timerInterval);
            } else if (game.in_draw()) {
                statusText = 'Ничья!';
                statusIcon = '🤝';
                if (timerInterval) clearInterval(timerInterval);
            } else if (game.in_check()) {
                statusText += ' (Шах!)';
                statusIcon = '⚠️';
                playCheckSound();
            }

            $('#gameStatus').html(statusIcon + ' ' + statusText);
            $('#moveCount').text(moveCount);
            updateMovesTable();
        }

        function updateMovesTable() {
            var history = game.history({ verbose: true });
            var tableHtml = '';
            for (var i = 0; i < history.length; i += 2) {
                var moveNum = (i / 2) + 1;
                var whiteMove = history[i] ? history[i].san : '';
                var blackMove = history[i + 1] ? history[i + 1].san : '';
                tableHtml += '<tr><td class="move-number">' + moveNum + '.</td><td class="move-white">' + whiteMove + '</td><td class="move-black">' + blackMove + '</td></tr>';
            }
            $('#movesTable').html(tableHtml);
            var container = $('.moves-box')[0];
            container.scrollTop = container.scrollHeight;
        }

        function onDragStart(source, piece, position, orientation) {
            if (game.game_over()) return false;
            if ((game.turn() === 'w' && piece.search(/^b/) !== -1) ||
                (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
                return false;
            }
        }

        function onDrop(source, target) {
            var move = game.move({
                from: source,
                to: target,
                promotion: 'q'
            });

            if (move === null) return 'snapback';

            if (move.flags.includes('c')) {
                playCaptureSound();
            } else {
                playMoveSound();
            }

            if (!gameStarted) {
                gameStarted = true;
                startTimer();
            }

            updateUI();
        }

        function onSnapEnd() {
            board.position(game.fen());
        }

        function gameOver(message) {
            $('#gameStatus').html('🏁 ' + message);
        }

        var config = {
            draggable: true,
            position: 'start',
            onDragStart: onDragStart,
            onDrop: onDrop,
            onSnapEnd: onSnapEnd,
            pieceTheme: 'https://cdnjs.cloudflare.com/ajax/libs/chessboard-js/1.0.0/img/chesspieces/wikipedia/{piece}.png'
        };

        initAudio();
        board = Chessboard('board', config);
        updateUI();
        updateTimerDisplay();

        $('#resetBtn').on('click', function() {
            if (timerInterval) clearInterval(timerInterval);
            game.reset();
            board.start();
            whiteTime = 600;
            blackTime = 600;
            gameStarted = false;
            updateTimerDisplay();
            updateUI();
        });

        $('#undoBtn').on('click', function() {
            game.undo();
            board.position(game.fen());
            updateUI();
        });

        $('#soundToggle').on('change', function() {
            soundEnabled = $(this).is(':checked');
            var icon = soundEnabled ? '🔊' : '🔇';
            $('#soundIcon').text(icon);
            if (soundEnabled) initAudio();
        });
    </script>
</body>
</html>
