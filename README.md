# Calccam
Smart calculator with camera-based object measurement
cat > index.html <<'EOF'
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>CALCCAM</title>

<style>
* {
    box-sizing: border-box;
    font-family: Arial, sans-serif;
}

body {
    margin: 0;
    background: #101114;
    color: white;
}

header {
    padding: 18px;
    text-align: center;
    font-size: 26px;
    font-weight: bold;
}

.screen {
    max-width: 500px;
    margin: auto;
    padding: 15px;
}

.display {
    background: #1d1f24;
    border-radius: 18px;
    padding: 25px;
    text-align: right;
    font-size: 38px;
    min-height: 90px;
    margin-bottom: 15px;
    overflow: hidden;
}

.buttons {
    display: grid;
    grid-template-columns: repeat(4, 1fr);
    gap: 10px;
}

button {
    border: none;
    border-radius: 15px;
    padding: 20px 5px;
    font-size: 20px;
    background: #292c33;
    color: white;
}

button:active {
    transform: scale(.96);
}

.operator {
    background: #4d55d8;
}

.special {
    background: #3a3d45;
}

.camera-btn {
    width: 100%;
    margin-top: 15px;
    background: #16a085;
    font-size: 20px;
}

#cameraScreen {
    display: none;
}

.camera-container {
    position: relative;
    width: 100%;
    overflow: hidden;
    border-radius: 18px;
    background: black;
}

video {
    width: 100%;
    display: block;
}

#selection {
    position: absolute;
    border: 3px solid #00ff88;
    display: none;
    pointer-events: none;
}

.measure-result {
    background: #1d1f24;
    padding: 18px;
    margin-top: 15px;
    border-radius: 15px;
    text-align: center;
}

.mode-buttons {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    margin: 15px 0;
}

.back {
    background: #333;
}

input {
    width: 100%;
    padding: 14px;
    margin-top: 8px;
    border-radius: 10px;
    border: none;
    font-size: 17px;
}

.hidden {
    display: none;
}
</style>
</head>

<body>

<header>CALCCAM</header>

<div class="screen" id="calculatorScreen">

    <div class="display" id="display">0</div>

    <div class="buttons">
        <button class="special" onclick="clearDisplay()">AC</button>
        <button class="special" onclick="deleteLast()">⌫</button>
        <button class="special" onclick="percentage()">%</button>
        <button class="operator" onclick="add('/')">÷</button>

        <button onclick="add('7')">7</button>
        <button onclick="add('8')">8</button>
        <button onclick="add('9')">9</button>
        <button class="operator" onclick="add('*')">×</button>

        <button onclick="add('4')">4</button>
        <button onclick="add('5')">5</button>
        <button onclick="add('6')">6</button>
        <button class="operator" onclick="add('-')">−</button>

        <button onclick="add('1')">1</button>
        <button onclick="add('2')">2</button>
        <button onclick="add('3')">3</button>
        <button class="operator" onclick="add('+')">+</button>

        <button onclick="add('0')">0</button>
        <button onclick="add('.')">.</button>
        <button class="operator" style="grid-column: span 2"
                onclick="calculate()">=</button>
    </div>

    <button class="camera-btn" onclick="openCamera()">
        📷 Camera Measurement
    </button>

</div>


<div class="screen" id="cameraScreen">

    <button class="back" onclick="closeCamera()">← Back to Calculator</button>

    <div class="mode-buttons">
        <button onclick="setMode('single')">📐 Select Object</button>
        <button onclick="setMode('multi')">📏 Multi Measure</button>
    </div>

    <label>Reference length (cm)</label>
    <input id="referenceCm"
           type="number"
           value="10"
           min="0.1"
           step="0.1">

    <p>
        Place a known-size reference near the object.
        Then drag on the camera view to select the object.
    </p>

    <div class="camera-container" id="cameraContainer">

        <video id="video" autoplay playsinline></video>

        <div id="selection"></div>

    </div>

    <div class="measure-result" id="result">
        Select an object to measure it.
    </div>

    <button class="camera-btn" onclick="startSelection()">
        🎯 Select / Measure
    </button>

</div>


<script>

let expression = "";
let stream = null;
let mode = "single";

const display = document.getElementById("display");
const video = document.getElementById("video");
const selection = document.getElementById("selection");
const result = document.getElementById("result");

