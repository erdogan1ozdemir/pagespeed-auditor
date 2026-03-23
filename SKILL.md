---
name: pagespeed-auditor
description: >
  Bir veya birden fazla URL icin PageSpeed Insights analizi yapip PPTX sunum ve destekleyici
  Excel ciktisi ureten skill. Bagimsiz cagrilabilir 6 sub-skill icerir: render pipeline,
  javascript yuku, LCP optimizasyonu, CLS kararliligi, INP etkilesim, kalite ve erisilebilirlik.
  Bu skill'i su durumlarda kullan: "pagespeed analiz et", "site hizi raporu hazirla",
  "LCP bak", "CLS sorunlari", "INP analiz et", "javascript audit yap", "accessibility kontrol et",
  "render pipeline", "sayfa hizi sunumu", "performans raporu", "core web vitals",
  "pagespeed sunum hazirla", "site hizi optimizasyonu", "web vitals analiz",
  "erisilebilirlik raporu", "3rd party scriptler", "gorsel optimizasyonu",
  kullanici bir URL verip performans, hiz, CWV, Lighthouse veya PageSpeed ile ilgili herhangi
  bir analiz, rapor veya sunum istediginde bu skill tetiklenir. Tek bir sub-skill veya tam audit
  istenebilir. Turkce, Ingilizce ve Almanca cikti destekler.
---

# PageSpeed Auditor

Site performansini analiz edip IT ekiplerine yonelik aksiyon odakli sunum ve rapor ureten skill.

## Calisma modu: OTONOM

Kullanici URL'leri ve (varsa) sub-skill secimini verdikten sonra tum is akisini otonom calistir.
Ara adimlarda onay isteme - API cagrilarini yap, verileri topla, analiz et, ciktiyi uret.

**Izin istemeden yap:**
- PSI API veya DataForSEO Lighthouse cagrilari
- web_fetch ile HTML kaynak kodu cekme
- Chrome DevTools kontrolu (varsa)
- Screenshot ve filmstrip alma
- Sunum ve Excel dosyasi olusturma
- Reference dosyalarini okuma
- Paket yukleme (pptxgenjs, openpyxl vb.)
- Google dokumantasyonu arastirma (web_search)

**Yalnizca su durumlarda kullaniciya sor:**
- URL verilmemisse
- API key gerekiyorsa ve bulunamiyorsa
- Kritik hata sonrasi tum retry yollari tukenmisse

---

## 1. Ortam algilama

Skill basinda ortami tespit et:

```
Chrome extension var mi?
  EVET → 3 katmanli veri toplama:
    PSI API veya DataForSEO Lighthouse
    Chrome DevTools (DOM, network, screenshot)
    DataForSEO on_page (ek veri)

  HAYIR → 2 katmanli veri toplama:
    PSI API veya DataForSEO Lighthouse
    web_fetch ile HTML parse
```

Chrome kontrol: tool_search ile "tabs_context_mcp" ara. Bulunamazsa Chrome yok.

---

## 2. API key yonetimi

Sirasiyla kontrol et:
1. DataForSEO MCP bagli mi? → on_page_lighthouse kullan, PSI key gerekmez
2. Kullanici onceden key verdiyse → kullan
3. Hicbiri yoksa → kullaniciya sor:
   "PSI API key gereklidir. Ucretsiz anahtar icin:
   https://developers.google.com/speed/docs/insights/v5/get-started
   Anahtarinizi buraya yapistirin."

DataForSEO varsa tercih et - PSI key gerektirmez, full Lighthouse JSON dondurur.

---

## 3. Sub-skill yonlendirme

| # | Sub-skill | Tetikleyiciler | Reference |
|---|-----------|----------------|-----------|
| 1 | Render pipeline | "render pipeline", "TTFB", "sunucu hizi", "render-blocking", "cache" | references/render-pipeline.md |
| 2 | JavaScript yuku | "javascript audit", "JS yuku", "3rd party", "script analizi" | references/javascript-burden.md |
| 3 | LCP optimizasyonu | "LCP", "gorsel optimizasyonu", "largest contentful paint", "gorsel boyut" | references/lcp-optimization.md |
| 4 | CLS kararliligi | "CLS", "layout shift", "sayfa kayiyor", "kayma" | references/cls-stability.md |
| 5 | INP etkilesim | "INP", "etkilesim", "TBT", "tepki suresi", "DOM boyutu" | references/inp-interactivity.md |
| 6 | Kalite ve Erisilebilirlik | "accessibility", "erisilebilirlik", "best practices", "alt text" | references/quality-accessibility.md |

