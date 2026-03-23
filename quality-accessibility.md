# Sub-skill 6: Kalite ve Erişilebilirlik

Accessibility + Best Practices birlestik. Image alt, renk kontrast, heading hiyerarşisi,
ARIA, form label, link name, HTTPS, console hatalari, deprecated API'ler.

## Lighthouse audit eşleştirme

### Accessibility audit'leri
PSI API'den `category=accessibility` ile cekilir.

| Audit ID | Kontrol alani |
|---|---|
| image-alt | Görsellerde alt attribute |
| color-contrast | Metin-arka plan kontrast orani |
| heading-order | Heading hiyerarşisi (h1→h2→h3 atlama) |
| html-has-lang | html elementinde lang attribute |
| html-lang-valid | lang değerinin gecerli olmasi |
| aria-allowed-attr | ARIA attributelerinin gecerli kullanımı |
| aria-required-attr | Zorunlu ARIA attributeleri |
| aria-roles | ARIA rollerinin gecerliligi |
| button-name | Butonlarda erişilebilir isim |
| link-name | Linklerde erişilebilir isim |
| label | Form alanlarinda label |
| document-title | Sayfa başlığınin varligi |
| duplicate-id | Tekrarlanan ID'ler |
| tabindex | Tab sirasi kontrolu |
| bypass | Skip navigation linki |
| meta-viewport | Viewport zoom engelleme kontrolu |

### Best Practices audit'leri
PSI API'den `category=best-practices` ile cekilir.

| Audit ID | Kontrol alani |
|---|---|
| is-on-https | HTTPS kullanımı |
| no-vulnerable-libraries | Guvenlik acigi olan kutuphaneler |
| errors-in-console | Console hatalari |
| deprecations | Kullanim disi API'ler |
| csp-xss | Content Security Policy |
| inspector-issues | DevTools uyarilari |
| valid-source-maps | Source map gecerliligi |

## PSI API'den cekilecek veriler

```
categories['accessibility'].score      → Genel accessibility skoru
categories['best-practices'].score     → Genel best practices skoru
audits['image-alt']                    → Alt eksik görseller
audits['color-contrast']               → Kontrast sorunu olan elementler
audits['heading-order']                → Heading yapisi
audits['html-has-lang']
audits['aria-allowed-attr']
audits['button-name']
audits['link-name']
audits['label']
audits['is-on-https']
audits['errors-in-console']
audits['no-vulnerable-libraries']
```

## Chrome DevTools kontrolleri

**Alt attribute taramasi:**
```javascript
const imgs = Array.from(document.querySelectorAll('img'));
const altAudit = {
  total: imgs.length,
  missingAlt: imgs.filter(i => !i.hasAttribute('alt')).length,
  emptyAlt: imgs.filter(i => i.hasAttribute('alt') && i.alt === '').length,
  decorative: imgs.filter(i => i.alt === '' && i.getAttribute('role') === 'presentation').length,
  withAlt: imgs.filter(i => i.alt && i.alt.length > 0).length,
  details: imgs.filter(i => !i.alt).map(i => ({
    src: i.src.substring(0, 80),
    aboveFold: i.getBoundingClientRect().top < window.innerHeight,
    size: Math.round(i.getBoundingClientRect().width) + 'x' +
      Math.round(i.getBoundingClientRect().height)
  }))
};
```

**Heading hiyerarşisi kontrolu:**
```javascript
const headings = Array.from(document.querySelectorAll('h1,h2,h3,h4,h5,h6'));
const structure = headings.map((h, i) => ({
  level: parseInt(h.tagName[1]),
  text: h.textContent.trim().substring(0, 60),
  skipped: i > 0 ? parseInt(h.tagName[1]) - parseInt(headings[i-1].tagName[1]) > 1 : false
}));
const h1Count = headings.filter(h => h.tagName === 'H1').length;
const hasSkips = structure.some(s => s.skipped);
```

**Renk kontrast kontrolu:**
```javascript
// Basit kontrast kontrolu (tam WCAG hesaplamasi için axe-core gerekir)
const textEls = document.querySelectorAll('p, span, a, h1, h2, h3, h4, h5, h6, li, td, th, label, button');
const lowContrast = Array.from(textEls).slice(0, 50).filter(el => {
  const style = getComputedStyle(el);
  const color = style.color;
  const bg = style.backgroundColor;
  // RGB değerlerini cikart ve basit luminance kontrolu yap
  return false; // Tam implementasyon axe-core gerektirir
}).length;
```

