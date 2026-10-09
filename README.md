
<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<meta name="theme-color" content="#101827">
<title>QH IMEI Device Location Tracker</title>

<link rel="stylesheet"
 href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css"
 integrity="sha256-p4NxAoJBhIIN+hmNHrzRCf9tD/miZyoHS5obTRR9BMY="
 crossorigin="">

<style>
:root {
  --bg:#0b1120;
  --panel:#131e30;
  --panel2:#19273c;
  --text:#edf4ff;
  --muted:#9eafc7;
  --accent:#f59e0b;
  --green:#22c55e;
  --red:#ef4444;
  --border:#2a3a52;
}
* { box-sizing:border-box; }
body {
  margin:0;
  font-family:Tahoma,Arial,sans-serif;
  background:var(--bg);
  color:var(--text);
  line-height:1.8;
}
header {
  padding:24px 16px;
  background:linear-gradient(135deg,#152642,#0b1120);
  border-bottom:1px solid var(--border);
  text-align:center;
}
header h1 { margin:0 0 8px; font-size:clamp(22px,5vw,32px); }
header p { color:var(--muted); margin:0; }
.container { max-width:1100px; padding:18px; margin:auto; }
.card {
  background:var(--panel);
  border:1px solid var(--border);
  border-radius:16px;
  padding:18px;
  margin-bottom:16px;
}
h2 { font-size:19px; margin:0 0 12px; }
label { display:block; margin:12px 0 5px; font-weight:bold; }
input,button,select {
  font:inherit;
  border-radius:9px;
}
input {
  width:100%;
  padding:12px;
  background:#0b1424;
  color:white;
  border:1px solid var(--border);
  direction:ltr;
  text-align:left;
}
button {
  padding:10px 14px;
  border:0;
  cursor:pointer;
  font-weight:bold;
}
button:disabled { opacity:.5; cursor:not-allowed; }
.primary { background:var(--accent); color:#1b160b; }
.success { background:var(--green); color:#06210e; }
.danger { background:var(--red); color:white; }
.secondary {
  background:var(--panel2);
  color:var(--text);
  border:1px solid var(--border);
}
.actions {
  display:flex;
  flex-wrap:wrap;
  gap:9px;
  margin-top:14px;
}
.notice {
  padding:12px;
  background:#332912;
  border:1px solid #6d5016;
  border-radius:10px;
  color:#ffe7ae;
  font-size:14px;
}
.muted { color:var(--muted); font-size:13px; }
.status {
  padding:10px;
  background:#0b1424;
  border:1px solid var(--border);
  border-radius:9px;
  overflow-wrap:anywhere;
}
.grid {
  display:grid;
  grid-template-columns:repeat(auto-fit,minmax(220px,1fr));
  gap:14px;
}
.metric {
  padding:14px;
  background:var(--panel2);
  border-radius:12px;
  min-width:0;
}
.metric .value {
  font-size:19px;
  font-weight:bold;
  overflow-wrap:anywhere;
  direction:ltr;
  text-align:right;
}
.metric .label { color:var(--muted); font-size:13px; }
#map {
  width:100%;
  height:390px;
  border-radius:12px;
  background:#dbeafe;
  z-index:1;
}
table { width:100%; border-collapse:collapse; font-size:13px; }
th,td {
  padding:9px;
  border-bottom:1px solid var(--border);
  text-align:right;
}
.table-wrap { overflow-x:auto; }
footer {
  text-align:center;
  padding:20px;
  color:var(--muted);
  font-size:12px;
}
a { color:#7dd3fc; }
[hidden] { display:none !important; }
</style>
</head>

<body>
<header>
  <h1>QH IMEI Device Locator</h1>
  <p>سیستم بررسی IMEI و نمایش موقعیت مجاز دستگاه روی نقشه</p>
</header>

<main class="container">

  <section class="card">
    <h2>۱. بررسی شمارهٔ IMEI</h2>
    <p class="muted">
      شمارهٔ IMEI معمولاً ۱۵ رقم دارد. اعتبارسنجی این صفحه
      فقط ساختار و رقم کنترلی را بررسی می‌کند و مالکیت یا
      موقعیت گوشی را تأیید نمی‌کند.
    </p>

    <label for="imei">شمارهٔ IMEI</label>
    <input id="imei" inputmode="numeric" maxlength="15"
      autocomplete="off" placeholder="مثلاً 490154203237518">

    <div class="actions">
      <button class="primary" id="checkImei">بررسی IMEI</button>
      <button class="secondary" id="clearImei">پاک‌کردن</button>
    </div>
    <div id="imeiResult" class="status" style="margin-top:12px"
      aria-live="polite">شمارهٔ IMEI را وارد کنید.</div>
  </section>

  <section class="card">
    <h2>۲. موقعیت دستگاه فعلی</h2>
    <div class="notice">
      این بخش مکان دستگاهی را می‌خواند که همین صفحه روی آن باز است.
      اجازهٔ موقعیت‌یابی را فقط در صورتی بدهید که با آن موافق هستید.
      IMEI برای دریافت مختصات استفاده نمی‌شود.
    </div>

    <div class="actions">
      <button class="success" id="start">شروع موقعیت‌یابی</button>
      <button class="danger" id="stop" disabled>توقف</button>
      <button class="secondary" id="centerMap">رفتن به موقعیت فعلی</button>
    </div>

    <div id="locationStatus" class="status" style="margin-top:12px"
      aria-live="polite">موقعیت‌یابی هنوز شروع نشده است.</div>
  </section>

  <section class="grid" style="margin-bottom:16px">
    <div class="metric">
      <div class="label">عرض جغرافیایی (Latitude)</div>
      <div class="value" id="lat">—</div>
    </div>
    <div class="metric">
      <div class="label">طول جغرافیایی (Longitude)</div>
      <div class="value" id="lng">—</div>
    </div>
    <div class="metric">
      <div class="label">دقت اعلام‌شدهٔ دستگاه</div>
      <div class="value" id="accuracy">—</div>
    </div>
    <div class="metric">
      <div class="label">آخرین به‌روزرسانی</div>
      <div class="value" id="updated" style="font-size:15px">—</div>
    </div>
  </section>

  <section class="card">
    <h2>۳. نقشهٔ زنده</h2>
    <p class="muted">
      نشانگر نقشه، مختصات گزارش‌شده از دستگاه فعلی را نمایش می‌دهد.
      دایرهٔ اطراف نشانگر، محدودهٔ تقریبی دقت گزارش‌شده است.
    </p>
    <div id="map" aria-label="نقشهٔ موقعیت دستگاه"></div>
    <div class="actions">
      <button class="secondary" id="openMap">بازکردن مختصات در نقشهٔ بیرونی</button>
    </div>
  </section>

  <section class="card">
    <h2>۴. تاریخچهٔ مسیر</h2>
    <p class="muted">
      نقاط مسیر در حافظهٔ محلی همین مرورگر ذخیره می‌شوند؛
      به سرور ارسال نمی‌شوند.
    </p>
    <div class="grid">
      <div class="metric">
        <div class="label">تعداد نقاط ثبت‌شده</div>
        <div class="value" id="pointCount">۰</div>
      </div>
      <div class="metric">
        <div class="label">طول تقریبی مسیر</div>
        <div class="value" id="distance">۰ متر</div>
      </div>
    </div>
    <div class="actions">
      <button class="primary" id="export">صادرکردن اطلاعات JSON</button>
      <button class="secondary" id="clearRoute">پاک‌کردن تاریخچه</button>
    </div>
    <div class="table-wrap" style="margin-top:15px">
      <table>
        <thead>
          <tr>
            <th>شماره</th>
            <th>عرض</th>
            <th>طول</th>
            <th>دقت (متر)</th>
            <th>زمان</th>
          </tr>
        </thead>
        <tbody id="historyBody">
          <tr><td colspan="5">هنوز نقطه‌ای ثبت نشده است.</td></tr>
        </tbody>
      </table>
    </div>
  </section>

  <section class="card">
    <h2>۵. نکات تخنیکی و امنیتی</h2>
    <ul>
      <li>IMEI شناسهٔ دستگاه است، نه گیرندهٔ GPS.</li>
      <li>این صفحه به شبکهٔ مخابراتی اپراتورها دسترسی ندارد.</li>
      <li>موقعیت‌یابی مرورگر معمولاً به HTTPS و اجازهٔ کاربر نیاز دارد.</li>
      <li>نقشه به اینترنت نیاز دارد تا کاشی‌های نقشه بارگذاری شوند.</li>
      <li>مختصات GPS یا شبکه ممکن است خطا داشته باشد.</li>
      <li>این صفحه موقعیت گوشی خاموش یا گوشی دیگری را صرفاً با IMEI پیدا نمی‌کند.</li>
    </ul>
  </section>

</main>

<footer>
  QH Device Location Tracker · نسخهٔ تک‌فایلی
</footer>

<script
 src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"
 integrity="sha256-20nQCchB9co0qIjJZRGuk2/Z9VM+kNiyxNV1lvTlZBo="
 crossorigin="">
</script>

<script>
"use strict";

const $ = id => document.getElementById(id);
const STORAGE_KEY = "qh_device_location_history_v1";

let map = null;
let marker = null;
let accuracyCircle = null;
let routeLine = null;
let watchId = null;
let currentPosition = null;
let history = loadHistory();

function loadHistory() {
  try {
    const saved = JSON.parse(localStorage.getItem(STORAGE_KEY) || "[]");
    if (!Array.isArray(saved)) return [];
    return saved.filter(p =>
      Number.isFinite(p.lat) &&
      Number.isFinite(p.lng) &&
      Number.isFinite(p.accuracy) &&
      Number.isFinite(p.timestamp)
    );
  } catch {
    return [];
  }
}

function saveHistory() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(history));
  } catch {
    setLocationStatus("ذخیره در حافظهٔ محلی ناموفق بود؛ ممکن است فضای ذخیره‌سازی موجود نباشد.");
  }
}

