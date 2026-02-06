[index.html](https://github.com/user-attachments/files/25133181/index.html)
<!doctype html>
<html lang="hu">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Időjárás App + műholdas háttér</title>

  <!-- MapLibre -->
  <link href="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.css" rel="stylesheet" />
  <script src="https://unpkg.com/maplibre-gl@4.7.1/dist/maplibre-gl.js"></script>

  <style>
    :root{
      color-scheme: dark;
      --bgA: #0b1020;
      --bgB: #000000;
      --accent: rgba(102,166,255,.30);
      --accent2: rgba(138,43,226,.24);
      --card: rgba(0,0,0,.55);
      --stroke: rgba(255,255,255,.16);
      --text: #f2f5ff;
      --muted: rgba(242,245,255,.72);
    }

    *{ box-sizing:border-box; }
    html, body{ height:100%; margin:0; background:#000; font-family: system-ui, Arial, sans-serif; color:var(--text); }

    /* Műholdas háttér térkép */
    #map{
      position: fixed;
      inset: 0;
      z-index: 0;
      filter: saturate(1.15) contrast(1.05);
      transform: scale(1.02);
    }

    /* Színes “aurák” (időjárástól függően JS állítja) */
    .aura{
      position: fixed;
      inset: 0;
      z-index: 1;
      pointer-events: none;
      background:
        radial-gradient(900px 600px at 18% 24%, var(--accent), transparent 62%),
        radial-gradient(900px 600px at 82% 20%, var(--accent2), transparent 62%),
        radial-gradient(900px 600px at 55% 88%, rgba(0,255,200,.14), transparent 62%),
        linear-gradient(180deg, var(--bgA), var(--bgB));
      opacity: .85;
      mix-blend-mode: screen;
    }

    /* Sötétítő réteg */
    .shade{
      position: fixed;
      inset: 0;
      z-index: 2;
      pointer-events: none;
      background: rgba(0,0,0,.40);
      backdrop-filter: blur(1px);
    }

    /* App kártya */
    .wrap{
      position: relative;
      z-index: 3;
      min-height: 100%;
      display: grid;
      place-items: center;
      padding: 22px;
    }

    .card{
      width: min(720px, 100%);
      background: var(--card);
      border: 1px solid var(--stroke);
      border-radius: 18px;
      box-shadow: 0 20px 70px rgba(0,0,0,.55);
      overflow: hidden;
    }

    .topbar{
      padding: 14px 16px;
      border-bottom: 1px solid rgba(255,255,255,.12);
      display:flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      background: linear-gradient(90deg, rgba(255,255,255,.08), rgba(255,255,255,.02));
    }

    .title{
      display:flex;
      flex-direction: column;
      gap: 4px;
      min-width: 0;
    }
    .title h1{ margin:0; font-size: 16px; letter-spacing: .2px; }
    .title .sub{ font-size: 12px; color: var(--muted); }

    .pill{
      font-size: 12px;
      padding: 6px 10px;
      border-radius: 999px;
      border: 1px solid rgba(255,255,255,.14);
      background: rgba(0,0,0,.25);
      color: var(--muted);
      white-space: nowrap;
    }

    .content{ padding: 14px 16px 16px; }

    .row{
      display:flex;
      gap: 10px;
      align-items: center;
      flex-wrap: wrap;
    }

    input{
      flex: 1 1 260px;
      padding: 12px 12px;
      border-radius: 14px;
      border: 1px solid rgba(255,255,255,.16);
      background: rgba(0,0,0,.35);
      color: var(--text);
      outline: none;
      transition: box-shadow .15s ease, border-color .15s ease;
    }
    input::placeholder{ color: rgba(242,245,255,.55); }
    input:focus{
      border-color: rgba(255,255,255,.28);
      box-shadow: 0 0 0 4px rgba(120,160,255,.18);
    }

    button{
      padding: 12px 14px;
      border-radius: 14px;
      border: 1px solid rgba(255,255,255,.18);
      background: linear-gradient(180deg, rgba(255,255,255,.16), rgba(255,255,255,.06));
      color: var(--text);
      cursor: pointer;
      font-weight: 850;
      letter-spacing: .2px;
      transition: transform .06s ease, filter .15s ease;
      white-space: nowrap;
    }
    button:hover{ filter: brightness(1.12); }
    button:active{ transform: translateY(1px); }

    .status{
      margin-top: 10px;
      font-size: 13px;
      color: var(--muted);
      min-height: 18px;
    }

    .hero{
      margin-top: 12px;
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 16px;
      padding: 12px;
      background: rgba(0,0,0,.22);
      display:flex;
      justify-content: space-between;
      gap: 10px;
      align-items: center;
    }
    .hero .place{
      display:flex;
      flex-direction: column;
      gap: 4px;
      min-width: 0;
    }
    .hero .place .name{
      font-size: 15px;
      font-weight: 950;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    .hero .place .meta{
      font-size: 12px;
      color: var(--muted);
    }
    .hero .big{
      font-size: 34px;
      font-weight: 1000;
      letter-spacing: .3px;
    }

    .grid{
      margin-top: 10px;
      display:grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
    }
    .box{
      border: 1px solid rgba(255,255,255,.12);
      border-radius: 16px;
      padding: 12px;
      background: rgba(0,0,0,.18);
    }
    .k{ font-size: 12px; color: var(--muted); margin-bottom: 6px; }
    .v{ font-size: 18px; font-weight: 950; }

    .tagrow{
      margin-top: 10px;
      display:flex;
      gap: 8px;
      flex-wrap: wrap;
    }
    .tag{
      font-size: 12px;
      padding: 7px 10px;
      border-radius: 999px;
      border: 1px solid rgba(255,255,255,.12);
      background: rgba(0,0,0,.20);
      color: var(--muted);
    }

    @media (max-width: 520px){
      .grid{ grid-template-columns: 1fr; }
      .hero{ flex-direction: column; align-items: flex-start; }
      .hero .big{ font-size: 30px; }
    }
  </style>
</head>

<body>
  <div id="map"></div>
  <div class="aura"></div>
  <div class="shade"></div>

  <div class="wrap">
    <div class="card">
      <div class="topbar">
        <div class="title">
          <h1>Időjárás</h1>
          <div class="sub">Keress városra → friss adatok + műholdas háttér rázumol</div>
        </div>
        <div id="modePill" class="pill">—</div>
      </div>

      <div class="content">
        <div class="row">
          <input id="city" placeholder="Város (pl. Budapest, Szeged, Debrecen…)" autocomplete="off" />
          <button id="btn">Lekérés</button>
          <button id="btnLoc" title="Saját hely (ha engeded a böngészőben)">📍</button>
        </div>

        <div id="status" class="status"></div>

        <div id="hero" class="hero" style="display:none;">
          <div class="place">
            <div id="placeName" class="name">—</div>
            <div id="placeMeta" class="meta">—</div>
          </div>
          <div id="bigTemp" class="big">—°</div>
        </div>

        <div id="grid" class="grid" style="display:none;"></div>
        <div id="tags" class="tagrow" style="display:none;"></div>
      </div>
    </div>
  </div>

<script>
  // ====== MŰHOLDAS MAP (ESRI World Imagery, kulcs nélkül) ======
  const satelliteStyle = {
    "version": 8,
    "sources": {
      "satellite": {
        "type": "raster",
        "tiles": [
          "https://services.arcgisonline.com/ArcGIS/rest/services/World_Imagery/MapServer/tile/{z}/{y}/{x}"
        ],
        "tileSize": 256,
        "attribution": "© Esri, Maxar, Earthstar Geographics"
      }
    },
    "layers": [
      { "id": "satellite", "type": "raster", "source": "satellite" }
    ]
  };

  const map = new maplibregl.Map({
    container: "map",
    style: satelliteStyle,
    center: [19.0402, 47.4979],
    zoom: 3.8,
    maxZoom: 19,
    interactive: false // háttérként
  });

  let marker = null;

  function flyToCity(lon, lat, population) {
    // népesség alapján becsült zoom, hogy "város környéke" látszódjon
    let z = 12.0;
    const pop = Number(population || 0);
    if (pop >= 5000000) z = 10.0;
    else if (pop >= 1500000) z = 10.7;
    else if (pop >= 500000) z = 11.3;
    else if (pop >= 200000) z = 11.8;
    else z = 12.6;

    map.flyTo({ center: [lon, lat], zoom: z, speed: 0.9, curve: 1.2 });

    if (marker) marker.remove();
    marker = new maplibregl.Marker({ color: "#ffffff" }).setLngLat([lon, lat]).addTo(map);
  }

  // ====== WEATHER + UI ======
  const $ = (id) => document.getElementById(id);
  const statusEl = $("status");

  function fmt(n, digits=1){
    if (n === null || n === undefined || Number.isNaN(Number(n))) return "—";
    return Number(n).toFixed(digits);
  }

  async function geocodeCity(name) {
    const url = `https://geocoding-api.open-meteo.com/v1/search?name=${encodeURIComponent(name)}&count=5&language=hu&format=json`;
    const r = await fetch(url);
    if (!r.ok) throw new Error("Nem sikerült a város keresése.");
    const data = await r.json();
    if (!data.results || data.results.length === 0) return null;
    return data.results[0];
  }

  async function fetchWeather(lat, lon) {
    const url = new URL("https://api.open-meteo.com/v1/forecast");
    url.searchParams.set("latitude", lat);
    url.searchParams.set("longitude", lon);
    url.searchParams.set("timezone", "auto");
    url.searchParams.set("current", [
      "temperature_2m",
      "apparent_temperature",
      "precipitation",
      "rain",
      "showers",
      "snowfall",
      "wind_speed_10m",
      "wind_gusts_10m",
      "cloud_cover"
    ].join(","));

    const r = await fetch(url.toString());
    if (!r.ok) throw new Error("Nem sikerült az időjárás lekérése.");
    return await r.json();
  }

  function setThemeFromWeather(cur){
    const t = Number(cur.temperature_2m);
    const p = Number(cur.precipitation);
    const c = Number(cur.cloud_cover);

    let accent = "rgba(102,166,255,.30)";
    let accent2 = "rgba(138,43,226,.24)";
    let pill = "Semleges";

    // csapadékos
    if (p > 0.2){
      accent  = "rgba(0,170,255,.33)";
      accent2 = "rgba(0,255,200,.20)";
      pill = "Eső / csapadék";
    } else if (c >= 70) {
      // felhős
      accent  = "rgba(170,190,255,.22)";
      accent2 = "rgba(160,160,255,.18)";
      pill = "Felhős";
    } else if (c <= 25) {
      // derült
      accent  = "rgba(255,210,80,.26)";
      accent2 = "rgba(255,120,80,.18)";
      pill = "Derült";
    } else {
      pill = "Változó";
    }

    // hőmérséklet erős “hangulat”
    if (!Number.isNaN(t)){
      if (t <= 0){
        accent  = "rgba(80,140,255,.36)";
        accent2 = "rgba(0,255,200,.18)";
        pill = "Hideg";
      } else if (t >= 28){
        accent  = "rgba(255,120,80,.36)";
        accent2 = "rgba(255,60,140,.22)";
        pill = "Meleg";
      } else if (t >= 18 && t < 28){
        accent  = "rgba(255,210,80,.28)";
        accent2 = "rgba(255,120,80,.20)";
        pill = "Kellemes";
      }
    }

    document.documentElement.style.setProperty("--accent", accent);
    document.documentElement.style.setProperty("--accent2", accent2);
    $("modePill").textContent = pill;
  }

  function render(city, w){
    const cur = w.current || {};
    const u = w.current_units || {};

    setThemeFromWeather(cur);
    flyToCity(city.longitude, city.latitude, city.population);

    $("hero").style.display = "flex";
    $("grid").style.display = "grid";
    $("tags").style.display = "flex";

    $("placeName").textContent = `${city.name}${city.admin1 ? ", " + city.admin1 : ""}${city.country ? ", " + city.country : ""}`;
    $("placeMeta").textContent = `Koordináta: ${fmt(city.latitude, 4)}, ${fmt(city.longitude, 4)} • Frissítve: ${cur.time || "—"}`;
    $("bigTemp").textContent = `${fmt(cur.temperature_2m, 0)}°`;

    const boxes = [
      ["Hőérzet", `${fmt(cur.apparent_temperature)} ${u.apparent_temperature || "°C"}`],
      ["Csapadék", `${fmt(cur.precipitation)} ${u.precipitation || "mm"}`],
      ["Eső / Zápor", `${fmt(cur.rain)} ${u.rain || "mm"} / ${fmt(cur.showers)} ${u.showers || "mm"}`],
      ["Felhőzet", `${fmt(cur.cloud_cover, 0)} ${u.cloud_cover || "%"}`],
      ["Szél", `${fmt(cur.wind_speed_10m)} ${u.wind_speed_10m || "km/h"}`],
      ["Széllökés", `${fmt(cur.wind_gusts_10m)} ${u.wind_gusts_10m || "km/h"}`],
      ["Havazás", `${fmt(cur.snowfall)} ${u.snowfall || "cm"}`],
      ["Hőmérséklet", `${fmt(cur.temperature_2m)} ${u.temperature_2m || "°C"}`],
    ];

    $("grid").innerHTML = boxes.map(([k,v]) => `
      <div class="box">
        <div class="k">${k}</div>
        <div class="v">${v}</div>
      </div>
    `).join("");

    const tags = [];
    const t = Number(cur.temperature_2m);
    const p = Number(cur.precipitation);
    const c = Number(cur.cloud_cover);

    if (!Number.isNaN(t)){
      if (t <= 0) tags.push("🧊 fagyos");
      else if (t < 10) tags.push("🥶 hideg");
      else if (t < 18) tags.push("🧥 hűvös");
      else if (t < 28) tags.push("🙂 kellemes");
      else tags.push("🔥 meleg");
    }
    if (p > 0.2) tags.push("🌧️ csapadék");
    else tags.push("☔ nincs csapadék");
    if (c >= 70) tags.push("☁️ felhős");
    else if (c <= 25) tags.push("🌤️ derült");
    else tags.push("⛅ változó");

    $("tags").innerHTML = tags.map(t => `<div class="tag">${t}</div>`).join("");
  }

  async function runByCityName(){
    const name = $("city").value.trim();
    $("grid").style.display = "none";
    $("tags").style.display = "none";
    $("hero").style.display = "none";

    if (!name){
      statusEl.textContent = "Írj be egy városnevet.";
      return;
    }

    statusEl.textContent = "Keresés…";
    try{
      const city = await geocodeCity(name);
      if (!city){
        statusEl.textContent = "Nem találtam ilyen várost. Próbáld más névvel.";
        return;
      }

      statusEl.textContent = "Időjárás lekérése…";
      const w = await fetchWeather(city.latitude, city.longitude);

      statusEl.textContent = "";
      render(city, w);
    }catch(e){
      statusEl.textContent = "Hiba: " + (e?.message || String(e));
    }
  }

  async function runByGeolocation(){
    $("grid").style.display = "none";
    $("tags").style.display = "none";
    $("hero").style.display = "none";

    if (!navigator.geolocation){
      statusEl.textContent = "A böngésződ nem támogatja a helymeghatározást.";
      return;
    }

    statusEl.textContent = "Helyzet lekérése…";
    navigator.geolocation.getCurrentPosition(async (pos) => {
      try{
        const lat = pos.coords.latitude;
        const lon = pos.coords.longitude;

        statusEl.textContent = "Időjárás lekérése…";
        const w = await fetchWeather(lat, lon);

        const fakeCity = { name: "Saját hely", admin1: "", country: "", latitude: lat, longitude: lon, population: 50000 };
        statusEl.textContent = "";
        render(fakeCity, w);
      }catch(e){
        statusEl.textContent = "Hiba: " + (e?.message || String(e));
      }
    }, () => {
      statusEl.textContent = "Nem engedted a helymeghatározást.";
    }, { enableHighAccuracy: true, timeout: 10000 });
  }

  $("btn").addEventListener("click", runByCityName);
  $("btnLoc").addEventListener("click", runByGeolocation);
  $("city").addEventListener("keydown", (e) => { if (e.key === "Enter") runByCityName(); });

  // Indulás
  $("city").value = "Budapest";
  runByCityName();
</script>
</body>
</html>
