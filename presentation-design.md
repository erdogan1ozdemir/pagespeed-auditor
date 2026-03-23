# Sunum Tasarım Rehberi

Inbound Marketing Agency sunum tasarım standartları. Referans görseller baz alınmıştır.

---

## 1. Renk paleti

| Renk | HEX | Kullanım |
|---|---|---|
| Coral | F4845F | Kapak, kapanış, breadcrumb, vurgu çizgileri |
| Dark Teal | 1B3A36 | Separator arka plan, başlık metni, gövde metin |
| Off-White | FAF8F5 | İçerik slide arka planı |
| White | FFFFFF | Kapak/separator metin, kart arka plan |
| Separator Numara | 2A4E48 | Separator'daki büyük numara (dark teal'den biraz açık) |
| Kırmızı | D32F2F | Negatif değer, poor metrik |
| Yeşil | 2E7D32 | Pozitif değer, good metrik |
| Açık Kırmızı | FFCDD2 | Tablo negatif hücre arka planı |
| Açık Yeşil | C8E6C9 | Tablo pozitif hücre arka planı |
| Koyu Gri | 4A4A4A | Tablo bar, gövde metin alternatif |

---

## 2. Slide tipleri (referans görsellerden)

### 2.1 Kapak slide

Referans: Image 1 - VitrA SEO Değerlendirme

```javascript
// Coral arka plan
slide.background = { color: 'F4845F' };

// Dekoratif yarım daire - sol üst köşe (açık coral, düşük opaklık)
// pptxgenjs'de arc/oval shape ile oluştur
slide.addShape(pptxgen.shapes.OVAL, {
  x: -1.5, y: -1, w: 5, h: 7,
  fill: { color: 'F9A68A', transparency: 60 },
  line: { color: 'F9A68A', transparency: 70, width: 1 }
});

// Başlık - ortalanmış, beyaz, büyük
slide.addText('CLS Kararlılığı Analizi', {
  x: 0.5, y: 2.2, w: 12.3, h: 1.2,
  fontSize: 44, fontFace: 'Bricolage Grotesque',
  color: 'FFFFFF', bold: false, align: 'center'
});

// Alt başlık
slide.addText('vitra.com.tr - Mart 2026', {
  x: 0.5, y: 3.6, w: 12.3, h: 0.5,
  fontSize: 18, fontFace: 'Outfit',
  color: 'FFFFFF', italic: true, align: 'center'
});

// Inbound logosu - alt orta
slide.addText('inbound', {
  x: 5.5, y: 6.5, w: 2.3, h: 0.6,
  fontSize: 20, fontFace: 'Outfit',
  color: 'FFFFFF', bold: true, align: 'center'
});
```

### 2.2 Sunum akışı / agenda slide

Referans: Image 2 - Split layout

```javascript
// Sol panel: Coral arka plan (~%40 genişlik)
slide.addShape(pptxgen.shapes.RECT, {
  x: 0, y: 0, w: 5.3, h: 7.5, fill: { color: 'F4845F' }
});

// Dekoratif yarım daire sol panelde
slide.addShape(pptxgen.shapes.OVAL, {
  x: -1, y: 0.5, w: 4, h: 5.5,
  fill: { color: 'F9A68A', transparency: 60 }
});

// "SUNUM AKIŞI" büyük metin sol panelde
slide.addText('SUNUM AKIŞI', {
  x: 0.5, y: 2.5, w: 4.3, h: 1.5,
  fontSize: 36, fontFace: 'Bricolage Grotesque',
  color: 'FFFFFF', bold: false
});

// Inbound logosu sol alt
slide.addText('inbound', {
  x: 0.5, y: 6.5, w: 2, h: 0.5,
  fontSize: 16, fontFace: 'Outfit', color: 'FFFFFF', bold: true
});

// Sağ panel: Off-white (slide arka planı zaten FAF8F5)
// Numaralı liste
const agendaItems = [
  { num: '01', title: 'Genel Değerlendirme' },
  { num: '02', title: 'Anasayfa Analizi' },
  { num: '03', title: 'Ürün Sayfası Analizi' },
  { num: '04', title: 'Kategori Sayfası Analizi' },
  { num: '05', title: 'Özet ve Yol Haritası' }
];
agendaItems.forEach((item, i) => {
  const y = 0.8 + (i * 0.9);
  slide.addText([
    { text: item.num + ' - ', options: { fontSize: 18, fontFace: 'Outfit', color: '1B3A36' } },
    { text: item.title, options: { fontSize: 18, fontFace: 'Outfit', color: '1B3A36', bold: true } }
  ], { x: 5.8, y: y, w: 6.5, h: 0.6 });
});
```

