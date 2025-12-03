
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Clock UI Apps</title>
<style>
  body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: #f0f0f0;
    text-align: center;
    padding: 20px;
  }
  h1 {
    color: #b8860b; /* golden */
  }
  #clockCanvas {
    background: white;
    border: 10px solid #b8860b; /* golden border */
    border-radius: 50%;
    margin: 20px auto;
    display: block;
  }
  #controls {
    margin-top: 20px;
  }
  label {
    display: inline-block;
    margin: 10px 5px 5px 0;
  }
  select, input[type=color], input[type=file], input[type=time] {
    margin-right: 20px;
  }
  #digitalTime {
    font-size: 1.5em;
    margin-top: 20px;
    font-weight: bold;
  }
  #timezoneSelect {
    width: 250px;
  }
  #alarmStatus {
    margin-top: 10px;
    font-weight: bold;
    color: red;
  }
  button {
    padding: 10px 15px;
    margin-top: 10px;
    font-size: 1em;
  }
  #secondTick {
    font-size: 2em;
    margin-top: 10px;
    color: #b8860b;
  }
</style>
</head>
<body>

<h1>Clock UI Apps</h1>

<canvas id="clockCanvas" width="400" height="400"></canvas>

<div id="controls">
  <div>
    <label for="hourColor">Hour Hand Color:</label>
    <input type="color" id="hourColor" value="#000000" />
    <label for="minuteColor">Minute Hand Color:</label>
    <input type="color" id="minuteColor" value="#000000" />
    <label for="secondColor">Second Hand Color:</label>
    <input type="color" id="secondColor" value="#b8860b" />
  </div>

  <div>
    <label for="timezoneSelect">Select Timezone:</label>
    <select id="timezoneSelect"></select>
  </div>

  <div>
    <label for="alarmTime">Set Alarm Time:</label>
    <input type="time" id="alarmTime" />
    <label for="alarmTune">Select Alarm Tune:</label>
    <input type="file" id="alarmTune" accept="audio/*" />
  </div>

  <div>
    <button id="setAlarmBtn">Set Alarm</button>
    <button id="stopAlarmBtn" disabled>Stop Alarm</button>
  </div>

  <div id="alarmStatus">No alarm set</div>

  <div id="digitalTime"></div>
  <div id="secondTick">Tick: 0</div>
</div>

<audio id="alarmAudio" loop></audio>

<script>
// Initialize canvas and context
const canvas = document.getElementById('clockCanvas');
const ctx = canvas.getContext('2d');
const radius = canvas.height / 2;
ctx.translate(radius, radius); // Move context to center of canvas

// Variables for colors
let hourColor = document.getElementById('hourColor').value;
let minuteColor = document.getElementById('minuteColor').value;
let secondColor = document.getElementById('secondColor').value;

// Timezone selector
const timezoneSelect = document.getElementById('timezoneSelect');

// Alarm variables
const alarmTimeInput = document.getElementById('alarmTime');
const alarmTuneInput = document.getElementById('alarmTune');
const setAlarmBtn = document.getElementById('setAlarmBtn');
const stopAlarmBtn = document.getElementById('stopAlarmBtn');
const alarmStatus = document.getElementById('alarmStatus');
const alarmAudio = document.getElementById('alarmAudio');

let alarmTime = null;
let alarmSet = false;

// Tick counter display
const secondTick = document.getElementById('secondTick');

// Digital clock display
const digitalTimeDisplay = document.getElementById('digitalTime');

function drawClock(time) {
  drawFace(ctx, radius);
  drawNumbers(ctx, radius);
  drawCuts(ctx, radius);
  drawTicks(ctx, radius);

  drawTime(ctx, radius, time);
}

function drawFace(ctx, radius) {
  ctx.beginPath();
  ctx.arc(0, 0, radius * 0.95, 0, 2 * Math.PI);
  ctx.fillStyle = 'white';
  ctx.fill();

  // Golden border
  ctx.lineWidth = radius * 0.1;
  ctx.strokeStyle = '#b8860b';
  ctx.stroke();

  // Center dot
  ctx.beginPath();
  ctx.arc(0, 0, radius * 0.05, 0, 2 * Math.PI);
  ctx.fillStyle = '#b8860b';
  ctx.fill();
}

function drawNumbers(ctx, radius) {
  ctx.font = radius * 0.15 + "px arial";
  ctx.textBaseline = "middle";
  ctx.textAlign = "center";
  ctx.fillStyle = '#b8860b';
  for(let num = 1; num <= 12; num++) {
    let ang = num * Math.PI / 6;
    let x = radius * 0.8 * Math.sin(ang);
    let y = -radius * 0.8 * Math.cos(ang);

    // Draw numbers inside golden border circle
    ctx.lineWidth = 3;
    ctx.strokeStyle = '#b8860b';
    ctx.fillStyle = 'white';

    // Draw golden circle around number
    ctx.beginPath();
    ctx.arc(x, y, radius * 0.07, 0, 2 * Math.PI);
    ctx.fill();
    ctx.stroke();

    // Draw number
    ctx.fillStyle = '#b8860b';
    ctx.fillText(num.toString(), x, y);
  }
}

function drawCuts(ctx, radius) {
  // Draw 3 cuts at 3 positions (e.g., 3, 6, 9)
  const cutPositions = [3, 6, 9];
  ctx.strokeStyle = '#b8860b';
  ctx.lineWidth = 6;
  cutPositions.forEach(pos => {
    let ang = pos * Math.PI / 6;
    let startX = radius * 0.6 * Math.sin(ang);
    let startY = -radius * 0.6 * Math.cos(ang);
    let endX = radius * 0.7 * Math.sin(ang);
    let endY = -radius * 0.7 * Math.cos(ang);

    ctx.beginPath();
    ctx.moveTo(startX, startY);
    ctx.lineTo(endX, endY);
    ctx.stroke();
  });
}

