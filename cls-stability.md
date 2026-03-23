# Sub-skill 4: CLS Kararlılığı

Layout shift nedenleri, boyutsuz öğeler, animasyonlar, aspect-ratio, font kaymasi, ad/embed slotlari.

## Lighthouse audit eşleştirme

| Insight Audit ID | Kontrol alani |
|---|---|
| cls-culprits-insight | Layout shift'e neden olan öğeler, kayma buyuklugu |
| unsized-images | width/height olmayan img, video, iframe (diagnostic) |
| non-composited-animations | GPU yerine CPU ile calisan animasyonlar (diagnostic) |

Ek kontroller: aspect-ratio CSS, min-height rezervasyon, dinamik inject, ad slot,
font-display kaymasi, 3rd party inject CLS etkisi.

## PSI API'den cekilecek veriler

```
audits['cls-culprits-insight']         → shift eden elementler, değerler
audits['unsized-images']               → boyutsuz görsel listesi
audits['non-composited-animations']    → CPU animasyonlari
audits['cumulative-layout-shift']      → CLS skoru
audits['screenshot-thumbnails']        → filmstrip (shift karşılaştırmasi icin)
audits['final-screenshot']             → screenshot
loadingExperience.metrics.CUMULATIVE_LAYOUT_SHIFT_SCORE → field CLS
```

## Chrome DevTools kontrolleri

**Layout shift PerformanceObserver:**
```javascript
const clsEntries = performance.getEntriesByType('layout-shift');
const shifts = clsEntries.filter(e => !e.hadRecentInput).map(e => ({
  value: Math.round(e.value * 10000) / 10000,
  startTime: Math.round(e.startTime),
  sources: e.sources?.map(s => ({
    tagName: s.node?.tagName,
    selector: s.node?.className ? '.' + s.node.className.split(' ')[0] : s.node?.tagName,
    previousRect: s.previousRect ? {
      x: s.previousRect.x, y: s.previousRect.y,
      w: s.previousRect.width, h: s.previousRect.height
    } : null,
    currentRect: s.currentRect ? {
      x: s.currentRect.x, y: s.currentRect.y,
      w: s.currentRect.width, h: s.currentRect.height
    } : null
  })) || []
}));
const totalCLS = shifts.reduce((sum, s) => sum + s.value, 0);
```

**Boyutsuz öğe taramasi:**
```javascript
const mediaTags = Array.from(document.querySelectorAll('img, video, iframe, canvas, svg'));
const unsized = mediaTags.map(el => {
  const rect = el.getBoundingClientRect();
  return {
    tagName: el.tagName,
    src: (el.src || el.getAttribute('data-src') || '').substring(0, 80),
    hasWidth: el.hasAttribute('width'),
    hasHeight: el.hasAttribute('height'),
    hasAspectRatio: getComputedStyle(el).aspectRatio !== 'auto',
    displaySize: Math.round(rect.width) + 'x' + Math.round(rect.height),
    aboveFold: rect.top < window.innerHeight && rect.bottom > 0
  };
}).filter(el => !el.hasWidth || !el.hasHeight);
```

**Ad/embed slot kontrolu:**
```javascript
const adSelectors = [
  '[id*="ad"]', '[class*="ad-"]', '[class*="ads-"]',
  '[id*="banner"]', '[class*="banner"]',
  'iframe[src*="doubleclick"]', 'iframe[src*="googlesyndication"]',
  '[class*="embed"]', '[class*="widget"]'
];
const adSlots = Array.from(document.querySelectorAll(adSelectors.join(',')));
const adAudit = adSlots.map(el => ({
  selector: el.id ? '#' + el.id : '.' + (el.className || '').split(' ')[0],
  tagName: el.tagName,
  hasMinHeight: parseInt(getComputedStyle(el).minHeight) > 0,
  currentMinHeight: getComputedStyle(el).minHeight,
  currentHeight: getComputedStyle(el).height,
  aboveFold: el.getBoundingClientRect().top < window.innerHeight
}));
```

**GIF kaydi stratejisi (CLS görselestirme):**
Chrome extension varsa coklu screenshot stratejisi kullan:
1. gif_creator start_recording
2. navigate → hemen screenshot
3. wait 0.5s → screenshot
4. wait 0.5s → screenshot
5. wait 1s → screenshot
6. gif_creator stop_recording → export