function setLocationStatus(message) {
  $("locationStatus").textContent = message;
}

function formatNumber(number, digits = 6) {
  return Number(number).toFixed(digits);
}

function formatDistance(meters) {
  if (!Number.isFinite(meters)) return "—";
  if (meters < 1000) return Math.round(meters) + " متر";
  return (meters / 1000).toFixed(2) + " کیلومتر";
}

/* اعتبارسنجی IMEI با الگوریتم Luhn */
function isValidIMEI(value) {
  if (!/^\d{15}$/.test(value)) return false;

  let sum = 0;

  for (let i = 0; i < value.length; i++) {
    let digit = Number(value[i]);

    // از سمت راست، رقم‌های جایگاه دوم را دو برابر می‌کنیم.
    if (i % 2 === 1) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }

    sum += digit;
  }

  return sum % 10 === 0;
}

$("checkImei").addEventListener("click", () => {
  const value = $("imei").value.trim();
  const result = $("imeiResult");

  if (!/^\d+$/.test(value)) {
    result.textContent = "خطا: فقط رقم‌های انگلیسی ۰ تا ۹ را وارد کنید.";
    return;
  }

  if (value.length !== 15) {
    result.textContent = "خطا: IMEI باید دقیقاً ۱۵ رقم داشته باشد.";
    return;
  }

  if (!isValidIMEI(value)) {
    result.textContent =
      "رقم کنترلی IMEI معتبر نیست. شماره را دوباره بررسی کنید.";
    return;
  }

  result.textContent =
    "ساختار و رقم کنترلی IMEI معتبر است. این نتیجه مالکیت دستگاه یا موقعیت آن را تأیید نمی‌کند.";
});

