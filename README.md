# P2LSM-Leitner-Quiz-System
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Leitner System Quiz</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #f0f8ff;
      margin: 0; padding: 20px;
      color: #333;
      text-align: center;
    }
    h1 { margin-bottom: 30px; }
    .hidden { display: none; }
    button {
      padding: 10px 18px;
      font-size: 16px;
      border-radius: 8px;
      border: none;
      background: #007bff;
      color: white;
      cursor: pointer;
      margin: 8px;
      transition: background 0.3s;
    }
    button:hover { background: #0056b3; }
    #modeSelect button { width: 150px; }
    input[type="text"], input[type="password"] {
      padding: 8px 12px;
      font-size: 16px;
      border-radius: 6px;
      border: 1px solid #aaa;
      width: 200px;
    }
    #quiz {
      display: flex;
      justify-content: center;
      gap: 20px;
      flex-wrap: wrap;
      margin-top: 25px;
    }
    .box {
      background: white;
      border-radius: 12px;
      box-shadow: 0 0 12px rgba(0,0,0,0.1);
      width: 320px;
      padding: 15px;
      text-align: left;
    }
    .box-title {
      font-weight: bold;
      font-size: 20px;
      margin-bottom: 15px;
      border-bottom: 2px solid #ccc;
      padding-bottom: 5px;
      text-align: center;
    }
    .box:nth-child(1) { background: #fff3cd; }
    .box:nth-child(2) { background: #d1ecf1; }
    .box:nth-child(3) { background: #d4edda; }
    .question { margin-bottom: 15px; }
    .question strong { font-size: 18px; }
    .question input[type="text"] { width: 70%; font-size: 16px; }
    .disabled-msg { color: red; font-weight: 600; margin-top: 6px; }
    #welcome { font-size: 22px; font-weight: 600; margin-bottom: 10px; color: #006400; }
    #studentControls, #teacherControls { margin-top: 15px; }
    #report table {
      border-collapse: collapse;
      width: 90%;
      margin: 0 auto;
      margin-top: 20px;
    }
    #report th, #report td {
      border: 1px solid #999;
      padding: 8px 12px;
      text-align: center;
    }
    #report th { background-color: #007bff; color: white; }
    .back-btn { background: #6c757d !important; }
  </style>
</head>
<body>

<h1>🧠 Leitner System Quiz</h1>

<div id="modeSelect">
  <button onclick="showStudentLogin()">I'm a Student</button>
  <button onclick="showTeacherLogin()">I'm a Teacher</button>
</div>

<div id="studentLogin" class="hidden">
  <input type="text" id="studentNameInput" placeholder="Enter your name" autocomplete="off" />
  <button onclick="startQuiz()">Start Quiz</button>
  <br />
  <button id="backBtnStudentLogin" class="back-btn" onclick="goBack()">← Back</button>
</div>

<div id="teacherLogin" class="hidden">
  <input type="password" id="teacherPass" placeholder="Enter teacher password" autocomplete="off" />
  <button onclick="viewAllStudentNames()">View All Students</button>
  <br />
  <button id="backBtnTeacherLogin" class="back-btn" onclick="goBack()">← Back</button>
</div>

<div id="welcome" class="hidden"></div>
<div id="quiz" class="hidden"></div>
<div id="studentControls" class="hidden">
  <button id="backBtnStudent" class="back-btn" onclick="backToLogin()">← Back</button>
</div>

<div id="report" class="hidden">
  <h2>Student Report: <span id="reportStudentName"></span></h2>
  <table id="reportTable">
    <thead>
      <tr>
        <th>Question</th>
        <th>Answer</th>
        <th>Box</th>
        <th>Disabled Today?</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>
  <button id="backBtnTeacher" class="back-btn" onclick="goBack()">← Back</button>
</div>

<script>
const today = new Date();
const day = today.getDay();
const teacherPassword = "teach123";

const baseQuestions = [
  { id: 1, q: "2 × 3", a: "6", box: 1 },
  { id: 2, q: "3 × 4", a: "12", box: 1 },
  { id: 3, q: "4 × 5", a: "20", box: 1 },
  { id: 4, q: "5 × 6", a: "30", box: 1 },
  { id: 5, q: "10 × 7", a: "70", box: 1 },
  { id: 6, q: "2 × 8", a: "16", box: 1 },
  { id: 7, q: "3 × 9", a: "27", box: 1 },
  { id: 8, q: "4 × 2", a: "8", box: 1 },
  { id: 9, q: "5 × 3", a: "15", box: 1 },
  { id: 10, q: "10 × 10", a: "100", box: 1 }
];

let studentName = "";
let attempts = {};
let quizState = [];

/* --- Google Sheets Integration --- */
const SHEET_URL = "https://script.google.com/macros/s/AKfycbxMzguJFIBYLkgvo0VempShEz0ed3duVFsHsxaOQiMtRU_0JhAiYW7aBeRS5OvpAflwyw/exec"; // replace with your deployed Apps Script URL

async function saveToSheet(studentData) {
  console.log("Saving to Sheet:", studentData);
  try {
    const res = await fetch(SHEET_URL, {
      method: "POST",
      body: JSON.stringify(studentData),
      headers: { "Content-Type": "application/json" }
    });
    const result = await res.json();
    console.log("Sheet response:", result);
  } catch (err) {
    console.error("Sheet save error:", err);
  }
}

async function loadFromSheet() {
  try {
    const res = await fetch(SHEET_URL);
    const data = await res.json();
    console.log("Loaded from Sheet:", data);
    return data;
  } catch (err) {
    console.error("Sheet load error:", err);
    return {};
  }
}
/* --- End Sheets Integration --- */

function hideAllSections() {
  ["modeSelect","studentLogin","teacherLogin","welcome","quiz","studentControls","report"].forEach(id=>{
    document.getElementById(id).classList.add("hidden");
  });
}

function clearInputs() {
  document.getElementById("studentNameInput").value="";
  document.getElementById("teacherPass").value="";
  document.getElementById("welcome").textContent="";
}

function hideAllBackButtons() {
  ["backBtnStudentLogin","backBtnTeacherLogin","backBtnStudent","backBtnTeacher"].forEach(id=>{
    document.getElementById(id).style.display="none";
  });
}

function showStudentLogin() {
  hideAllSections(); clearInputs(); hideAllBackButtons();
  document.getElementById("studentLogin").classList.remove("hidden");
  document.getElementById("backBtnStudentLogin").style.display="inline-block";
}

function showTeacherLogin() {
  hideAllSections(); clearInputs(); hideAllBackButtons();
  document.getElementById("teacherLogin").classList.remove("hidden");
  document.getElementById("backBtnTeacherLogin").style.display="inline-block";
}

function goBack() {
  hideAllSections(); clearInputs(); hideAllBackButtons();
  document.getElementById("modeSelect").classList.remove("hidden");
}

function backToLogin() {
  saveProgress();
  document.getElementById("quiz").classList.add("hidden");
  document.getElementById("studentControls").classList.add("hidden");
  document.getElementById("welcome").classList.add("hidden");
  goBack();
}

function saveProgress() {
  const studentData = {
    name: studentName,
    quizState,
    attempts,
    expiry: new Date(Date.now() + 7*24*60*60*1000).toISOString()
  };

  // Save locally too
  const allStudents = JSON.parse(localStorage.getItem("leitner_students") || "{}");
  allStudents[studentName] = studentData;
  localStorage.setItem("leitner_students", JSON.stringify(allStudents));

  // Send to Google Sheet
  fetch(SHEET_URL, {
    method: "POST",
    body: JSON.stringify(studentData),
    headers: { "Content-Type": "application/json" }
  })
    .then(res => res.json())
    .then(d => console.log("Sheet response:", d))
    .catch(err => console.error("Error saving to sheet:", err));
}


async function loadProgress(name){
  const allStudents = await loadFromSheet();
  const studentData = allStudents[name];
  if(studentData && new Date()<=new Date(studentData.expiry)) return studentData;

  const localStudents = JSON.parse(localStorage.getItem("leitner_students")||"{}");
  const localData = localStudents[name];
  if(localData && new Date()<=new Date(localData.expiry)) return localData;

  return null;
}

async function startQuiz() {
  studentName = document.getElementById("studentNameInput").value.trim();
  if(!studentName){alert("Please enter your name."); return;}
  const saved = await loadProgress(studentName);
  if(saved){ quizState = saved.quizState; attempts=saved.attempts||{}; }
  else{ quizState = JSON.parse(JSON.stringify(baseQuestions)); attempts={}; }
  await saveProgress();
  hideAllSections();
  document.getElementById("welcome").classList.remove("hidden");
  document.getElementById("welcome").textContent = `Welcome, ${studentName}!`;
  document.getElementById("quiz").classList.remove("hidden");
  document.getElementById("studentControls").classList.remove("hidden");
  renderQuiz();
}

function renderQuiz(){
  const quizDiv=document.getElementById("quiz"); quizDiv.innerHTML="";
  const boxLabels={1:"📅 Everyday",2:"📆 Tuesday / Thursday",3:"🎉 Friday"};
  for(let boxNum=1;boxNum<=3;boxNum++){
    const boxDiv=document.createElement("div"); boxDiv.className="box";
    const boxTitle=document.createElement("div"); boxTitle.className="box-title"; boxTitle.textContent=boxLabels[boxNum]; boxDiv.appendChild(boxTitle);
    quizState.filter(q=>q.box===boxNum).forEach(q=>{
      const questionDiv=document.createElement("div"); questionDiv.className="question";
      if(boxNum===1){
        const isDisabled=attempts[q.id]&&attempts[q.id].disabled;
        if(isDisabled){const lastDay=attempts[q.id].day; if(lastDay!==new Date().getDay()){ delete attempts[q.id]; }}
        if(attempts[q.id]&&attempts[q.id].disabled){ questionDiv.innerHTML=`<strong>${q.q}</strong><br><div class="disabled-msg">Try again tomorrow</div>`; }
        else{ questionDiv.innerHTML=`<strong>${q.q}</strong><br><div style="display:flex;align-items:center;gap:8px;margin-top:5px;"><input type="text" id="input-${q.id}" autocomplete="off"/><button onclick="checkAnswer(${q.id})">Submit</button></div>`; }
      } else if(boxNum===2){
        if(day===2||day===4){ questionDiv.innerHTML=`<strong>${q.q}</strong><br><div style="display:flex;align-items:center;gap:8px;margin-top:5px;"><input type="text" id="input-${q.id}" autocomplete="off"/><button onclick="checkAnswer(${q.id})">Submit</button></div>`; }
        else{ questionDiv.innerHTML=`<strong>${q.q}</strong><br><div class="disabled-msg">Only available Tue/Thu</div>`; }
      } else if(boxNum===3){ questionDiv.innerHTML=`✅ <strong>${q.q}</strong>`; }
      boxDiv.appendChild(questionDiv);
    });
    quizDiv.appendChild(boxDiv);
  }
}

async function checkAnswer(id){
  const input=document.getElementById(`input-${id}`); if(!input)return;
  const answer=input.value.trim(); if(answer==="") return alert("Please enter an answer.");
  const q=quizState.find(q=>q.id===id);
  const today=new Date().getDay();
  if(answer===q.a){ if(q.box===1)q.box=2; else if(q.box===2)q.box=3; delete attempts[q.id]; }
  else{ attempts[q.id]={disabled:true, day:today}; q.box=1; }
  await saveProgress(); renderQuiz();
}

async function viewAllStudentNames(){
  const pass=document.getElementById("teacherPass").value.trim();
  if(pass!==teacherPassword){alert("Wrong password!"); return;}
  hideAllSections(); document.getElementById("report").classList.remove("hidden");
  const allStudents=await loadFromSheet();
  console.log("Teacher view all students:", allStudents);
  const tbody=document.querySelector("#reportTable tbody"); tbody.innerHTML="";
  if(Object.keys(allStudents).length===0){ tbody.innerHTML=`<tr><td colspan="4">No students have logged in yet.</td></tr>`; return; }
  for(const name in allStudents){
    const s=allStudents[name]; const tr=document.createElement("tr");
    tr.innerHTML=`<td>${name}</td><td>${s.quizState.length} questions</td><td>${s.quizState.map(q=>q.box).join(",")}</td><td>${JSON.stringify(s.attempts)}</td>`;
    tbody.appendChild(tr);
  }
}
</script>
</body>
</html>
