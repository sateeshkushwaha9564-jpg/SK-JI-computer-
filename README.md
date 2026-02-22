<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>SK JI Computer Coaching Classes and Institute</title>

<!-- Firebase v9 modular -->
<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/10.7.1/firebase-app.js";
import {
  getAuth, createUserWithEmailAndPassword, signInWithEmailAndPassword,
  onAuthStateChanged, signOut
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-auth.js";
import {
  getFirestore, doc, setDoc, getDoc, addDoc, collection,
  query, where, getDocs, onSnapshot, updateDoc
} from "https://www.gstatic.com/firebasejs/10.7.1/firebase-firestore.js";

const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_DOMAIN",
  projectId: "YOUR_PROJECT_ID"
};

const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);

let currentUserRole = "";
let staffFixedHours = 6;

/* ================= REGISTER ================= */
window.registerUser = async function () {
  const email = regEmail.value.trim();
  const pass = regPass.value.trim();
  const role = regRole.value;

  if (!email || !pass) return alert("Fill all fields");

  try {
    const userCred = await createUserWithEmailAndPassword(auth, email, pass);
    await setDoc(doc(db, "users", userCred.user.uid), {
      email, role, createdAt: new Date()
    });
    alert("Registered Successfully");
  } catch (e) {
    alert(e.message);
  }
};

/* ================= LOGIN ================= */
window.loginUser = async function () {
  try {
    const userCred = await signInWithEmailAndPassword(auth, loginEmail.value, loginPass.value);
    const snap = await getDoc(doc(db, "users", userCred.user.uid));
    currentUserRole = snap.data().role;
  } catch (e) {
    alert("Login Failed");
  }
};

onAuthStateChanged(auth, async (user) => {
  if (user) {
    authSection.style.display = "none";
    dashboard.style.display = "block";
    userEmail.innerText = user.email;

    const snap = await getDoc(doc(db, "users", user.uid));
    currentUserRole = snap.data().role;
    roleText.innerText = currentUserRole;

    loadDashboard(user.uid);
  } else {
    authSection.style.display = "block";
    dashboard.style.display = "none";
  }
});

/* ================= LOGOUT ================= */
window.logoutUser = function () {
  signOut(auth);
};

/* ================= MARK IN ================= */
window.markIn = async function () {
  const user = auth.currentUser;
  if (!user) return;

  const now = new Date();
  const today = now.toDateString();

  const hours = currentUserRole === "student" ? 1 : staffFixedHours;
  const outTime = new Date(now.getTime() + hours * 3600000);

  await addDoc(collection(db, "attendance"), {
    uid: user.uid,
    email: user.email,
    role: currentUserRole,
    date: today,
    in_time: now,
    out_time: outTime,
    status: "present"
  });
};

/* ================= MARK OUT ================= */
window.markOut = async function () {
  const user = auth.currentUser;
  const q = query(collection(db, "attendance"),
    where("uid", "==", user.uid),
    where("status", "==", "present")
  );
  const snap = await getDocs(q);
  snap.forEach(async (d) => {
    await updateDoc(d.ref, { status: "manual_out" });
  });
};

/* ================= DASHBOARD ================= */
async function loadDashboard(uid) {

  onSnapshot(collection(db, "attendance"), (snapshot) => {
    let presentDays = 0;
    let todayStatus = "Absent";
    let monthly = 0;
    let now = new Date();

    snapshot.forEach(async (docu) => {
      const d = docu.data();
      if (d.uid === uid) {

        const inDate = d.in_time.toDate();
        const outTime = d.out_time.toDate();

        // Auto OUT
        if (now > outTime && d.status === "present") {
          await updateDoc(docu.ref, { status: "auto_out" });
        }

        presentDays++;

        if (d.date === now.toDateString()) {
          todayStatus = d.status;
        }

        if (inDate.getMonth() === now.getMonth()) {
          monthly++;
        }
      }
    });

    totalPresent.innerText = presentDays;
    todayStat.innerText = todayStatus;
    monthlyCount.innerText = monthly;
  });
}

</script>

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
.section{
background:white;
color:black;
width:90%;
max-width:600px;
margin:20px auto;
padding:20px;
border-radius:10px;
}
input,select{
padding:8px;
margin:5px;
width:80%;
}
button{
padding:8px 12px;
margin:5px;
border:none;
border-radius:6px;
background:#1976d2;
color:white;
cursor:pointer;
}
.card{
background:#f1f1f1;
padding:10px;
margin:8px;
border-radius:6px;
}
</style>
</head>

<body>

<header>
SK JI Computer Coaching Classes and Institute<br>
Smart Attendance Portal (Auto Mode)
</header>

<div id="authSection" class="section">

<h3>Register</h3>
<input id="regEmail" placeholder="Email">
<input id="regPass" type="password" placeholder="Password">
<select id="regRole">
<option value="student">Student</option>
<option value="staff">Staff</option>
</select>
<br>
<button onclick="registerUser()">Register</button>

<hr>

<h3>Login</h3>
<input id="loginEmail" placeholder="Email">
<input id="loginPass" type="password" placeholder="Password">
<br>
<button onclick="loginUser()">Login</button>

</div>

<div id="dashboard" class="section" style="display:none;">

<h3>My Dashboard</h3>

<p><b>Email:</b> <span id="userEmail"></span></p>
<p><b>Role:</b> <span id="roleText"></span></p>

<div class="card">Total Present Days: <span id="totalPresent">0</span></div>
<div class="card">Today's Status: <span id="todayStat">Absent</span></div>
<div class="card">Monthly Entries: <span id="monthlyCount">0</span></div>

<br>
<button onclick="markIn()">IN</button>
<button onclick="markOut()">OUT</button>
<button onclick="logoutUser()">Logout</button>

</div>

</body>
</html>
