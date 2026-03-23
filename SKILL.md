---
name: pagespeed-auditor
description: >
  Bir veya birden fazla URL için PageSpeed Insights analizi yapip PPTX sunum ve destekleyici
  Excel çıktısi üreten skill. Bağımsız cagrilabilir 6 sub-skill içerir: render pipeline,
  javascript yuku, LCP optimizasyonu, CLS kararlılığı, INP etkileşim, kalite ve erişilebilirlik.
  Bu skill'i su durumlarda kullan: "pagespeed analiz et", "site hizi raporu hazirla",
  "LCP bak", "CLS sorunlari", "INP analiz et", "javascript audit yap", "accessibility kontrol et",
  "render pipeline", "sayfa hizi sunumu", "performans raporu", "core web vitals",
  "pagespeed sunum hazirla", "site hizi optimizasyonu", "web vitals analiz",
  "erişilebilirlik raporu", "3rd party scriptler", "görsel optimizasyonu",
  kullanici bir URL verip performans, hiz, CWV, Lighthouse veya PageSpeed ile ilgili herhangi
  bir analiz, rapor veya sunum istediginde bu skill tetiklenir. Tek bir sub-skill veya tam audit
  istenebilir. Turkce, Ingilizce ve Almanca çıktı destekler.
---

# PageSpeed Auditor

Site performansini analiz edip IT ekiplerine yonelik aksiyon odakli sunum ve rapor üreten skill.

## Çalışma modu: OTONOM

Kullanici URL'leri ve (varsa) sub-skill secimini verdikten sonra tum is akisini otonom çalıştır.
Ara adimlarda onay isteme - API cagrilarini yap, verileri topla, analiz et, çıktıyi üret.

**Izin istemeden yap:**
- PSI API veya DataForSEO Lighthouse cagrilari
- web_fetch ile HTML kaynak kodu cekme
- Chrome DevTools kontrolu (varsa)
- Screenshot ve filmstrip alma
- Sunum ve Excel dosyası oluşturma
- Reference dosyalarıni okuma
- Paket yükleme (pptxgenjs, openpyxl vb.)
- Google dokumantasyonu arastirma (web_search)

**Yalnizca su durumlarda kullaniciya sor:**
- URL verilmemisse
- API key gerekiyorsa ve bulunamiyorsa
- Kritik hata sonrasi tum retry yollari tukenmisse

---

## 1. Ortam algılama

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
2. Kullanici önceden key verdiyse → kullan
3. Hicbiri yoksa → kullaniciya sor:
   "PSI API key gereklidir. Ucretsiz anahtar icin:
   https://developers.google.com/speed/docs/insights/v5/get-started
   Anahtarinizi buraya yapistirin."

DataForSEO varsa tercih et - PSI key gerektirmez, full Lighthouse JSON dondurur.

---

## 3. Sub-skill yönlendirme

| # | Sub-skill | Tetikleyiciler | Reference |
|---|-----------|----------------|-----------|
| 1 | Render pipeline | "render pipeline", "TTFB", "sunucu hizi", "render-blocking", "cache" | references/render-pipeline.md |
| 2 | JavaScript yuku | "javascript audit", "JS yuku", "3rd party", "script analizi" | references/javascript-burden.md |
| 3 | LCP optimizasyonu | "LCP", "görsel optimizasyonu", "largest contentful paint", "görsel boyut" | references/lcp-optimization.md |
| 4 | CLS kararlılığı | "CLS", "layout shift", "sayfa kayiyor", "kayma" | references/cls-stability.md |
| 5 | INP etkileşim | "INP", "etkileşim", "TBT", "tepki süresi", "DOM boyutu" | references/inp-interactivity.md |
| 6 | Kalite ve Erişilebilirlik | "accessibility", "erişilebilirlik", "best practices", "alt text" | references/quality-accessibility.md |

"Tam audit", "full pagespeed", "tum metriklere bak" → 6 sub-skill sırasıyla çalışır.

**Her sub-skill çalıştırilmadan once ilgili reference dosyası okunmalidir.**

---

## 4. Sayfa tipi algılama

