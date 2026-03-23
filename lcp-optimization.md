# Sub-skill 3: LCP Optimizasyonu

LCP element kesfi, faz analizi, gorsel format/boyut, srcset, lazy/preload stratejisi.

## Lighthouse audit eslestirme

| Insight Audit ID | Kontrol alani |
|---|---|
| lcp-discovery-insight | fetchpriority, lazy loading, preload, HTML kesfedilebilirlik |
| lcp-phases-insight | TTFB → load delay → load duration → render delay |
| image-delivery-insight | WebP/AVIF, sikistirma, responsive boyut, animasyonlu icerik |
| total-byte-weight | Toplam sayfa agirligi |

Ek kontroller: intrinsic vs display boyut, srcset/sizes dogrulugu,
below-fold lazy loading eksikligi, above-fold lazy loading hatasi.

## PSI API'den cekilecek veriler

```
audits['lcp-discovery-insight']        → fetchpriority, lazy, preload kontrol
audits['lcp-phases-insight']           → LCP element bilgisi + faz breakdown
audits['image-delivery-insight']       → Gorsel format/boyut onerileri
audits['total-byte-weight']
audits['final-screenshot']             → LCP gorsel kaniti
audits['screenshot-thumbnails']        → Filmstrip
loadingExperience.metrics.LARGEST_CONTENTFUL_PAINT_MS → Field LCP
```

LCP element bilgisi:
```
audits['lcp-phases-insight'].details.items[0].node → {
  selector: ".hero-slider img",
  snippet: "<img src=\"hero.jpg\" alt=\"...\">",
  boundingRect: { width, height, top, left }
}
```

## Chrome DevTools kontrolleri

**LCP element tespiti + attribute kontrolu:**
```javascript
// PerformanceObserver ile LCP element'i yakala
const lcpEntries = performance.getEntriesByType('largest-contentful-paint');
const lcp = lcpEntries[lcpEntries.length - 1];
const el = lcp?.element;
if (el && el.tagName === 'IMG') {
  return {
    selector: el.className ? '.' + el.className.split(' ').join('.') : el.tagName,
    src: el.src,
    loading: el.getAttribute('loading'),
    fetchpriority: el.getAttribute('fetchpriority'),
    hasWidth: el.hasAttribute('width'),
    hasHeight: el.hasAttribute('height'),
    hasSrcset: el.hasAttribute('srcset'),
    naturalSize: el.naturalWidth + 'x' + el.naturalHeight,
    displaySize: Math.round(el.getBoundingClientRect().width) + 'x' +
      Math.round(el.getBoundingClientRect().height),
    sizeWaste: el.naturalWidth > 0 ?
      Math.round((1 - el.getBoundingClientRect().width / el.naturalWidth) * 100) + '%' : 'N/A'
  };
}
```

**Tum gorsellerin attribute taramasi:**
```javascript
const imgs = Array.from(document.querySelectorAll('img'));
const audit = imgs.map(img => {
  const rect = img.getBoundingClientRect();
  return {
    src: img.src,
    loading: img.getAttribute('loading'),
    fetchpriority: img.getAttribute('fetchpriority'),
    hasWidth: img.hasAttribute('width'),
    hasHeight: img.hasAttribute('height'),
    hasSrcset: img.hasAttribute('srcset'),
    hasSizes: img.hasAttribute('sizes'),
    alt: img.alt,
    naturalWidth: img.naturalWidth,
    naturalHeight: img.naturalHeight,
    displayWidth: Math.round(rect.width),
    displayHeight: Math.round(rect.height),
    aboveFold: rect.top < window.innerHeight && rect.bottom > 0,
    format: img.src.match(/\.(webp|avif|jpg|jpeg|png|gif|svg)(\?|$)/i)?.[1] || 'unknown'
  };
});
// Sampling yapma - tum gorseller taranir
```

