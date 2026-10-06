# chess.play2<!DOCTYPE html>
<script>
    var audioContext;
    var soundEnabled = true;
    var whiteTime = 600;
    var blackTime = 600;
    var timerInterval = null;
    var gameStarted = false;
    var botColor = 'b'; // бот играет чёрными
    var botThinking = false;

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

    function squareFromCoords(file, rank) {
        return String.fromCharCode(97 + file) + (8 - rank);
    }

    function evaluateBoard() {
        var boardState = game.board();
        var pieceValues = { p: 100, n: 320, b: 330, r: 500, q: 900, k: 20000 };
        var score = 0;
        var centerSquares = {
            'd4': true, 'e4': true, 'd5': true, 'e5': true,
            'c4': true, 'f4': true, 'c5': true, 'f5': true,
            'c3': true, 'f3': true, 'c6': true, 'f6': true,
            'd3': true, 'e3': true, 'd6': true, 'e6': true
        };

        for (var row = 0; row < 8; row++) {
            for (var col = 0; col < 8; col++) {
                var piece = boardState[row][col];
                if (!piece) continue;

                var square = squareFromCoords(col, row);
                var value = pieceValues[piece.type];
                var sign = piece.color === 'w' ? 1 : -1;

                score += sign * value;

                if (centerSquares[square]) {
                    score += sign * 12;
                }

                if (piece.type === 'p') {
                    score += sign * (piece.color === 'w' ? row : (7 - row)) * 2;
                }
            }
        }

        if (game.in_check()) {
            score += game.turn() === 'w' ? -30 : 30;
        }

        return score;
    }

    function minimax(depth, alpha, beta, maximizingPlayer) {
        if (depth === 0 || game.game_over()) {
            return evaluateBoard();
        }

        var moves = game.moves({ verbose: true });

        if (game.turn() === maximizingPlayer) {
            var maxEval = -Infinity;

            for (var i = 0; i < moves.length; i++) {
                game.move(moves[i]);

                var evalScore = minimax(depth - 1, alpha, beta, maximizingPlayer);
                game.undo();

                maxEval = Math.max(maxEval, evalScore);
                alpha = Math.max(alpha, evalScore);

                if (beta <= alpha) {
                    break;
                }
            }

            return maxEval;
        } else {
            var minEval = Infinity;

            for (var i = 0; i < moves.length; i++) {
                game.move(moves[i]);

                var evalScore = minimax(depth - 1, alpha, beta, maximizingPlayer);
                game.undo();

                minEval = Math.min(minEval, evalScore);
                beta = Math.min(beta, evalScore);

                if (beta <= alpha) {
                    break;
                }
            }

            return minEval;
        }
    }

    function chooseBotMove() {
        var legalMoves = game.moves({ verbose: true });
        if (!legalMoves.length) return null;

        var bestMove = legalMoves[0];
        var bestScore = -Infinity;

        for (var i = 0; i < legalMoves.length; i++) {
            game.move(legalMoves[i]);
            var score = minimax(2, -Infinity, Infinity, botColor);
            game.undo();

            if (score > bestScore) {
                bestScore = score;
                bestMove = legalMoves[i];
            }
        }

        return bestMove;
    }

    function makeBotMove() {
        if (botThinking || game.game_over() || game.turn() !== botColor) return;

        botThinking = true;

        setTimeout(function() {
            var move = chooseBotMove();

            if (!move) {
                botThinking = false;
                updateUI();
                return;
            }

            game.move(move);

            if (move.flags.includes('c')) {
                playCaptureSound();
            } else {
                playMoveSound();
            }

            if (!gameStarted) {
                gameStarted = true;
                startTimer();
            }

            board.position(game.fen());
            updateUI();
            botThinking = false;

            if (game.game_over()) {
                if (timerInterval) clearInterval(timerInterval);
            }
        }, 500);
    }

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
        if (botThinking || game.game_over()) return false;
        if (game.turn() === botColor) return false;
        if ((game.turn() === 'w' && piece.search(/^b/) !== -1) ||
            (game.turn() === 'b' && piece.search(/^w/) !== -1)) {
            return false;
        }
    }

    function onDrop(source, target) {
        if (botThinking || game.turn() !== 'w') return 'snapback';

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

        if (game.turn() === botColor) {
            makeBotMove();
        }
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
        botThinking = false;
        updateTimerDisplay();
        updateUI();
    });

    $('#undoBtn').on('click', function() {
        if (botThinking) return;
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
