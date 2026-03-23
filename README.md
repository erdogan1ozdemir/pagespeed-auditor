# PageSpeed Auditor

Inbound Marketing Agency SEO ekibi için gelistirilmis bir Claude skill'idir. Bir veya birden fazla URL için PageSpeed Insights analizi yaparak IT ekiplerine yonelik PPTX sunum ve destekleyici Excel rapor üretir.

---

## Kurulum

### Claude.ai'da

1. Bu repository'deki `pagespeed-auditor.skill` dosyasıni indirin
2. Claude.ai'da Settings > Skills bölümune gidin
3. "Add Skill" ile .skill dosyasıni yukleyin

### Claude Code'da

```bash
# Repository'yi klonlayin
git clone https://github.com/[username]/pagespeed-auditor.git

# Skill dosyasıni Claude Code'a ekleyin
# Claude Code'un skill dizinine kopyalayin
```

### Gereksinimler

Skill çalıştırildiginda gereken paketleri otomatik yukler:
- **pptxgenjs** - sunum oluşturma
- **openpyxl** - Excel oluşturma
- **markitdown** - önceki rapor okuma (before/after karşılaştırma)

Ek olarak asagidakilerden en az biri gerekir:
- **PSI API key** (ucretsiz): https://developers.google.com/speed/docs/insights/v5/get-started
- **DataForSEO MCP** baglantisi (PSI key'e alternatif, zaten hesabiniz varsa tercih edin)

---

## Kullanim

### Temel kullanim ornekleri

```
"vitra.com.tr için CLS analizi yap"
→ Otomatik 3 sayfa tipi algıla, CLS sub-skill'i çalıştır, PPTX + Excel üret

"sadece LCP bak: https://www.example.com/urun/123"
→ Verilen URL için LCP analizi, PPTX + Excel üret

"example.com tam pagespeed audit"
→ 6 sub-skill sırasıyla çalıştır, kapsamli PPTX + Excel üret

"önceki raporla karsilastir: example.com CLS"
→ Önceki dosyayi yukle, yeni analiz yap, before/after sunum üret
```

### Sub-skill bazli kullanim

Her sub-skill bağımsız olarak cagrilabilir:

| Komut ornekleri | Calisan sub-skill |
|---|---|
| "render pipeline analiz et", "TTFB bak", "sunucu hizi" | Render Pipeline |
| "javascript audit yap", "3rd party scriptler", "JS yuku" | JavaScript Yuku |
| "LCP bak", "görsel optimizasyonu", "sayfa hizi" | LCP Optimizasyonu |
| "CLS analiz et", "layout shift", "sayfa kayiyor" | CLS Kararlılığı |
| "INP bak", "etkileşim hizi", "DOM boyutu" | INP Etkileşim |
| "accessibility kontrol et", "erişilebilirlik", "alt text" | Kalite ve Erişilebilirlik |

### Sayfa tipi algılama

Sadece domain verirseniz otomatik algıla:
- **E-ticaret siteleri**: anasayfa + ürün sayfası + kategori sayfası
- **Kurumsal siteler**: anasayfa + hizmet sayfası + içerik sayfası
- Maksimum 3 sayfa tipi analiz edilir
- Spesifik URL verirseniz otomatik algılama yapılmaz

### Dil destegi

- Varsayılan: Turkce
- Ingilizce veya Almanca istenebilir: "vitra.de için LCP analizi yap, Almanca"
- Dil tum çıktılari etkiler (sunum, Excel, çözüm onerileri)

---

## Çıktı formatlari

### PPTX sunum (ana çıktı)

IT ekiplerine sunulabilecek profesyonel danismanlik raporu. Her bulgu için 3 asamali yaklaşım:

1. **Tespit slayti**: Ne tespit edildi, hangi element etkileniyor, gerçek HTML referansi, screenshot kaniti
2. **Etki slayti**: Neden önemli, hangi metrikleri etkiliyor, kullanici deneyimi etkisi, Google referansi
3. **Çözüm önerisi slayti**: Önerilen düzeltme adimlari, mevcut vs önerilen kod karşılaştırmasi, efor ve öncelik

Sunum akisi:
```
Kapak → Genel değerlendirme → Metrik açıklama →
  Her sayfa tipi x cihaz icin:
    Bölüm ayiraci → Skor karti → Bulgu slaytlari →
Özet aksiyon tablosu → Öncelik sıralaması → Kapatis
```

PSI API'den gelen filmstrip ve screenshot görselleri sunuma gomulur.

### Excel rapor (destekleyici)

IT ekibinin sprint'e aktarabilecegi detayli teknik rapor:

- **Özet sheet**: Tum sayfa tiplerinin tum metrik skorlari
- **Sub-skill sheet'leri**: Her bulgu için satir - element selector, mevcut değer, önerilen değer, etkilenen metrikler, öncelik, efor, sorumluluk, durum (IT ekibi dolduracak)
- **Ham veri sheet**: PSI API'den gelen tum audit skorlari

Sunumdaki slaytlardan Excel'e referans verilir: "Detayli veri icin: dosya.xlsx, Sheet adi, satir X-Y"

### Before/After karşılaştırma

Önceki audit sonuclariyla karşılaştırma yapilabilir:
1. Önceki Excel veya PPTX dosyasıni ayri dizine yukleyin
2. Skill yeni analiz yapar ve önceki sonuclarla karsilastirir
3. Iyilesen metrikler yesil, kotulesen metrikler kirmizi olarak raporlanir

---

## Neye bakar?

### Sub-skill 1: Render Pipeline
Sunucu yanitini ve tarayıcınin ilk render'ini geçıktıren sorunlar.
- TTFB (sunucu yanit süresi)
- Render-blocking CSS ve JavaScript
- Kritik istek zinciri derinligi
- Preconnect/dns-prefetch/preload eksiklikleri
- Cache TTL yetersizlikleri
- HTTP/2 veya HTTP/3 kullanımı
- Font yükleme stratejisi (font-display)
- Text compression (Brotli/gzip)
- Sayfa ağırlığı breakdown (JS/CSS/font/görsel)

### Sub-skill 2: JavaScript Yuku
Main thread'i mesgul eden JS sorunlari.
- 3rd party script envanteri (domain bazli boyut ve etki)
- Kullanılmayan JavaScript (dead code)
- Tekrarlanan JavaScript modulleri
- Eski polyfill'ler (legacy JavaScript)
- Script defer/async/module stratejisi kontrolu
- JS execution time ve main thread breakdown
- Long task tespiti (50ms+)
- CSR vs SSR etki değerlendirmesi

### Sub-skill 3: LCP Optimizasyonu
Largest Contentful Paint metrigine odakli görsel ve element optimizasyonu.
- LCP element tespiti ve faz breakdown (TTFB → load delay → load → render)
- fetchpriority attribute kontrolu
- Lazy loading stratejisi (above-fold'da lazy olmamali)
- Preload kullanımı
- Görsel format analizi (WebP/AVIF vs JPEG/PNG)
- Intrinsic vs display boyut uyumsuzlugu (boyut waste %)
- srcset/sizes attribute kontrolu
- Below-fold lazy loading eksikligi

### Sub-skill 4: CLS Kararlılığı
Cumulative Layout Shift - sayfadaki beklenmedik kaymalari tespit eder.
- Layout shift eden elementlerin tespiti ve kayma buyuklugu
- Boyutsuz görseller (width/height attribute eksik)
- aspect-ratio CSS kontrolu
- Ad/embed slot alan rezervasyonu (min-height)
- Font degisimi kaymasi (FOUT)
- Dinamik inject edilen içerik
- Non-composited animasyonlar
- Filmstrip ile shift görselestirmesi

### Sub-skill 5: INP Etkileşim
Interaction to Next Paint - sayfa etkileşim hizi.
- INP faz breakdown (input delay + processing + presentation delay)
- DOM element sayisi ve nesting derinligi
- Forced reflow tespiti (DOM okuma-yazma interleaving)
- CSS selector karmaşıklığı
- Kullanılmayan CSS
- content-visibility: auto firsatlari

### Sub-skill 6: Kalite ve Erişilebilirlik
Accessibility + Best Practices birlestik audit.
- Görsel alt attribute kontrolu (eksik, bos, dekoratif)
- Renk kontrast orani (WCAG AA)
- Heading hiyerarşisi (h1-h6 sirasi, atlama)
- ARIA rolleri ve attribute gecerliligi
- Form label baglantilari
- Button/link erişilebilir isimleri
- HTML lang attribute
- HTTPS kullanımı
- Console hatalari
- Deprecated API kullanımı
- Guvenlik acigi olan kutuphaneler

---

## Nasil çalışır?

### Veri toplama mimarisi

Skill 3 katmanli veri toplama yapar, ortama gore en zengini kullanir:

```
Katman 1: PSI API veya DataForSEO Lighthouse
  → Lighthouse audit skorlari, metrik değerleri
  → Field data (CrUX - gerçek kullanici verisi)
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
1. Ortam algıla (Chrome var mi? DataForSEO var mi? PSI key var mi?)
2. Sayfa tipi algıla (e-ticaret mi? kurumsal mi?)
3. Her sayfa tipi x mobil/desktop icin:
   a. PSI API / DataForSEO Lighthouse cagrisi
   b. HTML kaynak analizi (Chrome / DataForSEO custom_js / web_fetch)
   c. Field data (CrUX) kontrol
4. Bulgulari kategorize et (sub-skill bazli)
5. Her bulgu icin: tespit + etki analizi + çözüm önerisi
6. PPTX sunum oluştur (pptxgenjs)
7. Excel rapor oluştur (openpyxl)
8. Dosyalari kullaniciya sun
```

### Field data önceliklendirme

Skill her zaman CrUX field datasina bakar (son 28 gunluk gerçek kullanici verisi).
Field data yoksa (yetersiz trafik) bunu sunumda açıkça belirtir ve lab datasini gösterir.

### Lighthouse versiyon uyumlulugu

Lighthouse 13 (Ekim 2025) ile gelen yeni insight audit yapısı kullanilir.
Audit ID'leri reference dosyalarında merkezi tutulur. Lighthouse guncellendikce
reference dosyaları guncellenerek uyumluluk korunur.

---

## Dosya yapisi

```
pagespeed-auditor/
├── SKILL.md                              # Ana orkestrasyon (sub-skill yönlendirme, veri toplama, çıktı formati)
├── README.md                             # Bu dosya
└── references/
    ├── render-pipeline.md                # Sub-skill 1: TTFB, render-blocking, cache, font
    ├── javascript-burden.md              # Sub-skill 2: 3rd party, unused JS, execution
    ├── lcp-optimization.md               # Sub-skill 3: LCP element, görsel, fetchpriority
    ├── cls-stability.md                  # Sub-skill 4: layout shift, boyutsuz öğeler
    ├── inp-interactivity.md              # Sub-skill 5: INP faz, DOM, reflow
    ├── quality-accessibility.md          # Sub-skill 6: a11y + best practices
    └── presentation-design.md            # Sunum tasarim rehberi (renkler, fontlar, slayt tipleri)
```

Her reference dosyası su bölümleri içerir:
- Lighthouse audit ID eşleştirmesi
- PSI API'den cekilecek veriler
- Chrome DevTools kontrol scriptleri
- DataForSEO kontrolleri
- web_fetch kontrolleri (fallback)
- Bulgu kaliplari ve çözüm onerileri (Google referansli)
- Sunum slayt sablonu
- Excel sheet yapisi

---

## Lisans

Bu skill Inbound Marketing Agency SEO ekibi için gelistirilmistir.
