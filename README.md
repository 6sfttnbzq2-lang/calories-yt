<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="utf-8"/>
  <meta name="viewport" content="width=device-width, initial-scale=1"/>
  <title>Calories YouTube — Fitness</title>

  <link rel="manifest" href="manifest.json">
  <meta name="theme-color" content="#111111">

  <style>
    :root{
      --bg:#0e0e10;
      --card:#161617;
      --muted:#bdbdbd;
      --accent:#ff3b30;
      --accent-2:#ff6b5a;
      --glass: rgba(255,255,255,0.03);
    }
    html, body{height:100%;margin:0;font-family:Inter,Arial,Helvetica,sans-serif;background:linear-gradient(180deg,var(--bg),#070707);color:#fff;}
    .wrap{max-width:920px;margin:18px auto;padding:18px;}
    header{display:flex;align-items:center;gap:12px;}
    h1{margin:0;font-size:18px;}
    .card{background:var(--card);padding:14px;border-radius:12px;margin-top:12px;box-shadow:0 6px 18px rgba(0,0,0,0.6);}
    label{display:block;font-size:13px;color:var(--muted);margin-bottom:6px;}
    input, select, button{width:100%;padding:10px;border-radius:8px;border:0;background:var(--glass);color:#fff;font-size:15px;}
    .row{display:grid;grid-template-columns:1fr 1fr;gap:10px;}
    .actions{display:flex;gap:8px;margin-top:10px;}
    button.primary{background:linear-gradient(180deg,var(--accent),var(--accent-2));color:#fff;font-weight:600;}
    #videos{margin-top:12px;}
    .vid{display:flex;justify-content:space-between;gap:12px;padding:10px;border-radius:10px;background:rgba(255,255,255,0.02);align-items:center;margin-bottom:8px;}
    .meta{font-size:13px;color:var(--muted);}
    .small{font-size:12px;color:var(--muted);}
    footer{margin-top:18px;color:var(--muted);font-size:13px;}
    @media(max-width:600px){.row{grid-template-columns:1fr;}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <h1>Calories — Mes vidéos YouTube</h1>
      <div class="small">Thème : sombre + accents rouges • Icône : flamme + cœur</div>
    </header>

    <div class="card">
      <label>Ton poids (kg)</label>
      <input id="poids" type="number" value="70" min="30" max="300">

      <div style="height:10px"></div>

      <label>Lien YouTube</label>
      <input id="lien" placeholder="https://youtu.be/...">

      <div style="height:10px"></div>

      <div class="row">
        <div>
          <label>Durée réelle (minutes)</label>
          <input id="minutes" type="number" value="30" min="1">
        </div>
        <div>
          <label>Type d'entraînement (MET)</label>
          <select id="type">
            <option value="7">Cardio — MET 7</option>
            <option value="9.5">HIIT — MET 9.5</option>
            <option value="6">Danse — MET 6</option>
            <option value="5">Musculation — MET 5</option>
            <option value="4">Yoga / Stretch — MET 4</option>
          </select>
        </div>
      </div>

      <div class="actions">
        <button class="primary" id="ajouterBtn">➕ Ajouter la vidéo</button>
        <button id="viderBtn">🧹 Vider la liste</button>
      </div>

      <div id="videos"></div>

      <div style="display:flex;justify-content:space-between;align-items:center;margin-top:8px">
        <div class="meta">Total calories :</div>
        <div style="font-weight:700;font-size:18px" id="total">0 kcal</div>
      </div>
    </div>

    <footer>
      <div class="small">Tu peux installer l'app depuis le menu du navigateur : "Ajouter à l'écran d'accueil". Les données restent sur ton téléphone.</div>
    </footer>
  </div>

  <script src="app.js"></script>
</body>
</html>
const $ = id => document.getElementById(id);
const VIDEOS_KEY = 'yt_videos_calories_v1';

function load() {
  const raw = localStorage.getItem(VIDEOS_KEY);
  return raw ? JSON.parse(raw) : [];
}

function save(arr) {
  localStorage.setItem(VIDEOS_KEY, JSON.stringify(arr));
}

function render() {
  const list = load();
  const container = $('videos');
  container.innerHTML = '';
  let total = 0;

  list.forEach((v, idx) => {
    const calories = v.met * v.poids * (v.minutes/60);
    total += calories;

    const el = document.createElement('div');
    el.className = 'vid';
    el.innerHTML = `
      <div>
        <div style="font-weight:600">${v.title || 'Vidéo'} <span class="small">(${v.minutes} min)</span></div>
        <div class="meta"><a href="${v.lien}" target="_blank">${v.lien}</a></div>
      </div>
      <div style="text-align:right">
        <div style="font-weight:700">${calories.toFixed(1)} kcal</div>
        <div class="small">${v.type}</div>
        <div style="margin-top:6px"><button data-idx="${idx}" class="remove">Supprimer</button></div>
      </div>
    `;
    container.appendChild(el);
  });

  $('total').innerText = total.toFixed(1) + ' kcal';

  Array.from(document.querySelectorAll('.remove')).forEach(btn => {
    btn.addEventListener('click', () => {
      const i = Number(btn.dataset.idx);
      const arr = load();
      arr.splice(i,1);
      save(arr);
      render();
    });
  });
}

function tryExtractTitle(url, cb) {
  try {
    const u = new URL(url);
    if(u.hostname.includes('youtube') || u.hostname.includes('youtu.be')) {
      const v = u.searchParams.get('v') || u.pathname.split('/').pop();
      cb('YouTube — ' + v);
      return;
    }
  } catch(e) {}
  cb('Vidéo');
}

document.addEventListener('DOMContentLoaded', () => {
  render();
  $('ajouterBtn').addEventListener('click', () => {
    const poids = Number($('poids').value) || 70;
    const lien = $('lien').value.trim();
    const minutes = Number($('minutes').value) || 30;
    const met = Number($('type').value) || 7;
    const type = $('type').options[$('type').selectedIndex].text;

    if(!lien){ alert('Colle le lien YouTube ou une description.'); return; }

    tryExtractTitle(lien, (title) => {
      const arr = load();
      arr.push({lien, minutes, met, type, poids, title});
      save(arr);
      render();
      $('lien').value=''; $('minutes').value='30';
    });
  });

  $('viderBtn').addEventListener('click', () => {
    if(confirm('Vider toute la liste ?')) {
      localStorage.removeItem(VIDEOS_KEY);
      render();
    }
  });
});
{
  "name": "Calories YouTube — Fitness",
  "short_name": "CaloriesYT",
  "start_url": "./index.html",
  "display": "standalone",
  "background_color": "#0e0e10",
  "theme_color": "#111111",
  "icons": [
    {
      "src": "data:image/svg+xml;utf8,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><rect width='100' height='100' rx='18' fill='%23221f1f'/><g transform='translate(12,10) scale(0.76)'><path d='M30 10 C35 2, 50 3, 47 20 C60 18, 66 30, 56 38 C68 39, 74 54, 56 64 C34 77, 18 62, 30 36 C24 54, 12 60, 12 60 C18 40, 28 22, 30 10 Z' fill='%23ff3b30'/><path d='M50 26 C53 22, 64 20, 64 34 C64 46, 52 56, 50 64 C48 56,36 46,36 34 C36 22,47 22,50 26 Z' fill='%23ffdfdf'/></g></svg>",
      "sizes": "192x192",
      "type": "image/svg+xml"
    }
  ]
}
const CACHE = 'calories-cache-v1';
const OFFLINE_URL = './index.html';

self.addEventListener('install', (e) => {
  e.waitUntil(
    caches.open(CACHE).then(cache => {
      return cache.addAll([OFFLINE_URL, './', './index.html', './app.js', './manifest.json']);
    })
  );
  self.skipWaiting();
});

self.addEventListener('activate', (e) => {
  e.waitUntil(self.clients.claim());
});

self.addEventListener('fetch', (e) => {
  e.respondWith(
    caches.match(e.request)
      .then(resp => resp || fetch(e.request).catch(() => caches.match(OFFLINE_URL)))
  );
});
