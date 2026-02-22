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
background:linear-gradient(135deg,#1d2671,#c33764);
color:white;
text-align:center;
}
header{
background:#ff9800;
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
.hidden{display:none;}
.card{
background:#f5f5f5;
padding:10px;
margin:8px;
border-radius:6px;
display:inline-block;
width:200px;
}
.present{color:green;font-weight:bold;}
.absent{color:red;font-weight:bold;}
.auto_out{color:orange;}
</style>
</head>

<body>

<header>
SK JI Computer Coaching Classes and Institute<br>
Smart Attendance System
</header>

<div class="container">

<h3>User Panel</h3>

<input type="text" id="name" placeholder="Enter Name">
<select id="role">
<option value="student">Student</option>
<option value="staff">Staff</option>
</select>

<button onclick="markIn()">IN</button>
<button onclick="markOut()">OUT</button>
<button onclick="showMyReport()">My Report</button>

<hr>
<button onclick="adminLogin()">Admin Login</button>

<div id="myReport"></div>

<div id="adminPanel" class="hidden">
<h3>Admin Panel</h3>

<h4>Add Staff (One Time)</h4>
<input type="text" id="staffName" placeholder="Staff Name">
<button onclick="addStaff()">Add Staff</button>

<h4>Staff List</h4>
<div id="staffList"></div>

<h4>Set Staff Fixed Hours</h4>
<input type="number" id="staffHours" placeholder="Hours">
<button onclick="saveHours()">Save</button>

<h4>Monthly Report</h4>
<div id="monthlyReport"></div>

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

let fixedHours=6;

function saveHours(){
fixedHours=document.getElementById("staffHours").value;
alert("Updated");
}

function addStaff(){
let name=document.getElementById("staffName").value.trim();
if(!name) return;

db.collection("staff").doc(name).get().then(doc=>{
if(doc.exists){
alert("Staff already exists");
}else{
db.collection("staff").doc(name).set({
name:name,
created:new Date()
});
alert("Staff Added");
}
});
}

db.collection("staff").onSnapshot(snapshot=>{
let html="";
snapshot.forEach(doc=>{
html+=doc.data().name+"<br>";
});
document.getElementById("staffList").innerHTML=html;
});

function markIn(){
let name=document.getElementById("name").value.trim();
let role=document.getElementById("role").value;
if(!name) return;

let now=new Date();
let hours= role==="student"?1:fixedHours;
let outTime=new Date(now.getTime()+hours*3600000);

if(role==="staff"){
db.collection("staff").doc(name).get().then(doc=>{
if(!doc.exists){
alert("Staff not registered");
return;
}
saveAttendance(name,role,now,outTime);
});
}else{
saveAttendance(name,role,now,outTime);
}
}

function saveAttendance(name,role,now,outTime){
db.collection("attendance").add({
name,role,
date:now.toDateString(),
in_time:now,
out_time:outTime,
status:"present"
});
alert("IN Done");
}

function markOut(){
let name=document.getElementById("name").value.trim();
db.collection("attendance")
.where("name","==",name)
.where("status","==","present")
.get()
.then(s=>s.forEach(doc=>doc.ref.update({status:"manual_out"})));
}

function showMyReport(){
let name=document.getElementById("name").value.trim();
let present=0,absent=0;
db.collection("attendance")
.where("name","==",name)
.get()
.then(snapshot=>{
snapshot.forEach(doc=>{
if(doc.data().status==="present" || doc.data().status==="manual_out" || doc.data().status==="auto_out")
present++;
});
});
document.getElementById("myReport").innerHTML=
"<div class='card'>Present Days: "+present+"</div>";
}

function adminLogin(){
let pass=prompt("Password");
if(pass==="SKJI@2026"){
document.getElementById("adminPanel").classList.remove("hidden");
}
}

function logout(){
document.getElementById("adminPanel").classList.add("hidden");
}

db.collection("attendance").onSnapshot(snapshot=>{
let monthly=0;
snapshot.forEach(doc=>{
let d=doc.data();
if(new Date(d.in_time.toDate()).getMonth()===new Date().getMonth())
monthly++;
});
document.getElementById("monthlyReport").innerHTML=
"<div class='card'>Total Monthly Entries: "+monthly+"</div>";
});
</script>

</body>
</html>