"Tam audit", "full pagespeed", "tum metriklere bak" → 6 sub-skill sirasiyla calisir.

**Her sub-skill calistirilmadan once ilgili reference dosyasi okunmalidir.**

---

## 4. Sayfa tipi algilama

Kullanici sadece domain verirse otomatik algila (maksimum 3 sayfa tipi):

1. Anasayfa (domain root) her zaman dahil
2. web_fetch ile anasayfa HTML'ini cek, navigasyon linklerinden sayfa tipleri bul:
   - E-ticaret tespiti: /product/, /p-, /urun/, /category/, /c-, /kategori/ patternleri
   - Kurumsal tespiti: /hizmet/, /service/, /blog/, /icerik/ patternleri
3. E-ticaret → anasayfa + urun sayfasi + kategori sayfasi
4. Kurumsal → anasayfa + hizmet sayfasi + en onemli icerik sayfasi
5. Kullanici spesifik URL verdiyse otomatik algilama yapma

---

## 5. Veri toplama

Her URL icin mobil VE desktop ayri ayri analiz edilir.

### 5.1 PSI API cagrisi
```
Parametreler:
  url: hedef URL
  strategy: mobile veya desktop
  category: sub-skill'e gore
    Sub-skill 1-5: performance
    Sub-skill 6: performance,accessibility,best-practices
    Tam audit: performance,accessibility,best-practices
```

Cekilecek veriler:
- `loadingExperience.metrics` → Field data (CrUX) - BU HER ZAMAN BIRINCIL KAYNAK
- `lighthouseResult.audits` → Audit detaylari
- `lighthouseResult.audits['final-screenshot'].details.data` → Screenshot (base64)
- `lighthouseResult.audits['screenshot-thumbnails'].details.items` → Filmstrip (base64 array)
- `lighthouseResult.categories` → Kategori skorlari

Field data yoksa (yetersiz trafik) bunu sunumda acikca belirt.

### 5.2 HTML kaynak analizi

Chrome varsa: navigate + javascript_tool ile DOM tara
Chrome yoksa: web_fetch ile HTML cek, parse et

**TUM elementler taranir. SAMPLING (slice/limit) KESINLIKLE YAPILMAZ.**

### 5.3 DataForSEO ek veri kaynaklari

**on_page_lighthouse:** PSI API alternatifi. full_data: true ile tam Lighthouse JSON dondurur.
Screenshot, filmstrip, tum audit detaylari dahil. PSI API key gerektirmez.

**on_page_instant_pages:** custom_js parametresiyle sayfada JS calistirabilir.
Chrome DevTools'taki gibi DOM tarama yapilabilir (img attribute kontrolu, script tarama,
layout shift olcumu vb.). Claude Code ortaminda Chrome yokken en guclu alternatif budur.
```
Ornek: on_page_instant_pages(
  url: "https://example.com",
  enable_javascript: true,
  custom_js: "document.querySelectorAll('img').length"
)
```

**on_page_content_parsing:** Heading yapisi, link yapisi, yapisal icerik analizi.

---

## 6. Batch yonetimi

- Istekleri siraya koy: URL1-mobile, URL1-desktop, URL2-mobile...
- PSI API cagrilari arasinda 3 saniye bekle
- Basarisiz cagrida 1 kez tekrar dene, sonra DataForSEO'ya fallback
- DataForSEO da basarisizsa "analiz edilemedi" olarak raporla

---

## 7. Cikti: PPTX sunum

Once pptx skill'ini oku: `view /mnt/skills/public/pptx/SKILL.md` ve `view /mnt/skills/public/pptx/pptxgenjs.md`

### Tasarim kurallari

