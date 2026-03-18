# Sub-skill 4: CLS Kararliligi

Layout shift nedenleri, boyutsuz ogeler, animasyonlar, aspect-ratio, font kaymasi, ad/embed slotlari.

## Lighthouse audit eslestirme

| Insight Audit ID | Kontrol alani |
|---|---|
| cls-culprits-insight | Layout shift'e neden olan ogeler, kayma buyuklugu |
| unsized-images | width/height olmayan img, video, iframe (diagnostic) |
| non-composited-animations | GPU yerine CPU ile calisan animasyonlar (diagnostic) |

Ek kontroller: aspect-ratio CSS, min-height rezervasyon, dinamik inject, ad slot,
font-display kaymasi, 3rd party inject CLS etkisi.

## PSI API'den cekilecek veriler

```
audits['cls-culprits-insight']         → shift eden elementler, degerler
audits['unsized-images']               → boyutsuz gorsel listesi
audits['non-composited-animations']    → CPU animasyonlari
audits['cumulative-layout-shift']      → CLS skoru
audits['screenshot-thumbnails']        → filmstrip (shift karsilastirmasi icin)
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

**Boyutsuz oge taramasi:**
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

**GIF kaydi stratejisi (CLS gorselestirme):**
Chrome extension varsa coklu screenshot stratejisi kullan:
1. gif_creator start_recording
2. navigate → hemen screenshot
3. wait 0.5s → screenshot
4. wait 0.5s → screenshot
5. wait 1s → screenshot
6. gif_creator stop_recording → export

Eger GIF yetersizse, filmstrip karelerini yan yana koyarak shift gorsellestir.

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: boyutsuz oge taramasi (img/video/iframe width/height),
aspect-ratio CSS kontrolu, ad slot min-height kontrolu.
on_page_lighthouse (full_data: true) ile CLS culprit detayi ve filmstrip kareleri.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: tum img/video/iframe taglerinde width, height attribute kontrolu.
Inline style'da aspect-ratio. Ad container'larda min-height.
Filmstrip kareleri (PSI API'den) arasindaki farklar ile CLS gorselestirme.

---

## Bulgu kaliplari

### Boyutsuz gorseller
**Cozumler:** Tum img/video/iframe'lere width ve height attribute ekle.
CSS'te aspect-ratio tanimla. Container'a min-height ekle.
```html
<!-- Mevcut -->
<img src="hero.jpg" alt="...">
<!-- Onerilen -->
<img src="hero.jpg" alt="..." width="1200" height="675">
```
```css
.hero-img { aspect-ratio: 16/9; width: 100%; height: auto; }
```
**Ref:** https://web.dev/articles/optimize-cls

### Ad/embed slot kaymasi
**Cozumler:** Ad container'a min-height tanimla. Placeholder/skeleton UI kullan.
Icerik yuklenene kadar alan rezerve et.
```css
.ad-container { min-height: 250px; }
```
**Ref:** https://web.dev/articles/optimize-cls#ads-embeds-and-iframes-without-dimensions

### Font kaymasi (FOUT)
**Cozumler:** font-display: optional (CLS'i onler), fallback font metrik eslestirme
(size-adjust, ascent-override, descent-override).
**Ref:** https://web.dev/articles/font-display

### Dinamik inject edilen icerik
**Cozumler:** JS ile inject edilen icerik icin onceden alan ayir.
content-visibility: auto kullanimi degerlendir.
**Ref:** https://web.dev/articles/content-visibility

### Non-composited animasyonlar
**Cozumler:** transform ve opacity disindaki animasyonlardan kacin.
will-change: transform ekle. GPU-accelerated animasyonlar kullan.
**Ref:** https://web.dev/articles/animations-guide

---

## Sunum slayt sablonu

### Skor karti
- CLS field degeri (good/needs improvement/poor)
- Toplam shift sayisi + en buyuk shift degeri
- Filmstrip (shift anlari isaretli)

### Shift gorselestirme slayti
- Filmstrip karelerinde shift oncesi/sonrasi karsilastirma
- GIF (Chrome varsa) veya yan yana screenshot karsilastirmasi

### Boyutsuz oge envanter slayti
- Tablo: element, boyut durumu, above-fold mi, aspect-ratio var mi
- Etkilenen alan screenshot ile isaretli

### Bulgu slaytlari (tespit → etki → cozum) her bulgu icin

---

## Excel sheet yapisi

Sheet adi: "CLS Kararliligi"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Element (selector) | Shift Degeri |
Kayma Yonu (px) | Above-fold | width Attr | height Attr | aspect-ratio CSS |
min-height | Onerilen Aksiyon | CLS Etkisi | Oncelik | Efor | Sorumluluk | Durum