Kullanici sadece domain verirse otomatik algıla (maksimum 3 sayfa tipi):

1. Anasayfa (domain root) her zaman dahil
2. web_fetch ile anasayfa HTML'ini cek, navigasyon linklerinden sayfa tipleri bul:
   - E-ticaret tespiti: /product/, /p-, /urun/, /category/, /c-, /kategori/ patternleri
   - Kurumsal tespiti: /hizmet/, /service/, /blog/, /içerik/ patternleri
3. E-ticaret → anasayfa + ürün sayfası + kategori sayfası
4. Kurumsal → anasayfa + hizmet sayfası + en önemli içerik sayfası
5. Kullanici spesifik URL verdiyse otomatik algılama yapma

---

## 5. Veri toplama

Her URL için mobil VE desktop ayri ayri analiz edilir.

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

Field data yoksa (yetersiz trafik) bunu sunumda açıkça belirt.

### 5.2 HTML kaynak analizi

Chrome varsa: navigate + javascript_tool ile DOM tara
Chrome yoksa: web_fetch ile HTML cek, parse et

**TUM elementler taranir. SAMPLING (slice/limit) KESINLIKLE YAPILMAZ.**

### 5.3 DataForSEO ek veri kaynaklari

**on_page_lighthouse:** PSI API alternatifi. full_data: true ile tam Lighthouse JSON dondurur.
Screenshot, filmstrip, tum audit detaylari dahil. PSI API key gerektirmez.

**on_page_instant_pages:** custom_js parametresiyle sayfada JS çalıştırabilir.
Chrome DevTools'taki gibi DOM tarama yapilabilir (img attribute kontrolu, script tarama,
layout shift ölçümü vb.). Claude Code ortaminda Chrome yokken en guclu alternatif budur.
```
Ornek: on_page_instant_pages(
  url: "https://example.com",
  enable_javascript: true,
  custom_js: "document.querySelectorAll('img').length"
)
```

**on_page_content_parsing:** Heading yapisi, link yapisi, yapisal içerik analizi.

---

## 6. Batch yonetimi

- İstekleri sıraya koy: URL1-mobile, URL1-desktop, URL2-mobile...
- PSI API cagrilari arasinda 3 saniye bekle
- Başarısız cagrida 1 kez tekrar dene, sonra DataForSEO'ya fallback
- DataForSEO da başarısızsa "analiz edilemedi" olarak raporla

---

## 7. Çıktı: PPTX sunum

Once pptx skill'ini oku: `view /mnt/skills/public/pptx/SKILL.md` ve `view /mnt/skills/public/pptx/pptxgenjs.md`

### Tasarim kurallari

