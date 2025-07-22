<<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8" />
  <title>نظام الحضور بالباركود</title>
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <link href="https://fonts.googleapis.com/css2?family=Cairo&display=swap" rel="stylesheet">
  <script src="https://cdn.jsdelivr.net/npm/qrcode@1.4.4/build/qrcode.min.js"></script>
  <script src="https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js"></script>
  <style>
    body {
      font-family: 'Cairo', sans-serif;
      margin: 0;
      padding: 0;
      background: linear-gradient(to right, #e0f7fa, #80deea);
    }
    
    /* تنسيقات التبويب */
    .tab {
      display: none;
      padding: 20px;
    }
    .tab.active {
      display: block;
    }
    
    /* تنسيقات شريط التنقل */
    .navbar {
      background-color: #00796b;
      overflow: hidden;
      display: flex;
      justify-content: center;
    }
    .navbar button {
      float: right;
      color: white;
      text-align: center;
      padding: 14px 16px;
      font-size: 17px;
      border: none;
      background: none;
      cursor: pointer;
    }
    .navbar button:hover {
      background-color: #004d40;
    }
    
    /* تنسيقات الصفحة الرئيسية */
    #home {
      text-align: center;
      padding: 40px;
    }
    #home h1 {
      font-size: 28px;
      margin-bottom: 30px;
    }
    #home button {
      font-size: 18px;
      margin: 15px;
      padding: 15px 40px;
      border: none;
      background-color: #00796b;
      color: white;
      border-radius: 12px;
      cursor: pointer;
    }
    #home button:hover {
      background-color: #004d40;
    }
    
    /* تنسيقات صفحة توليد الباركود */
    #generator {
      background-color: #e8f5e9;
      text-align: center;
      padding: 30px;
    }
    #generator input, #generator select {
      margin: 8px;
      padding: 10px;
      width: 300px;
      border-radius: 8px;
      border: 1px solid #aaa;
    }
    #generator button {
      padding: 10px 30px;
      font-size: 16px;
      background-color: #388e3c;
      color: white;
      border: none;
      border-radius: 8px;
      margin-top: 10px;
    }
    #qrcode {
      margin-top: 20px;
    }
    
    /* تنسيقات صفحة الماسح الضوئي */
    #scanner {
      background-color: #fff3e0;
      text-align: center;
      padding: 20px;
    }
    #reader {
      width: 300px;
      margin: auto;
    }
    .log {
      margin-top: 20px;
      font-size: 18px;
    }
    
    /* تنسيقات صفحة سجل الحضور */
    #attendance-log {
      background: #f3e5f5;
      padding: 20px;
      text-align: center;
    }
    #attendanceTable {
      width: 100%;
      margin-top: 20px;
      border-collapse: collapse;
    }
    #attendanceTable th, #attendanceTable td {
      border: 1px solid #aaa;
      padding: 8px;
    }
    #dateSelector {
      padding: 10px;
      margin-bottom: 20px;
    }
  </style>
