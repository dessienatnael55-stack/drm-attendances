
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>DRM Attendance</title>
<style>
* { box-sizing: border-box; }
body {
  margin: 0; padding: 16px;
  font-family: Arial, sans-serif;
  background: #f1f5f9; color: #172554;
}
header {
  background: #124b8b; color: white;
  padding: 22px 16px; border-radius: 16px;
  text-align: center;
}
header h2 { margin: 0 0 8px; font-size: 21px; }
header p { margin: 0; font-size: 13px; }
.card {
  background: white; border-radius: 14px;
  padding: 14px; margin-top: 15px;
  box-shadow: 0 2px 8px #0000000d;
}
label { display: block; margin-bottom: 7px; font-weight: bold; }
input, select, button {
  font: inherit; padding: 11px;
  border-radius: 8px; border: 1px solid #cbd5e1;
}
input { width: 100%; }
.stats { display: flex; gap: 8px; }
.stat {
  flex: 1; text-align: center; padding: 12px 4px;
  background: #eaf3ff; border-radius: 10px;
}
.stat strong { display: block; font-size: 23px; }
.stat span { font-size: 12px; }
.student {
  display: flex; align-items: center;
  justify-content: space-between; gap: 8px;
  padding: 12px 0; border-bottom: 1px solid #e2e8f0;
}
.student .name { flex: 1; font-size: 14px; }
.student select { width: 125px; padding: 9px 4px; }
button {
  cursor: pointer; border: none;
  background: #124b8b; color: white; font-weight: bold;
}
.actions { display: grid; grid-template-columns: 1fr 1fr; gap: 8px; }
.actions button { min-height: 44px; }
.secondary { background: #e2e8f0; color: #172554; }
#message { color: #087443; font-size: 13px; }
small { color: #64748b; }
</style>
</head>
<body>

<header>
  <h2>DRM Attendance</h2>
  <p>Disaster Risk Management and Sustainable Development</p>
</header>

<section class="card">
  <label for="date">Attendance Date</label>
  <input type="date" id="date">
  <p><small>Choose the class date, then mark each student's attendance.</small></p>
</section>

<section class="card">
  <div class="stats">
    <div class="stat"><strong id="present">0</strong><span>Present</span></div>
    <div class="stat"><strong id="absent">0</strong><span>Absent</span></div>
    <div class="stat"><strong id="late">0</strong><span>Late</span></div>
    <div class="stat"><strong id="unmarked">0</strong><span>Unmarked</span></div>
  </div>
</section>

<section class="card">
  <label for="newName">Add a student</label>
  <div style="display:flex;gap:8px">
    <input id="newName" placeholder="Enter student name">
    <button onclick="addStudent()">Add</button>
  </div>
</section>

<section class="card">
  <label>Student Attendance</label>
  <div id="students"></div>
  <p id="message" aria-live="polite"></p>
  <div class="actions">
    <button onclick="saveAttendance()">Save Attendance</button>
    <button onclick="exportCSV()">Export CSV</button>
    <button class="secondary" onclick="resetDay()">Clear This Date</button>
    <button class="secondary" onclick="exportRoster()">Export Student List</button>
  </div>
</section>

<script>
const starterNames = [
  "Yordan Yigzaw Nigatu",
  "Kenean Engida Tedla",
  "Esuyawt Demelie Belay",
  "Enku Yitbarek Kassaye",
  "Aynalen Fentaye Ngusie",
  "Mulat Adugna Andargi",
  "Amsalu Damtie Adafrie",
  "Hana Fekad Mekonni",
  "Seble Alamrew Muluneh",
  "Yikedes Demelas Abera",
  "Henok Ephraim Yemata",
  "Meskere Melese Yitayew",
  "Nahusen Marye Abebaw",
  "Abdu Seid Shikuru",
  "Eliyas Eyayu Meshesh",
  "Samual Ambelu Tegegne",
  "Ayana Kassie Amare",
  "Tadesse Alemu Tiruneh",
  "Kidist Yohanni Alehegn"
];

let names = JSON.parse(localStorage.getItem("drmNames") || "null") || starterNames;
let records = JSON.parse(localStorage.getItem("drmRecords") || "{}");

const dateInput = document.getElementById("date");
const today = new Date();
dateInput.value = [
  today.getFullYear(),
  String(today.getMonth()+1).padStart(2,"0"),
  String(today.getDate()).padStart(2,"0")
].join("-");

function currentRecord() {
  if (!records[dateInput.value]) records[dateInput.value] = {};
  return records[dateInput.value];
}

function render() {
  const list = document.getElementById("students");
  list.innerHTML = "";
  const record = currentRecord();

  names.forEach((name, index) => {
    const row = document.createElement("div");
    row.className = "student";

    const label = document.createElement("span");
    label.className = "name";
    label.textContent = (index + 1) + ". " + name;

    const select = document.createElement("select");
    select.setAttribute("aria-label", "Attendance for " + name);

    [
      ["", "Not marked"],
      ["Present", "Present"],
      ["Absent", "Absent"],
      ["Late", "Late"]
    ].forEach(([value, text]) => {
      const option = document.createElement("option");
      option.value = value;
      option.textContent = text;
      select.appendChild(option);
    });

    select.value = record[name] || "";
    select.onchange = () => {
      record[name] = select.value;
      saveQuietly();
      updateStats();
    };

    row.append(label, select);
    list.appendChild(row);
  });

  updateStats();
}

function updateStats() {
  const record = currentRecord();
  const values = names.map(name => record[name] || "");
  document.getElementById("present").textContent =
    values.filter(v => v === "Present").length;
  document.getElementById("absent").textContent =
    values.filter(v => v === "Absent").length;
  document.getElementById("late").textContent =
    values.filter(v => v === "Late").length;
  document.getElementById("unmarked").textContent =
    values.filter(v => !v).length;
}

function saveQuietly() {
  localStorage.setItem("drmRecords", JSON.stringify(records));
  localStorage.setItem("drmNames", JSON.stringify(names));
}

function saveAttendance() {
  saveQuietly();
  document.getElementById("message").textContent =
    "Attendance saved on this device for " + dateInput.value + ".";
}

function addStudent() {
  const input = document.getElementById("newName");
  const name = input.value.trim();
  if (!name) {
    alert("Please enter a student name.");
    return;
  }
  if (names.some(n => n.toLowerCase() === name.toLowerCase())) {
    alert("This student is already on the list.");
    return;
  }
  names.push(name);
  input.value = "";
  saveQuietly();
  render();
}

function resetDay() {
  if (!confirm("Clear attendance for " + dateInput.value + "?")) return;
  delete records[dateInput.value];
  saveQuietly();
  render();
  document.getElementById("message").textContent = "This date was cleared.";
}

function downloadCSV(filename, rows) {
  const csv = rows.map(row => row.map(value =>
    '"' + String(value ?? "").replace(/"/g, '""') + '"'
  ).join(",")).join("\r\n");
  const blob = new Blob(["\uFEFF" + csv], {
    type: "text/csv;charset=utf-8;"
  });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = filename;
  a.click();
  setTimeout(() => URL.revokeObjectURL(url), 1000);
}

function exportCSV() {
  const record = currentRecord();
  const rows = [["No.", "Student Name", "Date", "Attendance"]];
  names.forEach((name, i) =>
    rows.push([i+1, name, dateInput.value, record[name] || "Not marked"])
  );
  downloadCSV("DRM_Attendance_" + dateInput.value + ".csv", rows);
}

function exportRoster() {
  downloadCSV("DRM_Student_List.csv", [
    ["No.", "Student Name"],
    ...names.map((name, i) => [i+1, name])
  ]);
}

dateInput.addEventListener("change", () => {
  document.getElementById("message").textContent = "";
  render();
});

render();
</script>
</body>
</html>
