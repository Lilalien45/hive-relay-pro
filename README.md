<!DOCTYPE html>
<html>
<head>
<title>Twitch Hive Relay</title>

<script src="https://unpkg.com/tmi.js@1.8.5/dist/tmi.min.js"></script>

<style>
body{
    background:#0e0e10;
    color:#efeff1;
    font-family:Arial,sans-serif;
    margin:0;
    display:flex;
    flex-direction:column;
    height:100vh;
}

#topbar{
    padding:12px;
    background:#18181b;
    display:flex;
    gap:10px;
    flex-wrap:wrap;
}

button{
    padding:8px 14px;
    background:#9147ff;
    border:none;
    color:white;
    font-weight:bold;
    cursor:pointer;
}

button.off{
    background:#444;
}

#stream-area{
    height:300px;
    background:black;
}

#chat-log{
    flex:1;
    overflow-y:auto;
    padding:15px;
    font-size:13px;
}

#input-area{
    display:flex;
    gap:10px;
    padding:10px;
    background:#18181b;
}

input{
    flex:1;
    padding:10px;
    background:#0e0e10;
    border:1px solid #444;
    color:white;
}
</style>
</head>

<body>

<div id="topbar">
<button id="loginBtn">Login</button>
<button id="chatToggle" class="off">Chat OFF</button>
<button id="viewToggle" class="off">View OFF</button>
<button id="autoJoinToggle" class="off">Auto Join OFF</button>
<span id="status">🔴 Not Logged In</span>
</div>

<div id="stream-area"></div>

<div id="chat-log">Chat disabled.</div>

<div id="input-area">
<input id="messageInput" placeholder="Send message">
<button onclick="sendMessage()">SEND</button>
</div>

<script>

const CONFIG={
    clientId:"cvv666bkohm1zuauw2f5nqcwvi1hx8",
    redirectUri:"http://127.0.0.1:19132/another%20twitch.html"
};

let accessToken=null;
let twitchClient=null;
let channels=[];
let chatEnabled=false;
let autoJoin=false;
let viewEnabled=false;

/* COOKIE */
function saveToken(token){
    document.cookie="twitch_token="+token+"; path=/; max-age=31536000";
}
function getToken(){
    const c=document.cookie.split("; ");
    for(const x of c){
        if(x.startsWith("twitch_token=")) return x.split("=")[1];
    }
    return null;
}

/* LOGIN */
document.getElementById("loginBtn").onclick=()=>{
    const url=
    "https://id.twitch.tv/oauth2/authorize"+
    "?response_type=token"+
    "&client_id="+CONFIG.clientId+
    "&redirect_uri="+encodeURIComponent(CONFIG.redirectUri)+
    "&scope="+encodeURIComponent("chat:read chat:edit user:read:follows")+
    "&force_verify=true";

    window.location.href=url;
};

/* AUTH CHECK */
function checkAuth(){
    const hash=new URLSearchParams(location.hash.substring(1));
    let token=hash.get("access_token") || getToken();

    if(token){
        accessToken=token;
        saveToken(token);
        window.location.hash="";
        document.getElementById("status").innerText="🟢 Logged In";
    }
}

/* CHAT TOGGLE */
document.getElementById("chatToggle").onclick=()=>{
    chatEnabled=!chatEnabled;
    const btn=document.getElementById("chatToggle");

    if(chatEnabled){
        btn.classList.remove("off");
        btn.innerText="Chat ON";
        initChat();
    }else{
        btn.classList.add("off");
        btn.innerText="Chat OFF";
        if(twitchClient) twitchClient.disconnect();
        document.getElementById("chat-log").innerHTML="Chat disabled.";
    }
};

/* VIEW TOGGLE */
document.getElementById("viewToggle").onclick=()=>{
    viewEnabled=!viewEnabled;
    const btn=document.getElementById("viewToggle");

    if(viewEnabled){
        btn.classList.remove("off");
        btn.innerText="View ON";
        if(channels.length>0){
            loadStreamEmbed(channels[0]);
        }
    }else{
        btn.classList.add("off");
        btn.innerText="View OFF";
        document.getElementById("stream-area").innerHTML="";
    }
};

/* AUTO JOIN */
document.getElementById("autoJoinToggle").onclick=()=>{
    autoJoin=!autoJoin;
    const btn=document.getElementById("autoJoinToggle");

    if(autoJoin){
        btn.classList.remove("off");
        btn.innerText="Auto Join ON";
    }else{
        btn.classList.add("off");
        btn.innerText="Auto Join OFF";
    }
};

/* INIT CHAT */
async function initChat(){

    if(!accessToken){
        alert("Login first.");
        return;
    }

    const userRes=await fetch("https://api.twitch.tv/helix/users",{
        headers:{
            "Client-ID":CONFIG.clientId,
            "Authorization":"Bearer "+accessToken
        }
    });

    const userData=await userRes.json();
    const username=userData.data[0].login;

    if(autoJoin){
        const followRes=await fetch(
            "https://api.twitch.tv/helix/streams/followed?first=10",
            {
                headers:{
                    "Client-ID":CONFIG.clientId,
                    "Authorization":"Bearer "+accessToken
                }
            }
        );
        const followData=await followRes.json();
        channels=followData.data.map(x=>x.user_login);
    }else{
        channels=[username];
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
        const log=document.getElementById("chat-log");
        const div=document.createElement("div");
        div.innerHTML="<b>["+channel+"] "+tags["display-name"]+":</b> "+msg;
        log.appendChild(div);
        log.scrollTop=log.scrollHeight;
    });

    if(viewEnabled){
        loadStreamEmbed(channels[0]);
    }
}

/* STREAM EMBED */
function loadStreamEmbed(channel){
    document.getElementById("stream-area").innerHTML=
    `<iframe
        src="https://player.twitch.tv/?channel=${channel}&parent=127.0.0.1"
        height="100%"
        width="100%"
        frameborder="0"
        allowfullscreen>
     </iframe>`;
}

/* SEND */
function sendMessage(){
    const input=document.getElementById("messageInput");
    if(!input.value || !twitchClient) return;

    channels.forEach(c=>{
        twitchClient.say(c,input.value);
    });

    input.value="";
}

document.getElementById("messageInput").onkeydown=e=>{
    if(e.key==="Enter") sendMessage();
};

checkAuth();

</script>

</body>
</html>
