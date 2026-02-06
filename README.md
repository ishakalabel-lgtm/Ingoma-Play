<!DOCTYPE html>
<html lang="rn">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>INGOMA PLAY</title>
    <style>
        :root { --primary:#ffcc00; --bg:#000; --card:#111 }
        body{margin:0;font-family:Segoe UI,Tahoma;background:var(--bg);color:#fff}
        header{background:var(--primary);color:#000;padding:15px;text-align:center;position:sticky;top:0;z-index:9}
        header input{width:90%;padding:12px;border-radius:25px;border:none;margin-top:10px}
        nav{display:flex;background:#111}
        nav button{flex:1;padding:15px;background:none;border:none;color:#aaa;font-weight:bold}
        nav button.active-nav{color:var(--primary);border-bottom:3px solid var(--primary)}
        section{display:none;padding:15px}
        section.active{display:block}
        .card{background:var(--card);padding:15px;margin-bottom:15px;border-radius:12px}
        button{padding:12px;border:none;border-radius:8px;background:var(--primary);font-weight:bold;width:100%}
        input{width:100%;padding:12px;margin-top:8px;border-radius:8px;border:1px solid #333;background:#222;color:#fff}
        img{width:100%;border-radius:10px}
        audio{width:100%;margin-top:10px}
        .hidden{display:none}
    </style>
</head>
<body>

<header>
    <h2>INGOMA PLAY</h2>
    <small>Ijwi ryawe n’itunga ryawe</small><br>
    <input id="searchBox" placeholder="🔍 Rondera indirimbo..." oninput="searchSongs()">
</header>

<nav>
    <button class="active-nav" onclick="openTab('home',this)">HOME</button>
    <button onclick="openTab('artist',this)">UMURIRIMVYI</button>
    <button onclick="openTab('admin',this)">ADMIN</button>
</nav>

<section id="home" class="active">
    <div id="homeSongs"></div>
</section>

<section id="artist">
    <div id="artistAuth" class="card">
        <button onclick="showArtist('new')">YANDIKISHA</button>
        <button onclick="showArtist('old')" style="background:#333;color:#fff;margin-top:5px">INJIRA</button>
    </div>

    <div id="artistNew" class="card hidden">
        <input id="realName" placeholder="Izina ryawe">
        <input id="stageName" placeholder="Izina ry'ubuhanzi">
        <input id="phone" placeholder="Telefone">
        <input id="pin" type="password" placeholder="PIN">
        <button onclick="registerArtist()">EMEZA</button>
    </div>

    <div id="artistOld" class="card hidden">
        <input id="loginPhone" placeholder="Telefone">
        <input id="loginPin" type="password" placeholder="PIN">
        <button onclick="loginArtist()">INJIRA</button>
    </div>
</section>

<section id="admin">
    <div class="card">
        <input id="adminPin" type="password" placeholder="ADMIN PIN">
        <button onclick="adminLogin()">INJIRA</button>
    </div>
    <div id="adminPanel" class="hidden"></div>
</section>

<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
const supabase = supabaseJs.createClient(
 "https://fmrjmanvrydfmpiiaoap.supabase.co",
 "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImZtcmptYW52cnlkZm1waWlhb2FwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3Njk1ODczNzgsImV4cCI6MjA4NTE2MzM3OH0.hKz4cDWEfyb7yubBbY3OKQPzh_GnodmKfVvm_5NNoiY"
);

let allSongs=[];

function openTab(id,btn){
 document.querySelectorAll("section").forEach(s=>s.classList.remove("active"));
 document.querySelectorAll("nav button").forEach(b=>b.classList.remove("active-nav"));
 document.getElementById(id).classList.add("active");
 btn.classList.add("active-nav");
}

async function loadHome(){
 const {data}=await supabase.from("songs").select("*").eq("status","approved");
 allSongs=data||[];
 render(allSongs);
}
function render(list){
 document.getElementById("homeSongs").innerHTML=list.map(s=>`
  <div class="card">
    <img src="${s.cover_url}">
    <b>${s.title}</b>
    <audio controls src="${s.audio_url}"></audio>
  </div>`).join("");
}
function searchSongs(){
 const q=searchBox.value.toLowerCase();
 render(allSongs.filter(s=>s.title.toLowerCase().includes(q)));
}

function showArtist(t){
 artistAuth.classList.add("hidden");
 document.getElementById(t==="new"?"artistNew":"artistOld").classList.remove("hidden");
}
async function registerArtist(){
 await supabase.from("artists").insert({
  real_name:realName.value,
  stage_name:stageName.value,
  phone:phone.value,
  pin:pin.value
 });
 alert("Wiyandikishije!");
 location.reload();
}
async function loginArtist(){
 const {data}=await supabase.from("artists")
 .select("*").eq("phone",loginPhone.value).eq("pin",loginPin.value).single();
 if(!data) return alert("PIN canke telefone siyo");
 alert("Murakaze "+data.stage_name);
}

function adminLogin(){
 if(adminPin.value==="01010101"){
  adminPanel.classList.remove("hidden");
  loadAdmin();
 }
}
async function loadAdmin(){
 const {data}=await supabase.from("songs").select("*").eq("status","pending");
 adminPanel.innerHTML=data.map(s=>`
  <div class="card">
   ${s.title}
   <button onclick="approve(${s.id})">EMERA</button>
  </div>`).join("");
}
async function approve(id){
 await supabase.from("songs").update({status:"approved"}).eq("id",id);
 loadAdmin();loadHome();
}

loadHome();
</script>
</body>
</html>