Detayli tasarim rehberi icin: `view references/presentation-design.md`
Temel kurallar:
- Kapak/kapatis: Coral (#F4845F) arka plan, beyaz metin
- Bölüm ayiraclari: Dark Teal (#1B3A36), numarali, coral cizgi
- İçerik slaytlari: Off-White (#FAF8F5) arka plan
- Başlık fontu: Bricolage Grotesque (fallback: Calibri)
- Govde fontu: Outfit (fallback: Calibri)
- Başlıklar: Duz dark teal metin, coral highlight blogu KULLANMA
- Breadcrumb: sol ust, coral, "Bölüm | Slayt" formati
- Heat-map tablo: kirmizi (#FFCDD2 + #D32F2F) negatif, yesil (#C8E6C9 + #2E7D32) pozitif
- Insight: ➔ ile baslar
- Tire: EM DASH (—) KULLANMA, kisa tire (-) kullan
- Slide boyutu: LAYOUT_WIDE (13.33 x 7.5 inch)

### Filmstrip ve screenshot

**SCREENSHOT KALİTE KURALI (ÇOK ÖNEMLİ):**
PSI API iki tip görsel döndürür:
- `final-screenshot` → tam sayfa (~1200px genişlik, İYİ kalite) → büyük göster
- `screenshot-thumbnails` → filmstrip kareleri (~120px genişlik, DÜŞÜK kalite) → KÜÇÜK göster

Filmstrip karelerini büyütme - bulanıklaşır. Max 1.6-2 inch genişlik.
Final-screenshot büyük gösterilebilir (5-6 inch genişlik).

Mobil ve desktop screenshot AYRI AYRI alınır (strategy=mobile, strategy=desktop).
Aynı slayt veya ardışık slaytlarda yan yana gösterilir.

DİKDÖRTGEN göster, OVAL/DAİRE kırpma YAPMA (rounding: true KULLANMA).

Desktop ve mobil için AYRI separator oluşturma. Aynı sayfa tipi separator'ı altında
hem desktop hem mobil analizi ilerler.

Detaylı kod örnekleri: `view references/presentation-design.md` bölüm 3

### İçerik yogunlugu kurali (BOS ALAN BIRAKILMAZ)

Slaytlarin alt yarisi BOS KALMAMALI. İçerik tum slayti kaplamalı:
- Metin yeterli degilse: görsel, screenshot, tablo veya ek açıklama ekle
- Tespit slaytinda: screenshot kaniti, filmstrip, etkilenen element görseli ekle
- Etki slaytinda: metrik karşılaştırma tablosu, Google referans kutusu ekle
- Çözüm slaytinda: mevcut vs önerilen kod karşılaştırmasi kutu icinde, efor/öncelik kartlari ekle
- Bos alan kaliyorsa: ek not, ipucu veya ilgili kaynak linki ekle
- Her slaytta en az 1 görsel element olmali (tablo, chart, screenshot, kod kutusu vb.)

### Kaynak badge

Her içerik slaytinin sol alt kosesine coral pill badge ekle:
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

Mevcut ve önerilen HTML/CSS kodlarini acik gri arka planli kutu icinde göster:
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
KAPAK (coral, dekoratif yarim daire, inbound logosu)
SUNUM AKISI (split layout: sol coral, sag numarali liste)
GENEL DEĞERLENDİRME (skor tablosu + özet paragraf)
METRIK AÇIKLAMA NOTU (IT ekibi icin)

HER SAYFA TIPI ICIN (Anasayfa, Ürün, Kategori):
  BÖLÜM AYIRACI (dark teal, numarali, coral cizgi)
  SKOR KARTI: Desktop + Mobil yan yana (buyuk metrik, kartlar, screenshot + filmstrip)

  HER BULGU ICIN AYRI SLAYTLAR:
    TESPİT SLAYTI (detayli açıklama + HTML referansi + screenshot)
    ETKI SLAYTI (kullanici etkisi + metrik etkisi + Google ref)
    ÇÖZÜM ÖNERİSİ SLAYTI (adimlar + kod ornegi + efor/öncelik)

  CLS ICIN EK: Etkilenen tum elementler tek slaytta tablo halinde

ÖZET AKSIYON TABLOSU (sayfa x bulgu matrisi)
ÖNCELİK SIRALAMASI / YOL HARITASI
KAPANIŞ (coral, tesekkurler, inbound logosu)
```

ONEMLI KURALLAR:
- Mobil ve desktop için AYRI separator oluşturma. Ayni sayfa tipi separator'i altinda
  hem mobil hem desktop analizi ilerler. Skor kartinda ikisi yan yana gösterilir.
- Her bulgu için ayri slaytlar (tek slaytta birlestirme YAPMA)
- Basit bulgularda etki ve çözüm birlesebilir
- Ortak sorunlar ilk goruldugu yerde detayli anlatilir, diger sayfalarda referans verilir
- CLS analizinde kayma yasatan TUM elementler tek slayttta tablo halinde listelenir
  (CSS selector + kayma değeri + önerilen düzeltme)

### Tespit slayti içeriği (dolu olmali)

Referans Image 4 gibi yogun. Sol-sag kolon yapisinda:
- SOL (%55): Breadcrumb + başlık + 2-3 paragraf açıklama (➔ ile) + element bilgisi + kod kutusu
- SAG (%45): Screenshot kaniti + etkilenen alanin zoom gorunumu
- ALT: Kaynak badge + Excel referans notu

Açıklamalar anlasilabilir olmali - teknik olmayan kisiler de okudugunda anlayabilmeli.
Kisa cumleler, somut ornekler, "bu ne demek" sorusuna cevap veren ifadeler.

### Etki slayti içeriği

- Kullanici deneyimi etkisi (en az 2-3 cumle)
- Metrik etkisi (sayisal, tablolu)
- Diger metriklere etkisi
- Google referansi (web.dev linki, coral metin)

### Çözüm önerisi slayti içeriği

- Önerilen adimlar (numarali, her biri 2-3 cumle)
- Mevcut vs önerilen kod karşılaştırmasi (2 kod kutusu yan yana veya ust uste)
- Beklenen iyileşme değeri (yesil renk)
- Öncelik / Efor / Sorumluluk kartlari (3 kart yan yana)
- Excel referans notu (spesifik: "dosya.xlsx, Sheet Adi sayfası, satir X-Y")

---

## 8. Çıktı: Excel rapor

Once xlsx skill'ini oku: `view /mnt/skills/public/xlsx/SKILL.md`

### Sheet yapisi

**Sheet 1 - Özet:**
Tum sayfaların tum metrik skorlari tablosu. Satirlar: sayfa tipi x cihaz. Sutunlar: metrikler.

**Sheet 2+ - Sub-skill detay (her aktif sub-skill için ayri sheet):**
| Sutun | Açıklama |
|-------|----------|
| Sayfa Tipi | Anasayfa, Ürün, Kategori |
| Cihaz | Mobil, Desktop |
| Bulgu | Tespit edilen durum |
| Element | CSS selector |
| Mevcut Değer | Mevcut HTML/attribute durumu |
| Önerilen Değer | Önerilen degisiklik |
| Etkilenen Metrikler | LCP, CLS, INP vb. |
| Metrik Etkisi | Sayisal etki (ornegin CLS +0.15) |
| Öncelik | Yüksek / Orta / Düşük |
| Efor | Düşük / Orta / Yüksek |
| Sorumluluk | Frontend / Backend / DevOps |
| Durum | (bos - IT ekibi dolduracak) |

**Son sheet - Ham Veri:**
PSI API'den gelen tum audit skorlari (audit ID + skor + değer).

### Dosya isimlendirme
```
[domain]_[subskill]_[tarih].pptx
[domain]_[subskill]_[tarih].xlsx
Ornek: vitra.com.tr_cls-analizi_2026-03-18.pptx
Tam audit: vitra.com.tr_pagespeed-audit_2026-03-18.pptx
```

---

## 9. Dil ve karakter

- Varsayılan dil: Turkce (kullanici belirtmezse)
- EN veya DE istenirse tum çıktılar o dilde

### TURKCE KARAKTER KURALI (EN KRITIK KURAL)

Bu skill'in ürettigi PPTX ve Excel dosyalarındaki TUM metinler Turkce ozel karakterleri
ORIJINAL HALLERIYLE icermek ZORUNDADIR. ASCII'ye cevirme KESINLIKLE YAPILMAZ.

pptxgenjs ve openpyxl UTF-8'i sorunsuz destekler. Turkce karakter donusumu gerektiren
HICBIR islem yapma. JavaScript string'lerinde Turkce karakterleri DOGRUDAN yaz.

**JavaScript string ornekleri (DOGRU):**
```javascript
// DOGRU - Turkce karakterler doğrudan string icinde
slide.addText('CLS Kararlılığı Analizi', { ... });
slide.addText('Değerlendirme', { ... });
slide.addText('Çözüm Önerisi', { ... });
slide.addText('Öncelik: Yüksek', { ... });
slide.addText('Ürün Sayfası', { ... });
slide.addText('Geç Yüklenen CSS Dosyaları', { ... });
slide.addText('İyileştirme Fırsatı', { ... });
slide.addText('Düşük', { ... });

// YANLIS - ASCII karşılıklari KULLANMA
slide.addText('CLS Kararlılığı Analizi', { ... });  // YANLIS!
slide.addText('Değerlendirme', { ... });              // YANLIS!
slide.addText('Çözüm Önerisi', { ... });              // YANLIS!
```

**Slayt başlıklarinda Turkce karakter:**
- "CLS Kararlılığı Analizi" (DOGRU) vs "CLS Kararlılığı Analizi" (YANLIS)
- "Genel Değerlendirme" (DOGRU) vs "Genel Değerlendirme" (YANLIS)
- "Çözüm Önerisi" (DOGRU) vs "Çözüm Önerisi" (YANLIS)
- "Skor Kartı" (DOGRU) vs "Skor Karti" (YANLIS)
- "Yükleme Süreci" (DOGRU) vs "Yükleme Süreçi" (YANLIS)
- "Bölüm Ayıracı" (DOGRU) vs "Bölüm Ayiraci" (YANLIS)

**Tablo başlıklarinda:**
- "Sayfa", "Değer", "Durum", "Öncelik", "Çözüm", "Ürün", "Görsel", "İyileşme"

**Insight metinlerinde:**
- "➔ Yapılan incelemede..." (DOGRU) vs "➔ Yapilan incelemede..." (YANLIS)

**Doğrulama adimi:** Çıktı dosyasıni oluşturduktan sonra:
```bash
python -m markitdown output.pptx | grep -P '[a-z]' | head -20
```
Eger "Değerlendirme", "Çözüm", "Öncelik" gibi ASCII karşılıklar varsa DÜZELT.

---

## 10. Uslup

IT ekibine yonelik profesyonel danismanlik raporu tonu. Yumusak, kurumsal ve is birligi odakli.

### Yasak ifadeler → dogru karşılıklari

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

### Açıklama derinligi kurali

Her tespit slaytinda EN AZ su bilgiler olmali:
- Ne tespit edildi (2-3 cumle)
- Hangi element etkileniyor (CSS selector + boyut bilgisi)
- Neden olusuyor (teknik açıklama, 2-3 cumle)
- Mevcut HTML kodu (gerçek sayfadan alinmis)

Her etki slaytinda EN AZ su bilgiler olmali:
- Kullanici deneyimi etkisi (3-4 cumle, somut senaryo)
- Metrik etkisi (sayisal: "CLS'e +0.351 katki, toplam değerin %98'i")
- Diger metriklere etkisi (varsa)
- Google referansi (web.dev linki)

Her çözüm slaytinda EN AZ su bilgiler olmali:
- 2-4 numarali öneri adimi (her biri 2-3 cumle açıklama)
- Mevcut vs önerilen kod karşılaştırmasi
- Beklenen iyileşme değeri
- Öncelik + efor + sorumluluk bilgisi

---

## 11. Before / After karşılaştırma

1. Kullanicidan önceki Excel/PPTX dosyasıni ayri dizine yüklemesini iste
2. Önceki dosyayi oku (markitdown veya openpyxl)
3. Ayni URL'ler ve sub-skill'ler için yeni analiz çalıştır
4. Karşılaştırma sunumu:
   - Her bulgu için "Önceki → Simdiki" karşılaştırmasi
   - Iyilesen: yesil, kotulusen: kirmizi
   - "Önerilen X uygulanmış, CLS 0.24'ten 0.09'a dusmus" gibi yorumlar
5. Dosya adi: [domain]_[subskill]_karşılaştırma_[tarih].pptx

---

## 12. Lighthouse versiyon dayanikliligi

Audit ID'leri reference dosyalarında merkezi tutulur.
Bilinmeyen ID gelirse: mevcut ID'leri JSON'dan listele, en yakin eşleşen insight audit'i kullan.
Sunumda "Lighthouse [versiyon] ile analiz edilmistir" belirt.

---

## 13. Bağımlılıklar

```bash
npm install -g pptxgenjs 2>/dev/null
pip install openpyxl --break-system-packages 2>/dev/null
pip install "markitdown[pptx]" --break-system-packages 2>/dev/null
```

Sunum oluşturma icin: view /mnt/skills/public/pptx/SKILL.md ve view /mnt/skills/public/pptx/pptxgenjs.md
Excel oluşturma icin: view /mnt/skills/public/xlsx/SKILL.md