Detayli tasarim rehberi icin: `view references/presentation-design.md`
Temel kurallar:
- Kapak/kapatis: Coral (#F4845F) arka plan, beyaz metin
- Bolum ayiraclari: Dark Teal (#1B3A36), numarali, coral cizgi
- Icerik slaytlari: Off-White (#FAF8F5) arka plan
- Baslik fontu: Bricolage Grotesque (fallback: Calibri)
- Govde fontu: Outfit (fallback: Calibri)
- Basliklar: Duz dark teal metin, coral highlight blogu KULLANMA
- Breadcrumb: sol ust, coral, "Bolum | Slayt" formati
- Heat-map tablo: kirmizi (#FFCDD2 + #D32F2F) negatif, yesil (#C8E6C9 + #2E7D32) pozitif
- Insight: ➔ ile baslar
- Tire: EM DASH (—) KULLANMA, kisa tire (-) kullan
- Slide boyutu: LAYOUT_WIDE (13.33 x 7.5 inch)

### Filmstrip ve screenshot gomu

PSI API'den gelen base64 JPEG sunuma gomulur. DIKDORTGEN gosterilir, DAIRE veya OVAL KIRPMA YAPMA:
```javascript
// DOGRU: Dikdortgen gorsel
slide.addImage({
  data: 'image/jpeg;base64,' + base64Data,
  x: 0.5, y: 2.5, w: 4, h: 3
});

// YANLIS: Daire kirpma - YAPMA
// rounding: true veya shape: 'OVAL' KULLANMA
```

Filmstrip: 3-5 kareyi yan yana dikdortgen olarak goster. Her karenin altina zamanlama yaz (830ms, 4150ms vb.).

### Icerik yogunlugu kurali (BOS ALAN BIRAKILMAZ)

Slaytlarin alt yarisi BOS KALMAMALI. Icerik tum slayti kaplamali:
- Metin yeterli degilse: gorsel, screenshot, tablo veya ek aciklama ekle
- Tespit slaytinda: screenshot kaniti, filmstrip, etkilenen element gorseli ekle
- Etki slaytinda: metrik karsilastirma tablosu, Google referans kutusu ekle
- Cozum slaytinda: mevcut vs onerilen kod karsilastirmasi kutu icinde, efor/oncelik kartlari ekle
- Bos alan kaliyorsa: ek not, ipucu veya ilgili kaynak linki ekle
- Her slaytta en az 1 gorsel element olmali (tablo, chart, screenshot, kod kutusu vb.)

### Kaynak badge

Her icerik slaytinin sol alt kosesine coral pill badge ekle:
```javascript
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: 0.3, y: 6.8, w: 3.5, h: 0.35, fill: { color: 'F4845F' }, rectRadius: 0.15
});
slide.addText('Kaynak: PageSpeed Insights API', {
  x: 0.3, y: 6.8, w: 3.5, h: 0.35, fontSize: 10, color: 'FFFFFF',
  fontFace: 'Outfit', bold: true, align: 'center', valign: 'middle'
});
```

### Kod bloklari

Mevcut ve onerilen HTML/CSS kodlarini acik gri arka planli kutu icinde goster:
```javascript
// Kod kutusu
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: 0.5, y: 3.5, w: 12, h: 1.5, fill: { color: 'F5F5F5' },
  line: { color: 'E0E0E0', width: 0.5 }, rectRadius: 0.1
});
slide.addText(codeContent, {
  x: 0.7, y: 3.6, w: 11.6, h: 1.3, fontSize: 11,
  fontFace: 'Consolas', color: '1B3A36', valign: 'top'
});
```

### Sunum akisi (her sub-skill icin)

```
KAPAK (coral)
GENEL DEGERLENDIRME (skor tablosu + ozet paragraf)
METRIK ACIKLAMA NOTU (IT ekibi icin)

HER SAYFA TIPI x CIHAZ ICIN (ornek: Anasayfa Mobil, Anasayfa Desktop, Urun Mobil...):
  BOLUM AYIRACI (dark teal, numarali)
  SKOR KARTI + FILMSTRIP

  HER BULGU ICIN AYRI SLAYTLAR (tek slaytta birlestirme YAPMA):
    TESPIT SLAYTI
    ETKI SLAYTI
    COZUM ONERISI SLAYTI

OZET AKSIYON TABLOSU (sayfa x bulgu matrisi)
ONCELIK SIRALAMASI / YOL HARITASI
KAPATIS (coral, tesekkurler)
```

ONEMLI KURALLAR:
- Her sayfa tipi icin mobil ve desktop AYRI AYRI analiz edilir ve ayri bolumler halinde sunulur
- Urun ve kategori sayfalarindaki bulgular da AYRI SLAYTLARDA gosterilir, tek slaytta birlestirilMEZ
- Her bulgu icin 2-3 slayt kullanilir (tespit + etki + cozum). Basit bulgularda etki ve cozum birlesebilir
- Ortak sorunlar (CSS, font vb.) her sayfada tekrarlanmaz, ilk goruldugu yerde detayli anlatilir,
  diger sayfalarda "Anasayfa bolumunde detaylanan CSS optimizasyonu bu sayfa icin de gecerlidir" denir

### Tespit slayti icerigi (dolu olmali)

Slayt iceriginin tum slayti kaplamasi icin su elementler bulunmali:
- Breadcrumb (sol ust, coral)
- Baslik (dark teal, buyuk)
- 2-3 paragraf aciklama metni (➔ ile baslayan maddeler)
- Etkilenen element bilgisi (CSS selector, coral metin)
- Mevcut HTML kod kutusu (acik gri arka plan, monospace font)
- Screenshot veya filmstrip kaniti (dikdortgen gorsel)
- Kaynak badge (sol alt)

### Etki slayti icerigi

- Kullanici deneyimi etkisi (en az 2-3 cumle)
- Metrik etkisi (sayisal, tablolu)
- Diger metriklere etkisi
- Google referansi (web.dev linki, coral metin)

### Cozum onerisi slayti icerigi

- Onerilen adimlar (numarali, her biri 2-3 cumle)
- Mevcut vs onerilen kod karsilastirmasi (2 kod kutusu yan yana veya ust uste)
- Beklenen iyilesme degeri (yesil renk)
- Oncelik / Efor / Sorumluluk kartlari (3 kart yan yana)
- Excel referans notu (spesifik: "dosya.xlsx, Sheet Adi sayfasi, satir X-Y")

---

## 8. Cikti: Excel rapor

Once xlsx skill'ini oku: `view /mnt/skills/public/xlsx/SKILL.md`

### Sheet yapisi

**Sheet 1 - Ozet:**
Tum sayfalarin tum metrik skorlari tablosu. Satirlar: sayfa tipi x cihaz. Sutunlar: metrikler.

**Sheet 2+ - Sub-skill detay (her aktif sub-skill icin ayri sheet):**
| Sutun | Aciklama |
|-------|----------|
| Sayfa Tipi | Anasayfa, Urun, Kategori |
| Cihaz | Mobil, Desktop |
| Bulgu | Tespit edilen durum |
| Element | CSS selector |
| Mevcut Deger | Mevcut HTML/attribute durumu |
| Onerilen Deger | Onerilen degisiklik |
| Etkilenen Metrikler | LCP, CLS, INP vb. |
| Metrik Etkisi | Sayisal etki (ornegin CLS +0.15) |
| Oncelik | Yuksek / Orta / Dusuk |
| Efor | Dusuk / Orta / Yuksek |
| Sorumluluk | Frontend / Backend / DevOps |
| Durum | (bos - IT ekibi dolduracak) |

**Son sheet - Ham Veri:**
PSI API'den gelen tum audit skorlari (audit ID + skor + deger).

### Dosya isimlendirme
```
[domain]_[subskill]_[tarih].pptx
[domain]_[subskill]_[tarih].xlsx
Ornek: vitra.com.tr_cls-analizi_2026-03-18.pptx
Tam audit: vitra.com.tr_pagespeed-audit_2026-03-18.pptx
```

---

## 9. Dil ve karakter

- Varsayilan dil: Turkce (kullanici belirtmezse)
- EN veya DE istenirse tum ciktilar o dilde

### TURKCE KARAKTER KURALI (EN KRITIK KURAL)

Bu skill'in urettigi PPTX ve Excel dosyalarindaki TUM metinler Turkce ozel karakterleri
ORIJINAL HALLERIYLE icermek ZORUNDADIR. ASCII'ye cevirme KESINLIKLE YAPILMAZ.

pptxgenjs ve openpyxl UTF-8'i sorunsuz destekler. Turkce karakter donusumu gerektiren
HICBIR islem yapma. String'leri oldugu gibi kullan.

Dogru ornekler (sunumda boyle gorunmeli):
- "Değerlendirme" ("Degerlendirme" DEGIL)
- "Yükleme Süreci" ("Yukleme Sureci" DEGIL)
- "Çözüm Önerisi" ("Cozum Onerisi" DEGIL)
- "Öncelik: Yüksek" ("Oncelik: Yuksek" DEGIL)
- "İyileştirme" ("Iyilestirme" DEGIL)
- "Kararlılığı" ("Kararliligi" DEGIL)
- "Görsel" ("Gorsel" DEGIL)
- "Düşük" ("Dusuk" DEGIL)
- "Ürün Sayfası" ("Urun Sayfasi" DEGIL)

Kontrol: cikti dosyasini olusturduktan sonra markitdown ile oku ve Turkce karakter
icerip icermedigini dogrula. Tek bir bile ASCII-ye donusturulmus karakter varsa duzelt.

---

## 10. Uslup

IT ekibine yonelik profesyonel danismanlik raporu tonu. Yumusak, kurumsal ve is birligi odakli.

### Yasak ifadeler → dogru karsiliklari

| YANLIS (kullanma) | DOGRU (kullan) |
|---|---|
| "Sorun: görsellerde width/height yok" | "Hero slider görsellerinde boyut tanımı bulunmamaktadır" |
| "Bu CLS'i bozuyor" | "Bu durum CLS değerini olumsuz yönde etkileyebilmektedir" |
| "Hemen düzeltin" | "İlgili düzenlemenin kısa vadede ele alınması önerilmektedir" |
| "CSS'i inline yerleştirin" | "Kritik CSS'in inline olarak yerleştirilmesi değerlendirilebilir" |
| "Font'u preload ile yükleyin" | "İlgili font dosyasının preload ile ön yüklenmesi tavsiye edilmektedir" |
| "Kullanılmayan CSS'i kaldırın" | "Kullanılmayan CSS kurallarının temizlenmesi faydalı olacaktır" |
| "Sorun", "hata", "eksiklik", "bozuk" | "Bulgu", "tespit", "iyileştirme fırsatı", "değerlendirme" |
| "Düzeltin", "yapın", "ekleyin" (emir kipi) | "...önerilmektedir", "...değerlendirilebilir", "...tavsiye edilmektedir" |

### Insight madde formati

Her insight maddesi ➔ karakteri ile baslar (tire "-" veya bullet "•" KULLANMA):
```
➔ Anasayfada toplam 6 CSS dosyasının geç yüklenmekte olduğu tespit edilmiştir.
  Bu durum, tüm sayfa düzeni üzerinde büyük çaplı kaymaya neden olmaktadır.

➔ İncelenen dönemde CLS değerinin 0.356 olduğu görülmektedir. Google'ın
  belirlediği 0.1 eşik değerinin üzerinde yer almaktadır.
```

### Aciklama derinligi kurali

Her tespit slaytinda EN AZ su bilgiler olmali:
- Ne tespit edildi (2-3 cumle)
- Hangi element etkileniyor (CSS selector + boyut bilgisi)
- Neden olusuyor (teknik aciklama, 2-3 cumle)
- Mevcut HTML kodu (gercek sayfadan alinmis)

Her etki slaytinda EN AZ su bilgiler olmali:
- Kullanici deneyimi etkisi (3-4 cumle, somut senaryo)
- Metrik etkisi (sayisal: "CLS'e +0.351 katki, toplam degerin %98'i")
- Diger metriklere etkisi (varsa)
- Google referansi (web.dev linki)

Her cozum slaytinda EN AZ su bilgiler olmali:
- 2-4 numarali oneri adimi (her biri 2-3 cumle aciklama)
- Mevcut vs onerilen kod karsilastirmasi
- Beklenen iyilesme degeri
- Oncelik + efor + sorumluluk bilgisi

---

## 11. Before / After karsilastirma

1. Kullanicidan onceki Excel/PPTX dosyasini ayri dizine yuklemesini iste
2. Onceki dosyayi oku (markitdown veya openpyxl)
3. Ayni URL'ler ve sub-skill'ler icin yeni analiz calistir
4. Karsilastirma sunumu:
   - Her bulgu icin "Onceki → Simdiki" karsilastirmasi
   - Iyilesen: yesil, kotulusen: kirmizi
   - "Onerilen X uygulanmis, CLS 0.24'ten 0.09'a dusmus" gibi yorumlar
5. Dosya adi: [domain]_[subskill]_karsilastirma_[tarih].pptx

---

## 12. Lighthouse versiyon dayanikliligi

Audit ID'leri reference dosyalarinda merkezi tutulur.
Bilinmeyen ID gelirse: mevcut ID'leri JSON'dan listele, en yakin eslesen insight audit'i kullan.
Sunumda "Lighthouse [versiyon] ile analiz edilmistir" belirt.

---

## 13. Bagimliliklar

```bash
npm install -g pptxgenjs 2>/dev/null
pip install openpyxl --break-system-packages 2>/dev/null
pip install "markitdown[pptx]" --break-system-packages 2>/dev/null
```

Sunum olusturma icin: view /mnt/skills/public/pptx/SKILL.md ve view /mnt/skills/public/pptx/pptxgenjs.md
Excel olusturma icin: view /mnt/skills/public/xlsx/SKILL.md