$("clearImei").addEventListener("click", () => {
  $("imei").value = "";
  $("imeiResult").textContent = "شمارهٔ IMEI را وارد کنید.";
});

/* آماده‌سازی نقشه */
function initMap() {
  if (typeof L === "undefined") {
    $("map").textContent =
      "کتابخانهٔ نقشه بارگذاری نشد. اتصال اینترنت را بررسی و صفحه را دوباره باز کنید.";
    setLocationStatus("نقشه بارگذاری نشده است؛ موقعیت‌یابی ممکن است همچنان در دسترس باشد.");
    return;
  }

  map = L.map("map").setView([34.5553, 69.2075], 6);

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    maxZoom: 19,
    attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a>'
  }).addTo(map);

  marker = L.marker([34.5553, 69.2075]).addTo(map);
  marker.bindPopup("موقعیت هنوز ثبت نشده است.");

  accuracyCircle = L.circle([34.5553, 69.2075], {
    radius: 0,
    color: "#f59e0b"
  }).addTo(map);

  routeLine = L.polyline([], {
    color: "#22c55e",
    weight: 4
  }).addTo(map);

  // نمایش نقاط ذخیره‌شده از نشست‌های پیشین
  if (history.length) {
    routeLine.setLatLngs(history.map(p => [p.lat, p.lng]));
    updateHistoryUI();
  }

  setTimeout(() => map.invalidateSize(), 200);
}

