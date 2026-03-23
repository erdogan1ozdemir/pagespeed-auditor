# Sub-skill 2: JavaScript Yuku

Main thread'i mesgul eden JS sorunlari: gereksiz kod, 3rd party, execution time, CSR/SSR.

## Lighthouse audit eşleştirme

| Insight Audit ID | Kontrol alani |
|---|---|
| third-parties-insight | 3rd party script envanteri, boyut, CPU etkisi |
| unused-javascript | Yuklenen ama çalıştırilmayan JS |
| duplicated-javascript-insight | Ayni modul birden fazla yukleniyor |
| legacy-javascript-insight | Modern tarayıcılara gereksiz polyfill |
| bootup-time | Toplam JS parse + compile + execute |
| mainthread-work-breakdown | Script, layout, style, paint dagilimi |
| long-tasks | 50ms'den uzun main thread gorevleri |
| unminified-javascript | Minify edilmemis JS dosyaları |

Ek kontroller: script defer/async/module kontrolu, CSR vs SSR tespiti, JS ağırlığı breakdown.

## PSI API'den cekilecek veriler

```
audits['third-parties-insight']
audits['unused-javascript']
audits['duplicated-javascript-insight']
audits['legacy-javascript-insight']
audits['bootup-time']
audits['mainthread-work-breakdown']
audits['long-tasks']
audits['unminified-javascript']
audits['total-byte-weight']
loadingExperience.metrics
```

## Chrome DevTools kontrolleri

**Script defer/async tarama:**
```javascript
const scripts = Array.from(document.querySelectorAll('script[src]'));
const audit = scripts.map(s => ({
  src: s.src,
  defer: s.defer,
  async: s.async,
  type: s.type || 'classic',
  isThirdParty: !s.src.includes(location.hostname),
  domain: new URL(s.src).hostname
}));
const summary = {
  total: audit.length,
  withDefer: audit.filter(s => s.defer).length,
  withAsync: audit.filter(s => s.async).length,
  noStrategy: audit.filter(s => !s.defer && !s.async && s.type !== 'module').length,
  thirdPartyDomains: [...new Set(audit.filter(s => s.isThirdParty).map(s => s.domain))]
};
```

**Network JS breakdown:**
```javascript
const jsEntries = performance.getEntriesByType('resource')
  .filter(e => e.name.match(/\.js(\?|$)/i))
  .map(e => ({
    url: e.name.substring(0, 100),
    transferSize: e.transferSize,
    duration: Math.round(e.duration),
    isThirdParty: !e.name.includes(location.hostname)
  }))
  .sort((a, b) => b.transferSize - a.transferSize);
```

**CSR tespiti:**
```javascript
// Body'nin JS olmadan içerik içerip icermedigini kontrol et
const bodyText = document.body.innerText.trim();
const hasNoscript = document.querySelector('noscript');
const rootEl = document.getElementById('root') || document.getElementById('app') || document.getElementById('__next');
const isCSR = rootEl && bodyText.length < 200;
```

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: script defer/async taramasi, 3rd party domain
envanteri, DOM analizi. on_page_content_parsing ile inline script tespiti.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: tum script[src] taglerinin defer/async durumu, inline script boyutu,
noscript içeriği (SSR tespiti), 3rd party script domainleri.

---

## Bulgu kaliplari

### 3rd party script yuku
**Öneriler:** Kullanılmayan ucuncu parti scriptlerin kaldırılması önerilmektedir.
Gerekli olan scriptlerin defer veya async ile yüklenmesi değerlendirilebilir.
Facade pattern uygulanması (ornegin YouTube embed yerine onizleme görseli kullanımı,
tiklandiginda gerçek player yüklemesi) kaynak tuketimini önemli ölçüde azaltacaktır.
**Ref:** https://web.dev/articles/third-party-summary

### Kullanılmayan JavaScript
**Öneriler:** Code coverage analizi ile kullanılmayan kodun tespit edilmesi ve temizlenmesi
tavsiye edilmektedir. Tree-shaking ve code-splitting uygulanması, dynamic import ile
lazy loading stratejisi değerlendirilmelidir.
**Ref:** https://web.dev/articles/unused-javascript

### Script defer/async eksikligi
**Öneriler:** Kritik olmayan scriptlere defer attribute'u eklenmesi önerilmektedir.
Birbirinden bağımsız scriptler için async kullanımı uygun olacaktır.
ES module kullanımı (type="module") otomatik defer davranisi sağlayacaktır.

### CSR/SSR etkisi
**Öneriler:** Server-side rendering (SSR) veya static site generation (SSG) yaklaşımlarinin
değerlendirilmesi önerilmektedir. Kritik içeriğin sunucu tarafinda render edilmesi,
hydration overhead'inin partial hydration ile azaltılmasi faydalı olacaktır.

---

## Sunum slayt sablonu

### Skor karti slayti
- TBT, bootup-time, main thread breakdown değerleri
- Long task sayisi
- Filmstrip

### 3rd party envanter slayti
- Domain bazli tablo: domain, script sayisi, toplam boyut, defer/async durumu
- Gerçek HTML'den alinan script tag ornekleri

### Bulgu slaytlari (tespit → etki → çözüm) her bulgu icin

---

## Excel sheet yapisi

Sheet adi: "JavaScript Yuku"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Audit ID | Script URL/Domain |
Transfer Boyutu (KB) | Defer | Async | 1st/3rd Party | Main Thread Etkisi (ms) |
Önerilen Aksiyon | Öncelik | Efor | Sorumluluk | Durum
