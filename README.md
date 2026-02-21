<!DOCTYPE html>
<html>
<head>
<title>Hive Relay Multi Viewer Pro</title>

<style>
body{
    background:#0e0e10;
    color:white;
    font-family:Arial;
    padding:15px;
}

.topbar{
    margin-bottom:15px;
}

.grid{
    display:grid;
    grid-template-columns:repeat(auto-fit,minmax(400px,1fr));
    gap:15px;
}

.slot{
    background:#18181b;
    padding:10px;
    border-radius:10px;
}

input{
    padding:8px;
    background:#0e0e10;
    border:1px solid #444;
    color:white;
}

button{
    padding:8px 14px;
    background:#9147ff;
    border:none;
    color:white;
    cursor:pointer;
    margin:4px;
}
</style>

<script src="https://unpkg.com/tmi.js@1.8.5/dist/tmi.min.js"></script>

</head>

<body>

<h2>Hive Relay Pro Viewer</h2>

<div class="topbar">

<button onclick="loginClient(0)">Login Client 1</button>
<button onclick="loginClient(1)">Login Client 2</button>
<button onclick="loginClient(2)">Login Client 3</button>

<button onclick="loadStreams()">Load Selected Streams</button>

</div>

<h3>Channel Browser</h3>
<input id="searchBox" placeholder="Search Twitch Channel">
<button onclick="addSearchToSlot()">Add To Next Slot</button>

<div id="inputs"></div>

<div id="streams" class="grid"></div>

<h3>Relay Chat</h3>

<input id="chatMessage" placeholder="Send message to all clients">
<button onclick="sendRelay()">Send Relay</button>

<script>

/* CLIENT IDS */
const CLIENTS=[
"cvv666bkohm1zuauw2f5nqcwvi1hx8",
"8etzpqt1zgsldyz25h7t3f23r4qng8",
"6qlrlu03by71dxa52tb5aqf9nodq1o"
];

const REDIRECT="https://lilalien45.github.io/hive-relay-pro/";

let tokens=[null,null,null];
let relayClients=[];
let slotChannels=new Array(6).fill("");

/* Create 6 Inputs */
const inputContainer=document.getElementById("inputs");

for(let i=0;i<6;i++){
    inputContainer.innerHTML+=`
    <div>
    Slot ${i+1}
    <input id="channel${i}" placeholder="Channel name">
    </div>
    `;
}

/* LOGIN */
function loginClient(index){

    const url=
    "https://id.twitch.tv/oauth2/authorize"+
    "?response_type=token"+
    "&client_id="+CLIENTS[index]+
    "&redirect_uri="+encodeURIComponent(REDIRECT)+
    "&scope="+encodeURIComponent("chat:read chat:edit user:read:follows")+
    "&state="+index+
    "&force_verify=true";

    window.location.href=url;
}

/* AUTH CHECK */
function checkAuth(){

    const params=new URLSearchParams(location.hash.substring(1));

    const token=params.get("access_token");
    const state=params.get("state");

    if(token && state!==null){
        tokens[state]=token;
        window.location.hash="";
    }
}

/* LOAD STREAMS */
function loadStreams(){

    const container=document.getElementById("streams");
    container.innerHTML="";

    for(let i=0;i<6;i++){

        const channel=document.getElementById("channel"+i).value.trim();
        if(!channel) continue;

        slotChannels[i]=channel;

        container.innerHTML+=`
        <div class="slot">
        <iframe
        src="https://player.twitch.tv/?channel=${channel}&parent=${location.hostname}"
        width="100%"
        height="300"
        allowfullscreen>
        </iframe>

        <iframe
        src="https://www.twitch.tv/embed/${channel}/chat?parent=${location.hostname}"
        width="100%"
        height="250">
        </iframe>
        </div>
        `;
    }
}

/* RELAY CHAT */
function sendRelay(){

    const msg=document.getElementById("chatMessage").value;
    if(!msg) return;

    relayClients.forEach(client=>{
        if(!client) return;

        slotChannels.forEach(ch=>{
            if(ch){
                client.say(ch,msg);
            }
        });
    });

    document.getElementById("chatMessage").value="";
}

/* CHANNEL BROWSER ADD */
let nextSlot=0;

function addSearchToSlot(){
    const val=document.getElementById("searchBox").value.trim();
    if(!val) return;

    if(nextSlot>=6) nextSlot=0;

    document.getElementById("channel"+nextSlot).value=val;
    nextSlot++;
}

checkAuth();

</script>

</body>
</html>
