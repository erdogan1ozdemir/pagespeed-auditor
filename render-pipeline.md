# Sub-skill 1: Render Pipeline

Tarayicinin ilk render'ini geciktiren sorunlar: TTFB, render-blocking, ag bagimliligi, font, cache, resource hints.

## Lighthouse audit eslestirme

| Insight Audit ID | Kontrol alani |
|---|---|
| document-latency-insight | TTFB, redirect zinciri, compression |
| render-blocking-insight | Render engelleyen CSS/JS |
| network-dependency-tree-insight | Kritik istek zinciri, preconnect |
| use-cache-insight | Yetersiz cache TTL |
| modern-http-insight | HTTP/2 veya HTTP/3 |
| font-display-insight | Font gorunurluk stratejisi |

Ek kontroller (Lighthouse disinda): preload/prefetch, 103 Early Hints, priority hints,
sayfa agirligi breakdown (JS/CSS/font/gorsel), CDN kontrolu, Brotli vs gzip.

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

**Sayfa agirligi breakdown:**
```javascript
const entries = performance.getEntriesByType('resource');
const breakdown = {};
entries.forEach(e => {
  const ext = e.name.split('.').pop().split('?')[0].toLowerCase();
  const type = ['js'].includes(ext) ? 'JavaScript' :
    ['css'].includes(ext) ? 'CSS' :
    ['woff','woff2','ttf','otf','eot'].includes(ext) ? 'Font' :
    ['jpg','jpeg','png','gif','webp','avif','svg'].includes(ext) ? 'Gorsel' : 'Diger';
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

on_page_instant_pages ile custom_js calistirarak Chrome DevTools kontrollerinin
cogunu sunucu tarafinda yapabilirsin. Ayni JS scriptleri custom_js parametresine ver.
on_page_lighthouse ile full_data: true gonderip Lighthouse JSON'dan audit verilerini cek.
on_page_content_parsing ile heading yapisi ve link yapisi al.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: link rel="preconnect/dns-prefetch/preload" varligi, render-blocking
stylesheet ve script (defer/async olmayan) tespiti, meta http-equiv redirect.

---

## Bulgu kaliplari

### TTFB yavasi
**Oneriler:** Sunucu tarafinda cache mekanizmasi (Redis, Varnish, CDN edge cache) kullanilmasi
onerilmektedir. Redirect zincirlerinin kisaltilmasi, Brotli/gzip compression'in aktif edilmesi
ve CDN kullaniminin degerlendirilmesi yanitlama suresini iyilestirecektir.
**Ref:** https://web.dev/articles/ttfb

### Render-blocking kaynaklar
**Oneriler:** Above-the-fold icerigi icin gereken kritik CSS'in inline olarak yerlestirilmesi
tavsiye edilmektedir. JavaScript dosyalarina defer attribute'u eklenmesi, kritik olmayan CSS'in
media attribute ile asenkron yuklenmesi degerlendirilebilir.
**Ref:** https://web.dev/articles/render-blocking-resources

### Cache yetersizligi
**Oneriler:** Statik kaynaklar icin uzun sureli cache (max-age=31536000, immutable) ve
dosya adi fingerprinting uygulanmasi onerilmektedir. HTML dokumanlari icin daha kisa
sureli cache (max-age=3600) yeterli olacaktir.
**Ref:** https://web.dev/articles/uses-long-cache-ttl

### HTTP/2-3 eksikligi
**Oneriler:** Sunucu veya CDN konfigurasyonunda HTTP/2 desteginin aktif edilmesi tavsiye
edilmektedir. HTTP/3 (QUIC) destegi icin Cloudflare veya Fastly gibi CDN cozumleri
degerlendirilebilir.
**Ref:** https://web.dev/articles/performance-http2

### Font gorunurluk
**Oneriler:** Font-display degerinin swap veya optional olarak ayarlanmasi, kritik fontlarin
preload ile on yuklenmesi ve font subset kullanilmasi onerilmektedir. Fallback font icin
metrik eslestirme (size-adjust) tanimlanmasi da CLS'i azaltacaktir.
**Ref:** https://web.dev/articles/font-display

### Preconnect/dns-prefetch eksik
**Oneriler:** Ucuncu parti kaynaklarin bulundugu domainlere preconnect tanimlanmasi,
baglanti kurulma suresini kisaltacaktir:
`<link rel="preconnect" href="https://cdn.example.com" crossorigin>`
**Ref:** https://web.dev/articles/uses-rel-preconnect

---

## Sunum slayt sablonu

### Skor karti slayti
- Breadcrumb: "{Sayfa} {Cihaz} | Render Pipeline"
- TTFB, FCP, SI field degerleri tablosu (good/needs improvement/poor renk kodlu)
- Filmstrip gomu

### Bulgu - tespit slayti
- Breadcrumb: "{Sayfa} {Cihaz} | Bulgu {n}/{toplam}"
- Tespit aciklamasi (➔ maddeler)
- Etkilenen kaynaklar listesi/tablosu
- Mevcut HTML/config kod blogu

### Bulgu - etki slayti
- Kullanici deneyimi + metrik etkisi + Google referansi

### Bulgu - cozum slayti
- Oneriler (numarali)
- Mevcut vs onerilen kod
- Beklenen iyilesme + efor + oncelik + sorumluluk
- Excel referans notu

---

## Excel sheet yapisi

Sheet adi: "Render Pipeline"
Sutunlar: Sayfa Tipi | Cihaz | Bulgu | Audit ID | Etkilenen Kaynak | Mevcut Deger |
Google Esigi | Onerilen Aksiyon | Etkilenen Metrikler | Tahmini Tasarruf (ms) |
Oncelik | Efor | Sorumluluk | Durum