function add(value) {
    expression += value;
    display.textContent = expression;
}

function clearDisplay() {
    expression = "";
    display.textContent = "0";
}

function deleteLast() {
    expression = expression.slice(0, -1);
    display.textContent = expression || "0";
}

function calculate() {
    try {
        if (!expression) return;

        // Basic calculator only.
        if (!/^[0-9+\-*/().% ]+$/.test(expression)) {
            throw new Error();
        }

        let answer = Function('"use strict"; return (' + expression + ')')();

        if (!Number.isFinite(answer)) throw new Error();

        expression = String(answer);
        display.textContent = expression;

    } catch {
        display.textContent = "Error";
        expression = "";
    }
}

function percentage() {
    try {
        if (!expression) return;

        let answer = Function(
            '"use strict"; return (' + expression + ')'
        )();

        answer = answer / 100;

        expression = String(answer);
        display.textContent = expression;

    } catch {
        display.textContent = "Error";
        expression = "";
    }
}


async function openCamera() {

    document.getElementById("calculatorScreen").style.display = "none";
    document.getElementById("cameraScreen").style.display = "block";

    try {

        stream = await navigator.mediaDevices.getUserMedia({
            video: {
                facingMode: { ideal: "environment" }
            },
            audio: false
        });

        video.srcObject = stream;

    } catch (error) {

        result.innerHTML =
            "❌ Camera permission was denied or camera is unavailable.";

        console.error(error);
    }
}


function closeCamera() {

    if (stream) {
        stream.getTracks().forEach(track => track.stop());
        stream = null;
    }

    document.getElementById("cameraScreen").style.display = "none";
    document.getElementById("calculatorScreen").style.display = "block";
}


function setMode(newMode) {

    mode = newMode;

    if (mode === "single") {

        result.innerHTML =
            "Single mode: select one object.";

    } else {

        result.innerHTML =
            "Multi mode: select multiple objects one after another.";
    }
}


function startSelection() {

    result.innerHTML =
        "Drag across an object in the camera view.";

    selection.style.display = "none";

    let startX = 0;
    let startY = 0;
    let dragging = false;

    const container = document.getElementById("cameraContainer");

    function pointerDown(e) {

        dragging = true;

        const rect = container.getBoundingClientRect();

        startX = e.clientX - rect.left;
        startY = e.clientY - rect.top;

        selection.style.left = startX + "px";
        selection.style.top = startY + "px";
        selection.style.width = "0px";
        selection.style.height = "0px";
        selection.style.display = "block";
    }

    function pointerMove(e) {

        if (!dragging) return;

        const rect = container.getBoundingClientRect();

        const currentX = e.clientX - rect.left;
        const currentY = e.clientY - rect.top;

        const width = Math.abs(currentX - startX);
        const height = Math.abs(currentY - startY);

        selection.style.left =
            Math.min(startX, currentX) + "px";

        selection.style.top =
            Math.min(startY, currentY) + "px";

        selection.style.width = width + "px";
        selection.style.height = height + "px";
    }

    function pointerUp() {

        if (!dragging) return;

        dragging = false;

        const widthPx = selection.offsetWidth;
        const heightPx = selection.offsetHeight;

        if (widthPx < 10 || heightPx < 10) {
            result.innerHTML = "Selection too small.";
            return;
        }

        const referenceCm =
            parseFloat(document.getElementById("referenceCm").value);

        if (!referenceCm || referenceCm <= 0) {
            result.innerHTML =
                "Enter a valid reference length.";
            return;
        }

        /*
         * Prototype calculation:
         * referenceCm represents the physical length
         * represented by the selected width.
         *
         * A future version will replace this with
         * AR/depth-based real-world measurement.
         */

        const length = referenceCm;
        const ratio = heightPx / widthPx;
        const breadth = length * ratio;

        result.innerHTML = `
            <strong>Estimated Measurement</strong><br><br>
            Length: ${length.toFixed(1)} cm<br>
            Breadth: ${breadth.toFixed(1)} cm
        `;

        if (mode === "multi") {
            result.innerHTML +=
                "<br><br>➕ Select another object.";
        }
    }

    container.onpointerdown = pointerDown;
    container.onpointermove = pointerMove;
    container.onpointerup = pointerUp;
}

</script>

</body>
</html>
EOF
