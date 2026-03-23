# Sunum Tasarim Rehberi (pptxgenjs icin)

Tutarli ve profesyonel sunumlar uretmek icin tasarim kurallari.

---

## 1. Renk paleti

### Ana renkler

| Renk | HEX | Kullanim |
|---|---|---|
| Coral/Salmon | #F4845F | Kapak, kapatis, breadcrumb, ikonlar |
| Dark Teal | #1B3A36 | Bolum ayiraci, baslik metni, govde metin |
| Off-White/Cream | #FAF8F5 | Icerik slide arka plani |
| White | #FFFFFF | Kapak/kapatis metin, kart arka plani |

### Veri gorsellestirme renkleri

| Renk | HEX | Kullanim |
|---|---|---|
| Kirmizi (dusus) | #D32F2F | Negatif degisim, poor metrikler |
| Yesil (artis) | #2E7D32 | Pozitif degisim, good metrikler |
| Turuncu (vurgu) | #F4845F | Insight vurgulari |
| Koyu gri (bar) | #4A4A4A | Bar chart mevcut donem |
| Sari/altin (bar) | #F5A623 | Bar chart onceki donem |
| Acik kirmizi hucre | #FFCDD2 | Tablo negatif deger arka plani |
| Acik yesil hucre | #C8E6C9 | Tablo pozitif deger arka plani |

### Heat-map renklendirme

- Negatif: hucre #FFCDD2, metin #D32F2F
- Pozitif: hucre #C8E6C9, metin #2E7D32
- Notr: beyaz arka plan, dark teal metin

---

## 2. Slide tipleri

