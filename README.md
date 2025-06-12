<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Hidden Dragon Chess II: The Rise of the Archer</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: #5a3a22;
            color: #f0e2d0;
        }
        #game-container {
            display: flex;
            flex-direction: column;
            align-items: center;
            text-align: center;
        }
        #chess-board {
            display: grid;
            grid-template-columns: repeat(8, 70px);
            grid-template-rows: repeat(4, 70px);
            background-color: #e3b778;
            border: 3px solid #3d281a;
            padding: 35px;
            position: relative;
        }
        .square {
            width: 70px;
            height: 70px;
            display: flex;
            justify-content: center;
            align-items: center;
            box-sizing: border-box;
            border: 1px solid #b38b5d;
            cursor: pointer;
        }
        .piece {
            width: 60px;
            height: 60px;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-size: 36px;
            font-weight: bold;
            box-shadow: 0 2px 4px rgba(0,0,0,0.4);
            -webkit-user-select: none;
            -ms-user-select: none;
            user-select: none;
        }
        .red-piece {
            background-color: #f0d5b1;
            border: 2px solid #c0392b;
            color: #c0392b;
        }
        .black-piece {
            background-color: #f0d5b1;
            border: 2px solid #2c3e50;
            color: #2c3e50;
        }
        .hidden-piece {
            background-color: #4a779c;
            border: 2px solid #2e4a60;
            cursor: pointer;
        }
         .hidden-piece:hover { background-color: #6a9ac3; }
        .selected .piece {
            box-shadow: 0 0 15px 5px #f1c40f;
        }
        .valid-move::after {
            content: '';
            position: absolute;
            width: 20px;
            height: 20px;
            background-color: rgba(46, 204, 113, 0.7);
            border-radius: 50%;
        }
        #info-panel { display: flex; justify-content: space-around; width: 100%; max-width: 640px; margin-top:20px; }
        .player-info { width: 250px; padding: 10px; border: 2px solid #f0e2d0; border-radius: 8px; }
        #player1-info.active, #player2-info.active { border-color: #f1c40f; }
        #game-status { font-size: 1.2em; font-weight: bold; height: 50px; }
        #game-over { font-size: 1.5em; color: #e74c3c; font-weight: bold; }
        button { margin-top: 15px; padding: 10px 20px; font-size: 1em; cursor: pointer; border:none; border-radius:5px; background-color:#2e4a60; color:white; }
        button:hover { background-color:#4a779c; }
    </style>
</head>
<body>
<div id="game-container">
    <h1>Hidden Dragon Chess II: The Rise of the Archer</h1>
    <div id="chess-board"></div>
    <div id="info-panel">
        <div id="player1-info" class="player-info">
            <h3>Player 1</h3>
            <p id="player1-color">Color: (Undecided)</p>
            <p id="player1-score">Score: 0</p>
        </div>
        <div id="player2-info" class="player-info">
            <h3>Player 2</h3>
            <p id="player2-color">Color: (Undecided)</p>
            <p id="player2-score">Score: 0</p>
        </div>
    </div>
    <p id="game-status"></p>
    <p id="game-over"></p>
    <button id="reset-button">New Game</button>
</div>

<script>
    document.addEventListener('DOMContentLoaded', () => {
        const boardElement = document.getElementById('chess-board');
        const p1ColorEl = document.getElementById('player1-color');
        const p2ColorEl = document.getElementById('player2-color');
        const p1ScoreEl = document.getElementById('player1-score');
        const p2ScoreEl = document.getElementById('player2-score');
        const gameStatusEl = document.getElementById('game-status');
        const gameOverEl = document.getElementById('game-over');
        const resetButton = document.getElementById('reset-button');
        const p1InfoEl = document.getElementById('player1-info');
        const p2InfoEl = document.getElementById('player2-info');

        const COLS = 8;
        const ROWS = 4;
        const SQUARES = COLS * ROWS;

        let board = [];
        let selectedPieceIndex = null;
        let validMoves = [];
        let isTurnInProgress = false;
        
        let isGameStarted = false;
        let player1Color = null; 
        let isRedTurn = true; 
        let isGameOver = false;

        const scores = { 'k': 10, 'r': 9, 'c': 5, 'h': 4, 'w': 3, 'e': 2, 'a': 2, 's': 1 };
        
        // **MODIFIED**: New data structure for piece characters
        const pieceDisplay = {
            'k': { red: '帥', black: '將' },
            'a': { red: '仕', black: '士' },
            'e': { red: '相', black: '象' },
            'h': { red: '傌', black: '馬' },
            'r': { red: '俥', black: '車' },
            'c': { red: '砲', black: '炮' },
            's': { red: '兵', black: '卒' },
            'w': { red: '弩', black: '弓' }
        };

        function initializeBoard() {
            const redPieces = ['R','H','E','A','K','A','E','H','R','C','C','S','S','S','S','W'];
            const blackPieces = redPieces.map(p => p.toLowerCase());
            const allPieces = [...redPieces, ...blackPieces];
            
            for (let i = allPieces.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [allPieces[i], allPieces[j]] = [allPieces[j], allPieces[i]];
            }

            board = allPieces.map(p => ({ piece: p, isFlipped: false }));
            
            isGameStarted = false;
            isGameOver = false;
            isTurnInProgress = false;
            player1Color = null;
            selectedPieceIndex = null;
            validMoves = [];

            gameStatusEl.textContent = 'Player 1: Flip a piece to begin!';
            gameOverEl.textContent = '';
            p1ColorEl.textContent = 'Color: (Undecided)';
            p2ColorEl.textContent = 'Color: (Undecided)';
            p1ScoreEl.textContent = 'Score: 0';
            p2ScoreEl.textContent = 'Score: 0';
            p1InfoEl.classList.add('active');
            p2InfoEl.classList.remove('active');
            
            renderBoard();
        }

        function renderBoard() {
            boardElement.innerHTML = '';
            for (let i = 0; i < SQUARES; i++) {
                const square = document.createElement('div');
                square.classList.add('square');
                
                const cell = board[i];

                const pieceSpan = document.createElement('span');
                pieceSpan.classList.add('piece');

                if (!cell.isFlipped) {
                    pieceSpan.classList.add('hidden-piece');
                } else if (cell.piece) {
                    const pieceColorClass = isRedPiece(cell.piece) ? 'red-piece' : 'black-piece';
                    pieceSpan.classList.add(pieceColorClass);
                    
                    // **MODIFIED**: Use the new data structure to get the correct character
                    const pieceType = cell.piece.toLowerCase();
                    const isRed = isRedPiece(cell.piece);
                    pieceSpan.textContent = pieceDisplay[pieceType][isRed ? 'red' : 'black'];
                }
                square.appendChild(pieceSpan);

                if (selectedPieceIndex === i) square.classList.add('selected');
                if (validMoves.includes(i)) square.classList.add('valid-move');

                square.addEventListener('click', () => onSquareClick(i));
                boardElement.appendChild(square);
            }
        }

        function onSquareClick(index) {
            if (isGameOver || isTurnInProgress) return;

            if (!isGameStarted) {
                handleFirstFlip(index);
                return;
            }

            const clickedCell = board[index];

            if (selectedPieceIndex !== null) {
                if (validMoves.includes(index)) {
                    movePiece(selectedPieceIndex, index);
                } else { 
                    selectedPieceIndex = null;
                    validMoves = [];
                    renderBoard();
                }
                return;
            }
            
            if (!clickedCell.isFlipped) {
                isTurnInProgress = true;
                clickedCell.isFlipped = true;
                renderBoard(); 

                setTimeout(() => {
                    switchTurn();
                    isTurnInProgress = false;
                }, 500);
                return;
            }

            if (clickedCell.piece && isPieceOfCurrentTurn(clickedCell.piece)) {
                selectedPieceIndex = index;
                validMoves = getValidMoves(clickedCell.piece, index);
                renderBoard();
            }
        }

        function handleFirstFlip(index) {
            const cell = board[index];
            if (!cell.piece) return; 

            isTurnInProgress = true;
            cell.isFlipped = true;
            isGameStarted = true;
            renderBoard(); 

            if (isRedPiece(cell.piece)) {
                player1Color = 'black';
                isRedTurn = true; 
            } else {
                player1Color = 'red';
                isRedTurn = false; 
            }
            
            setTimeout(() => {
                switchTurn();
                isTurnInProgress = false;
            }, 500);
        }

        function movePiece(from, to) {
            board[to].piece = board[from].piece;
            board[from].piece = null;
            switchTurn();
        }
        
        function switchTurn() {
            selectedPieceIndex = null;
            validMoves = [];
            isRedTurn = !isRedTurn;
            
            updatePlayerUI();
            renderBoard();
            checkGameOver();
        }
        
        function checkGameOver() {
            if (!isGameStarted || player1Color === null) return;

            let hasLegalAction = false;
            const canFlip = board.some(cell => !cell.isFlipped);
            if (canFlip) {
                hasLegalAction = true;
            }
            
            if (!hasLegalAction) {
                for (let i = 0; i < SQUARES; i++) {
                    const cell = board[i];
                    if (cell.piece && isPieceOfCurrentTurn(cell.piece)) {
                        if (getValidMoves(cell.piece, i).length > 0) {
                            hasLegalAction = true;
                            break;
                        }
                    }
                }
            }
            
            if (!hasLegalAction) {
                let redPieceCount = 0;
                let blackPieceCount = 0;
                board.forEach(cell => {
                    if (cell.piece && cell.isFlipped) {
                        if (isRedPiece(cell.piece)) redPieceCount++;
                        else blackPieceCount++;
                    }
                });
                
                if (redPieceCount === 0 || blackPieceCount === 0) {
                    gameOverEl.textContent = "One side has been eliminated!";
                } else {
                    gameOverEl.textContent = "No legal moves remaining!";
                }
                endGameAndScore();
            }
        }
        
        function endGameAndScore() {
            isGameOver = true;
            let redScore = 0;
            let blackScore = 0;

            board.forEach(cell => {
                if(cell.isFlipped && cell.piece) {
                    const score = scores[cell.piece.toLowerCase()];
                    if(isRedPiece(cell.piece)) redScore += score;
                    else blackScore += score;
                }
            });
            
            p1ScoreEl.textContent = `Final Score: ${player1Color === 'red' ? redScore : blackScore}`;
            p2ScoreEl.textContent = `Final Score: ${player1Color === 'red' ? blackScore : redScore}`;

            const p1Final = player1Color === 'red' ? redScore : blackScore;
            const p2Final = player1Color === 'red' ? blackScore : redScore;
            
            let winnerMessage = '';
            if (p1Final > p2Final) winnerMessage = "Player 1 Wins!";
            else if (p2Final > p1Final) winnerMessage = "Player 2 Wins!";
            else winnerMessage = "It's a Draw!";
            
            const finalMessage = gameOverEl.textContent ? `${gameOverEl.textContent} ${winnerMessage}` : `Game Over - ${winnerMessage}`;
            gameOverEl.textContent = finalMessage;
            gameStatusEl.textContent = '';
        }

        function getValidMoves(piece, index) {
            const legalMoves = [];
            const r = Math.floor(index / COLS);
            const c = index % COLS;
            const pieceIsRed = isRedPiece(piece);

            const isOnBoard = (row, col) => row >= 0 && row < ROWS && col >= 0 && col < COLS;
            const getIndex = (row, col) => row * COLS + col;

            switch (piece.toLowerCase()) {
                case 's': 
                    [[-1,0],[1,0],[0,-1],[0,1]].forEach(([rd,cd]) => {
                        if(isOnBoard(r+rd, c+cd)){
                            const targetIndex = getIndex(r+rd, c+cd);
                            const targetCell = board[targetIndex];
                            if(targetCell.isFlipped && (!targetCell.piece || pieceIsRed !== isRedPiece(targetCell.piece))) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    break;
                
                case 'w': 
                    const archerMoveDirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];
                    const archerCaptureDirs = [[-2,0],[2,0],[0,-2],[0,2],[-2,-2],[-2,2],[2,-2],[2,2]];
                    archerMoveDirs.forEach(([rd, cd]) => {
                        if (isOnBoard(r+rd, c+cd)) {
                            const targetIndex = getIndex(r+rd, c+cd);
                            const targetCell = board[targetIndex];
                            if (targetCell.isFlipped && !targetCell.piece) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    archerCaptureDirs.forEach(([rd,cd]) => {
                         if(isOnBoard(r+rd, c+cd)){
                            const targetIndex = getIndex(r+rd, c+cd);
                            const targetCell = board[targetIndex];
                            if(targetCell.isFlipped && targetCell.piece && !isPieceOfCurrentTurn(targetCell.piece)) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    break;

                case 'k':
                    [[-1,0],[1,0],[0,-1],[0,1],[-1,-1],[-1,1],[1,-1],[1,1]].forEach(([rd,cd]) => {
                         if(isOnBoard(r+rd, c+cd)){
                            const targetIndex = getIndex(r+rd, c+cd);
                            const targetCell = board[targetIndex];
                            if(targetCell.isFlipped && (!targetCell.piece || pieceIsRed !== isRedPiece(targetCell.piece))) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    break;

                case 'a':
                    const straightDirs = [[-1, 0], [1, 0], [0, -1], [0, 1]];
                    const diagonalDirs = [[-1, -1], [-1, 1], [1, -1], [1, 1]];
                    straightDirs.forEach(([rd, cd]) => {
                        if (isOnBoard(r + rd, c + cd)) {
                            const targetIndex = getIndex(r + rd, c + cd);
                            const targetCell = board[targetIndex];
                            if (targetCell.isFlipped && !targetCell.piece) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    diagonalDirs.forEach(([rd, cd]) => {
                        if (isOnBoard(r + rd, c + cd)) {
                            const targetIndex = getIndex(r + rd, c + cd);
                            const targetCell = board[targetIndex];
                            if (targetCell.isFlipped && (!targetCell.piece || pieceIsRed !== isRedPiece(targetCell.piece))) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    break;
                
                case 'e': 
                    [[-1,-1],[-1,1],[1,-1],[1,1]].forEach(([rd,cd]) => {
                        let temp_r = r + rd, temp_c = c + cd;
                        while(isOnBoard(temp_r, temp_c)){
                            const i = getIndex(temp_r, temp_c);
                            if(!board[i].isFlipped) break;
                            if(board[i].piece){
                                if(isRedPiece(board[i].piece) !== isRedPiece(piece)) legalMoves.push(i);
                                break;
                            }
                            legalMoves.push(i);
                            temp_r += rd; temp_c += cd;
                        }
                    });
                    break;
                
                case 'h': 
                    [[-2,-1],[-2,1],[-1,-2],[-1,2],[1,-2],[1,2],[2,-1],[2,1]].forEach(([rd,cd]) => {
                         if(isOnBoard(r+rd, c+cd)){
                            const targetIndex = getIndex(r+rd, c+cd);
                            const targetCell = board[targetIndex];
                            if(targetCell.isFlipped && (!targetCell.piece || pieceIsRed !== isRedPiece(targetCell.piece))) {
                                legalMoves.push(targetIndex);
                            }
                        }
                    });
                    break;

                case 'r':
                    [[-1,0],[1,0],[0,-1],[0,1]].forEach(([rd,cd]) => {
                        let temp_r = r + rd, temp_c = c + cd;
                        while(isOnBoard(temp_r, temp_c)){
                            const i = getIndex(temp_r, temp_c);
                            if(!board[i].isFlipped) break;
                            if(board[i].piece){
                                if(isRedPiece(board[i].piece) !== pieceIsRed) legalMoves.push(i);
                                break;
                            }
                            legalMoves.push(i);
                            temp_r += rd; temp_c += cd;
                        }
                    });
                    break;
                
                case 'c':
                    [[-1,0],[1,0],[0,-1],[0,1]].forEach(([rd,cd]) => {
                        let temp_r = r + rd, temp_c = c + cd;
                        let screenFound = false;
                        while(isOnBoard(temp_r, temp_c)){
                            const i = getIndex(temp_r, temp_c);
                            const currentCell = board[i];
                            if(currentCell.piece){
                                if(!screenFound){
                                    screenFound = true; 
                                } else { 
                                    if(currentCell.isFlipped && isRedPiece(currentCell.piece) !== pieceIsRed){
                                        legalMoves.push(i);
                                    }
                                    break; 
                                }
                            } else {
                                if (!screenFound && currentCell.isFlipped) {
                                    legalMoves.push(i);
                                }
                            }
                            temp_r += rd; temp_c += cd;
                        }
                    });
                    break;
            }
            return legalMoves;
        }
        
        function updatePlayerUI() {
             const currentTurnColor = isRedTurn ? 'Red' : 'Black';
             gameStatusEl.innerHTML = `${currentTurnColor}'s Turn. Flip a piece or move your own.`;
             
             if (player1Color) {
                 const p1DisplayColor = player1Color.charAt(0).toUpperCase() + player1Color.slice(1);
                 const p2DisplayColor = player1Color === 'red' ? 'Black' : 'Red';
                 p1ColorEl.textContent = `Color: ${p1DisplayColor}`;
                 p2ColorEl.textContent = `Color: ${p2DisplayColor}`;
             }

             const isP1Turn = player1Color ? (isRedTurn === (player1Color === 'red')) : false;
             p1InfoEl.classList.toggle('active', isP1Turn);
             p2InfoEl.classList.toggle('active', !isP1Turn);
        }

        function isRedPiece(piece) {
            return piece && piece.toUpperCase() === piece;
        }

        function isPieceOfCurrentTurn(piece) {
            if (!piece) return false;
            return isRedTurn === isRedPiece(piece);
        }

        resetButton.addEventListener('click', initializeBoard);
        initializeBoard();
    });
</script>
</body>
</html>