### 2.3 Bölüm ayıracı (separator)

Referans: Image 3 - Dark teal arka plan, büyük numara, coral çizgiler

```javascript
// Dark teal tam arka plan
slide.background = { color: '1B3A36' };

// Büyük numara - sol tarafta, koyu teal tonunda (arka planla kaynaşan)
slide.addText('01', {
  x: -0.5, y: 1.5, w: 4, h: 4,
  fontSize: 160, fontFace: 'Bricolage Grotesque',
  color: '2A4E48', bold: true
});

// Coral çizgi - başlık üstünde (kısa, yatay, yuvarlak uçlu)
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: 5.5, y: 3.0, w: 0.8, h: 0.08,
  fill: { color: 'F4845F' }, rectRadius: 0.04
});

// Başlık - beyaz, bold, orta-sağ
slide.addText('Anasayfa Analizi', {
  x: 4, y: 3.3, w: 8, h: 0.8,
  fontSize: 36, fontFace: 'Bricolage Grotesque',
  color: 'FFFFFF', bold: true, align: 'center'
});

// Coral çizgi - başlık altında
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: 5.5, y: 4.3, w: 0.8, h: 0.08,
  fill: { color: 'F4845F' }, rectRadius: 0.04
});

// Alt metin (opsiyonel - site adı)
slide.addText('vitra.com.tr', {
  x: 4, y: 4.5, w: 8, h: 0.5,
  fontSize: 16, fontFace: 'Outfit',
  color: 'FFFFFF', align: 'center'
});
```

ÖNEMLİ: Desktop ve mobil için AYRI separator oluşturma. Aynı separator altında
"Anasayfa Analizi" başlığıyla hem mobil hem desktop slaytları ilerler.

### 2.4 İçerik slide

Referans: Image 4, 5 - Yoğun içerikli sayfalar

```javascript
// Arka plan
slide.background = { color: 'FAF8F5' };

// Breadcrumb - sol üst, coral renk
slide.addText('Anasayfa | Bulgu 1 - Tespit', {
  x: 0.3, y: 0.2, w: 8, h: 0.4,
  fontSize: 11, fontFace: 'Outfit',
  color: 'F4845F', bold: true
});

// Başlık - dark teal, büyük, bold
// NOT: Coral highlight bloğu KULLANMA, düz dark teal metin
slide.addText('Geç Yüklenen CSS Dosyaları Kaydırması', {
  x: 0.3, y: 0.6, w: 12.5, h: 0.8,
  fontSize: 28, fontFace: 'Bricolage Grotesque',
  color: '1B3A36', bold: true, margin: 0
});
```

### 2.5 Skor kartı slide

Referans: Image 6 - Büyük metrik, kartlar, screenshot + filmstrip

```javascript
// Büyük CLS değeri - sol tarafta
slide.addText('0.391', {
  x: 0.3, y: 1.5, w: 3, h: 1.5,
  fontSize: 72, fontFace: 'Bricolage Grotesque',
  color: 'D32F2F', bold: true, align: 'center'
});
slide.addText('Poor', {
  x: 0.3, y: 3.0, w: 3, h: 0.4,
  fontSize: 16, fontFace: 'Outfit',
  color: 'D32F2F', bold: true, align: 'center'
});

// Metrik kartları - yanyana (Performance, LCP, FCP, TBT)
const metrics = [
  { label: 'Performance', value: '77/100' },
  { label: 'LCP', value: '1.0 s' },
  { label: 'FCP', value: '0.6 s' },
  { label: 'TBT', value: '40 ms' }
];
metrics.forEach((m, i) => {
  const x = 3.5 + (i * 2.5);
  // Kart arka planı
  slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
    x: x, y: 1.2, w: 2.2, h: 1.8,
    fill: { color: 'FFFFFF' },
    line: { color: 'E0E0E0', width: 0.5 },
    rectRadius: 0.1, shadow: { type: 'outer', blur: 3, opacity: 0.1 }
  });
  slide.addText(m.label, {
    x: x, y: 1.3, w: 2.2, h: 0.4,
    fontSize: 12, fontFace: 'Outfit', color: '4A4A4A', align: 'center'
  });
  slide.addText(m.value, {
    x: x, y: 1.8, w: 2.2, h: 0.8,
    fontSize: 32, fontFace: 'Bricolage Grotesque',
    color: '1B3A36', bold: true, align: 'center'
  });
});
```