### Kapak slide
- Arka plan: Coral (#F4845F)
- Metin: Beyaz (#FFFFFF)
- Baslik: Ortalanmis, buyuk punto, bold
- Alt baslik/tarih: Basligin altinda, kucuk punto

### Bolum ayiraci (separator)
- Arka plan: Dark Teal (#1B3A36)
- Buyuk numara: Sol tarafta 120-180pt, biraz acik ton (#2A4E48)
- Baslik: Ortada, beyaz, bold
- Coral cizgi: Basligin ust/altinda kisa yatay coral cizgi (#F4845F, ~60px)

### Icerik slide
- Arka plan: Off-White (#FAF8F5)
- Breadcrumb: Sol ust, coral renk, "Bolum Adi | Slide Basligi" formati
- Baslik: Dark teal, buyuk punto, bold
- ONEMLI: Baslikta coral highlight blogu KULLANMA, duz dark teal metin kullan
- Govde: Dark teal veya koyu gri

### Kart duzeni slide
- Arka plan: Off-White
- Kartlar: Beyaz, hafif golge, yuvarlak koseler (~12px)
- Kart baslik: Dark teal, bold
- Kart metin: Koyu gri

### Veri/tablo slide
- Tablo header: Koyu arka plan (#1B3A36 veya #4A4A4A), beyaz metin
- Veri hucreleri: Beyaz, ince border
- Degisim sutunu: Heat-map renklendirme
- Insight paneli: Sag tarafta, ➔ ile baslayan maddeler
- Kaynak notu: Sol alt, coral pill icinde beyaz metin ("Kaynak: ...")

### Kapatis slide
- Arka plan: Coral (#F4845F)
- "Tesekkurler" - beyaz, ortalanmis

---

## 3. Yazi tipleri

| Kullanim | Font | Agirlik | Punto |
|---|---|---|---|
| Kapak baslik | Bricolage Grotesque | Regular | 44-52 |
| Separator baslik | Bricolage Grotesque | Bold | 36-44 |
| Separator numara | Bricolage Grotesque | Bold | 120-180 |
| Slide baslik | Bricolage Grotesque | Bold | 28-36 |
| KPI buyuk sayi | Bricolage Grotesque | Bold | 36-48 |
| Kart baslik | Bricolage Grotesque | SemiBold | 18-22 |
| Breadcrumb | Outfit | SemiBold | 10-12 |
| Govde metin | Outfit | Regular | 14-16 |
| Tablo metin | Outfit | Regular | 11-13 |
| Insight madde | Outfit | Regular | 13-15 |
| Dipnot/kaynak | Outfit | Regular/SemiBold | 9-11 |

Fallback font: Calibri (Bricolage/Outfit yuklu degilse).

---

## 4. Uslup kurallari

- Tire: Kisa tire (-) kullan, EM DASH (—) KULLANMA
- Insight baslangici: ➔ karakteri + bosluk
- Rakamlar insight'larda bold
- Dusus: Kirmizi, artis: Yesil, vurgu: Coral
- Profesyonel ve analitik ton, edilgen yapilar

---

## 5. pptxgenjs teknik notlar

### Slide boyutu
```javascript
pres.layout = 'LAYOUT_WIDE'; // 13.33 x 7.5 inch
```

### Renk tanimlari
```javascript
const COLORS = {
  coral: 'F4845F',
  darkTeal: '1B3A36',
  offWhite: 'FAF8F5',
  white: 'FFFFFF',
  red: 'D32F2F',
  green: '2E7D32',
  darkGray: '4A4A4A',
  gold: 'F5A623',
  lightRed: 'FFCDD2',
  lightGreen: 'C8E6C9',
  separatorNum: '2A4E48'
};
```

### Metin stilleri
```javascript
const STYLES = {
  coverTitle:   { fontSize: 48,  color: 'FFFFFF', fontFace: 'Bricolage Grotesque', bold: false },
  sectionTitle: { fontSize: 40,  color: 'FFFFFF', fontFace: 'Bricolage Grotesque', bold: true  },
  sectionNum:   { fontSize: 160, color: '2A4E48', fontFace: 'Bricolage Grotesque', bold: true  },
  slideTitle:   { fontSize: 32,  color: '1B3A36', fontFace: 'Bricolage Grotesque', bold: true  },
  breadcrumb:   { fontSize: 11,  color: 'F4845F', fontFace: 'Outfit',              bold: true  },
  bodyText:     { fontSize: 14,  color: '1B3A36', fontFace: 'Outfit',              bold: false },
  insightText:  { fontSize: 13,  color: '1B3A36', fontFace: 'Outfit',              bold: false },
  tableHeader:  { fontSize: 12,  color: 'FFFFFF', fontFace: 'Outfit',              bold: true  },
  tableData:    { fontSize: 11,  color: '1B3A36', fontFace: 'Outfit',              bold: false },
  kpiNumber:    { fontSize: 42,  color: '1B3A36', fontFace: 'Bricolage Grotesque', bold: true  },
  sourceNote:   { fontSize: 10,  color: 'FFFFFF', fontFace: 'Outfit',              bold: true  },
  footnote:     { fontSize: 9,   color: 'D32F2F', fontFace: 'Outfit',              bold: false }
};
```

### Margin ve bosluk
- Genel kenar boslugu: 0.5 inch
- Breadcrumb: x=0.3, y=0.2
- Icerik baslangic: x=0.5, y=0.8
- Elementler arasi: 0.3-0.5 inch

### Screenshot ve filmstrip gomu

**DIKDORTGEN goster, DAIRE/OVAL kirpma YAPMA:**
```javascript
// DOGRU: Dikdortgen gorsel
slide.addImage({
  data: 'image/jpeg;base64,' + base64Data,
  x: 0.5, y: 2.5, w: 4, h: 3
});

// YANLIS - YAPMA:
// rounding: true → daire kirpma yapar
// shape icinde oval/circle secme
```

**Filmstrip gosterimi:**
3-5 kareyi yan yana dikdortgen olarak goster:
```javascript
filmstripItems.forEach((item, i) => {
  const xPos = 0.5 + (i * 2.5);
  slide.addImage({
    data: 'image/jpeg;base64,' + item.data.split(',')[1],
    x: xPos, y: 3.5, w: 2.2, h: 1.5
  });
  slide.addText(item.timing + 'ms', {
    x: xPos, y: 5.1, w: 2.2, h: 0.3, fontSize: 10,
    fontFace: 'Outfit', color: '4A4A4A', align: 'center'
  });
});
```

### Icerik yogunlugu

**Slaytlarin alt yarisi BOS KALMAMALI.** Her slayt tipine gore doldurucu elementler:
- Tespit: aciklama + element bilgisi + kod kutusu + screenshot
- Etki: kullanici etkisi + metrik tablosu + Google referans kutusu
- Cozum: oneriler + mevcut vs onerilen kod kutusu + oncelik/efor kartlari

### Kod kutusu stili

Mevcut ve onerilen HTML/CSS kodlarini kutu icinde goster:
```javascript
// Acik gri arka planli kod kutusu
slide.addShape(pptxgen.shapes.ROUNDED_RECTANGLE, {
  x: 0.5, y: posY, w: 12, h: 1.5,
  fill: { color: 'F5F5F5' },
  line: { color: 'E0E0E0', width: 0.5 },
  rectRadius: 0.1
});
slide.addText(codeString, {
  x: 0.7, y: posY + 0.1, w: 11.6, h: 1.3,
  fontSize: 11, fontFace: 'Consolas', color: '1B3A36', valign: 'top'
});
```

### Kaynak badge

Her icerik slaytinin sol alt kosesine:
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

### Insight madde formati

➔ karakteri ile baslar (tire "-" veya bullet "•" KULLANMA):
```javascript
slide.addText([
  { text: '➔ ', options: { color: 'F4845F', bold: true } },
  { text: 'Anasayfada toplam ', options: { color: '1B3A36' } },
  { text: '6 CSS dosyasının', options: { color: '1B3A36', bold: true } },
  { text: ' geç yüklenmekte olduğu tespit edilmiştir.', options: { color: '1B3A36' } }
], { x: 0.5, y: posY, w: 12, h: 0.5, fontSize: 13, fontFace: 'Outfit' });
```
