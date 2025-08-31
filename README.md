# Day-2-Project
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Plot the Dot</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            background: #f9f9f9;
        }
        h1 {
            font-size: 3em;
            font-weight: bold;
            margin-top: 30px;
        }
        .instructions {
            font-size: 1.2em;
            margin: 20px 0 10px 0;
        }
        #coordinate {
            font-size: 1.5em;
            font-weight: bold;
            margin-bottom: 20px;
        }
        #plane-container {
            display: flex;
            justify-content: center;
            margin-bottom: 20px;
        }
        #cartesian-plane {
            background: #fff;
            border: 2px solid #333;
            box-shadow: 0 2px 8px rgba(0,0,0,0.1);
        }
        #check-btn, #reset-btn {
            font-size: 1em;
            padding: 10px 20px;
            margin: 10px 5px;
            border: none;
            border-radius: 5px;
            background: #007bff;
            color: #fff;
            cursor: pointer;
            transition: background 0.2s;
        }
        #check-btn:hover, #reset-btn:hover {
            background: #0056b3;
        }
        #score {
            margin-top: 30px;
            font-size: 1.1em;
            color: #222;
        }
        #result {
            font-size: 1.2em;
            margin: 10px 0;
            font-weight: bold;
        }
    </style>
</head>
<body>
    <h1>Plot the Dot</h1>
    <div class="instructions">
        Move the red dot to the following spot on the cartesian plane:
    </div>
    <div id="coordinate">(?, ?)</div>
    <div id="plane-container">
        <canvas id="cartesian-plane" width="500" height="500"></canvas>
    </div>
    <button id="check-btn">Check Answer</button>
    <button id="reset-btn" style="display:none;">Next</button>
    <div id="result"></div>
    <div id="score">
        Attempts: <span id="attempts">0</span> |
        Correct: <span id="correct">0</span> |
        Percentage: <span id="percentage">0%</span>
    </div>
    <script>
        // Constants
        const minCoord = -10, maxCoord = 10;
        const canvasSize = 500;
        const gridStep = canvasSize / (maxCoord - minCoord);
        const dotRadius = 10;
        const axisLineWidth = 3;
        const gridLineWidth = 1;
        const axisColor = '#222';
        const gridColor = '#bbb';
    const dotColor = 'red';
    const correctDotColor = 'blue';
        const labelFont = '12px Arial';
        const labelColor = '#222';
        
        // State
        let target = {x: 0, y: 0};
        let dot = {x: 0, y: 0};
        let dragging = false;
        let offset = {x: 0, y: 0};
    let attempts = 0;
    let correct = 0;
    let showCorrectDot = false;

        const canvas = document.getElementById('cartesian-plane');
        const ctx = canvas.getContext('2d');
        const coordinateDiv = document.getElementById('coordinate');
        const checkBtn = document.getElementById('check-btn');
        const resetBtn = document.getElementById('reset-btn');
        const resultDiv = document.getElementById('result');
        const attemptsSpan = document.getElementById('attempts');
        const correctSpan = document.getElementById('correct');
        const percentageSpan = document.getElementById('percentage');

        function randomCoord() {
            return Math.floor(Math.random() * (maxCoord - minCoord + 1)) + minCoord;
        }

        function newTarget() {
            target.x = randomCoord();
            target.y = randomCoord();
            coordinateDiv.textContent = `(${target.x}, ${target.y})`;
        }

        function coordToCanvas(x, y) {
            // (0,0) is center
            return {
                x: canvasSize/2 + x * gridStep,
                y: canvasSize/2 - y * gridStep
            };
        }

        function canvasToCoord(x, y) {
            // Convert canvas px to nearest integer coordinate
            let cx = Math.round((x - canvasSize/2) / gridStep);
            let cy = Math.round((canvasSize/2 - y) / gridStep);
            // Clamp to grid
            cx = Math.max(minCoord, Math.min(maxCoord, cx));
            cy = Math.max(minCoord, Math.min(maxCoord, cy));
            return {x: cx, y: cy};
        }

        function drawPlane() {
            ctx.clearRect(0, 0, canvasSize, canvasSize);
            ctx.save();
            // Draw grid lines
            ctx.strokeStyle = gridColor;
            ctx.lineWidth = gridLineWidth;
            for (let i = minCoord; i <= maxCoord; i++) {
                // Vertical
                ctx.beginPath();
                ctx.moveTo(coordToCanvas(i, minCoord).x, coordToCanvas(i, minCoord).y);
                ctx.lineTo(coordToCanvas(i, maxCoord).x, coordToCanvas(i, maxCoord).y);
                ctx.stroke();
                // Horizontal
                ctx.beginPath();
                ctx.moveTo(coordToCanvas(minCoord, i).x, coordToCanvas(minCoord, i).y);
                ctx.lineTo(coordToCanvas(maxCoord, i).x, coordToCanvas(maxCoord, i).y);
                ctx.stroke();
            }
            // Draw axes
            ctx.strokeStyle = axisColor;
            ctx.lineWidth = axisLineWidth;
            // y axis
            ctx.beginPath();
            ctx.moveTo(coordToCanvas(0, minCoord).x, coordToCanvas(0, minCoord).y);
            ctx.lineTo(coordToCanvas(0, maxCoord).x, coordToCanvas(0, maxCoord).y);
            ctx.stroke();
            // x axis
            ctx.beginPath();
            ctx.moveTo(coordToCanvas(minCoord, 0).x, coordToCanvas(minCoord, 0).y);
            ctx.lineTo(coordToCanvas(maxCoord, 0).x, coordToCanvas(maxCoord, 0).y);
            ctx.stroke();
            // Draw labels
            ctx.font = labelFont;
            ctx.fillStyle = labelColor;
            ctx.textAlign = 'center';
            ctx.textBaseline = 'top';
            for (let i = minCoord; i <= maxCoord; i++) {
                if (i !== 0) {
                    // x axis labels
                    let pos = coordToCanvas(i, 0);
                    ctx.fillText(i, pos.x, pos.y + 5);
                }
            }
            ctx.textAlign = 'right';
            ctx.textBaseline = 'middle';
            for (let i = minCoord; i <= maxCoord; i++) {
                if (i !== 0) {
                    // y axis labels
                    let pos = coordToCanvas(0, i);
                    ctx.fillText(i, pos.x - 5, pos.y);
                }
            }
            // Draw axis labels
            ctx.font = 'bold 16px Arial';
            ctx.fillStyle = axisColor;
            // X label
            ctx.textAlign = 'right';
            ctx.textBaseline = 'top';
            ctx.fillText('x', coordToCanvas(maxCoord, 0).x - 5, coordToCanvas(maxCoord, 0).y + 5);
            // Y label
            ctx.textAlign = 'left';
            ctx.textBaseline = 'top';
            ctx.fillText('y', coordToCanvas(0, maxCoord).x + 5, coordToCanvas(0, maxCoord).y + 5);
            ctx.restore();
        }

        function drawDot() {
            ctx.save();
            // Draw user's dot (red)
            ctx.beginPath();
            let pos = coordToCanvas(dot.x, dot.y);
            ctx.arc(pos.x, pos.y, dotRadius, 0, 2 * Math.PI);
            ctx.fillStyle = dotColor;
            ctx.fill();
            // Draw correct dot (blue) if needed
            if (showCorrectDot) {
                ctx.beginPath();
                let correctPos = coordToCanvas(target.x, target.y);
                ctx.arc(correctPos.x, correctPos.y, dotRadius, 0, 2 * Math.PI);
                ctx.fillStyle = correctDotColor;
                ctx.fill();
            }
            ctx.restore();
        }

        function redraw() {
            drawPlane();
            drawDot();
        }

        function resetDot() {
            dot.x = 0;
            dot.y = 0;
        }

        // Mouse events for dragging
        canvas.addEventListener('mousedown', function(e) {
            let rect = canvas.getBoundingClientRect();
            let mx = e.clientX - rect.left;
            let my = e.clientY - rect.top;
            let pos = coordToCanvas(dot.x, dot.y);
            let dist = Math.sqrt((mx - pos.x) ** 2 + (my - pos.y) ** 2);
            if (dist <= dotRadius + 2) {
                dragging = true;
                offset.x = mx - pos.x;
                offset.y = my - pos.y;
            }
        });
        canvas.addEventListener('mousemove', function(e) {
            if (dragging) {
                let rect = canvas.getBoundingClientRect();
                let mx = e.clientX - rect.left - offset.x;
                let my = e.clientY - rect.top - offset.y;
                let coord = canvasToCoord(mx, my);
                dot.x = coord.x;
                dot.y = coord.y;
                redraw();
            }
        });
        canvas.addEventListener('mouseup', function(e) {
            dragging = false;
        });
        canvas.addEventListener('mouseleave', function(e) {
            dragging = false;
        });

        checkBtn.addEventListener('click', function() {
            attempts++;
            let isCorrect = (dot.x === target.x && dot.y === target.y);
            if (isCorrect) {
                correct++;
                resultDiv.textContent = 'Correct!';
                resultDiv.style.color = 'green';
                showCorrectDot = false;
            } else {
                resultDiv.textContent = `Incorrect. The correct spot was (${target.x}, ${target.y}).`;
                resultDiv.style.color = 'red';
                showCorrectDot = true;
            }
            updateScore();
            checkBtn.style.display = 'none';
            resetBtn.style.display = 'inline-block';
            redraw();
        });

        resetBtn.addEventListener('click', function() {
            resetDot();
            newTarget();
            showCorrectDot = false;
            redraw();
            resultDiv.textContent = '';
            checkBtn.style.display = 'inline-block';
            resetBtn.style.display = 'none';
        });

        function updateScore() {
            attemptsSpan.textContent = attempts;
            correctSpan.textContent = correct;
            let percent = attempts === 0 ? 0 : Math.round((correct / attempts) * 100);
            percentageSpan.textContent = percent + '%';
        }

        // Initialize
        function init() {
            resetDot();
            newTarget();
            redraw();
            updateScore();
        }
        init();
    </script>
</body>
</html>