</head>
<body>
  <!-- شريط التنقل -->
  <div class="navbar">
    <button onclick="openTab('home')">🏠 الرئيسية</button>
    <button onclick="openTab('generator')">🎫 توليد باركود</button>
    <button onclick="openTab('scanner')">📷 مسح الباركود</button>
    <button onclick="openTab('attendance-log')">📅 سجل الحضور</button>
  </div>

  <!-- الصفحة الرئيسية -->
  <div id="home" class="tab active">
    <h1>📋 نظام تسجيل الحضور بالباركود</h1>
    <button onclick="openTab('generator')">🎫 توليد باركود</button><br>
    <button onclick="openTab('scanner')">📷 مسح الباركود</button><br>
    <button onclick="openTab('attendance-log')">📅 سجل الحضور</button>
  </div>

  <!-- صفحة توليد الباركود -->
  <div id="generator" class="tab">
    <h2>🧾 توليد باركود لعضو</h2>
    <input type="text" id="name" placeholder="الاسم الثلاثي"><br>
    <input type="number" id="age" placeholder="العمر"><br>
    <select id="group">
      <option value="كشاف">كشاف</option>
      <option value="أشبال">أشبال</option>
    </select><br>
    <input type="text" id="number" placeholder="رقم العضو"><br>
    <input type="text" id="guardian" placeholder="رقم ولي الأمر"><br>
    <input type="text" id="address" placeholder="السكن"><br>
    <button onclick="generateQRCode()">🎫 توليد</button>
    <div id="qrcode"></div>
  </div>

  <!-- صفحة الماسح الضوئي -->
  <div id="scanner" class="tab">
    <h2>📷 مسح الباركود وتسجيل الحضور</h2>
    <div id="reader"></div>
    <div class="log" id="log"></div>
  </div>

  <!-- صفحة سجل الحضور -->
  <div id="attendance-log" class="tab">
    <h2>📅 سجل الحضور حسب التاريخ</h2>
    <select id="dateSelector" onchange="loadAttendance()"></select>
    <table id="attendanceTable">
      <thead>
        <tr>
          <th>الاسم</th><th>العمر</th><th>الفئة</th><th>الرقم</th><th>ولي الأمر</th><th>السكن</th><th>الوقت</th>
        </tr>
      </thead>
      <tbody></tbody>
    </table>
  </div>

  <script>
    // وظيفة تبديل التبويبات
    function openTab(tabName) {
      const tabs = document.getElementsByClassName('tab');
      for (let i = 0; i < tabs.length; i++) {
        tabs[i].classList.remove('active');
      }
      document.getElementById(tabName).classList.add('active');
      
      // إعادة تهيئة الماسح عند فتح تبويب المسح
      if (tabName === 'scanner') {
        initScanner();
      }
      
      // تحديث سجل الحضور عند فتح التبويب
      if (tabName === 'attendance-log') {
        fillDateOptions();
      }
    }

    // وظائف توليد الباركود
    function generateQRCode() {
      const data = {
        name: document.getElementById('name').value,
        age: document.getElementById('age').value,
        group: document.getElementById('group').value,
        number: document.getElementById('number').value,
        guardian: document.getElementById('guardian').value,
        address: document.getElementById('address').value
      };
      const jsonData = JSON.stringify(data);
      document.getElementById('qrcode').innerHTML = "";
      QRCode.toCanvas(document.createElement('canvas'), jsonData, { width: 256 }, function (err, canvas) {
        if (!err) document.getElementById('qrcode').appendChild(canvas);
      });
    }

    // وظائف الماسح الضوئي
    let html5QrcodeScanner;
    
    function initScanner() {
      document.getElementById('reader').innerHTML = '';
      document.getElementById('log').innerText = '';
      
      html5QrcodeScanner = new Html5Qrcode("reader");
      html5QrcodeScanner.start(
        { facingMode: "environment" },
        { fps: 10, qrbox: 250 },
        onScanSuccess
      ).catch(err => {
        document.getElementById('log').innerText = "❌ حدث خطأ في تشغيل الكاميرا";
      });
    }
    
    function onScanSuccess(decodedText) {
      try {
        const data = JSON.parse(decodedText);
        saveAttendance(data);
      } catch {
        document.getElementById('log').innerText = "❌ لم يتم التعرف على الباركود.";
      }
    }
    
    function saveAttendance(data) {
      const today = new Date().toISOString().split('T')[0];
      let attendance = JSON.parse(localStorage.getItem(`attendance-${today}`) || "[]");
      const found = attendance.find(item => item.name === data.name);
      if (!found) {
        data.time = new Date().toLocaleTimeString();
        attendance.push(data);
        localStorage.setItem(`attendance-${today}`, JSON.stringify(attendance));
        document.getElementById('log').innerText = `✅ تم تسجيل حضور: ${data.name}`;
      } else {
        document.getElementById('log').innerText = `⚠️ ${data.name} تم تسجيله سابقًا`;
      }
    }

    // وظائف سجل الحضور
    function getDatesWithAttendance() {
      return Object.keys(localStorage).filter(key => key.startsWith("attendance-")).map(key => key.replace("attendance-", ""));
    }

    function loadAttendance() {
      const date = document.getElementById("dateSelector").value;
      const data = JSON.parse(localStorage.getItem(`attendance-${date}`) || "[]");
      const tbody = document.querySelector("#attendanceTable tbody");
      tbody.innerHTML = "";
      data.forEach(item => {
        const row = `<tr><td>${item.name}</td><td>${item.age}</td><td>${item.group}</td><td>${item.number}</td><td>${item.guardian}</td><td>${item.address}</td><td>${item.time}</td></tr>`;
        tbody.innerHTML += row;
      });
    }

    function fillDateOptions() {
      const dates = getDatesWithAttendance();
      const selector = document.getElementById("dateSelector");
      selector.innerHTML = "";
      dates.forEach(d => selector.innerHTML += `<option value="${d}">${d}</option>`);
      if (dates.length) loadAttendance();
    }
  </script>
</body>
</html>