function handlePosition(position) {
  const coords = position.coords;

  const point = {
    lat: coords.latitude,
    lng: coords.longitude,
    accuracy: Number.isFinite(coords.accuracy) ? coords.accuracy : 0,
    timestamp: position.timestamp || Date.now()
  };

  currentPosition = point;

  $("lat").textContent = formatNumber(point.lat);
  $("lng").textContent = formatNumber(point.lng);
  $("accuracy").textContent = Math.round(point.accuracy) + " متر";
  $("updated").textContent = new Date(point.timestamp).toLocaleString();

  if (map && marker && accuracyCircle && routeLine) {
    const latlng = [point.lat, point.lng];

    marker.setLatLng(latlng);
    marker.setPopupContent(
      "موقعیت فعلی<br>عرض: " + formatNumber(point.lat) +
      "<br>طول: " + formatNumber(point.lng) +
      "<br>دقت: " + Math.round(point.accuracy) + " متر"
    );

    accuracyCircle.setLatLng(latlng);
    accuracyCircle.setRadius(point.accuracy);

    // از ذخیرهٔ چندبارهٔ مختصات تقریباً یکسان جلوگیری می‌شود.
    const last = history[history.length - 1];
    const shouldAdd = !last ||
      last.lat !== point.lat ||
      last.lng !== point.lng ||
      point.timestamp - last.timestamp > 30000;

    if (shouldAdd) {
      history.push(point);

      // محدودیت حافظه: نگه‌داری حداکثر ۵۰۰۰ نقطه
      if (history.length > 5000) history = history.slice(-5000);

      saveHistory();
      routeLine.setLatLngs(history.map(p => [p.lat, p.lng]));
      updateHistoryUI();
    }
  }

  setLocationStatus(
    "موقعیت دریافت شد. زمان: " +
    new Date(point.timestamp).toLocaleTimeString() +
    ". دقت اعلام‌شده: حدود " + Math.round(point.accuracy) + " متر."
  );
}

function handlePositionError(error) {
  const messages = {
    1: "دسترسی به موقعیت رد شد. برای استفاده از GPS باید اجازه بدهید.",
    2: "موقعیت در حال حاضر در دسترس نیست. GPS یا خدمات مکان‌یابی را بررسی کنید.",
    3: "زمان دریافت موقعیت تمام شد. دوباره تلاش کنید."
  };

  setLocationStatus(
    messages[error.code] || "خطای ناشناخته در دریافت موقعیت رخ داد."
  );
}

$("start").addEventListener("click", () => {
  if (!("geolocation" in navigator)) {
    setLocationStatus("این مرورگر از موقعیت‌یابی پشتیبانی نمی‌کند.");
    return;
  }

  if (!window.isSecureContext) {
    setLocationStatus(
      "برای موقعیت‌یابی، سایت را از طریق HTTPS باز کنید؛ آدرس GitHub Pages معمولاً HTTPS است."
    );
    return;
  }

  if (watchId !== null) {
    setLocationStatus("موقعیت‌یابی از قبل فعال است.");
    return;
  }

  setLocationStatus("در انتظار اجازه و دریافت مختصات دستگاه فعلی...");
  $("start").disabled = true;

  watchId = navigator.geolocation.watchPosition(
    handlePosition,
    error => {
      handlePositionError(error);
      if (error.code === 1) stopTracking();
      else $("start").disabled = false;
    },
    {
      enableHighAccuracy: true,
      maximumAge: 5000,
      timeout: 20000
    }
  );

  $("stop").disabled = false;
});

