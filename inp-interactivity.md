# Sub-skill 5: INP Etkilesim

Etkilesim faz analizi, DOM boyutu, forced reflow, CSS selecici karmasikligi, rendering verimliligi.

## Lighthouse audit eslestirme

| Insight Audit ID | Kontrol alani |
|---|---|
| interaction-to-next-paint-insight | INP faz breakdown (input delay + processing + paint) |
| dom-size-insight | Asiri element sayisi / derinlik |
| forced-reflow | DOM okuma-yazma interleaving'i (diagnostic) |
| css-selector-complexity | Eslestirmesi pahali CSS kurallari (diagnostic) |
| unused-css-rules | Kullanilmayan CSS |
| unminified-css | Minify edilmemis CSS |

Ek kontroller: event listener sayisi, content-visibility: auto firsatlari,
will-change kullanimi, requestAnimationFrame firsatlari.

## PSI API'den cekilecek veriler

```
audits['interaction-to-next-paint-insight'] → INP faz analizi
audits['dom-size-insight']                  → DOM istatistikleri
audits['forced-reflow']                     → Reflow tespitleri
audits['css-selector-complexity']           → Pahali selectorler
audits['unused-css-rules']                  → Kullanilmayan CSS
audits['unminified-css']                    → Minify edilmemis CSS
audits['total-blocking-time']               → TBT (INP lab proxy)
loadingExperience.metrics.INTERACTION_TO_NEXT_PAINT → field INP
```

## Chrome DevTools kontrolleri

**DOM boyutu ve derinlik:**
```javascript
const allElements = document.querySelectorAll('*').length;
const bodyChildren = document.body.children.length;
const maxDepth = (function getMax(el, d) {
  let m = d;
  for (const c of el.children) m = Math.max(m, getMax(c, d + 1));
  return m;
})(document.body, 0);
const deepestElement = (function findDeepest(el, d, max) {
  let result = { el, depth: d };
  for (const c of el.children) {
    const child = findDeepest(c, d + 1, max);
    if (child.depth > result.depth) result = child;
  }
  return result;
})(document.body, 0);
```

**Event listener sayisi:**
```javascript
// Not: getEventListeners sadece DevTools console'da calisir
// Alternatif: bilinen event attributelerini tara
const interactiveEls = document.querySelectorAll(
  '[onclick],[onchange],[onsubmit],[onscroll],[onmouseover],[ontouchstart]'
);
const formEls = document.querySelectorAll('input, select, textarea, button, a[href]');
```

**Unused CSS kontrolu:**
```javascript
const stylesheets = Array.from(document.styleSheets);
const totalRules = stylesheets.reduce((sum, ss) => {
  try { return sum + ss.cssRules.length; } catch(e) { return sum; }
}, 0);
```

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: DOM element sayisi, max nesting derinligi,
event handler tespiti. on_page_content_parsing ile yapisal analiz.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: toplam element sayisi (tag count), nesting derinligi,
inline event handler tespiti, stylesheet sayisi ve boyutu.

---

## Bulgu kaliplari

### Asiri DOM boyutu
**Esikler:** >800 element: dikkat, >1400 element: uyari, >1800 element: kritik.
Max derinlik >32: uyari.
**Cozumler:** Gereksiz wrapper div'leri kaldir, lazy rendering (content-visibility: auto),
virtualized list (uzun listeler icin), DOM temizligi.
**Ref:** https://web.dev/articles/dom-size-and-interactivity

### Forced reflow
**Cozumler:** DOM okuma ve yazma islemlerini ayir. Batch DOM writes kullan.
requestAnimationFrame icinde layout okumalari yap.
**Ref:** https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing

### CSS selector karmasikligi
**Cozumler:** Derin nesting'den kacin (.a .b .c .d .e gibi). BEM veya utility-first
yaklasim kullan. Evrensel selectorler (*) azalt.
**Ref:** https://web.dev/articles/reduce-the-scope-and-complexity-of-style-calculations

### Kullanilmayan CSS
**Cozumler:** Coverage analizi ile kullanilmayan kurallari tespit et ve kaldir.
Critical CSS ayir, geri kalanini lazy yukle. PurgeCSS / uncss kullan.
**Ref:** https://web.dev/articles/unused-css-rules

### content-visibility firsatlari
**Cozumler:** Uzun sayfalarfda below-fold icerige content-visibility: auto ekle.
contain-intrinsic-size ile tahmini boyut tanimla.
```css
.below-fold-section {
  content-visibility: auto;
  contain-intrinsic-size: 0 500px;
}
```
**Ref:** https://web.dev/articles/content-visibility

---

## Sunum slayt sablonu

### Skor karti
- TBT (lab) ve INP field degeri
- DOM istatistikleri: element sayisi, max derinlik
- Long task sayisi

### INP faz breakdown slayti
- Input delay + processing + presentation delay
- Hangi faz darbogaz vurgusu

### Bulgu slaytlari (tespit → etki → cozum) her bulgu icin

---

## Excel sheet yapisi

Sheet adi: "INP Etkilesim"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Audit ID | Element/Selector |
Mevcut Deger | Google Esigi | Onerilen Aksiyon | INP Faz Etkisi |
Oncelik | Efor | Sorumluluk | Durum