---

## 3. Screenshot ve filmstrip kuralları

### Screenshot kalitesi

PSI API iki tür görsel döndürür:
1. `final-screenshot` - tam sayfa görüntüsü (~1200px genişlik, iyi kalite)
2. `screenshot-thumbnails` - filmstrip kareleri (~120px genişlik, DÜŞÜK kalite)

**KALİTE KURALI:**
- Büyük sayfa görünümü için HER ZAMAN `final-screenshot` kullan
- Filmstrip kareleri KÜÇÜK göster (genişlik max 2 inch/kare) - büyütme, bulanıklaşır
- Mobil ve desktop için AYRI `final-screenshot` al (strategy=mobile ve strategy=desktop)
- Screenshot'ları DİKDÖRTGEN göster, OVAL/DAİRE kırpma YAPMA

### Filmstrip gösterimi

```javascript
// final-screenshot: Sol tarafta büyük (kaliteli)
slide.addImage({
  data: 'image/jpeg;base64,' + finalScreenshotBase64,
  x: 0.3, y: 3.5, w: 5, h: 3.5
});

// Filmstrip: Sağ tarafta küçük kareler yan yana
// Her kare MAX 2 inch genişlik - büyütme!
const filmstrip = thumbnailItems.slice(0, 4); // max 4 kare
filmstrip.forEach((item, i) => {
  const x = 6 + (i * 1.8);
  slide.addImage({
    data: item.data, // zaten base64
    x: x, y: 3.5, w: 1.6, h: 2.5
  });
  slide.addText(item.timing + 'ms', {
    x: x, y: 6.1, w: 1.6, h: 0.3,
    fontSize: 9, fontFace: 'Outfit', color: '4A4A4A', align: 'center'
  });
});
```

ÖNEMLİ: Filmstrip kareleri KÜÇÜK boyutta gösterilmeli. 120px genişliğindeki
thumbnail'ı 4 inch'e büyütürsen BULANIK olur. Max 1.6-2 inch genişlik.

### Mobil ve desktop screenshot ayrımı

Her sayfa tipi için mobil VE desktop screenshot ayrı ayrı alınır ve
aynı slayt veya ardışık slaytlarda YAN YANA gösterilir:

```javascript
// Sol: Desktop screenshot
slide.addText('Desktop', { x: 0.3, y: 3.2, w: 5, h: 0.3, fontSize: 11, color: '4A4A4A' });
slide.addImage({ data: desktopScreenshot, x: 0.3, y: 3.5, w: 5, h: 3 });

// Sağ: Mobil screenshot (daha dar)
slide.addText('Mobil', { x: 7, y: 3.2, w: 2.5, h: 0.3, fontSize: 11, color: '4A4A4A' });
slide.addImage({ data: mobileScreenshot, x: 7, y: 3.5, w: 2.5, h: 3 });
```

---

## 4. CLS etkilenen elementler - tek slide

CLS'te kayma yaşatan tüm div'ler tek bir slide'da listelenir:

```javascript
// Başlık
slide.addText('Sabitlenmesi Gereken Alanlar', {
  x: 0.3, y: 0.6, w: 12.5, h: 0.6,
  fontSize: 28, fontFace: 'Bricolage Grotesque', color: '1B3A36', bold: true
});

// Açıklama paragrafı
slide.addText('Aşağıdaki elementlerin en-boy oranlarının sabitlenmesi veya ' +
  'placeholder alanlarının belirlenmesi önerilmektedir.', {
  x: 0.3, y: 1.3, w: 7, h: 0.6,
  fontSize: 13, fontFace: 'Outfit', color: '4A4A4A'
});

// Element tablosu
const tableRows = [
  [
    { text: 'Element', options: { bold: true, color: 'FFFFFF', fill: { color: '1B3A36' } } },
    { text: 'Kayma Değeri', options: { bold: true, color: 'FFFFFF', fill: { color: '1B3A36' } } },
    { text: 'Önerilen Düzeltme', options: { bold: true, color: 'FFFFFF', fill: { color: '1B3A36' } } }
  ],
  // Her etkilenen element bir satır
  [
    { text: 'div.hero-slider > img' },
    { text: '0.351', options: { color: 'D32F2F' } },
    { text: 'width="1200" height="675" ekle + aspect-ratio: 16/9' }
  ]
];
slide.addTable(tableRows, {
  x: 0.3, y: 2.2, w: 12.5,
  fontSize: 11, fontFace: 'Outfit',
  border: { color: 'E0E0E0', pt: 0.5 }
});
```

