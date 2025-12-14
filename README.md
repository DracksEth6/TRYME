<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Project TRYME – Phase 2 Full</title>
<style>
body { font-family: Arial, sans-serif; background: #f4f6f8; margin: 0; padding: 0; }
header { background: #1f2937; color: white; padding: 15px; text-align: center; }
.container { padding: 20px; }
.card { background: white; padding: 15px; margin-bottom: 20px; border-radius: 8px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
input, button, select, textarea { width: 100%; padding: 8px; margin-top: 8px; }
button { background: #2563eb; color: white; border: none; cursor: pointer; border-radius: 4px; }
button:hover { background: #1e40af; }
.hidden { display: none; }
table { width: 100%; border-collapse: collapse; margin-top: 10px; }
th, td { border: 1px solid #ddd; padding: 8px; text-align: left; }
th { background: #e5e7eb; }
</style>
</head>
<body>
<header>
<h1>Project TRYME</h1>
<p>Academic Integrity Monitoring System – Phase 2</p>
</header>
<div class="container">

<!-- STUDENT LOGIN / REGISTRATION -->
<div class="card" id="loginPage">
<h2>Login / Register</h2>
<input type="text" id="loginStudentUsername" placeholder="Username">
<input type="password" id="loginStudentPass" placeholder="Password">
<button onclick="studentLogin()">Login as Student</button>
<button onclick="showStudentRegister()">Register New Student</button>
<button onclick="showTeacherLogin()">Teacher Login</button>
</div>

<!-- STUDENT REGISTRATION -->
<div class="card hidden" id="studentRegister">
<h2>Student Registration</h2>
<input type="text" id="regUsername" placeholder="Choose a Username">
<input type="text" id="regGiven" placeholder="Given Name">
<input type="text" id="regMiddle" placeholder="Middle Initial">
<input type="text" id="regLast" placeholder="Last Name">
<input type="text" id="regGrade" placeholder="Grade Level / Section">
<input type="password" id="regPass" placeholder="Password">
<input type="password" id="regPassConfirm" placeholder="Confirm Password">
<button onclick="registerStudent()">Register</button>
<button onclick="backToLogin()">Back to Login</button>
</div>

<!-- STUDENT CONSENT -->
<div class="card hidden" id="consentPage">
<h2>Exam Monitoring Consent</h2>
<p>This exam requires access to your camera and microphone to monitor suspicious behavior.</p>
<label><input type="checkbox" id="consentCheck"> I agree to enable camera and microphone.</label>
<button onclick="startConsent()">Proceed to Exam</button>
</div>

<!-- STUDENT EXAM PAGE -->
<div class="card hidden" id="studentPage">
<h2>Student Exam</h2>
<h3 id="studentFullName"></h3>
<input type="text" id="searchExam" placeholder="Enter Exam Name">
<button onclick="searchExam()">Search Exam</button>
<div id="examContainer" class="hidden">
<h3 id="examTitle"></h3>
<div id="questionsContainer"></div>
<button onclick="submitExam()">Submit Exam</button>
</div>
<button onclick="logout()">Logout</button>
</div>

<!-- TEACHER LOGIN -->
<div class="card hidden" id="teacherLoginPage">
<h2>Teacher Login</h2>
<input type="text" id="teacherUsername" placeholder="Username">
<input type="password" id="teacherPassword" placeholder="Password">
<button onclick="teacherLogin()">Login</button>
<button onclick="showTeacherRegister()">Register New Teacher</button>
<button onclick="backToLogin()">Back to Login</button>
</div>

<!-- TEACHER REGISTRATION -->
<div class="card hidden" id="teacherRegisterPage">
<h2>Teacher Registration</h2>
<input type="text" id="regTeacherUsername" placeholder="Username">
<input type="text" id="regTeacherName" placeholder="Full Name">
<input type="password" id="regTeacherPass" placeholder="Password">
<input type="password" id="regTeacherPassConfirm" placeholder="Confirm Password">
<button onclick="registerTeacher()">Register</button>
<button onclick="backToTeacherLogin()">Back to Login</button>
</div>

<!-- TEACHER DASHBOARD -->
<div class="card hidden" id="teacherPage">
<h2>Teacher Dashboard</h2>
<input type="text" id="newExamName" placeholder="Exam Name">
<input type="text" id="newExamClass" placeholder="Class / Section">

<h3>Add Questions</h3>
<select id="questionType">
<option value="mcq">Multiple Choice</option>
<option value="open">Open-Ended</option>
</select>
<input type="text" id="questionText" placeholder="Question Text">
<div id="mcqOptionsContainer">
<input type="text" class="mcqOption" placeholder="Option 1">
<input type="text" class="mcqOption" placeholder="Option 2">
<input type="text" class="mcqOption" placeholder="Option 3">
<input type="text" class="mcqOption" placeholder="Option 4">
</div>
<button onclick="addQuestion()">Add Question</button>

<h4>Questions List</h4>
<ul id="questionsList"></ul>
<button onclick="uploadExam()">Upload Exam</button>

<h3>Submissions</h3>
<div id="submissionsContainer"></div>
<button onclick="logout()">Logout</button>
</div>

</div>

<script>
// ===== GLOBAL VARIABLES =====
let students = JSON.parse(localStorage.getItem('students')) || [];
let teachers = JSON.parse(localStorage.getItem('teachers')) || [];
let currentStudent = null;
let currentTeacher = null;
let exams = JSON.parse(localStorage.getItem('exams')) || [];
let currentExam = null;
let currentExamQuestions = [];
let violationCount = 0;
let monitoringLogs = [];
let videoStream = null;
let audioStream = null;
let examStartTime = null;

// ===== STUDENT FUNCTIONS =====
function showStudentRegister(){
document.getElementById('loginPage').classList.add('hidden');
document.getElementById('studentRegister').classList.remove('hidden');
}
function backToLogin(){
document.getElementById('studentRegister').classList.add('hidden');
document.getElementById('loginPage').classList.remove('hidden');
document.getElementById('teacherLoginPage').classList.add('hidden');
document.getElementById('teacherRegisterPage').classList.add('hidden');
}
function registerStudent(){
const u=document.getElementById('regUsername').value.trim();
const g=document.getElementById('regGiven').value.trim();
const m=document.getElementById('regMiddle').value.trim();
const l=document.getElementById('regLast').value.trim();
const gr=document.getElementById('regGrade').value.trim();
const p=document.getElementById('regPass').value;
const c=document.getElementById('regPassConfirm').value;
if(!u||!g||!l||!gr||!p||!c){alert("Fill all fields");return;}
if(p!==c){alert("Passwords do not match");return;}
if(students.find(s=>s.username===u)){alert("Username taken");return;}
students.push({username:u,given:g,middle:m,last:l,grade:gr,pass:p});
localStorage.setItem('students',JSON.stringify(students));
alert("Registered Successfully!\nFull Name: "+g+" "+m+" "+l);
backToLogin();
}
function studentLogin(){
const u=document.getElementById('loginStudentUsername').value.trim();
const p=document.getElementById('loginStudentPass').value;
const s=students.find(x=>x.username===u && x.pass===p);
if(s){currentStudent=s;
document.getElementById('loginPage').classList.add('hidden');
document.getElementById('consentPage').classList.remove('hidden');
}else alert("Invalid credentials");
}

// ===== TEACHER REGISTRATION / LOGIN =====
function showTeacherLogin(){
document.getElementById('loginPage').classList.add('hidden');
document.getElementById('teacherLoginPage').classList.remove('hidden');
}
function showTeacherRegister(){
document.getElementById('teacherLoginPage').classList.add('hidden');
document.getElementById('teacherRegisterPage').classList.remove('hidden');
}
function backToTeacherLogin(){
document.getElementById('teacherRegisterPage').classList.add('hidden');
document.getElementById('teacherLoginPage').classList.remove('hidden');
}
function registerTeacher(){
const u=document.getElementById('regTeacherUsername').value.trim();
const n=document.getElementById('regTeacherName').value.trim();
const p=document.getElementById('regTeacherPass').value;
const c=document.getElementById('regTeacherPassConfirm').value;
if(!u||!n||!p||!c){alert("Fill all fields");return;}
if(p!==c){alert("Passwords do not match");return;}
if(teachers.find(t=>t.username===u)){alert("Username already taken");return;}
teachers.push({username:u,fullname:n,password:p});
localStorage.setItem('teachers',JSON.stringify(teachers));
alert("Teacher Registered Successfully!");
backToTeacherLogin();
}
function teacherLogin(){
const u=document.getElementById('teacherUsername').value.trim();
const p=document.getElementById('teacherPassword').value;
const t=teachers.find(x=>x.username===u && x.password===p);
if(t){currentTeacher=t;
document.getElementById('teacherLoginPage').classList.add('hidden');
document.getElementById('teacherPage').classList.remove('hidden');
loadSubmissions();
}else alert("Invalid teacher credentials");
}

// ===== STUDENT CONSENT & MONITORING =====
function startConsent(){
if(!document.getElementById('consentCheck').checked){alert("Consent required"); return;}
document.getElementById('consentPage').classList.add('hidden');
document.getElementById('studentPage').classList.remove('hidden');
document.getElementById('studentFullName').innerText=currentStudent.given+" "+currentStudent.middle+" "+currentStudent.last;
examStartTime=Date.now();
violationCount=0;
monitoringLogs=[];
startMonitoring();
}

// Monitoring only active for students during exam
function startMonitoring(){
// Camera
navigator.mediaDevices.getUserMedia({video:true}).then(stream=>{
videoStream=stream;
const video=document.createElement('video');
video.srcObject=stream; video.play();
setInterval(()=>{if(video.videoWidth===0) logViolation("Face not detected");},5000);
}).catch(()=>logViolation("Camera access denied"));
// Audio
navigator.mediaDevices.getUserMedia({audio:true}).then(stream=>{
audioStream=stream;
const ctx=new AudioContext();
const analyser=ctx.createAnalyser();
const source=ctx.createMediaStreamSource(stream);
source.connect(analyser);
const data=new Uint8Array(analyser.frequencyBinCount);
setInterval(()=>{
analyser.getByteFrequencyData(data);
const vol=data.reduce((a,b)=>a+b)/data.length;
if(vol>25) logViolation("Voice detected");
},4000);
}).catch(()=>logViolation("Microphone access denied"));
// Browser rules
window.addEventListener("blur",()=>logViolation("Switched tab/window"));
document.addEventListener("visibilitychange",()=>{if(document.visibilityState!=="visible") logViolation("Window minimized/hidden");});
document.addEventListener("copy",e=>{e.preventDefault();logViolation("Copy attempt");});
document.addEventListener("cut",e=>{e.preventDefault();logViolation("Cut attempt");});
document.addEventListener("paste",e=>{e.preventDefault();logViolation("Paste attempt");});
document.addEventListener("contextmenu",e=>e.preventDefault());
window.addEventListener("offline",()=>logViolation("Network disconnected"));
window.addEventListener("online",()=>logViolation("Network reconnected"));
setInterval(()=>{
let now=Date.now();
if(Math.abs(now-(examStartTime+(now-examStartTime)))>5*60*1000) logViolation("Time anomaly");
},30000);
}

function logViolation(reason){
const t=new Date().toLocaleTimeString();
monitoringLogs.push(`${t} – ${reason}`);
violationCount++;
if(violationCount>=3){
alert("You violated rules 3 times. You are now disqualified from this exam.");
stopMonitoring();
disableExamForStudent();
document.getElementById('studentPage').classList.add('hidden');
document.getElementById('loginPage').classList.remove('hidden');
}
}
function stopMonitoring(){
if(videoStream) videoStream.getTracks().forEach(t=>t.stop());
if(audioStream) audioStream.getTracks().forEach(t=>t.stop());
}
function disableExamForStudent(){
if(!currentExam.disqualified) currentExam.disqualified=[];
currentExam.disqualified.push(currentStudent.username);
localStorage.setItem('exams',JSON.stringify(exams));
}

// ===== EXAM FUNCTIONS =====
function searchExam(){
const kw=document.getElementById('searchExam').value.toLowerCase();
const ex=exams.find(e=>e.name.toLowerCase().includes(kw));
if(ex){
if(ex.disqualified&&ex.disqualified.includes(currentStudent.username)){alert("You are disqualified."); return;}
currentExam=ex;
document.getElementById('examContainer').classList.remove('hidden');
document.getElementById('examTitle').innerText=ex.name+" ("+ex.class+")";
loadQuestions(ex);
}else alert("No exam found.");
}
function loadQuestions(exam){
const container=document.getElementById('questionsContainer');
container.innerHTML="";
currentExamQuestions=exam.questions;
exam.questions.forEach((q,i)=>{
if(q.type==="mcq"){
let opts=q.options.map((o,j)=>`<label><input type="radio" name="q${i}" value="${o}"> ${o}</label><br>`).join('');
container.innerHTML+=`<div><b>Q${i+1}:</b> ${q.text}<br>${opts}</div>`;
}else container.innerHTML+=`<div><b>Q${i+1}:</b> ${q.text}<br><textarea name="q${i}"></textarea></div>`;
});
}
function submitExam(){
stopMonitoring();
const answers=[];
currentExamQuestions.forEach((q,i)=>{
if(q.type==="mcq"){const sel=document.querySelector(`[name="q${i}"]:checked`);answers.push(sel?sel.value:"No Answer");}
else{const el=document.querySelector(`[name="q${i}"]`);answers.push(el.value);}
});
if(!currentExam.submissions) currentExam.submissions=[];
currentExam.submissions.push({student:currentStudent.username,answers:answers,violations:violationCount,logs:monitoringLogs});
localStorage.setItem('exams',JSON.stringify(exams));
alert("Exam Submitted Successfully!");
document.getElementById('studentPage').classList.add('hidden');
document.getElementById('loginPage').classList.remove('hidden');
}

// ===== TEACHER DASHBOARD =====
function addQuestion(){
const type=document.getElementById('questionType').value;
const text=document.getElementById('questionText').value.trim();
if(!text){alert("Enter question text"); return;}
let question={type,text};
if(type==="mcq"){
question.options=Array.from(document.querySelectorAll(".mcqOption")).map(o=>o.value.trim()).filter(v=>v);
if(question.options.length<2){alert("Add at least 2 options"); return;}
}
currentExamQuestions.push(question);
updateQuestionsList();
document.getElementById('questionText').value="";
}
function updateQuestionsList(){
const ul=document.getElementById('questionsList');
ul.innerHTML="";
currentExamQuestions.forEach((q,i)=>{ul.innerHTML+=`<li>${i+1}. ${q.type.toUpperCase()}: ${q.text}</li>`;});
}
function uploadExam(){
const name=document.getElementById('newExamName').value.trim();
const cls=document.getElementById('newExamClass').value.trim();
if(!name||!cls){alert("Fill exam name and class"); return;}
exams.push({name:name,class:cls,questions:currentExamQuestions,submissions:[],disqualified:[]});
localStorage.setItem('exams',JSON.stringify(exams));
currentExamQuestions=[];
updateQuestionsList();
loadSubmissions();
alert("Exam Uploaded!");
}
function loadSubmissions(){
const container=document.getElementById('submissionsContainer');
container.innerHTML="";
exams.forEach(exam=>{
if(exam.submissions){
exam.submissions.forEach(sub=>{
const level=sub.violations>=3?"DISQUALIFIED":sub.violations>=2?"MEDIUM":"LOW";
container.innerHTML+=`<div class="card"><b>${sub.student}</b> (${exam.name})<br>Violation Level: <b>${level}</b><ul>${sub.logs.map(l=>`<li>${l}</li>`).join('')}</ul></div>`;
});
}
});
}

// ===== LOGOUT =====
function logout(){
document.getElementById('loginPage').classList.remove('hidden');
document.getElementById('studentPage').classList.add('hidden');
document.getElementById('teacherPage').classList.add('hidden');
document.getElementById('studentRegister').classList.add('hidden');
document.getElementById('consentPage').classList.add('hidden');
document.getElementById('teacherLoginPage').classList.add('hidden');
document.getElementById('teacherRegisterPage').classList.add('hidden');
currentExamQuestions=[];
currentStudent=null;
currentTeacher=null;
}
</script>
</body>
</html>
