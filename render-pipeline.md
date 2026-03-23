# Sub-skill 1: Render Pipeline

Tarayıcınin ilk render'ini geçıktıren sorunlar: TTFB, render-blocking, ag bagimliligi, font, cache, resource hints.

## Lighthouse audit eşleştirme

| Insight Audit ID | Kontrol alani |
|---|---|
| document-latency-insight | TTFB, redirect zinciri, compression |
| render-blocking-insight | Render engelleyen CSS/JS |
| network-dependency-tree-insight | Kritik istek zinciri, preconnect |
| use-cache-insight | Yetersiz cache TTL |
| modern-http-insight | HTTP/2 veya HTTP/3 |
| font-display-insight | Font görünürlük stratejisi |

Ek kontroller (Lighthouse disinda): preload/prefetch, 103 Early Hints, priority hints,
sayfa ağırlığı breakdown (JS/CSS/font/görsel), CDN kontrolu, Brotli vs gzip.

## PSI API'den cekilecek veriler

```
audits['document-latency-insight']
audits['render-blocking-insight']
audits['network-dependency-tree-insight']
audits['use-cache-insight']
audits['modern-http-insight']
audits['font-display-insight']
audits['total-byte-weight']
audits['screenshot-thumbnails']        → filmstrip
audits['final-screenshot']             → screenshot
loadingExperience.metrics              → field CrUX
```

## Chrome DevTools kontrolleri

**Resource hints:**
```javascript
const links = Array.from(document.querySelectorAll('link'));
const hints = links.filter(l =>
  ['preload','preconnect','dns-prefetch','prefetch'].includes(l.rel)
).map(l => ({
  rel: l.rel, href: l.href, as: l.getAttribute('as'),
  crossorigin: l.getAttribute('crossorigin'),
  fetchpriority: l.getAttribute('fetchpriority')
}));
```

**Sayfa ağırlığı breakdown:**
```javascript
const entries = performance.getEntriesByType('resource');
const breakdown = {};
entries.forEach(e => {
  const ext = e.name.split('.').pop().split('?')[0].toLowerCase();
  const type = ['js'].includes(ext) ? 'JavaScript' :
    ['css'].includes(ext) ? 'CSS' :
    ['woff','woff2','ttf','otf','eot'].includes(ext) ? 'Font' :
    ['jpg','jpeg','png','gif','webp','avif','svg'].includes(ext) ? 'Görsel' : 'Diger';
  if (!breakdown[type]) breakdown[type] = { count: 0, totalSize: 0 };
  breakdown[type].count++;
  breakdown[type].totalSize += e.transferSize || 0;
});
```

**Compression kontrolu:**
```javascript
const uncompressed = performance.getEntriesByType('resource').filter(e =>
  e.encodedBodySize > 0 && e.decodedBodySize > 0 &&
  e.encodedBodySize === e.decodedBodySize && e.decodedBodySize > 1024
).map(e => ({ url: e.name.substring(0, 80), size: e.decodedBodySize }));
```

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js çalıştırarak Chrome DevTools kontrollerinin
cogunu sunucu tarafinda yapabilirsin. Ayni JS scriptleri custom_js parametresine ver.
on_page_lighthouse ile full_data: true gonderip Lighthouse JSON'dan audit verilerini cek.
on_page_content_parsing ile heading yapısı ve link yapısı al.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: link rel="preconnect/dns-prefetch/preload" varligi, render-blocking
stylesheet ve script (defer/async olmayan) tespiti, meta http-equiv redirect.

---

## Bulgu kaliplari

### TTFB yavasi
**Öneriler:** Sunucu tarafinda cache mekanizmasi (Redis, Varnish, CDN edge cache) kullanilmasi
önerilmektedir. Redirect zincirlerinin kısaltılması, Brotli/gzip compression'in aktif edilmesi
ve CDN kullanımınin değerlendirilmesi yanitlama süresini iyileştirecektir.
**Ref:** https://web.dev/articles/ttfb

### Render-blocking kaynaklar
**Öneriler:** Above-the-fold içeriği için gereken kritik CSS'in inline olarak yerleştirilmesi
tavsiye edilmektedir. JavaScript dosyalarına defer attribute'u eklenmesi, kritik olmayan CSS'in
media attribute ile asenkron yüklenmesi değerlendirilebilir.
**Ref:** https://web.dev/articles/render-blocking-resources

### Cache yetersizligi
**Öneriler:** Statik kaynaklar için uzun sureli cache (max-age=31536000, immutable) ve
dosya adi fingerprinting uygulanması önerilmektedir. HTML dokumanlari için daha kisa
sureli cache (max-age=3600) yeterli olacaktır.
**Ref:** https://web.dev/articles/uses-long-cache-ttl

### HTTP/2-3 eksikligi
**Öneriler:** Sunucu veya CDN konfigurasyonunda HTTP/2 desteginin aktif edilmesi tavsiye
edilmektedir. HTTP/3 (QUIC) destegi için Cloudflare veya Fastly gibi CDN çözümleri
değerlendirilebilir.
**Ref:** https://web.dev/articles/performance-http2

### Font görünürlük
**Öneriler:** Font-display değerinin swap veya optional olarak ayarlanmasi, kritik fontlarin
preload ile on yüklenmesi ve font subset kullanilmasi önerilmektedir. Fallback font icin
metrik eşleştirme (size-adjust) tanımlanmasi da CLS'i azaltacaktır.
**Ref:** https://web.dev/articles/font-display

### Preconnect/dns-prefetch eksik
**Öneriler:** Ucuncu parti kaynaklarin bulundugu domainlere preconnect tanımlanmasi,
baglanti kurulma süresini kısaltacaktır:
`<link rel="preconnect" href="https://cdn.example.com" crossorigin>`
**Ref:** https://web.dev/articles/uses-rel-preconnect

---

## Sunum slayt sablonu

### Skor karti slayti
- Breadcrumb: "{Sayfa} {Cihaz} | Render Pipeline"
- TTFB, FCP, SI field değerleri tablosu (good/needs improvement/poor renk kodlu)
- Filmstrip gomu

### Bulgu - tespit slayti
- Breadcrumb: "{Sayfa} {Cihaz} | Bulgu {n}/{toplam}"
- Tespit açıklamasi (➔ maddeler)
- Etkilenen kaynaklar listesi/tablosu
- Mevcut HTML/config kod blogu

### Bulgu - etki slayti
- Kullanici deneyimi + metrik etkisi + Google referansi

### Bulgu - çözüm slayti
- Öneriler (numarali)
- Mevcut vs önerilen kod
- Beklenen iyileşme + efor + öncelik + sorumluluk
- Excel referans notu

---

## Excel sheet yapisi

Sheet adi: "Render Pipeline"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Audit ID | Etkilenen Kaynak | Mevcut Değer |
Google Esigi | Önerilen Aksiyon | Etkilenen Metrikler | Tahmini Tasarruf (ms) |
Öncelik | Efor | Sorumluluk | Durum
