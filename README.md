<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>SK JI Computer Coaching Classes and Institute</title>

<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js"></script>

<style>
body{
margin:0;
font-family:Arial;
background:linear-gradient(135deg,#141e30,#243b55);
color:white;
text-align:center;
}
header{
background:#ff6f00;
padding:15px;
font-size:20px;
font-weight:bold;
}
.container{
background:white;
color:black;
width:95%;
max-width:1000px;
margin:20px auto;
padding:20px;
border-radius:10px;
}
input,select{
padding:8px;
margin:5px;
width:200px;
}
button{
padding:8px 12px;
margin:4px;
border:none;
border-radius:6px;
background:#1976d2;
color:white;
cursor:pointer;
}
button:hover{background:#0d47a1;}
.card{
background:#f5f5f5;
padding:10px;
margin:8px;
border-radius:6px;
display:inline-block;
width:180px;
}
.hidden{display:none;}
.present{color:green;font-weight:bold;}
.auto_out{color:orange;}
.manual_out{color:blue;}
.absent{color:red;font-weight:bold;}
</style>
</head>

<body>

<header>
SK JI Computer Coaching Classes and Institute<br>
Professional Attendance System
</header>

<div class="container">

<div id="userPanel">
<input type="text" id="name" placeholder="Enter Full Name">
<select id="role">
<option value="student">Student</option>
<option value="staff">Staff</option>
</select>

<button onclick="markIn()">IN</button>
<button onclick="markOut()">OUT</button>
<button onclick="applyLeave()">Leave</button>

<hr>
<button onclick="adminLogin()">Admin Login</button>
</div>

<div id="adminPanel" class="hidden">

<h3>Admin Dashboard</h3>

<div>
<div class="card">Students Present<br><span id="studentCount">0</span></div>
<div class="card">Staff Present<br><span id="staffCount">0</span></div>
<div class="card">Today Absent<br><span id="absentCount">0</span></div>
<div class="card">Monthly Records<br><span id="monthlyCount">0</span></div>
</div>

<br>

<input type="number" id="fixedHours" placeholder="Staff Fixed Hours">
<button onclick="setHours()">Update Hours</button>

<br><br>

<input type="text" id="search" placeholder="Search Name">
<select id="filterRole">
<option value="all">All</option>
<option value="student">Student</option>
<option value="staff">Staff</option>
</select>
<button onclick="exportCSV()">Export CSV</button>

<h4>Attendance Records</h4>
<div id="records"></div>

<h4>Pending Leave</h4>
<div id="leaveList"></div>

<button onclick="logout()">Logout</button>

</div>

</div>

<script>
const firebaseConfig={
apiKey:"YOUR_API_KEY",
authDomain:"YOUR_DOMAIN",
projectId:"YOUR_PROJECT_ID",
};
firebase.initializeApp(firebaseConfig);
const db=firebase.firestore();

let staffHours=6;

function setHours(){
staffHours=document.getElementById("fixedHours").value;
alert("Staff Hours Updated");
}

function markIn(){
const name=document.getElementById("name").value.trim();
const role=document.getElementById("role").value;
if(!name) return alert("Enter Name");

const now=new Date();
let hours= role==="student"?1:staffHours;
let outTime=new Date(now.getTime()+hours*3600000);

db.collection("attendance").add({
id:Date.now(),
name,role,
in_time:now,
out_time:outTime,
status:"present"
});
}

function markOut(){
const name=document.getElementById("name").value.trim();
db.collection("attendance")
.where("name","==",name)
.where("status","==","present")
.get()
.then(s=>s.forEach(doc=>doc.ref.update({status:"manual_out"})));
}

function applyLeave(){
const name=document.getElementById("name").value.trim();
const role=document.getElementById("role").value;
const today=new Date();
let required= role==="student"?1:2;
let selected=prompt("Enter Leave Date (YYYY-MM-DD)");
let leaveDate=new Date(selected);

if((leaveDate-today)/(1000*60*60*24) < required){
alert("Leave rule not satisfied");
return;
}

db.collection("leave").add({
name,role,date:leaveDate,status:"pending"
});
}

function adminLogin(){
let pass=prompt("Password");
if(pass==="SKJI@2026"){
document.getElementById("userPanel").classList.add("hidden");
document.getElementById("adminPanel").classList.remove("hidden");
}
}

function logout(){
document.getElementById("adminPanel").classList.add("hidden");
document.getElementById("userPanel").classList.remove("hidden");
}

db.collection("attendance").onSnapshot(snapshot=>{
let student=0,staff=0,monthly=0;
let today=new Date().toDateString();
let presentNames=[];
let html="";
snapshot.forEach(doc=>{
let d=doc.data();
let inDate=d.in_time.toDate();
let outTime=d.out_time.toDate();
let now=new Date();

if(now>outTime && d.status==="present"){
doc.ref.update({status:"auto_out"});
}

if(d.status==="present"){
if(d.role==="student") student++;
if(d.role==="staff") staff++;
presentNames.push(d.name);
}

if(inDate.getMonth()===new Date().getMonth()) monthly++;

html+=`${d.name} - ${d.role} - <span class="${d.status}">${d.status}</span>
<button onclick="del('${doc.id}')">Delete</button><br>`;
});

document.getElementById("studentCount").innerText=student;
document.getElementById("staffCount").innerText=staff;
document.getElementById("monthlyCount").innerText=monthly;
document.getElementById("records").innerHTML=html;

generateAbsent(presentNames);
});

function generateAbsent(presentNames){
let expected=["Student1","Student2","Staff1"];
let absent=0,html="";
expected.forEach(n=>{
if(!presentNames.includes(n)){
absent++;
html+=n+"<br>";
}
});
document.getElementById("absentCount").innerText=absent;
}

function del(id){
db.collection("attendance").doc(id).delete();
}

db.collection("leave").where("status","==","pending")
.onSnapshot(snapshot=>{
let html="";
snapshot.forEach(doc=>{
let d=doc.data();
html+=`${d.name} - ${d.role}
<button onclick="approve('${doc.id}')">Approve</button>
<button onclick="reject('${doc.id}')">Reject</button><br>`;
});
document.getElementById("leaveList").innerHTML=html;
});

function approve(id){
db.collection("leave").doc(id).update({status:"approved"});
}

function reject(id){
db.collection("leave").doc(id).update({status:"rejected"});
}

function exportCSV(){
db.collection("attendance").get().then(snapshot=>{
let csv="Name,Role,Status\n";
snapshot.forEach(doc=>{
let d=doc.data();
csv+=`${d.name},${d.role},${d.status}\n`;
});
let blob=new Blob([csv],{type:"text/csv"});
let url=URL.createObjectURL(blob);
let a=document.createElement("a");
a.href=url;
a.download="attendance.csv";
a.click();
});
}
</script>

</body>
</html>
