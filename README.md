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
    display:flex;
    flex-direction:column;
    height:100vh;
}

#header{
    padding:15px;
    background:#18181b;
    border-bottom:2px solid #303032;
    display:flex;
    flex-wrap:wrap;
    gap:15px;
    align-items:center;
}

#chat-log{
    flex:1;
    overflow-y:auto;
    padding:20px;
    font-size:13px;
}

#input-area{
    padding:20px;
    background:#18181b;
    display:flex;
    gap:10px;
}

input{
    flex:1;
    padding:15px;
    background:#0e0e10;
    border:1px solid #464649;
    color:white;
    border-radius:4px;
}

button{
    padding:12px 18px;
    background:#9147ff;
    border:none;
    color:white;
    font-weight:bold;
    border-radius:4px;
    cursor:pointer;
}

button:hover{
    background:#772ce8;
}

.msg{ margin-bottom:6px; }
.chan{ color:#bf94ff; font-weight:bold; margin-right:6px; }
</style>
</head>

<body>

<div id="header">

<div id="status">🔴 Not Logged In</div>

<button id="login-btn">Login Twitch</button>

<div>
<input type="checkbox" id="auto-join">
<label>Auto Join Followed Channels</label>
</div>

<div>
<input type="checkbox" id="enable-chat">
<label>Enable Chat</label>
</div>

</div>

<div id="chat-log">Chat will appear here...</div>

<div id="input-area">
<input id="mass-input" placeholder="Send message">
<button onclick="broadcast()">SEND</button>
</div>

<script>

const CONFIG={
    clientId:"cvv666bkohm1zuauw2f5nqcwvi1hx8",
    redirectUri:"http://127.0.0.1:19132"
};

let accessToken=null;
let twitchClient=null;
let channels=[];
let relayQueue=[];
let relayCooldown=false;

/* ---------- COOKIE STORAGE ---------- */

function saveToken(token){
    document.cookie=
        "twitch_token="+token+
        "; path=/; max-age=31536000";
}

function getTokenFromCookie(){
    const cookies=document.cookie.split("; ");

    for(const c of cookies){
        if(c.startsWith("twitch_token=")){
            return c.split("=")[1];
        }
    }
    return null;
}

/* ---------- LOGIN ---------- */

document.getElementById("login-btn").onclick=()=>{

    const auth=
    "https://id.twitch.tv/oauth2/authorize"+
    "?response_type=token"+
    "&client_id="+CONFIG.clientId+
    "&redirect_uri="+encodeURIComponent(CONFIG.redirectUri)+
    "&scope="+encodeURIComponent(
        "chat:read chat:edit user:read:follows"
    )+
    "&force_verify=true";

    window.location.href=auth;
};

/* ---------- AUTH CHECK ---------- */

function checkAuth(){

    const params=new URLSearchParams(
        location.hash.substring(1)
    );

    let token=params.get("access_token") || getTokenFromCookie();

    if(token){
        accessToken=token;
        saveToken(token);

        document.getElementById("login-btn").style.display="none";
        document.getElementById("status").innerText="🟢 Logged In";
    }
}

/* ---------- RELAY ---------- */

async function initRelay(){

    if(!accessToken) return;

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
        const username=userData.data[0].login;

        if(document.getElementById("auto-join").checked){

            const followRes=await fetch(
                "https://api.twitch.tv/helix/streams/followed?first=25",
                {
                    headers:{
                        "Client-ID":CONFIG.clientId,
                        "Authorization":"Bearer "+accessToken
                    }
                }
            );

            const followData=await followRes.json();
            channels=followData.data.map(x=>x.user_login);
        }

        twitchClient=new tmi.Client({
            identity:{
                username:username,
                password:"oauth:"+accessToken
            },
            channels:channels
        });

        await twitchClient.connect();

        twitchClient.on("message",(channel,tags,msg)=>{

            const log=document.getElementById("chat-log");

            const div=document.createElement("div");
            div.className="msg";

            div.innerHTML=
            `<span class="chan">[${channel}]</span>
             <b>${tags["display-name"]}</b>: ${msg}`;

            log.appendChild(div);
            log.scrollTop=log.scrollHeight;
        });

    }catch(err){
        console.error(err);
    }
}

/* ---------- CHAT TOGGLE ---------- */

document.getElementById("enable-chat").onchange=async e=>{

    if(e.target.checked){

        if(!accessToken){
            alert("Login first");
            e.target.checked=false;
            return;
        }

        await initRelay();
        document.getElementById("status").innerText="🟢 Chat Enabled";

    }else{

        if(twitchClient){
            await twitchClient.disconnect().catch(()=>{});
            twitchClient=null;
        }

        document.getElementById("status").innerText="🟠 Chat Disabled";
    }
};

/* ---------- QUEUE ---------- */

function processQueue(){

    if(relayCooldown || relayQueue.length===0) return;

    relayCooldown=true;

    const text=relayQueue.shift();

    if(twitchClient && channels.length>0){

        channels.forEach((chan,i)=>{
            setTimeout(()=>{
                twitchClient.say(chan,text);
            },i*600);
        });
    }

    setTimeout(()=>relayCooldown=false,2000);
}

setInterval(processQueue,400);

/* ---------- SEND ---------- */

function broadcast(){

    const text=document.getElementById("mass-input").value;
    if(!text) return;

    relayQueue.push(text);
    document.getElementById("mass-input").value="";
}

document.getElementById("mass-input").onkeydown=e=>{
    if(e.key==="Enter") broadcast();
};

/* ---------- START ---------- */

    /* visit the website */
checkAuth();

</script>
</body>
</html>

