# L
<!DOCTYPE html>
<html lang="ar">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>مواقيت الصلاة</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="container">
        <h1>الوقت والتاريخ</h1>
        <h2 id="current-time">جاري التحميل...</h2>
        <h3 id="hijri-date">جاري جلب التاريخ...</h3>

        <h1>مواقيت الصلاة</h1>
        <div id="prayer-times">جاري جلب البيانات...</div>

        <h1>اتجاه القبلة</h1>
        <p id="qibla-direction">جاري تحديد القبلة...</p>
        <img id="qibla-arrow" src="qibla-arrow.png" alt="اتجاه القبلة" style="width: 100px; display: none;">
    </div>

    <script src="script.js"></script>
</body>
</html>
body {
    font-family: Arial, sans-serif;
    text-align: center;
    direction: rtl;
    background: url('mosque-background.jpg') no-repeat center center fixed;
    background-size: cover;
    padding: 20px;
}
#container {
    background: rgba(255, 255, 255, 0.8);
    padding: 20px;
    border-radius: 10px;
    box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
    max-width: 500px;
    margin: auto;
}
h1 {
    color: #333;
}
#qibla-arrow {
    margin-top: 10px;
}
// 🔹 تحديث وعرض الوقت الحالي
function updateClock() {
    let now = new Date();
    let hours = now.getHours().toString().padStart(2, '0');
    let minutes = now.getMinutes().toString().padStart(2, '0');
    let seconds = now.getSeconds().toString().padStart(2, '0');
    document.getElementById("current-time").innerText = `${hours}:${minutes}:${seconds}`;
}
setInterval(updateClock, 1000);
updateClock();

// 🔹 جلب مواقيت الصلاة
function getPrayerTimes() {
    navigator.geolocation.getCurrentPosition(position => {
        let lat = position.coords.latitude;
        let lon = position.coords.longitude;
        let apiUrl = `https://api.aladhan.com/v1/timings?latitude=${lat}&longitude=${lon}&method=2`;
        fetch(apiUrl)
        .then(response => response.json())
        .then(data => {
            let times = data.data.timings;
            let output = `<h2>مواقيت الصلاة</h2>`;
            let prayerNames = {Fajr: "الفجر", Sunrise: "الشروق", Dhuhr: "الظهر", Asr: "العصر", Maghrib: "المغرب", Isha: "العشاء"};
            for (let prayer in prayerNames) {
                output += `<p><strong>${prayerNames[prayer]}</strong>: ${times[prayer]}</p>`;
            }
            document.getElementById("prayer-times").innerHTML = output;
        })
        .catch(error => console.error("حدث خطأ:", error));
    }, error => {
        document.getElementById("prayer-times").innerHTML = "يرجى السماح بالموقع لعرض الأوقات.";
    });
}

// 🔹 جلب التاريخ الهجري
function getHijriDate() {
    let today = new Date();
    let apiUrl = `https://api.aladhan.com/v1/gToH/${today.getFullYear()}-${today.getMonth() + 1}-${today.getDate()}`;
    fetch(apiUrl)
    .then(response => response.json())
    .then(data => {
        let hijriDate = data.data.hijri.date;
        let hijriMonth = data.data.hijri.month.ar;
        let hijriYear = data.data.hijri.year;
        document.getElementById("hijri-date").innerText = `التاريخ الهجري: ${hijriDate} ${hijriMonth} ${hijriYear} هـ`;
    })
    .catch(error => console.error("حدث خطأ:", error));
}

// 🔹 تحديد اتجاه القبلة
function getQiblaDirection() {
    navigator.geolocation.getCurrentPosition(position => {
        let lat = position.coords.latitude;
        let lon = position.coords.longitude;
        let apiUrl = `https://api.aladhan.com/v1/qibla/${lat}/${lon}`;
        fetch(apiUrl)
        .then(response => response.json())
        .then(data => {
            let direction = data.data.direction;
            document.getElementById("qibla-direction").innerText = `اتجاه القبلة: ${direction.toFixed(2)}° من الشمال`;
            let arrow = document.getElementById("qibla-arrow");
            arrow.style.transform = `rotate(${direction}deg)`;
            arrow.style.display = "block";
        })
        .catch(error => console.error("حدث خطأ:", error));
    }, error => {
        document.getElementById("qibla-direction").innerText = "يرجى السماح بالموقع لتحديد اتجاه القبلة.";
    });
}

// 🔹 تشغيل الوظائف عند تحميل الصفحة
window.onload = function() {
    getPrayerTimes();
    getHijriDate();
    getQiblaDirection();
};
