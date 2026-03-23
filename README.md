# PageSpeed Auditor

Inbound Marketing Agency SEO ekibi icin gelistirilmis bir Claude skill'idir. Bir veya birden fazla URL icin PageSpeed Insights analizi yaparak IT ekiplerine yonelik PPTX sunum ve destekleyici Excel rapor uretir.

---

## Kurulum

### Claude.ai'da

1. Bu repository'deki `pagespeed-auditor.skill` dosyasini indirin
2. Claude.ai'da Settings > Skills bolumune gidin
3. "Add Skill" ile .skill dosyasini yukleyin

### Claude Code'da

```bash
# Repository'yi klonlayin
git clone https://github.com/[username]/pagespeed-auditor.git

# Skill dosyasini Claude Code'a ekleyin
# Claude Code'un skill dizinine kopyalayin
```

### Gereksinimler

Skill calistirildiginda gereken paketleri otomatik yukler:
- **pptxgenjs** - sunum olusturma
- **openpyxl** - Excel olusturma
- **markitdown** - onceki rapor okuma (before/after karsilastirma)

Ek olarak asagidakilerden en az biri gerekir:
- **PSI API key** (ucretsiz): https://developers.google.com/speed/docs/insights/v5/get-started
- **DataForSEO MCP** baglantisi (PSI key'e alternatif, zaten hesabiniz varsa tercih edin)

---

## Kullanim

### Temel kullanim ornekleri

```
"vitra.com.tr icin CLS analizi yap"
→ Otomatik 3 sayfa tipi algila, CLS sub-skill'i calistir, PPTX + Excel uret

"sadece LCP bak: https://www.example.com/urun/123"
→ Verilen URL icin LCP analizi, PPTX + Excel uret

"example.com tam pagespeed audit"
→ 6 sub-skill sirasiyla calistir, kapsamli PPTX + Excel uret

"onceki raporla karsilastir: example.com CLS"
→ Onceki dosyayi yukle, yeni analiz yap, before/after sunum uret
```

### Sub-skill bazli kullanim

Her sub-skill bagimsiz olarak cagrilabilir:

| Komut ornekleri | Calisan sub-skill |
|---|---|
| "render pipeline analiz et", "TTFB bak", "sunucu hizi" | Render Pipeline |
| "javascript audit yap", "3rd party scriptler", "JS yuku" | JavaScript Yuku |
| "LCP bak", "gorsel optimizasyonu", "sayfa hizi" | LCP Optimizasyonu |
| "CLS analiz et", "layout shift", "sayfa kayiyor" | CLS Kararliligi |
| "INP bak", "etkilesim hizi", "DOM boyutu" | INP Etkilesim |
| "accessibility kontrol et", "erisilebilirlik", "alt text" | Kalite ve Erisilebilirlik |

### Sayfa tipi algilama

Sadece domain verirseniz otomatik algila:
- **E-ticaret siteleri**: anasayfa + urun sayfasi + kategori sayfasi
- **Kurumsal siteler**: anasayfa + hizmet sayfasi + icerik sayfasi
- Maksimum 3 sayfa tipi analiz edilir
- Spesifik URL verirseniz otomatik algilama yapilmaz

### Dil destegi

- Varsayilan: Turkce
- Ingilizce veya Almanca istenebilir: "vitra.de icin LCP analizi yap, Almanca"
- Dil tum ciktilari etkiler (sunum, Excel, cozum onerileri)

---

## Cikti formatlari

### PPTX sunum (ana cikti)

IT ekiplerine sunulabilecek profesyonel danismanlik raporu. Her bulgu icin 3 asamali yaklasim:

1. **Tespit slayti**: Ne tespit edildi, hangi element etkileniyor, gercek HTML referansi, screenshot kaniti
2. **Etki slayti**: Neden onemli, hangi metrikleri etkiliyor, kullanici deneyimi etkisi, Google referansi
3. **Cozum onerisi slayti**: Onerilen duzeltme adimlari, mevcut vs onerilen kod karsilastirmasi, efor ve oncelik

Sunum akisi:
```
Kapak → Genel degerlendirme → Metrik aciklama →
  Her sayfa tipi x cihaz icin:
    Bolum ayiraci → Skor karti → Bulgu slaytlari →
Ozet aksiyon tablosu → Oncelik siralamasi → Kapatis
```

PSI API'den gelen filmstrip ve screenshot gorselleri sunuma gomulur.

### Excel rapor (destekleyici)

IT ekibinin sprint'e aktarabilecegi detayli teknik rapor:

- **Ozet sheet**: Tum sayfa tiplerinin tum metrik skorlari
- **Sub-skill sheet'leri**: Her bulgu icin satir - element selector, mevcut deger, onerilen deger, etkilenen metrikler, oncelik, efor, sorumluluk, durum (IT ekibi dolduracak)
- **Ham veri sheet**: PSI API'den gelen tum audit skorlari

Sunumdaki slaytlardan Excel'e referans verilir: "Detayli veri icin: dosya.xlsx, Sheet adi, satir X-Y"

### Before/After karsilastirma

Onceki audit sonuclariyla karsilastirma yapilabilir:
1. Onceki Excel veya PPTX dosyasini ayri dizine yukleyin
2. Skill yeni analiz yapar ve onceki sonuclarla karsilastirir
3. Iyilesen metrikler yesil, kotulesen metrikler kirmizi olarak raporlanir

---

## Neye bakar?

### Sub-skill 1: Render Pipeline
Sunucu yanitini ve tarayicinin ilk render'ini geciktiren sorunlar.
- TTFB (sunucu yanit suresi)
- Render-blocking CSS ve JavaScript
- Kritik istek zinciri derinligi
- Preconnect/dns-prefetch/preload eksiklikleri
- Cache TTL yetersizlikleri
- HTTP/2 veya HTTP/3 kullanimi
- Font yukleme stratejisi (font-display)
- Text compression (Brotli/gzip)
- Sayfa agirligi breakdown (JS/CSS/font/gorsel)

### Sub-skill 2: JavaScript Yuku
Main thread'i mesgul eden JS sorunlari.
- 3rd party script envanteri (domain bazli boyut ve etki)
- Kullanilmayan JavaScript (dead code)
- Tekrarlanan JavaScript modulleri
- Eski polyfill'ler (legacy JavaScript)
- Script defer/async/module stratejisi kontrolu
- JS execution time ve main thread breakdown
- Long task tespiti (50ms+)
- CSR vs SSR etki degerlendirmesi

### Sub-skill 3: LCP Optimizasyonu
Largest Contentful Paint metrigine odakli gorsel ve element optimizasyonu.
- LCP element tespiti ve faz breakdown (TTFB → load delay → load → render)
- fetchpriority attribute kontrolu
- Lazy loading stratejisi (above-fold'da lazy olmamali)
- Preload kullanimi
- Gorsel format analizi (WebP/AVIF vs JPEG/PNG)
- Intrinsic vs display boyut uyumsuzlugu (boyut waste %)
- srcset/sizes attribute kontrolu
- Below-fold lazy loading eksikligi

### Sub-skill 4: CLS Kararliligi
Cumulative Layout Shift - sayfadaki beklenmedik kaymalari tespit eder.
- Layout shift eden elementlerin tespiti ve kayma buyuklugu
- Boyutsuz gorseller (width/height attribute eksik)
- aspect-ratio CSS kontrolu
- Ad/embed slot alan rezervasyonu (min-height)
- Font degisimi kaymasi (FOUT)
- Dinamik inject edilen icerik
- Non-composited animasyonlar
- Filmstrip ile shift gorselestirmesi

### Sub-skill 5: INP Etkilesim
Interaction to Next Paint - sayfa etkilesim hizi.
- INP faz breakdown (input delay + processing + presentation delay)
- DOM element sayisi ve nesting derinligi
- Forced reflow tespiti (DOM okuma-yazma interleaving)
- CSS selector karmasikligi
- Kullanilmayan CSS
- content-visibility: auto firsatlari

### Sub-skill 6: Kalite ve Erisilebilirlik
Accessibility + Best Practices birlestik audit.
- Gorsel alt attribute kontrolu (eksik, bos, dekoratif)
- Renk kontrast orani (WCAG AA)
- Heading hiyerarsisi (h1-h6 sirasi, atlama)
- ARIA rolleri ve attribute gecerliligi
- Form label baglantilari
- Button/link erisilebilir isimleri
- HTML lang attribute
- HTTPS kullanimi
- Console hatalari
- Deprecated API kullanimi
- Guvenlik acigi olan kutuphaneler

---

## Nasil calisir?

### Veri toplama mimarisi

Skill 3 katmanli veri toplama yapar, ortama gore en zengini kullanir:

```
Katman 1: PSI API veya DataForSEO Lighthouse
  → Lighthouse audit skorlari, metrik degerleri
  → Field data (CrUX - gercek kullanici verisi)
  → Screenshot ve filmstrip (base64 JPEG)

Katman 2: Chrome DevTools (claude.ai + Chrome extension varsa)
  → Canli DOM tarama (javascript_tool)
  → Network waterfall (read_network_requests)
  → Screenshot/GIF kaydi
  → Accessibility tree (read_page)

Katman 3: web_fetch + HTML parse (Chrome yoksa)
  → Statik HTML attribute kontrolu
  → Script/link/img tag analizi
```

DataForSEO MCP bagliysa: on_page_lighthouse (PSI alternatifi), on_page_instant_pages
(custom_js ile sunucu tarafli DOM tarama), on_page_content_parsing (yapisal analiz).

### Analiz akisi

```
1. Ortam algila (Chrome var mi? DataForSEO var mi? PSI key var mi?)
2. Sayfa tipi algila (e-ticaret mi? kurumsal mi?)
3. Her sayfa tipi x mobil/desktop icin:
   a. PSI API / DataForSEO Lighthouse cagrisi
   b. HTML kaynak analizi (Chrome / DataForSEO custom_js / web_fetch)
   c. Field data (CrUX) kontrol
4. Bulgulari kategorize et (sub-skill bazli)
5. Her bulgu icin: tespit + etki analizi + cozum onerisi
6. PPTX sunum olustur (pptxgenjs)
7. Excel rapor olustur (openpyxl)
8. Dosyalari kullaniciya sun
```

### Field data onceliklendirme

Skill her zaman CrUX field datasina bakar (son 28 gunluk gercek kullanici verisi).
Field data yoksa (yetersiz trafik) bunu sunumda acikca belirtir ve lab datasini gosterir.

### Lighthouse versiyon uyumlulugu

Lighthouse 13 (Ekim 2025) ile gelen yeni insight audit yapisi kullanilir.
Audit ID'leri reference dosyalarinda merkezi tutulur. Lighthouse guncellendikce
reference dosyalari guncellenerek uyumluluk korunur.

---

## Dosya yapisi

```
pagespeed-auditor/
├── SKILL.md                              # Ana orkestrasyon (sub-skill yonlendirme, veri toplama, cikti formati)
├── README.md                             # Bu dosya
└── references/
    ├── render-pipeline.md                # Sub-skill 1: TTFB, render-blocking, cache, font
    ├── javascript-burden.md              # Sub-skill 2: 3rd party, unused JS, execution
    ├── lcp-optimization.md               # Sub-skill 3: LCP element, gorsel, fetchpriority
    ├── cls-stability.md                  # Sub-skill 4: layout shift, boyutsuz ogeler
    ├── inp-interactivity.md              # Sub-skill 5: INP faz, DOM, reflow
    ├── quality-accessibility.md          # Sub-skill 6: a11y + best practices
    └── presentation-design.md            # Sunum tasarim rehberi (renkler, fontlar, slayt tipleri)
```

Her reference dosyasi su bolumleri icerir:
- Lighthouse audit ID eslestirmesi
- PSI API'den cekilecek veriler
- Chrome DevTools kontrol scriptleri
- DataForSEO kontrolleri
- web_fetch kontrolleri (fallback)
- Bulgu kaliplari ve cozum onerileri (Google referansli)
- Sunum slayt sablonu
- Excel sheet yapisi

---

## Lisans

Bu skill Inbound Marketing Agency SEO ekibi icin gelistirilmistir.
