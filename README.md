<!DOCTYPE html>
<html lang="rw">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>INGOMA PLAY</title>

<style>
body{margin:0;font-family:Arial;background:#000;color:#fff}
header{background:#ffcc00;color:#000;padding:14px;text-align:center}
header input{width:92%;margin-top:10px;padding:10px;border-radius:20px;border:none}

nav{display:flex;background:#111}
nav button{flex:1;padding:14px;background:none;border:none;color:#fff}

section{display:none;padding:15px}
section.active{display:block}

.card{background:#111;padding:14px;margin-bottom:14px;border-radius:10px}
button{padding:10px;border:none;border-radius:6px;background:#ffcc00;font-weight:bold;margin-top:8px;width:100%}
input,select{width:100%;padding:10px;margin-top:6px;border-radius:6px;border:none}

.hidden{display:none}
img,audio{width:100%;border-radius:10px;margin-top:8px}
small{color:#aaa}
hr{border:1px solid #222}
</style>
</head>

<body>

<header>
<h2>INGOMA PLAY</h2>
<small>Ijwi ryawe n’itunga ryawe</small>
<input id="searchBox" placeholder="🔍 Rondera indirimbo" oninput="searchSongs()">
</header>

<nav>
<button onclick="openTab('home')">HOME</button>
<button onclick="openTab('artist')">UMURIRIMVYI</button>
<button onclick="openTab('admin')">ADMIN</button>
</nav>

<!-- HOME -->
<section id="home" class="active">
<div class="card">
<h3>🔥 ZIHOTSE</h3>
<div id="trending"></div>
</div>

<div class="card">
<h3>🆕 INDIRIMBO NSHASHA</h3>
<div id="newSongs"></div>
</div>

<div class="card">
<h3>🎧 INDIRIMBO ZOSE</h3>
<div id="homeSongs"></div>
</div>
</section>

<!-- ARTIST -->
<section id="artist">
<div class="card">
<button onclick="showArtist('new')">URI MUSHASHA</button>
<button onclick="showArtist('old')">URUMUSANGWA</button>
</div>

<div id="artistNew" class="card hidden">
<h3>KWISIGA</h3>
<input id="realName" placeholder="Izina nyakuri">
<input id="stageName" placeholder="Izina ry'ubuhanzi">
<input id="phone" placeholder="Numero ya terefone">
<input id="pin" placeholder="PIN">
<button onclick="registerArtist()">KOMEZA</button>
</div>

<div id="artistOld" class="card hidden">
<h3>KWINJIRA</h3>
<input id="loginPhone" placeholder="Numero ya terefone">
<input id="loginPin" placeholder="PIN">
<button onclick="loginArtist()">INJIRA</button>
</div>

<div id="channel" class="card hidden">
<h3>CHANNEL</h3>
<p>Balance: <b id="balance">0</b> FBU</p>
<p>Uploads remaining: <b id="uploads">0</b></p>

<button onclick="openPayment()">GUSHIRAKO INDIRIMBO</button>

<div id="paymentBox" class="hidden">
<hr>
<p><b>UPLOAD 1 = <span id="price">5000</span> FBU</b></p>
<button onclick="payWhatsApp()">RIHA</button>
<button onclick="showPinBox()">WARISHE</button>
</div>

<div id="pinBox" class="hidden">
<input id="paymentPin" placeholder="SHIRAMWO PIN">
<button onclick="verifyPin()">EMEZA PIN</button>
</div>

<div id="uploadBox" class="hidden">
<hr>
<h4>UPLOAD INDIMBO</h4>
<input id="songTitle" placeholder="Izina ry'indirimbo">
<input type="file" id="songFile" accept="audio/*">
<input type="file" id="coverFile" accept="image/*">
<button onclick="uploadSong()">UPLOAD</button>
<small id="uploadMsg"></small>
</div>
</div>
</section>

<!-- ADMIN -->
<section id="admin">
<div id="adminLoginBox" class="card">
<input id="adminPin" placeholder="ADMIN PIN">
<button onclick="adminLogin()">INJIRA</button>
</div>

<div id="adminPanel" class="hidden">

<div class="card">
<h3>⚙ SETTINGS</h3>
<input id="newPrice" placeholder="Igiciro ca upload">
<button onclick="updatePrice()">SAVE PRICE</button>
</div>

<div class="card">
<h3>🔐 CREATE PIN</h3>
<button onclick="createPin()">KORA PIN</button>
<div id="pinList"></div>
</div>

<div class="card">
<h3>🎵 INDIRIMBO ZOSE</h3>
<div id="songsAdmin"></div>
</div>

<div class="card">
<h3>👥 ARTISTES</h3>
<div id="artistsAdmin"></div>
</div>

</div>
</section>

<script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>
<script>
const supabase = supabaseJs.createClient(
 "https://fmrjmanvrydfmpiiaoap.supabase.co",
 "YOUR_ANON_KEY_HERE"
);

let currentArtist=null, approvedSongs=[], uploadPrice=5000;

function openTab(id){
 document.querySelectorAll("section").forEach(s=>s.classList.remove("active"));
 document.getElementById(id).classList.add("active");
}

function showArtist(t){
 artistNew.classList.add("hidden"); artistOld.classList.add("hidden");
 document.getElementById(t==="new"?"artistNew":"artistOld").classList.remove("hidden");
}

async function registerArtist(){
 await supabase.from("artists").insert([{
  real_name:realName.value,
  stage_name:stageName.value,
  phone:phone.value,
  pin:pin.value,
  balance:0,
  uploads_remaining:0
 }]);
 alert("Vyashobotse");
}

async function loginArtist(){
 const {data}=await supabase.from("artists")
 .select("*").eq("phone",loginPhone.value).eq("pin",loginPin.value).single();
 if(!data) return alert("Ntivyemewe");
 currentArtist=data;
 balance.innerText=data.balance;
 uploads.innerText=data.uploads_remaining;
 channel.classList.remove("hidden");
}

function openPayment(){ paymentBox.classList.remove("hidden"); }

function payWhatsApp(){
 window.open(`https://wa.me/25766429717?text=Muraho Admin, ndashaka kuriha ${uploadPrice}FBU zo gushirako indirimbo. Artist:${currentArtist.stage_name}`);
}

function showPinBox(){ pinBox.classList.remove("hidden"); }

async function verifyPin(){
 const {data}=await supabase.from("payment_pins")
 .select("*").eq("pin",paymentPin.value).eq("used",false).single();
 if(!data) return alert("PIN ntiyemewe");

 await supabase.from("payment_pins").update({used:true,artist_id:currentArtist.id}).eq("id",data.id);
 await supabase.from("artists").update({uploads_remaining:1}).eq("id",currentArtist.id);

 uploads.innerText=1;
 uploadBox.classList.remove("hidden");
 alert("PIN yemewe");
}

async function uploadSong(){
 if(currentArtist.uploads_remaining<1) return;
 const a=songFile.files[0], c=coverFile.files[0];
 const af=Date.now()+"_"+a.name, cf=Date.now()+"_"+c.name;

 await supabase.storage.from("songs").upload(af,a);
 await supabase.storage.from("covers").upload(cf,c);

 const au=supabase.storage.from("songs").getPublicUrl(af).data.publicUrl;
 const cu=supabase.storage.from("covers").getPublicUrl(cf).data.publicUrl;

 await supabase.from("songs").insert([{
  title:songTitle.value,
  artist_id:currentArtist.id,
  audio_url:au,
  cover_url:cu,
  status:"pending",
  views:0
 }]);

 await supabase.from("artists").update({uploads_remaining:0}).eq("id",currentArtist.id);
 uploads.innerText=0;
 uploadMsg.innerText="Irindiriye kwemerwa";
}

function adminLogin(){
 if(adminPin.value==="01010101"){
  adminLoginBox.classList.add("hidden");
  adminPanel.classList.remove("hidden");
  loadAdmin();
 }
}

async function loadAdmin(){
 const {data:songs}=await supabase.from("songs").select("*");
 songsAdmin.innerHTML=songs.map(s=>`
  <div class="card">${s.title}
  <button onclick="approve(${s.id})">EMERA</button>
  <button onclick="hideSong(${s.id})">HIDE</button>
  </div>`).join("");

 const {data:arts}=await supabase.from("artists").select("*");
 artistsAdmin.innerHTML=arts.map(a=>`
  <div class="card">${a.stage_name} - ${a.phone}</div>`).join("");
}

async function approve(id){
 await supabase.from("songs").update({status:"approved"}).eq("id",id);
 loadAdmin();
}
async function hideSong(id){
 await supabase.from("songs").update({status:"hidden"}).eq("id",id);
 loadAdmin();
}

async function createPin(){
 const pin=Math.floor(100000+Math.random()*900000);
 await supabase.from("payment_pins").insert([{pin,used:false}]);
 alert("PIN yakozwe: "+pin);
}

async function updatePrice(){
 uploadPrice=parseInt(newPrice.value);
 await supabase.from("settings").upsert([{key:"upload_price",value:uploadPrice}]);
 alert("Igiciro cahindutse");
}
</script>

</body>
</html>