Eger GIF yetersizse, filmstrip karelerini yan yana koyarak shift görsellestir.

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: boyutsuz öğe taramasi (img/video/iframe width/height),
aspect-ratio CSS kontrolu, ad slot min-height kontrolu.
on_page_lighthouse (full_data: true) ile CLS culprit detayi ve filmstrip kareleri.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: tum img/video/iframe taglerinde width, height attribute kontrolu.
Inline style'da aspect-ratio. Ad container'larda min-height.
Filmstrip kareleri (PSI API'den) arasindaki farklar ile CLS görselestirme.

---

## Bulgu kaliplari

### Boyutsuz görseller
**Öneriler:** İlgili img, video ve iframe elementlerine width ve height attribute'larinin
eklenmesi tavsiye edilmektedir. CSS tarafinda aspect-ratio tanımının yapilmasi, attribute'lar
olmasa bile alanin önceden ayrilmasini sağlayacaktır. Container elementlere min-height
tanımlanmasi da değerlendirilebilir.
```html
<!-- Mevcut durum -->
<img src="hero.jpg" alt="...">
<!-- Önerilen durum -->
<img src="hero.jpg" alt="..." width="1200" height="675">
```
```css
.hero-img { aspect-ratio: 16/9; width: 100%; height: auto; }
```
**Ref:** https://web.dev/articles/optimize-cls

### Ad/embed slot kaymasi
**Öneriler:** Reklam ve embed alanlarina min-height tanımlanmasi uygun olacaktır.
İçerik yüklenene kadar placeholder veya skeleton UI kullanilmasi, kullanici deneyimini
olumlu yonde etkileyecektir.
```css
.ad-container { min-height: 250px; }
```
**Ref:** https://web.dev/articles/optimize-cls#ads-embeds-and-iframes-without-dimensions

### Font kaymasi (FOUT)
**Öneriler:** font-display değerinin optional olarak ayarlanmasi, font kaymasini
tamamen önleyecektir. Fallback font için metrik eşleştirme (size-adjust, ascent-override,
descent-override) tanımlanmasi da faydalı olacaktır. Kullanılmayan font varyantlarinin
kaldırılması dosya boyutunu ve yükleme süresini azaltacaktır.
**Ref:** https://web.dev/articles/font-display

### Dinamik inject edilen içerik
**Öneriler:** JavaScript ile inject edilen içerik alanlarinin önceden boyutlandirilmasi
önerilmektedir. content-visibility: auto kullanımınin değerlendirilmesi, ozellikle uzun
sayfalarda performans kazanci saglayabilir.
**Ref:** https://web.dev/articles/content-visibility

### Non-composited animasyonlar
**Öneriler:** Animasyonlarda transform ve opacity disindaki CSS ozelliklerinden kaçınılmasi
tavsiye edilmektedir. will-change: transform tanımının eklenmesi, GPU-accelerated
animasyonlara gecis yapilmasi performansi iyileştirecektir.
**Ref:** https://web.dev/articles/animations-guide

---

## Sunum slayt sablonu

### Skor kartı
- Desktop + mobil CLS değerleri yan yana (büyük metrik + kartlar)
- Final-screenshot: desktop sol, mobil sağ (DİKDÖRTGEN, kaliteli)
- Filmstrip: KÜÇÜK kareler (max 1.6 inch/kare), büyütme

### Etkilenen tüm elementler - TEK SLAYT

Kayma yaşatan TÜM div/element'ler tek slayttta tablo halinde gösterilir:
- Başlık: "Sabitlenmesi Gereken Alanlar"
- Açıklama: "Aşağıdaki elementlerin en-boy oranlarının sabitlenmesi veya
  placeholder alanlarının belirlenmesi önerilmektedir."
- Tablo: Element (CSS selector) | Kayma Değeri | Boyut | Önerilen Düzeltme
- Scroll öncesi kısımda ek div'ler varsa onların da en-boy oranları sabitlenmeli notu

Bu slayt, IT ekibinin doğrudan task olarak kullanabileceği bir checklist görevi görür.

### Shift görselleştirme slaytı
- Filmstrip'te shift olan kareler arası fark
- Desktop ve mobil ayrı gösterilir

### Bulgu slaytları (tespit → etki → çözüm) her önemli bulgu için

Her slayt DOLU olmalı - sol metin + sağ görsel kolon yapısında.
Açıklamalar anlaşılır, somut ve aksiyon odaklı.

---

## Excel sheet yapısı

Sheet adı: "CLS Kararlılığı"
Sütunlar: Sayfa Tipi | Cihaz | Bulgu | Element (selector) | Shift Değeri |
Kayma Yönü (px) | Above-fold | width Attr | height Attr | aspect-ratio CSS |
min-height | Önerilen Aksiyon | CLS Etkisi | Öncelik | Efor | Sorumluluk | Durum