---

## 5. İçerik yoğunluğu ve detay kuralları

Referans Image 4 ve 5 çok yoğun içerikli. SLAYTLAR BOŞ KALMAMALI.

### Tespit slaytı dolu olmalı:
- Sol kolon (~%55): Açıklama paragrafları (2-3), etkilenen elementler, CSS selector,
  mevcut HTML kod kutusu
- Sağ kolon (~%45): Screenshot, filmstrip veya etkilenen alanın zoom görünümü
- Alt kısım: Kaynak badge + Excel referans notu

### Etki slaytı dolu olmalı:
- Kullanıcı etkisi paragrafı (3-4 cümle)
- Metrik tablosu (küçük): hangi metrik ne kadar etkileniyor
- Google referans kutusu (açık gri arka plan, web.dev linki)

### Çözüm slaytı dolu olmalı:
- Numaralı öneriler (her biri 2 satır açıklama)
- Kod kutusu: mevcut vs önerilen (yan yana veya üst üste)
- Alt kısım: Öncelik/Efor/Sorumluluk kartları + beklenen iyileşme

### Insight madde formatı:
➔ karakteri ile başlar, tire "-" veya bullet "•" KULLANMA.

---

## 6. Yazı tipleri

| Kullanım | Font | Ağırlık | Punto |
|---|---|---|---|
| Kapak başlık | Bricolage Grotesque | Regular | 44 |
| Separator başlık | Bricolage Grotesque | Bold | 36 |
| Separator numara | Bricolage Grotesque | Bold | 160 |
| Slide başlık | Bricolage Grotesque | Bold | 28 |
| KPI büyük sayı | Bricolage Grotesque | Bold | 42-72 |
| Breadcrumb | Outfit | SemiBold | 11 |
| Gövde metin | Outfit | Regular | 13-14 |
| Tablo metin | Outfit | Regular | 11 |
| Insight madde | Outfit | Regular | 13 |
| Kaynak notu | Outfit | SemiBold | 10 |

Fallback: Calibri (Bricolage/Outfit yüklü değilse).

---

## 7. Teknik notlar

### Slide boyutu
```javascript
pres.layout = 'LAYOUT_WIDE'; // 13.33 x 7.5 inch
```

### EM DASH KULLANMA
Tire: kısa tire (-) kullan. EM DASH (—) KESINLIKLE KULLANMA.

### Coral highlight bloğu KULLANMA
Başlıklarda turuncu arka plan + beyaz yazı bloğu YAPMA. Düz dark teal metin kullan.

### TÜRKÇE KARAKTERLER
Tüm metinlerde Türkçe özel karakterler (ç, ğ, ı, ö, ş, ü, İ, Ş) orijinal
halleriyle kullanılır. ASCII'ye çevirme YAPILMAZ. pptxgenjs UTF-8 destekler.

### Kod kutusu stili
```javascript
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: posX, y: posY, w: width, h: height,
  fill: { color: 'F5F5F5' },
  line: { color: 'E0E0E0', width: 0.5 },
  rectRadius: 0.1
});
slide.addText(codeString, {
  x: posX + 0.15, y: posY + 0.1, w: width - 0.3, h: height - 0.2,
  fontSize: 10, fontFace: 'Consolas', color: '1B3A36', valign: 'top'
});
```

### Kaynak badge
```javascript
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: 0.3, y: 6.8, w: 3.5, h: 0.35,
  fill: { color: 'F4845F' }, rectRadius: 0.15
});
slide.addText('Kaynak: PageSpeed Insights API', {
  x: 0.3, y: 6.8, w: 3.5, h: 0.35,
  fontSize: 10, color: 'FFFFFF', fontFace: 'Outfit',
  bold: true, align: 'center', valign: 'middle'
});
```