function stopTracking() {
  if (watchId !== null) {
    navigator.geolocation.clearWatch(watchId);
    watchId = null;
  }

  $("start").disabled = false;
  $("stop").disabled = true;
  setLocationStatus("موقعیت‌یابی متوقف شد. تاریخچهٔ قبلی حفظ شده است.");
}

$("stop").addEventListener("click", stopTracking);

$("centerMap").addEventListener("click", () => {
  if (!currentPosition) {
    setLocationStatus("هنوز موقعیت فعلی دریافت نشده است.");
    return;
  }

  if (map) {
    map.setView([currentPosition.lat, currentPosition.lng], 17);
    if (marker) marker.openPopup();
  }
});

$("openMap").addEventListener("click", () => {
  if (!currentPosition) {
    setLocationStatus("ابتدا موقعیت دستگاه فعلی را دریافت کنید.");
    return;
  }

  const url =
    "https://www.openstreetmap.org/?mlat=" +
    encodeURIComponent(currentPosition.lat) +
    "&mlon=" + encodeURIComponent(currentPosition.lng) +
    "#map=17/" + encodeURIComponent(currentPosition.lat) +
    "/" + encodeURIComponent(currentPosition.lng);

  window.open(url, "_blank", "noopener,noreferrer");
});

function calculateDistance(a, b) {
  const R = 6371000;
  const rad = degrees => degrees * Math.PI / 180;

  const dLat = rad(b.lat - a.lat);
  const dLng = rad(b.lng - a.lng);
  const lat1 = rad(a.lat);
  const lat2 = rad(b.lat);

  const value =
    Math.sin(dLat / 2) ** 2 +
    Math.cos(lat1) * Math.cos(lat2) *
    Math.sin(dLng / 2) ** 2;

  return 2 * R * Math.asin(Math.sqrt(Math.min(1, value)));
}

function updateHistoryUI() {
  $("pointCount").textContent = history.length.toLocaleString();

  let total = 0;
  for (let i = 1; i < history.length; i++) {
    total += calculateDistance(history[i - 1], history[i]);
  }

  $("distance").textContent = formatDistance(total);

  const tbody = $("historyBody");
  tbody.replaceChildren();

  if (!history.length) {
    const tr = document.createElement("tr");
    const td = document.createElement("td");
    td.colSpan = 5;
    td.textContent = "هنوز نقطه‌ای ثبت نشده است.";
    tr.appendChild(td);
    tbody.appendChild(tr);
    return;
  }

  // نمایش ۱۰۰ نقطهٔ آخر در جدول
  history.slice(-100).reverse().forEach((p, index) => {
    const tr = document.createElement("tr");
    const values = [
      history.length - index,
      formatNumber(p.lat),
      formatNumber(p.lng),
      Math.round(p.accuracy),
      new Date(p.timestamp).toLocaleString()
    ];

    values.forEach(value => {
      const td = document.createElement("td");
      td.textContent = String(value);
      tr.appendChild(td);
    });

    tbody.appendChild(tr);
  });
}

$("export").addEventListener("click", () => {
  const data = {
    application: "QH IMEI Device Location Tracker",
    exportedAt: new Date().toISOString(),
    note: "این اطلاعات از دستگاهی است که صفحه روی آن اجرا شده؛ مکان از IMEI استخراج نشده است.",
    history: history
  };

  const blob = new Blob(
    [JSON.stringify(data, null, 2)],
    { type: "application/json;charset=utf-8" }
  );

  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "qh-location-history.json";
  document.body.appendChild(a);
  a.click();
  a.remove();
  URL.revokeObjectURL(url);
});

$("clearRoute").addEventListener("click", () => {
  if (!confirm("آیا از پاک‌کردن تمام تاریخچهٔ مسیر مطمئن هستید؟")) {
    return;
  }

  history = [];
  saveHistory();

  if (routeLine) routeLine.setLatLngs([]);

  updateHistoryUI();
  setLocationStatus("تاریخچهٔ مسیر پاک شد.");
});

initMap();
updateHistoryUI();
</script>

</body>
</html>
