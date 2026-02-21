<!DOCTYPE html>
<html>
<head>
<title>Hive Relay Pro</title>
<script src="https://unpkg.com/tmi.js@1.8.5/dist/tmi.min.js"></script>

<style>
body{
    background:#0e0e10;
    color:#efeff1;
    font-family:Inter,sans-serif;
    margin:0;
    height:100vh;
    display:flex;
    flex-direction:column;
}
#header{
    padding:12px;
    background:#18181b;
    border-bottom:2px solid #303032;
    display:flex;
    justify-content:space-between;
    align-items:center;
    flex-wrap:wrap;
    gap:8px;
}
#controls{display:flex;gap:8px;flex-wrap:wrap}
#content{flex:1;display:flex}
#stream{flex:2}
#chat{
    flex:1;
    background:#0e0e10;
    border-left:2px solid #303032;
    overflow-y:auto;
    padding:10px;
    font-size:13px;
}
#input-area{
    padding:12px;
    background:#18181b;
    display:flex;
    gap:10px;
}
input{
    padding:8px;
    background:#0e0e10;
    border:1px solid #464649;
    color:white;
}
button{
    padding:8px 12px;
    background:#9147ff;
    border:none;
    color:white;
    font-weight:bold;
    cursor:pointer;
    border-radius:4px;
}
.msg{margin-bottom:6px}
.chan{color:#bf94ff;margin-right:6px}
#error{
    background:#400;
    color:white;
    padding:6px;
    display:none;
}
</style>
</head>

<body>

<div id="header">
<div id="status">🔴 Not Logged In</div>
<div id="controls">
<button id="login-btn">Login</button>
<button onclick="logout()">Logout</button>
<button onclick="autoJoin()">Auto Join Followed</button>
<button onclick="enableChat()">Enable Chat</button>
<button onclick="refreshStreams()">Refresh</button>
</div>
</div>

<div id="error"></div>

<div id="content">
<div id="stream">
<iframe id="player" width="100%" height="100%" frameborder="0" allowfullscreen></iframe>
</div>
<div id="chat">Chat disabled.</div>
</div>

<div id="input-area">
<input id="vod-input" placeholder="Enter VOD ID">
<button onclick="loadVODFromInput()">Load VOD</button>
<input id="mass-input" placeholder="Relay message">
<button onclick="broadcast()">SEND</button>
</div>

<script>

/* CONFIG */

const CONFIG={
clientId:"cvv666bkohm1zuauw2f5nqcwvi1hx8",
redirect:"https://lilalien45.github.io/hive-relay-pro/",
parent:"lilalien45.github.io"
};

let accessToken=null;
let twitchClient=null;
let channels=[];
let username=null;

/* LOGIN */

document.getElementById("login-btn").onclick=()=>{

const authUrl=
"https://id.twitch.tv/oauth2/authorize"+
"?response_type=token"+
"&client_id="+CONFIG.clientId+
"&redirect_uri="+encodeURIComponent(CONFIG.redirect)+
"&scope="+encodeURIComponent(
"chat:read chat:edit user:read:follows"
)+
"&force_verify=true";

window.location.href=authUrl;
};

/* AUTH */

function checkAuth(){
const params=new URLSearchParams(location.hash.substring(1));
const token=params.get("access_token");

if(token){
localStorage.setItem("twitch_token",token);
window.location.hash="";
}

accessToken=localStorage.getItem("twitch_token");

if(accessToken){
document.getElementById("status").innerText="🟢 Logged In";
}
}

checkAuth();

/* STREAM FUNCTIONS */

function loadStream(channel){
document.getElementById("player").src =
`https://player.twitch.tv/?channel=${channel}&parent=${CONFIG.parent}`;
}

function loadVOD(videoId){
document.getElementById("player").src =
`https://player.twitch.tv/?video=${videoId}&parent=${CONFIG.parent}`;
}

function loadVODFromInput(){
const id=document.getElementById("vod-input").value;
if(id) loadVOD(id);
}

/* AUTO JOIN FOLLOWED */

async function autoJoin(){

if(!accessToken){
showError("Login required.");
return;
}

try{

const userRes=await fetch(
"https://api.twitch.tv/helix/users",
{
headers:{
"Client-ID":CONFIG.clientId,
"Authorization":"Bearer "+accessToken
}
}
);

const userData=await userRes.json();
username=userData.data[0].login;

const followRes=await fetch(
"https://api.twitch.tv/helix/streams/followed?first=20",
{
headers:{
"Client-ID":CONFIG.clientId,
"Authorization":"Bearer "+accessToken
}
}
);

const followData=await followRes.json();

if(!followData.data.length){
showError("No live followed streams.");
return;
}

channels=followData.data.map(x=>x.user_login);
loadStream(channels[0]);

}catch(err){
showError(err.message);
}

}

/* ENABLE CHAT */