function drawTicks(ctx, radius) {
  // Draw ticks for every second (60 ticks)
  ctx.lineWidth = 2;
  for(let i=0; i < 60; i++) {
    let ang = i * Math.PI / 30;
    let startRadius = (i % 5 === 0) ? radius * 0.88 : radius * 0.92;
    let endRadius = radius * 0.95;

    let startX = startRadius * Math.sin(ang);
    let startY = -startRadius * Math.cos(ang);
    let endX = endRadius * Math.sin(ang);
    let endY = -endRadius * Math.cos(ang);

    ctx.beginPath();
    ctx.moveTo(startX, startY);
    ctx.lineTo(endX, endY);
    ctx.strokeStyle = (i % 5 === 0) ? '#b8860b' : '#ccc';
    ctx.stroke();
  }
}

function drawHand(ctx, pos, length, width, color) {
  ctx.beginPath();
  ctx.lineWidth = width;
  ctx.lineCap = "round";
  ctx.strokeStyle = color;
  ctx.moveTo(0,0);
  ctx.rotate(pos);
  ctx.lineTo(0, -length);
  ctx.stroke();
  ctx.rotate(-pos);
}

function drawTime(ctx, radius, time) {
  let hour = time.getHours();
  let minute = time.getMinutes();
  let second = time.getSeconds();

  // Calculate positions
  let hourPos = ((hour % 12) + minute / 60 + second / 3600) * Math.PI / 6;
  let minutePos = (minute + second / 60) * Math.PI / 30;
  let secondPos = second * Math.PI / 30;

  // Hour hand
  drawHand(ctx, hourPos, radius * 0.5, radius * 0.07, hourColor);
  // Minute hand
  drawHand(ctx, minutePos, radius * 0.75, radius * 0.05, minuteColor);
  // Second hand
  drawHand(ctx, secondPos, radius * 0.85, radius * 0.02, secondColor);
}

function updateClock() {
  // Get time in selected timezone
  let tz = timezoneSelect.value || Intl.DateTimeFormat().resolvedOptions().timeZone;
  let now = new Date();

  // Convert to timezone time using Intl API
  let timeStr = now.toLocaleString('en-US', {timeZone: tz});
  let tzTime = new Date(timeStr);

  ctx.clearRect(-radius, -radius, canvas.width, canvas.height);
  drawClock(tzTime);

  // Update digital clock
  digitalTimeDisplay.textContent = `Digital Time (${tz}): ${tzTime.toLocaleTimeString()}`;

  // Update second tick
  secondTick.textContent = `Tick: ${tzTime.getSeconds()}`;

  // Check alarm
  if (alarmSet && alarmTime) {
    let alarmHours = alarmTime.getHours();
    let alarmMinutes = alarmTime.getMinutes();
    if (tzTime.getHours() === alarmHours && tzTime.getMinutes() === alarmMinutes && tzTime.getSeconds() === 0) {
      alarmAudio.play();
      alarmStatus.textContent = 'Alarm ringing!';
      stopAlarmBtn.disabled = false;
      setAlarmBtn.disabled = true;
    }
  }
  requestAnimationFrame(updateClock);
}

// Populate timezone selector with all timezones
function populateTimezones() {
  // Use Intl API supported timezones
  const timezones = Intl.supportedValuesOf('timeZone');
  timezones.forEach(tz => {
    let option = document.createElement('option');
    option.value = tz;
    option.textContent = tz;
    timezoneSelect.appendChild(option);
  });

  // Default select user's timezone
  timezoneSelect.value = Intl.DateTimeFormat().resolvedOptions().timeZone;
}

// Event listeners for color pickers
document.getElementById('hourColor').addEventListener('input', (e) => {
  hourColor = e.target.value;
});
document.getElementById('minuteColor').addEventListener('input', (e) => {
  minuteColor = e.target.value;
});
document.getElementById('secondColor').addEventListener('input', (e) => {
  secondColor = e.target.value;
});

// Alarm set button
setAlarmBtn.addEventListener('click', () => {
  let timeVal = alarmTimeInput.value;
  if (!timeVal) {
    alert('Please set alarm time.');
    return;
  }
  alarmTime = new Date();
  let [h, m] = timeVal.split(':').map(Number);
  alarmTime.setHours(h, m, 0, 0);

  // Check if tune selected
  if (alarmTuneInput.files.length > 0) {
    let file = alarmTuneInput.files[0];
    let fileURL = URL.createObjectURL(file);
    alarmAudio.src = fileURL;
  } else {
    // Default beep tone if no tune selected
    alarmAudio.src = '';
    alert('Please select an alarm tune.');
    return;
  }

  alarmSet = true;
  alarmStatus.textContent = `Alarm set for ${timeVal}`;
  stopAlarmBtn.disabled = true;
  setAlarmBtn.disabled = false;
});

// Stop alarm button
stopAlarmBtn.addEventListener('click', () => {
  alarmAudio.pause();
  alarmAudio.currentTime = 0;
  alarmSet = false;
  alarmStatus.textContent = 'Alarm stopped';
  stopAlarmBtn.disabled = true;
  setAlarmBtn.disabled = false;
});

// Initialize
populateTimezones();
updateClock();
</script>

</body>
</html>
