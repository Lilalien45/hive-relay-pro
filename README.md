<!DOCTYPE html>
<html>
<head>
<title>Hive Relay Separate Apps</title>
<script src="https://unpkg.com/tmi.js@1.8.5/dist/tmi.min.js"></script>

<style>
body{
    background:#0e0e10;
    color:#efeff1;
    font-family:Arial,sans-serif;
    margin:0;
    padding:20px;
}
.app{
    border:1px solid #333;
    padding:15px;
    margin-bottom:20px;
}
button{
    padding:6px 10px;
    margin:4px 4px 4px 0;
    background:#9147ff;
    border:none;
    color:white;
    cursor:pointer;
}
button.off{ background:#444; }

.stream{
    height:200px;
    background:black;
    margin-top:10px;
}

.chat{
    height:150px;
    overflow-y:auto;
    background:#111;
    padding:8px;
    font-size:12px;
    margin-top:10px;
}
input{
    padding:6px;
    background:#0e0e10;
    border:1px solid #444;
    color:white;
}
</style>
</head>
<body>

<h2>Hive Relay – Separate Apps</h2>

<div id="apps"></div>

<script>

const CLIENT_IDS=[
"cvv666bkohm1zuauw2f5nqcwvi1hx8",
"8etzpqt1zgsldyz25h7t3f23r4qng8",
"4r4n0hx6o7ff9ho3tdiaambnx2lw56"
];

const REDIRECT="https://lilalien45.github.io/hive-relay-pro/";

let appState={};

/* CREATE UI */
const container=document.getElementById("apps");

CLIENT_IDS.forEach((id,index)=>{

    appState[id]={
        token:null,
        client:null,
        channels:[],
        chatEnabled:false,
        viewEnabled:false,
        autoJoin:false
    };

    container.innerHTML+=`
    <div class="app" id="app_${id}">
        <h3>App ${index+1}</h3>
        <div>Status: <span id="status_${id}">🔴 Not Logged In</span></div>
        <button onclick="login('${id}')">Login</button>
        <button id="chat_${id}" class="off" onclick="toggleChat('${id}')">Chat OFF</button>
        <button id="view_${id}" class="off" onclick="toggleView('${id}')">View OFF</button>
        <button id="auto_${id}" class="off" onclick="toggleAuto('${id}')">Auto Join OFF</button>

        <div class="stream" id="stream_${id}"></div>
        <div class="chat" id="chatlog_${id}">Chat disabled.</div>

        <input id="input_${id}" placeholder="Message">
        <button onclick="sendMsg('${id}')">SEND</button>
    </div>
    `;
});

/* LOGIN */
function login(clientId){

    const url=
    "https://id.twitch.tv/oauth2/authorize"+
    "?response_type=token"+
    "&client_id="+clientId+
    "&redirect_uri="+encodeURIComponent(REDIRECT)+
    "&scope="+encodeURIComponent("chat:read chat:edit user:read:follows")+
    "&force_verify=true"+
    "&state="+clientId;

    window.location.href=url;
}

/* CHECK AUTH */
function checkAuth(){

    const hash=new URLSearchParams(location.hash.substring(1));
    const token=hash.get("access_token");
    const state=hash.get("state");

    if(token && state){
        localStorage.setItem("twitch_token_"+state,token);
        window.location.hash="";
    }

    CLIENT_IDS.forEach(id=>{
        const stored=localStorage.getItem("twitch_token_"+id);
        if(stored){
            appState[id].token=stored;
            document.getElementById("status_"+id).innerText="🟢 Logged In";
        }
    });
}

/* TOGGLES */
function toggleChat(id){
    const state=appState[id];
    state.chatEnabled=!state.chatEnabled;
    const btn=document.getElementById("chat_"+id);

    if(state.chatEnabled){
        btn.classList.remove("off");
        btn.innerText="Chat ON";
        initChat(id);
    }else{
        btn.classList.add("off");
        btn.innerText="Chat OFF";
        if(state.client) state.client.disconnect();
        document.getElementById("chatlog_"+id).innerHTML="Chat disabled.";
    }
}

function toggleView(id){
    const state=appState[id];
    state.viewEnabled=!state.viewEnabled;
    const btn=document.getElementById("view_"+id);

    if(state.viewEnabled){
        btn.classList.remove("off");
        btn.innerText="View ON";
        if(state.channels.length>0){
            loadStream(id,state.channels[0]);
        }
    }else{
        btn.classList.add("off");
        btn.innerText="View OFF";
        document.getElementById("stream_"+id).innerHTML="";
    }
}

function toggleAuto(id){
    const state=appState[id];
    state.autoJoin=!state.autoJoin;
    const btn=document.getElementById("auto_"+id);

    if(state.autoJoin){
        btn.classList.remove("off");
        btn.innerText="Auto Join ON";
    }else{
        btn.classList.add("off");
        btn.innerText="Auto Join OFF";
    }
}

/* INIT CHAT */
async function initChat(id){

    const state=appState[id];
    if(!state.token) return alert("Login first");

    const userRes=await fetch("https://api.twitch.tv/helix/users",{
        headers:{
            "Client-ID":id,
            "Authorization":"Bearer "+state.token
        }
    });

    const userData=await userRes.json();
    const username=userData.data[0].login;

    state.channels=[username];

    const client=new tmi.Client({
        connection:{secure:true,reconnect:true},
        identity:{
            username:username,
            password:"oauth:"+state.token
        },
        channels:state.channels
    });

    await client.connect();

    client.on("message",(channel,tags,msg)=>{
        const log=document.getElementById("chatlog_"+id);
        const div=document.createElement("div");
        div.innerHTML="<b>"+tags["display-name"]+":</b> "+msg;
        log.appendChild(div);
        log.scrollTop=log.scrollHeight;
    });

    state.client=client;

    if(state.viewEnabled){
        loadStream(id,state.channels[0]);
    }
}

/* STREAM */
function loadStream(id,channel){
    document.getElementById("stream_"+id).innerHTML=
    `<iframe
        src="https://player.twitch.tv/?channel=${channel}&parent=lilalien45.github.io"
        height="100%"
        width="100%"
        frameborder="0"
        allowfullscreen>
    </iframe>`;
}

/* SEND */
function sendMsg(id){
    const state=appState[id];
    const input=document.getElementById("input_"+id);
    if(!input.value || !state.client) return;

    state.channels.forEach(c=>{
        state.client.say(c,input.value);
    });

    input.value="";
}

checkAuth();

</script>
</body>
</html>    const url=
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


<!DOCTYPE html>
<html>
<head>
<title>Hive Relay Ultimate</title>
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
    gap:8px;
    flex-wrap:wrap;
}
button{
    padding:8px 12px;
    background:#9147ff;
    border:none;
    color:white;
    font-weight:bold;
    cursor:pointer;
}
button.off{ background:#444; }

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
<button onclick="loginAll()">Login All 3</button>
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

const APPS = [
"cvv666bkohm1zuauw2f5nqcwvi1hx8",
"8etzpqt1zgsldyz25h7t3f23r4qng8",
"4r4n0hx6o7ff9ho3tdiaambnx2lw56"
];

const REDIRECT="https://lilalien45.github.io/hive-relay-pro/";

let activeAccounts=[];
let twitchClients=[];
let channels=[];
let chatEnabled=false;
let viewEnabled=false;
let autoJoin=false;

/* LOGIN ALL */
function loginAll(){

    APPS.forEach(clientId=>{

        const url=
        "https://id.twitch.tv/oauth2/authorize"+
        "?response_type=token"+
        "&client_id="+clientId+
        "&redirect_uri="+encodeURIComponent(REDIRECT)+
        "&scope="+encodeURIComponent("chat:read chat:edit user:read:follows")+
        "&force_verify=true"+
        "&state="+clientId;

        window.open(url,"_blank");
    });
}

/* SAVE TOKEN */
function saveToken(clientId,token){
    localStorage.setItem("twitch_token_"+clientId,token);
}

/* CHECK AUTH */
function checkAuth(){

    const hash=new URLSearchParams(location.hash.substring(1));
    const token=hash.get("access_token");
    const state=hash.get("state");

    if(token && state){
        saveToken(state,token);
        window.location.hash="";
    }

    APPS.forEach(id=>{
        const stored=localStorage.getItem("twitch_token_"+id);
        if(stored){
            activeAccounts.push({
                clientId:id,
                token:stored
            });
        }
    });

    if(activeAccounts.length>0){
        document.getElementById("status").innerText=
        "🟢 Logged In ("+activeAccounts.length+" accounts)";
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
        twitchClients.forEach(c=>c.disconnect());
        twitchClients=[];
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
            loadStream(channels[0]);
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

    for(const acc of activeAccounts){

        const userRes=await fetch("https://api.twitch.tv/helix/users",{
            headers:{
                "Client-ID":acc.clientId,
                "Authorization":"Bearer "+acc.token
            }
        });

        const userData=await userRes.json();
        const username=userData.data[0].login;

        if(autoJoin){
            const followRes=await fetch(
            "https://api.twitch.tv/helix/streams/followed?first=5",
            {
                headers:{
                    "Client-ID":acc.clientId,
                    "Authorization":"Bearer "+acc.token
                }
            });
            const followData=await followRes.json();
            channels=followData.data.map(x=>x.user_login);
        }else{
            channels=[username];
        }

        const client=new tmi.Client({
            connection:{secure:true,reconnect:true},
            identity:{
                username:username,
                password:"oauth:"+acc.token
            },
            channels:channels
        });

        await client.connect();

        client.on("message",(channel,tags,msg)=>{
            const log=document.getElementById("chat-log");
            const div=document.createElement("div");
            div.innerHTML="<b>["+channel+"] "+tags["display-name"]+":</b> "+msg;
            log.appendChild(div);
            log.scrollTop=log.scrollHeight;
        });

        twitchClients.push(client);
    }

    if(viewEnabled && channels.length>0){
        loadStream(channels[0]);
    }
}

/* STREAM */
function loadStream(channel){
    document.getElementById("stream-area").innerHTML=
    `<iframe
        src="https://player.twitch.tv/?channel=${channel}&parent=lilalien45.github.io"
        height="100%"
        width="100%"
        frameborder="0"
        allowfullscreen>
     </iframe>`;
}

/* SEND */
function sendMessage(){
    const input=document.getElementById("messageInput");
    if(!input.value) return;

    twitchClients.forEach(client=>{
        channels.forEach(c=>{
            client.say(c,input.value);
        });
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
