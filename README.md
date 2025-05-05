https://[اسم_المستخدم].github.io/mdt-system/

from zipfile import ZipFile
import os

# إعداد ملفات المشروع
project_name = "mdt_system"
form_html = """
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>تعبئة تقرير MDT</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>نموذج تعبئة تقرير MDT</h1>
        <form id="reportForm">
            <label>اسم العسكري:</label>
            <input type="text" name="officer" required>

            <label>حالة المهمة:</label>
            <select name="missionStatus" required>
                <option>جارية</option>
                <option>ناجحة</option>
                <option>فاشلة</option>
            </select>

            <label>نوع الحالة:</label>
            <select name="caseType" required>
                <option>جنائية</option>
                <option>مرورية</option>
                <option>آخر</option>
            </select>

            <label>المخالفة:</label>
            <select name="violations" multiple size="8">
                <option>محاولة اختطاف : 25شهر : 500$</option>
                <option>اختطاف : 40شهر : 700$</option>
                <option>اختطاف رجل دولة : 70شهر : 1500$</option>
                <option>قتال الشوارع : 40شهر : 750$</option>
                <option>محاولة تهريب مجرم : 40شهر : 750$</option>
                <option>محاولة تهريب مجرم : 70شهر : 1500$</option>
                <option>الهروب من السجن : 50شهر : 850$</option>
                <option>حمل بطاقة غير عائدة له : 30شهر : 400$</option>
                <option>سرقة المواطنين : 30شهر : 400$</option>
                <option>سرقة سيارة : 20شهر : 100$</option>
                <option>سرقة بقالة : 30شهر : 1000$</option>
                <option>سرقة الصراف الآلي : 45شهر : 2500$</option>
                <option>سرقة منزل : 50شهر : 5000$</option>
                <option>سرقة بوب كات : 85شهر : 10000$</option>
                <option>سرقة بنك فليكا : 100شهر : 15000$</option>
                <option>الهروب من الشرطة : 25شهر : 300$</option>
                <option>التسوّول : 20شهر : 100$</option>
                <option>ذوق عام : 20شهر : 50$</option>
                <option>تجارة بالأسلحة من الدرجة الأولى : 60شهر : 3000$</option>
                <option>تجارة بالأسلحة من الدرجة الثانية : 60شهر : 5000$</option>
                <option>تجارة بالأسلحة من الدرجة الثالثة : 160شهر : 10000$</option>
                <option>حيازة سلاح غير مرخّص : 45شهر : $100</option>
            </select>

            <label>اسم المجرم:</label>
            <input type="text" name="criminalName" required>

            <label>صورة المجرم:</label>
            <input type="file" name="criminalImage" accept="image/*">

            <label>معلومات إضافية:</label>
            <textarea name="notes"></textarea>

            <button type="submit">إرسال التقرير</button>
        </form>
        <a href="reports.html">عرض التقارير</a>
    </div>
    <script src="script.js"></script>
</body>
</html>
"""

reports_html = """
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>تقارير MDT</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <h1>سجل تقارير MDT</h1>
        <input type="text" id="search" placeholder="ابحث باسم المجرم...">
        <button onclick="clearReports()">مسح كل التقارير</button>
        <div id="reportsContainer"></div>
        <a href="index.html">الرجوع للتعبئة</a>
    </div>
    <script src="reports.js"></script>
</body>
</html>
"""

style_css = """
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    direction: rtl;
}
.container {
    width: 90%;
    max-width: 800px;
    margin: 20px auto;
    background: white;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 0 10px rgba(0,0,0,0.1);
}
form input, form select, form textarea, button {
    width: 100%;
    margin-bottom: 15px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
}
button {
    background-color: #2c3e50;
    color: white;
    border: none;
    cursor: pointer;
}
button:hover {
    background-color: #1a242f;
}
#reportsContainer .report {
    border-bottom: 1px solid #ddd;
    padding: 10px 0;
}
"""

script_js = """
document.getElementById('reportForm').addEventListener('submit', function(e) {
    e.preventDefault();
    const form = e.target;
    const reader = new FileReader();
    const file = form.criminalImage.files[0];

    reader.onloadend = function() {
        const report = {
            officer: form.officer.value,
            missionStatus: form.missionStatus.value,
            caseType: form.caseType.value,
            violations: Array.from(form.violations.selectedOptions).map(opt => opt.value),
            criminalName: form.criminalName.value,
            criminalImage: reader.result || "",
            notes: form.notes.value
        };
        const reports = JSON.parse(localStorage.getItem('mdtReports') || '[]');
        reports.push(report);
        localStorage.setItem('mdtReports', JSON.stringify(reports));
        alert("تم حفظ التقرير!");
        form.reset();
    };

    if (file) reader.readAsDataURL(file);
    else reader.onloadend();
});
"""

reports_js = """
function loadReports() {
    const container = document.getElementById('reportsContainer');
    const reports = JSON.parse(localStorage.getItem('mdtReports') || '[]');
    const search = document.getElementById('search').value.trim();

    container.innerHTML = '';
    reports.forEach((report, index) => {
        if (search && !report.criminalName.includes(search)) return;
        const div = document.createElement('div');
        div.className = 'report';
        div.innerHTML = `
            <h3>المجرم: ${report.criminalName}</h3>
            <p><strong>اسم العسكري:</strong> ${report.officer}</p>
            <p><strong>حالة المهمة:</strong> ${report.missionStatus}</p>
            <p><strong>نوع الحالة:</strong> ${report.caseType}</p>
            <p><strong>المخالفات:</strong> ${report.violations.join(', ')}</p>
            <p><strong>ملاحظات:</strong> ${report.notes}</p>
            ${report.criminalImage ? `<img src="${report.criminalImage}" alt="صورة" style="max-width:200px;">` : ''}
        `;
        container.appendChild(div);
    });
}

function clearReports() {
    if (confirm('هل أنت متأكد من مسح جميع التقارير؟')) {
        localStorage.removeItem('mdtReports');
        loadReports();
    }
}

document.getElementById('search').addEventListener('input', loadReports);
window.onload = loadReports;
"""

# إنشاء ملفات المشروع داخل مجلد مؤقت
project_files = {
    "index.html": form_html,
    "reports.html": reports_html,
    "style.css": style_css,
    "script.js": script_js,
    "reports.js": reports_js
}

os.makedirs(f"/mnt/data/{project_name}", exist_ok=True)
for filename, content in project_files.items():
    with open(f"/mnt/data/{project_name}/{filename}", "w", encoding="utf-8") as f:
        f.write(content)

# ضغط الملفات في ملف ZIP
zip_path = f"/mnt/data/{project_name}.zip"
with ZipFile(zip_path, 'w') as zipf:
    for filename in project_files.keys():
        zipf.write(f"/mnt/data/{project_name}/{filename}", arcname=filename)

zip_path