**Accessibility tree:**
```javascript
// read_page tool'u ile tarayıcınin gerçek accessibility tree'si alinabilir
// Bu, Lighthouse'un bakamadigindan cok daha detayli bilgi verir
```

**Console hatalari:**
```javascript
// Not: console.error'lari yakalamak için önceden listener eklemek gerekir
// PSI API'nin errors-in-console audit'i bunu zaten yapar
```

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: alt attribute taramasi, heading hiyerarşisi,
form label kontrolu, ARIA attribute kontrolu, lang attribute.
on_page_content_parsing ile heading yapısı ve link yapisi.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: tum img'lerde alt attribute, html lang attribute, heading yapısı (h1-h6 sirasi),
form label baglantilari, button/link içerik kontrolu, meta viewport,
duplicate ID tespiti, HTTPS kontrolu (URL'den).

---

## Bulgu kaliplari

### Alt attribute eksik
**Öneriler:** İçerikli görsellere aciklayici ve anlamli alt metni eklenmesi önerilmektedir.
Dekoratif görsellere `alt=""` ve `role="presentation"` tanımlanmasi, ekran okuyucularin
bu görselleri atlamasini sağlayacaktır.
**Ref:** https://web.dev/articles/image-alt

### Renk kontrast yetersizligi
**Öneriler:** WCAG AA standartlarina gore normal metin için 4.5:1, buyuk metin icin
3:1 minimum kontrast oraninin saglanmasi gerekmektedir. Renk paletinin guncellenmesi
veya font boyutunun arttirilmasi değerlendirilmelidir.
**Ref:** https://web.dev/articles/color-contrast

### Heading hiyerarşisi bozuk
**Öneriler:** h1, h2, h3 sirasinin korunmasi ve seviye atlanmamasi önerilmektedir.
Her sayfada tek bir h1 elementi bulunmasi, hem erişilebilirlik hem de SEO acisindan
önemlidir.
**Ref:** https://web.dev/articles/heading-order

### ARIA hatalari
**Öneriler:** Native HTML'in yeterli oldugu durumlarda gereksiz ARIA attribute'larinin
kaldırılması tavsiye edilmektedir. Kullanilan ARIA rolleri ve attribute'larinin
spesifikasyona uygun eşleştirmesinin kontrolu önerilmektedir.
**Ref:** https://web.dev/articles/aria

### Form label eksik
**Öneriler:** Her form alanina iliskili label elementinin eklenmesi (for/id eslesmesi
veya label icine yerlestirme) önerilmektedir. Placeholder attribute'unun label'in
yerini almadigi unutulmamalidir.
**Ref:** https://web.dev/articles/label

### HTTPS eksik
**Öneriler:** SSL sertifikasinin eklenmesi ve HTTP'den HTTPS'e yönlendirme yapilmasi
önerilmektedir. Mixed content (HTTP uzerinden yüklenen kaynaklar) temizlenmesi
gerekmektedir.
**Ref:** https://web.dev/articles/is-on-https

### Console hatalari
**Öneriler:** Tespit edilen console hatalarinin incelenmesi ve ilgili kodun düzeltilmesi
tavsiye edilmektedir. Ucuncu parti kaynakli hatalarin ilgili servis saglayicilara
raporlanmasi, 404 donduren kaynak URL'lerinin guncellenmesi önerilmektedir.

---

## Sunum slayt sablonu

### Skor karti
- Accessibility skoru + Best Practices skoru (field data yoksa lab)
- Gecen/kalan audit sayisi
- Kritik vs uyari dagilimi

### Görsel erişilebilirlik slayti
- Alt eksik görsel sayisi ve ornekler
- Screenshot ile isaretli ornekler (varsa)

### Yapisal erişilebilirlik slayti
- Heading hiyerarşisi agaci (h1→h2→h3 gorunum)
- Atlanan seviyeler vurgulu

### ARIA ve form slayti
- Başarısız ARIA audit listesi
- Form label eksiklikleri

### Best practices slayti
- HTTPS durumu, console hatalari, deprecated API'ler
- Vulnerable library listesi (varsa)

### Bulgu slaytlari (tespit → etki → çözüm) kritik bulgular icin

---

## Excel sheet yapisi

Sheet adi: "Kalite ve Erişilebilirlik"
Sutunlar: Sayfa Tipi | Cihaz | Kategori (A11y/BP) | Bulgu | Audit ID |
Element (selector) | Mevcut Durum | Önerilen Düzeltme | WCAG Kriteri |
Etki Seviyesi (Kritik/Ciddi/Orta/Düşük) | Öncelik | Efor | Sorumluluk | Durum
