<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>NeoTune YouTube</title>

<script src="https://apis.google.com/js/api.js"></script>

<style>
body {
    margin:0;
    font-family:Arial;
    background:#121212;
    color:white;
}

header {
    padding:15px;
    font-size:20px;
    font-weight:bold;
}

button {
    padding:10px;
    margin:10px;
    background:#1db954;
    border:none;
    color:white;
    border-radius:5px;
}

.playlist {
    margin:10px;
    padding:10px;
    background:#1f1f1f;
    border-radius:10px;
}

#user {
    padding:10px;
    color:#aaa;
}
</style>
</head>

<body>

<header>NeoTune</header>

<div id="user">Não logado</div>

<button onclick="login()">Login com Google</button>
<button onclick="logout()">Sair</button>
<button onclick="carregar()">Minhas Playlists</button>

<div id="playlists"></div>

<script>
const CLIENT_ID = "SEU_CLIENT_ID";
const API_KEY = "SUA_API_KEY";

const DISCOVERY = "https://www.googleapis.com/discovery/v1/apis/youtube/v3/rest";
const SCOPES = "https://www.googleapis.com/auth/youtube.readonly";

function init(){
    gapi.load("client:auth2", async () => {
        await gapi.client.init({
            apiKey: API_KEY,
            clientId: CLIENT_ID,
            discoveryDocs: [DISCOVERY],
            scope: SCOPES
        });

        if(gapi.auth2.getAuthInstance().isSignedIn.get()){
            mostrarUsuario();
        }
    });
}

function login(){
    gapi.auth2.getAuthInstance().signIn().then(mostrarUsuario);
}

function logout(){
    gapi.auth2.getAuthInstance().signOut();
    document.getElementById("user").innerText="Não logado";
}

function mostrarUsuario(){
    const user = gapi.auth2.getAuthInstance().currentUser.get();
    const profile = user.getBasicProfile();
    document.getElementById("user").innerText =
        "Logado: " + profile.getName();
}

function carregar(){
    gapi.client.youtube.playlists.list({
        part:"snippet",
        mine:true,
        maxResults:10
    }).then(res => {
        const div = document.getElementById("playlists");
        div.innerHTML="";

        res.result.items.forEach(p=>{
            const el = document.createElement("div");
            el.className="playlist";
            el.innerHTML = `<b>${p.snippet.title}</b>`;
            div.appendChild(el);
        });
    });
}

init();
</script>

</body>
</html>