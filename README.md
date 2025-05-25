<!DOCTYPE html>
<html lang="ar">
<head>
  <meta charset="UTF-8" />
  <title>القرآن الكريم</title>
  <style>
    @import url('https://fonts.googleapis.com/css2?family=Amiri&display=swap');

    body {
      font-family: 'Amiri', serif;
      direction: rtl;
      background-color: #f4f4f4;
      padding: 20px;
      text-align: center;
      transition: background-color 0.3s, color 0.3s;
    }
    body.dark {
      background-color: #121212;
      color: #ffffff;
    }
    h1 {
      color: #2c3e50;
    }
    body.dark h1 {
      color: #f9f9f9;
    }
    .search-box {
      margin: 20px auto;
    }
    .search-box input {
      padding: 10px;
      width: 60%;
      border-radius: 5px;
      border: 1px solid #ccc;
      text-align: right;
    }
    .surah-list {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      gap: 10px;
    }
    .surah-button {
      background-color: #fff;
      border: 1px solid #ccc;
      padding: 10px 15px;
      border-radius: 5px;
      cursor: pointer;
    }
    .surah-button:hover {
      background-color: #e0e0e0;
    }
    .surah-content {
      margin-top: 20px;
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    body.dark .surah-content {
      background: #1e1e1e;
    }
    .ayah {
      margin: 10px 0;
      cursor: pointer;
    }
    .ayah:hover {
      background: #eee;
    }
    body.dark .ayah:hover {
      background: #333;
    }
    .fav-list {
      margin-top: 20px;
      padding: 10px;
      background: #ddf;
      border-radius: 10px;
    }
    body.dark .fav-list {
      background: #333366;
    }
    audio {
      width: 100%;
      margin-top: 10px;
    }
    .tafsir {
      margin-top: 15px;
      background: #eef;
      padding: 10px;
      border-radius: 5px;
      text-align: right;
    }
    body.dark .tafsir {
      background: #2a2a6a;
    }
    .toggle-dark {
      position: absolute;
      left: 20px;
      top: 20px;
      padding: 5px 10px;
      cursor: pointer;
      background: #ddd;
      border: none;
      border-radius: 5px;
    }
  </style>
</head>
<body>
  <button class="toggle-dark" onclick="toggleDarkMode()">🌙</button>
  <h1>القرآن الكريم</h1>

  <div class="search-box">
    <input type="text" id="searchInput" placeholder="ابحث في الآيات أو اسم السورة أو رقم الآية..." oninput="searchAyat()" />
    <div style="margin-top: 10px;">
      <label>التفسير:
        <select id="tafsirSelect">
          <option value="ar.jalalayn">الجلالين</option>
          <option value="ar.saadi">السعدي</option>
        </select>
      </label>
      <label style="margin-right: 15px;">الترجمة:
        <select id="translationSelect">
          <option value="">بدون ترجمة</option>
          <option value="en.sahih">Sahih International</option>
          <option value="en.yusufali">Yusuf Ali</option>
        </select>
      </label>
    </div>
  </div>

  <div class="surah-list" id="surahList"></div>
  <div class="surah-content" id="surahContent"></div>
  <div class="fav-list" id="favList"></div>

  <script>
    let surahsData = [];

    async function loadQuranData() {
      const response = await fetch('https://api.alquran.cloud/v1/quran/ar.alafasy');
      const data = await response.json();
      surahsData = data.data.surahs;
      displaySurahList();
      loadLastRead();
      loadFavorites();
    }

    function displaySurahList() {
      const surahList = document.getElementById('surahList');
      surahList.innerHTML = '';
      surahsData.forEach(surah => {
        const button = document.createElement('button');
        button.textContent = `${surah.number} - ${surah.englishName}`;
        button.className = 'surah-button';
        button.onclick = () => displaySurah(surah);
        surahList.appendChild(button);
      });
    }

    function displaySurah(surah) {
      const surahContent = document.getElementById('surahContent');
      surahContent.innerHTML = `<h2>${surah.name}</h2>`;
      surah.ayahs.forEach(ayah => {
        const p = document.createElement('p');
        p.className = 'ayah';
        p.innerHTML = `${ayah.text} ﴿${ayah.numberInSurah}﴾ <button onclick="toggleFavorite(${ayah.number}, event)">☆</button>`;
        p.querySelector('button').style.marginLeft = '10px';
        p.querySelector('button').style.cursor = 'pointer';
        p.onclick = (e) => {
          if (e.target.tagName.toLowerCase() !== 'button') {
            showTafsir(ayah, p);
          }
        };
        surahContent.appendChild(p);
      });
      surahContent.innerHTML += `
        <audio controls>
          <source src="https://server8.mp3quran.net/afs/${String(surah.number).padStart(3, '0')}.mp3" type="audio/mpeg" />
          المتصفح لا يدعم تشغيل الصوت.
        </audio>
      `;
      localStorage.setItem('lastRead', surah.number);
    }

    async function showTafsir(ayah, ayahElement) {
      const tafsirChoice = document.getElementById('tafsirSelect').value;
      const translationChoice = document.getElementById('translationSelect').value;

      if (ayahElement.nextElementSibling && ayahElement.nextElementSibling.classList.contains('tafsir')) {
        ayahElement.nextElementSibling.remove();
        return;
      }

      const tafsirBox = document.createElement('div');
      tafsirBox.className = 'tafsir';
      tafsirBox.textContent = 'جاري تحميل التفسير...';
      ayahElement.after(tafsirBox);

      try {
        const tafsirRes = await fetch(`https://api.alquran.cloud/v1/ayah/${ayah.number}/${tafsirChoice}`);
        const tafsirData = await tafsirRes.json();
        tafsirBox.innerHTML = `<strong>التفسير:</strong><br>${tafsirData.data.text || 'غير متوفر'}`;
      } catch {
        tafsirBox.innerHTML = '<strong>التفسير:</strong><br>حدث خطأ.';
      }

      if (translationChoice) {
        try {
          const transRes = await fetch(`https://api.alquran.cloud/v1/ayah/${ayah.number}/${translationChoice}`);
          const transData = await transRes.json();
          tafsirBox.innerHTML += `<hr><strong>الترجمة:</strong><br>${transData.data.text || 'غير متوفرة'}`;
        } catch {
          tafsirBox.innerHTML += `<hr><strong>الترجمة:</strong><br>فشل في جلب الترجمة.`;
        }
      }
    }

    function searchAyat() {
      const query = document.getElementById('searchInput').value.trim();
      const surahContent = document.getElementById('surahContent');
      if (query.length < 2) return;

      surahContent.innerHTML = `<h2>نتائج البحث عن: "${query}"</h2>`;
      let found = false;

      surahsData.forEach(surah => {
        surah.ayahs.forEach(ayah => {
          if (
            ayah.text.includes(query) ||
            surah.name.includes(query) ||
            ayah.numberInSurah.toString() === query
          ) {
            const p = document.createElement('p');
            p.className = 'ayah';
            p.textContent = `${ayah.text} ﴿${ayah.numberInSurah}﴾ - ${surah.name}`;
            p.onclick = () => showTafsir(ayah, p);
            surahContent.appendChild(p);
            found = true;
          }
        });
      });

      if (!found) {
        surahContent.innerHTML += '<p>لا توجد نتائج مطابقة.</p>';
      }
    }

    function toggleDarkMode() {
      document.body.classList.toggle('dark');
    }

    function toggleFavorite(ayahNumber, event) {
      event.stopPropagation();
      let favs = JSON.parse(localStorage.getItem('favorites')) || [];
      if (favs.includes(ayahNumber)) {
        favs = favs.filter(id => id !== ayahNumber);
      } else {
        favs.push(ayahNumber);
      }
      localStorage.setItem('favorites', JSON.stringify(favs));
      loadFavorites();
    }

    function loadFavorites() {
      const favs = JSON.parse(localStorage.getItem('favorites')) || [];
      const favList = document.getElementById('favList');
      favList.innerHTML = '<h3>الآيات المفضلة</h3>';
      if (favs.length === 0) {
        favList.innerHTML += '<p>لا توجد آيات مفضلة حتى الآن.</p>';
        return;
      }

      surahsData.forEach(surah => {
        surah.ayahs.forEach(ayah => {
          if (favs.includes(ayah.number)) {
            const p = document.createElement('p');
            p.textContent = `${ayah.text} ﴿${ayah.numberInSurah}﴾ - ${surah.name}`;
            favList.appendChild(p);
          }
        });
      });
    }

    function loadLastRead() {
      const last = localStorage.getItem('lastRead');
      if (last) {
        const surah = surahsData.find(s => s.number == last);
        if (surah) displaySurah(surah);
      }
    }

    window.onload = loadQuranData;
  </script>
</body>
</html> 
"""
# 📖 موقع القرآن الكريم

موقع ويب بسيط وتفاعلي يعرض سور وآيات القرآن الكريم مع إمكانية البحث، الاستماع للتلاوة، عرض التفسير، الترجمة، وتحديد المفضلات.

## 💡 الميزات

- عرض جميع سور وآيات القرآن الكريم.
- البحث داخل الآيات.
- التفسير من تفسير الجلالين.
- الترجمة (يمكنك تفعيلها باستخدام API مخصص).
- حفظ الآيات المفضلة.
- حفظ آخر قراءة.
- الوضع الداكن / الفاتح.
- الاستماع لتلاوة الشيخ مشاري العفاسي.

## 🛠️ التقنيات المستخدمة

- HTML + CSS
- JavaScript (Vanilla)
- [API من alquran.cloud](https://alquran.cloud/)
- خط [Amiri من Google Fonts](https://fonts.google.com/specimen/Amiri)

## 🚀 طريقة الاستخدام

1. حمّل الملفات أو انسخها إلى مجلد في جهازك.
2. افتح الملف `index.html` في متصفح حديث (مثل Chrome أو Firefox).
3. تأكد من اتصالك بالإنترنت لاستخدام التلاوة والتفسير.

## 🌐 المصادر

- [القرآن API - Al Quran Cloud](https://alquran.cloud/)
- [مشاري العفاسي - mp3quran.net](https://mp3quran.net/afs)

## 📝 الملاحظات

- إذا رغبت في دعم الترجمات، استخدم Endpoint آخر من `alquran.cloud` (مثل `/en.yusufali`).
- لضمان تجربة مثالية، ينصح باستخدام الموقع على متصفحات حديثة فقط.

## 📄 الترخيص

مشروع مفتوح المصدر لأغراض تعليمية ودعوية، يمكن التعديل والاستخدام بحرية مع ذكر المصدر.

---

> هذا المشروع تم إنشاؤه يدويًا لتسهيل الوصول إلى القرآن الكريم عبر الويب.
"""

# حفظ الملف
file_path = "/mnt/data/README.md"
with open(file_path, "w", encoding="utf-8") as f:
    f.write(readme_content)

file_path