async function enableChat(){

if(!channels.length){
showError("Join channels first.");
return;
}

twitchClient=new tmi.Client({
connection:{secure:true,reconnect:true},
identity:{
username:username,
password:"oauth:"+accessToken
},
channels:channels
});

await twitchClient.connect();

twitchClient.on("message",(channel,tags,msg)=>{
const chat=document.getElementById("chat");
const div=document.createElement("div");
div.className="msg";
div.innerHTML=
`<span class="chan">[${channel}]</span>
<b>${tags["display-name"]}</b>: ${msg}`;
chat.appendChild(div);
chat.scrollTop=chat.scrollHeight;
});

document.getElementById("chat").innerText="";
}

/* REFRESH */

function refreshStreams(){
autoJoin();
}

/* BROADCAST */

function broadcast(){

if(!twitchClient) return;

const text=document.getElementById("mass-input").value;
if(!text) return;

channels.forEach((chan,i)=>{
setTimeout(()=>{
twitchClient.say(chan,text);
},i*600);
});

document.getElementById("mass-input").value="";
}

/* LOGOUT */

function logout(){
localStorage.removeItem("twitch_token");
location.reload();
}

/* ERROR */

function showError(msg){
const el=document.getElementById("error");
el.style.display="block";
el.innerText=msg;
}

</script>

</body>
</html>let channels=[];
let username=null;

/* ================= LOGIN ================= */

document.getElementById("login-btn").onclick=()=>{

const authUrl=
"https://id.twitch.tv/oauth2/authorize"+
"?response_type=token"+
"&client_id="+CONFIG.clientId+
"&redirect_uri="+encodeURIComponent(CONFIG.redirect)+
"&scope="+encodeURIComponent(
"chat:read chat:edit user:read:follows"
)+
"&force_verify=true";

window.location.href=authUrl;
};

/* ================= AUTH ================= */

function checkAuth(){

const params=new URLSearchParams(location.hash.substring(1));
const token=params.get("access_token");

if(token){
localStorage.setItem("twitch_token",token);
window.location.hash="";
}

accessToken=localStorage.getItem("twitch_token");

if(accessToken){
document.getElementById("status").innerText="🟢 Logged In";
initRelay();
}

}

/* ================= STREAM LOADERS ================= */

function loadStream(channel){
document.getElementById("player").src =
`https://player.twitch.tv/?channel=${channel}&parent=${CONFIG.parent}`;
}

function loadVOD(videoId){
document.getElementById("player").src =
`https://player.twitch.tv/?video=${videoId}&parent=${CONFIG.parent}`;
}

/* ================= INIT SYSTEM ================= */

async function initRelay(){

try{

if(!accessToken){
showError("Login expired.");
return;
}

/* Get user */
const userRes=await fetch(
"https://api.twitch.tv/helix/users",
{
headers:{
"Client-ID":CONFIG.clientId,
"Authorization":"Bearer "+accessToken
}
}
);

const userData=await userRes.json();

if(!userData.data || !userData.data.length){
showError("Invalid token.");
return;
}

username=userData.data[0].login;

/* Get followed live streams */
const followRes=await fetch(
"https://api.twitch.tv/helix/streams/followed?first=20",
{
headers:{
"Client-ID":CONFIG.clientId,
"Authorization":"Bearer "+accessToken
}
}
);

const followData=await followRes.json();

if(!followData.data || followData.data.length===0){
document.getElementById("chat").innerText="No followed streams live.";
return;
}

/* Load first live stream */
loadStream(followData.data[0].user_login);

/* Collect channels */
channels=followData.data.map(x=>x.user_login);

/* Connect chat */
twitchClient=new tmi.Client({
options:{debug:false},
connection:{secure:true,reconnect:true},
identity:{
username:username,
password:"oauth:"+accessToken
},
channels:channels
});

await twitchClient.connect();

twitchClient.on("message",(channel,tags,msg)=>{
const chat=document.getElementById("chat");
const div=document.createElement("div");
div.className="msg";
div.innerHTML=
`<span class="chan">[${channel}]</span>
<b>${tags["display-name"]}</b>: ${msg}`;
chat.appendChild(div);
chat.scrollTop=chat.scrollHeight;
});

}catch(err){
showError("Init failed: "+err.message);
}

}

/* ================= BROADCAST ================= */

function broadcast(){

if(!twitchClient) return;

const text=document.getElementById("mass-input").value;
if(!text) return;

channels.forEach((chan,i)=>{
setTimeout(()=>{
twitchClient.say(chan,text);
},i*600);
});

document.getElementById("mass-input").value="";
}

/* ================= ERROR ================= */

function showError(msg){
const el=document.getElementById("error");
el.style.display="block";
el.innerText=msg;
}

/* ================= LOGOUT ================= */

function logout(){
localStorage.removeItem("twitch_token");
location.reload();
}

/* ================= START ================= */

checkAuth();

</script>

</body>
</html>