**Boyut waste analizi:**
```javascript
const oversized = audit.filter(i =>
  i.naturalWidth > 0 && i.displayWidth > 0 &&
  i.naturalWidth > i.displayWidth * 1.5
).map(i => ({
  src: i.src.substring(0, 80),
  natural: i.naturalWidth + 'x' + i.naturalHeight,
  display: i.displayWidth + 'x' + i.displayHeight,
  waste: Math.round((1 - i.displayWidth / i.naturalWidth) * 100) + '%'
}));
```

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: tum img elementlerinin attribute taramasi
(loading, fetchpriority, width, height, srcset, sizes, naturalWidth vs clientWidth),
LCP element tespiti. on_page_lighthouse (full_data: true) ile LCP faz breakdown.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: tum img tagleri parse et - src, loading, fetchpriority, width, height,
srcset, sizes, alt attribute'lari. picture/source elementleri. Gorsel format tespiti
(URL uzantisidan).

---

## Bulgu kaliplari

### LCP gorseli lazy loaded
**Oneriler:** Above-fold konumundaki LCP gorselinden loading="lazy" attribute'unun
kaldirilmasi onerilmektedir. fetchpriority="high" attribute'u eklenerek tarayicinin
bu gorseli oncelikli olarak yuklemesi saglanabilir.
**Ref:** https://web.dev/articles/lcp-lazy-loading

### fetchpriority eksik
**Oneriler:** LCP gorsel elementine fetchpriority="high" attribute'u eklenmesi tavsiye
edilmektedir. Eger preload link kullaniliyorsa, preload'a da fetchpriority eklenmesi
uygun olacaktir.
**Ref:** https://web.dev/articles/fetch-priority

### Gorsel format (WebP/AVIF)
**Oneriler:** Gorsellerin picture elementi ile WebP veya AVIF formatinda sunulmasi,
JPEG/PNG'nin fallback olarak korunmasi onerilmektedir. CDN uzerinden otomatik format
donusumu (Cloudinary, imgix, Cloudflare Images) degerlendirilmelidir.
**Ref:** https://web.dev/articles/serve-images-webp

### Boyut uyumsuzlugu (intrinsic vs display)
**Oneriler:** Gorsellerin display boyutuna uygun boyutta sunulmasi, gereksiz boyut
aktariminin onlenmesi tavsiye edilmektedir. srcset attribute'u ile farkli viewport
boyutlarina uygun gorsel boyutlarinin tanimlanmasi, CDN image resize kullaniminin
degerlendirilmesi onerilmektedir.
**Ref:** https://web.dev/articles/serve-responsive-images

### Below-fold lazy loading eksik
**Oneriler:** Viewport disindaki tum gorsellere loading="lazy" attribute'u eklenmesi
onerilmektedir. Onemli not: Above-fold gorsellerde lazy loading kullanilmamalidir,
bu durum LCP'yi olumsuz etkileyecektir.
**Ref:** https://web.dev/articles/browser-level-image-lazy-loading

---

## Sunum slayt sablonu

### Skor karti
- LCP field degeri (good/needs improvement/poor)
- LCP element screenshot'i (final-screenshot + element bbox isaretli)
- Filmstrip

### LCP faz breakdown slayti
- TTFB → resource load delay → resource load duration → render delay
- Bar chart ile faz dagilimi gorseli
- Hangi faz darbogaz? vurgusu

### Gorsel envanter slayti (onemli bulgularda)
- Above-fold gorseller tablosu: format, boyut, lazy, fetchpriority, width/height
- Boyut waste ozetli

### Bulgu slaytlari (tespit → etki → cozum) her bulgu icin

---

## Excel sheet yapisi

Sheet adi: "LCP Optimizasyonu"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Element (selector) | Gorsel URL |
Mevcut Format | Onerilen Format | Natural Boyut | Display Boyut | Boyut Waste % |
loading Attr | fetchpriority Attr | width/height | srcset | Onerilen Aksiyon |
LCP Faz Etkisi | Oncelik | Efor | Sorumluluk | Durum
