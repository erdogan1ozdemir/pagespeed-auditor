# Sub-skill 6: Kalite ve Erisilebilirlik

Accessibility + Best Practices birlestik. Image alt, renk kontrast, heading hiyerarsisi,
ARIA, form label, link name, HTTPS, console hatalari, deprecated API'ler.

## Lighthouse audit eslestirme

### Accessibility audit'leri
PSI API'den `category=accessibility` ile cekilir.

| Audit ID | Kontrol alani |
|---|---|
| image-alt | Gorsellerde alt attribute |
| color-contrast | Metin-arka plan kontrast orani |
| heading-order | Heading hiyerarsisi (h1→h2→h3 atlama) |
| html-has-lang | html elementinde lang attribute |
| html-lang-valid | lang degerinin gecerli olmasi |
| aria-allowed-attr | ARIA attributelerinin gecerli kullanimi |
| aria-required-attr | Zorunlu ARIA attributeleri |
| aria-roles | ARIA rollerinin gecerliligi |
| button-name | Butonlarda erisilebilir isim |
| link-name | Linklerde erisilebilir isim |
| label | Form alanlarinda label |
| document-title | Sayfa basliginin varligi |
| duplicate-id | Tekrarlanan ID'ler |
| tabindex | Tab sirasi kontrolu |
| bypass | Skip navigation linki |
| meta-viewport | Viewport zoom engelleme kontrolu |

### Best Practices audit'leri
PSI API'den `category=best-practices` ile cekilir.

| Audit ID | Kontrol alani |
|---|---|
| is-on-https | HTTPS kullanimi |
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
audits['image-alt']                    → Alt eksik gorseller
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

**Heading hiyerarsisi kontrolu:**
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
// Basit kontrast kontrolu (tam WCAG hesaplamasi icin axe-core gerekir)
const textEls = document.querySelectorAll('p, span, a, h1, h2, h3, h4, h5, h6, li, td, th, label, button');
const lowContrast = Array.from(textEls).slice(0, 50).filter(el => {
  const style = getComputedStyle(el);
  const color = style.color;
  const bg = style.backgroundColor;
  // RGB degerlerini cikart ve basit luminance kontrolu yap
  return false; // Tam implementasyon axe-core gerektirir
}).length;
```

**Accessibility tree:**
```javascript
// read_page tool'u ile tarayicinin gercek accessibility tree'si alinabilir
// Bu, Lighthouse'un bakamadigindan cok daha detayli bilgi verir
```

**Console hatalari:**
```javascript
// Not: console.error'lari yakalamak icin onceden listener eklemek gerekir
// PSI API'nin errors-in-console audit'i bunu zaten yapar
```

## DataForSEO kontrolleri (Chrome yoksa, API varsa)

on_page_instant_pages ile custom_js: alt attribute taramasi, heading hiyerarsisi,
form label kontrolu, ARIA attribute kontrolu, lang attribute.
on_page_content_parsing ile heading yapisi ve link yapisi.

## web_fetch kontrolleri (Chrome ve DataForSEO yoksa)

HTML'den: tum img'lerde alt attribute, html lang attribute, heading yapisi (h1-h6 sirasi),
form label baglantilari, button/link icerik kontrolu, meta viewport,
duplicate ID tespiti, HTTPS kontrolu (URL'den).

---

## Bulgu kaliplari

### Alt attribute eksik
**Cozumler:** Icerikli gorsellere aciklayici alt metni ekle.
Dekoratif gorsellere `alt=""` ve `role="presentation"` ekle.
**Ref:** https://web.dev/articles/image-alt

### Renk kontrast yetersizligi
**Cozumler:** WCAG AA: normal metin 4.5:1, buyuk metin 3:1 minimum kontrast.
Renk paleti guncelle veya font boyutunu artir.
**Ref:** https://web.dev/articles/color-contrast

### Heading hiyerarsisi bozuk
**Cozumler:** h1→h2→h3 sirasini koru, seviye atlama. Her sayfada tek h1.
**Ref:** https://web.dev/articles/heading-order

### ARIA hatalari
**Cozumler:** Gereksiz ARIA kaldir (native HTML yeterli oldugunda).
ARIA rolleri ve attributeleri dogru eslestir.
**Ref:** https://web.dev/articles/aria

### Form label eksik
**Cozumler:** Her input'a iliskili label ekle (for/id eslesmesi veya wrapping).
Placeholder, label'in yerini almaz.
**Ref:** https://web.dev/articles/label

### HTTPS eksik
**Cozumler:** SSL sertifikasi ekle, HTTP→HTTPS redirect. Mixed content temizle.
**Ref:** https://web.dev/articles/is-on-https

### Console hatalari
**Cozumler:** Her hatayi tespit et ve ilgili kodu duzelt. 3rd party kaynakli hatalari
raporla. 404 kaynak hatalari icin URL'leri duzelt.

---

## Sunum slayt sablonu

### Skor karti
- Accessibility skoru + Best Practices skoru (field data yoksa lab)
- Gecen/kalan audit sayisi
- Kritik vs uyari dagilimi

### Gorsel erisilebilirlik slayti
- Alt eksik gorsel sayisi ve ornekler
- Screenshot ile isaretli ornekler (varsa)

### Yapisal erisilebilirlik slayti
- Heading hiyerarsisi agaci (h1→h2→h3 gorunum)
- Atlanan seviyeler vurgulu

### ARIA ve form slayti
- Basarisiz ARIA audit listesi
- Form label eksiklikleri

### Best practices slayti
- HTTPS durumu, console hatalari, deprecated API'ler
- Vulnerable library listesi (varsa)

### Bulgu slaytlari (tespit → etki → cozum) kritik bulgular icin

---

## Excel sheet yapisi

Sheet adi: "Kalite ve Erisilebilirlik"
Sutunlar: Sayfa Tipi | Cihaz | Kategori (A11y/BP) | Bulgu | Audit ID |
Element (selector) | Mevcut Durum | Onerilen Duzeltme | WCAG Kriteri |
Etki Seviyesi (Kritik/Ciddi/Orta/Dusuk) | Oncelik | Efor | Sorumluluk | Durum
