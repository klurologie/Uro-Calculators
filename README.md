# Stein-Rechner


<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Stone Volume Calculator</title>
<style>
body {
  font-family: Arial, sans-serif;
  background: #f3f6f8;
  color: #17212b;
  max-width: 650px;
  margin: 30px auto;
  padding: 16px;
}
h1 { font-size: 27px; }
.card {
  background: white;
  padding: 22px;
  border-radius: 16px;
  box-shadow: 0 3px 14px #0001;
}
label { display: block; font-weight: bold; margin-top: 18px; }
input[type=number] {
  width: 85px;
  padding: 9px;
  font-size: 16px;
  border: 1px solid #bbc5cc;
  border-radius: 7px;
}
input[type=range] { width: 65%; accent-color: #087f70; }
.dimension {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
}
select, button {
  font-size: 16px;
  padding: 11px;
  border-radius: 8px;
  border: 1px solid #ccd5db;
}
button {
  background: #087f70;
  color: white;
  border: 0;
  cursor: pointer;
  margin: 14px 6px 0 0;
}
.result {
  margin-top: 24px;
  padding: 18px;
  background: #e6f5f1;
  border-radius: 12px;
}
#volume { font-size: 36px; font-weight: bold; }
small { color: #53616c; }
</style>
</head>
<body>
<h1>Stone Volume Calculator</h1>
<p>Enter dimensions in millimetres (mm).</p>

<div class="card">
  <label for="shape">Stone geometry</label>
  <select id="shape" onchange="calculate()">
    <option value="sphere">Sphere (mean diameter)</option>
    <option value="ellipsoid">Ellipsoid (three axes)</option>
  </select>

  <div id="dimensions"></div>

  <div class="result">
    <small>Estimated volume</small>
    <div id="volume">0.524 cm³</div>
    <small id="formula"></small>
  </div>

  <button onclick="resetValues()">Reset</button>
  <button onclick="copyResult()">Copy result</button>
  <p id="message" aria-live="polite"></p>
</div>

<p><small>
This calculator estimates geometric volume. Validate the formulas
and clinical suitability before clinical or research use.
No measurements are saved or transmitted.
</small></p>

<script>
const names = [
  ["height", "Height"],
  ["depth", "Depth"],
  ["width", "Width"]
];

const dimensions = document.getElementById("dimensions");

names.forEach(([id, name]) => {
  const label = document.createElement("label");
  label.htmlFor = id;
  label.textContent = name + " (mm)";

  const row = document.createElement("div");
  row.className = "dimension";

  const slider = document.createElement("input");
  slider.type = "range";
  slider.min = "1";
  slider.max = "100";
  slider.step = "0.1";
  slider.value = "10";
  slider.id = id + "Slider";
  slider.setAttribute("aria-label", name + " slider");

  const number = document.createElement("input");
  number.type = "number";
  number.id = id;
  number.min = "0.1";
  number.max = "1000";
  number.step = "0.1";
  number.value = "10";
  number.inputMode = "decimal";

  slider.addEventListener("input", () => {
    number.value = slider.value;
    calculate();
  });

  number.addEventListener("input", () => {
    const value = Number(number.value);
    if (number.value !== "" && Number.isFinite(value) && value > 0) {
      slider.value = Math.min(100, Math.max(1, value));
    }
    calculate();
  });

  row.append(slider, number);
  dimensions.append(label, row);
});

function calculate() {
  const vals = names.map(([id]) =>
    Number(document.getElementById(id).value)
  );

  const output = document.getElementById("volume");
  const formula = document.getElementById("formula");

  if (vals.some(v => !Number.isFinite(v) || v <= 0 || v > 1000)) {
    output.textContent = "Enter valid dimensions";
    formula.textContent = "";
    return;
  }

  let mm3;

  if (document.getElementById("shape").value === "sphere") {
    const d = (vals[0] + vals[1] + vals[2]) / 3;
    mm3 = Math.PI / 6 * d ** 3;
    formula.textContent =
      "V = π/6 × mean diameter³; mean diameter = " +
      d.toFixed(2) + " mm";
  } else {
    mm3 = Math.PI / 6 * vals[0] * vals[1] * vals[2];
    formula.textContent = "V = π/6 × height × depth × width";
  }

  output.textContent = (mm3 / 1000).toFixed(3) + " cm³";
}

function resetValues() {
  names.forEach(([id]) => {
    document.getElementById(id).value = "10";
    document.getElementById(id + "Slider").value = "10";
  });
  document.getElementById("shape").value = "sphere";
  document.getElementById("message").textContent = "";
  calculate();
}

async function copyResult() {
  const vals = names.map(([id]) =>
    document.getElementById(id).value
  );
  const text = "Stone volume: " +
    document.getElementById("volume").textContent +
    "; geometry: " + document.getElementById("shape").value +
    "; height/depth/width (mm): " + vals.join(" / ");

  try {
    await navigator.clipboard.writeText(text);
    document.getElementById("message").textContent = "Result copied.";
  } catch {
    window.prompt("Copy your result:", text);
  }
}

calculate();
</script>
</body>
</html>

