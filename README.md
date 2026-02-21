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
</html>    "https://id.twitch.tv/oauth2/authorize"+
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
