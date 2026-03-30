<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>أوقات الصلاة + القبلة + الخريطة</title>

<link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css"/>
<script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>

<style>
body{
  margin:0;
  font-family:tahoma;
  background:linear-gradient(#0f2027,#203a43,#2c5364);
  color:white;
  text-align:center;
  direction:rtl;
}
.container{max-width:900px;margin:auto;padding:10px}
input,button{
  padding:10px;border:none;border-radius:8px;margin:5px;font-size:16px
}
button{background:#ffffff25;color:white;cursor:pointer}
.card{
  background:#ffffff15;
  border:1px solid #ffffff35;
  border-radius:12px;
  padding:10px;margin:10px 0
}
#map{height:300px;border-radius:12px;margin-top:10px}
#qibla{font-size:70px;transition:1s}
</style>
</head>
<body>

<div class="container">
  <h2>🕌 أوقات الصلاة واتجاه القبلة</h2>

  <div class="card">
    <input id="city" placeholder="اكتب اسم المدينة بالإنجليزية (مثال: Setif)">
    <input id="country" placeholder="الدولة (مثال: Algeria)">
    <button onclick="loadData()">عرض</button>
  </div>

  <div class="card" id="times"></div>

  <div class="card">
    <div>اتجاه القبلة</div>
    <div id="qibla">⬆️</div>
    <div id="deg"></div>
  </div>

  <div id="map"></div>
</div>

<script>
// خريطة
var map = L.map('map').setView([20,0], 2);
L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',{
  attribution:'© OpenStreetMap'
}).addTo(map);
var marker;

// حساب اتجاه القبلة
function qiblaDirection(lat,lng){
  const kaabaLat=21.4225;
  const kaabaLng=39.8262;
  const toRad=d=>d*Math.PI/180;
  const toDeg=r=>r*180/Math.PI;

  const dLng=toRad(kaabaLng-lng);
  lat=toRad(lat);
  const kLat=toRad(kaabaLat);

  const y=Math.sin(dLng);
  const x=Math.cos(lat)*Math.tan(kLat)-Math.sin(lat)*Math.cos(dLng);
  let brng=toDeg(Math.atan2(y,x));
  return (brng+360)%360;
}

async function loadData(){
  const city=document.getElementById("city").value;
  const country=document.getElementById("country").value;

  // إحداثيات المدينة
  const geo=await fetch(`https://nominatim.openstreetmap.org/search?city=${city}&country=${country}&format=json`);
  const geoData=await geo.json();
  if(!geoData.length){ alert("المدينة غير موجودة"); return; }

  const lat=parseFloat(geoData[0].lat);
  const lon=parseFloat(geoData[0].lon);

  // تحديث الخريطة
  if(marker) map.removeLayer(marker);
  map.setView([lat,lon],10);
  marker=L.marker([lat,lon]).addTo(map);

  // أوقات الصلاة من API رسمي
  const today=new Date();
  const d=String(today.getDate()).padStart(2,"0");
  const m=String(today.getMonth()+1).padStart(2,"0");
  const y=today.getFullYear();

  const url=`https://api.aladhan.com/v1/timingsByCity/${d}-${m}-${y}?city=${city}&country=${country}&method=2`;
  const res=await fetch(url);
  const data=await res.json();
  const t=data.data.timings;

  document.getElementById("times").innerHTML=`
    <h3>${city}</h3>
    الفجر: ${t.Fajr}<br>
    الظهر: ${t.Dhuhr}<br>
    العصر: ${t.Asr}<br>
    المغرب: ${t.Maghrib}<br>
    العشاء: ${t.Isha}
  `;

  // القبلة
  const q=qiblaDirection(lat,lon);
  document.getElementById("qibla").style.transform=`rotate(${q}deg)`;
  document.getElementById("deg").innerText=q.toFixed(1)+"°";
}
</script>

</body>
</html>
