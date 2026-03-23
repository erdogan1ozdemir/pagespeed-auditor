# Sub-skill 5: INP Etkileşim

Etkileşim faz analizi, DOM boyutu, forced reflow, CSS selecici karmaşıklığı, rendering verimliligi.

## Lighthouse audit eşleştirme

| Insight Audit ID | Kontrol alani |
|---|---|
| interaction-to-next-paint-insight | INP faz breakdown (input delay + processing + paint) |
| dom-size-insight | Aşırı element sayisi / derinlik |
| forced-reflow | DOM okuma-yazma interleaving'i (diagnostic) |
| css-selector-complexity | Eşleştirmesi pahali CSS kurallari (diagnostic) |
| unused-css-rules | Kullanılmayan CSS |
| unminified-css | Minify edilmemis CSS |

Ek kontroller: event listener sayisi, content-visibility: auto firsatlari,
will-change kullanımı, requestAnimationFrame firsatlari.

## PSI API'den cekilecek veriler

```
audits['interaction-to-next-paint-insight'] → INP faz analizi
audits['dom-size-insight']                  → DOM istatistikleri
audits['forced-reflow']                     → Reflow tespitleri
audits['css-selector-complexity']           → Pahali selectorler
audits['unused-css-rules']                  → Kullanılmayan CSS
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
// Not: getEventListeners sadece DevTools console'da çalışır
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

### Aşırı DOM boyutu
**Esikler:** >800 element: dikkat, >1400 element: uyari, >1800 element: kritik.
Max derinlik >32: uyari.
**Öneriler:** Gereksiz wrapper div elementlerinin temizlenmesi önerilmektedir.
Uzun listeler için virtualized list yaklaşımi, below-fold içerik icin
content-visibility: auto kullanımı değerlendirilmelidir.
**Ref:** https://web.dev/articles/dom-size-and-interactivity

### Forced reflow
**Öneriler:** DOM okuma ve yazma işlemlerinin birbirinden ayrilmasi tavsiye edilmektedir.
Toplu DOM yazma işlemleri (batch DOM writes) kullanilmasi, layout okumalarinin
requestAnimationFrame icinde yapilmasi performansi iyileştirecektir.
**Ref:** https://web.dev/articles/avoid-large-complex-layouts-and-layout-thrashing

### CSS selector karmaşıklığı
**Öneriler:** Derin CSS nesting'den kaçınılmasi (.a .b .c .d .e gibi zincirler)
önerilmektedir. BEM veya utility-first yaklaşımin değerlendirilmesi, evrensel
selectorlerin (*) azaltılmasi tavsiye edilmektedir.
**Ref:** https://web.dev/articles/reduce-the-scope-and-complexity-of-style-calculations

### Kullanılmayan CSS
**Öneriler:** Coverage analizi ile kullanılmayan CSS kurallarinin tespit edilmesi ve
temizlenmesi tavsiye edilmektedir. Kritik CSS'in ayrilmasi, geri kalanin lazy
yüklenmesi önerilmektedir. PurgeCSS veya uncss gibi araclar değerlendirilmelidir.
**Ref:** https://web.dev/articles/unused-css-rules

### content-visibility firsatlari
**Öneriler:** Uzun sayfalarda below-fold icerige content-visibility: auto eklenmesi
önerilmektedir. contain-intrinsic-size ile tahmini boyut tanımlanmasi, render
performansini önemli ölçüde iyileştirecektir.
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
- TBT (lab) ve INP field değeri
- DOM istatistikleri: element sayisi, max derinlik
- Long task sayisi

### INP faz breakdown slayti
- Input delay + processing + presentation delay
- Hangi faz darbogaz vurgusu

### Bulgu slaytlari (tespit → etki → çözüm) her bulgu icin

---

## Excel sheet yapisi

Sheet adi: "INP Etkileşim"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Audit ID | Element/Selector |
Mevcut Değer | Google Esigi | Önerilen Aksiyon | INP Faz Etkisi |
Öncelik | Efor | Sorumluluk | Durum
