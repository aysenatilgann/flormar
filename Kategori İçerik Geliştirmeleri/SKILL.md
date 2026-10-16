---
name: flormar-kategori-icerik-guncelleme
description: Flormar (flormar.com.tr) kategori sayfası (PLP) SEO içeriklerini AI ile günceller - mevcut yayındaki metni çeker, Ahrefs (ve varsa DataForSEO) ile keyword / SERP / PAA / kannibalizasyon analizi yapar, marka rehberine ve Eylül 2026 revize notlarından çıkan dikkat listesine göre "Mevcut İçerik" + "Yeni İçerik" bölümlü doküman üretir, SSS'leri geliştirir, iç linkleri günceller ve teslim öncesi otomatik kalite kontrol çalıştırır. "Flormar kategori içerik güncelle", "LP update", "kategori içeriği AI ile güncelle", "Flormar X sayfasının içeriğini yenile", "kategori metni + internal link güncelleme", "bu ayın LP update'i" gibi taleplerde mutlaka kullan. Flormar blog yazıları, meta title/description işleri ve başka markaların kategori içerikleri bu skill'in kapsamı değildir.
---

# Flormar Kategori İçerik Güncelleme

Amaç: Flormar kategori sayfalarının alt SEO metnini, markadan revize almayacak kalitede güncellemek.
Çıktı her sayfa için bir doküman (ajansın Google Doc şablonu): üstte **YENİ İÇERİK** (sayfanın tamamı, yayına hazır), altta **MEVCUT İÇERİK** (yayındaki hali, değiştirilmeden).
Çalışma prensibi: mevcut başlıklar korunup geliştirilir, **eklenebilecek başlık varsa eklenir**, mümkünse **SSS de geliştirilir** (yoksa eklenir).

Başlamadan önce iki referansı oku:
- `references/marka-rehberi.md` - ton, kanıt noktaları, Akdeniz teması, sürdürülebilirlik, yasaklar
- `references/dikkat-listesi.md` - markanın Eylül 2026 revizelerinden çıkan kontrol listesi

## Girdi
Kullanıcıdan alınır (eksikse sor, tek mesajda):
1. Kategori URL listesi (genelde ayda 5 sayfa) ve dönem adı (ör. "Ekim 2026").
2. Varsa bu sayfalar için önceden hazırlanmış Google Doc'lar (.docx olarak indirilmiş) ya da özel notlar. Doküman gelirse `scripts/docx_to_md.py` ile yeni/mevcut markdown'a böl. **Bu ara markdown dosyaları scratchpad'e yazılır, teslim klasörüne değil.**
3. Odak KW tercihi varsa (yoksa Adım 2'de veriden çıkarılır).

Takip dosyası: `~/Downloads/Flormar - LP Update .xlsx` → "İçeriği Düzenlenen URLler" sayfası. Daha önce güncellenen URL'ler ve "Tüm Kategori URLleri" (iç link havuzu) buradadır. Excel'deki marka notu: kusursuz / mükemmel / en iyi yok, ünlem az.

## Akış

### Adım 1 - Hedef teyidi, mevcut içerik ve ürün listesi
Her URL için (Playwright ya da WebFetch; site JS ağırlıklı ise Playwright):
- **Hedef teyidi:** sayfa 200 dönüyor mu, canonical kendini mi gösteriyor, noindex var mı, ana KW'de Google'da sıralanan Flormar URL'si bu sayfa mı? Canonical başka sayfayı gösteriyorsa, sayfa noindex ise ya da sıralanan URL farklıysa **yazmaya başlamadan kullanıcıya sorulur.**
- Sayfanın alt kısmındaki **SEO metin bloğunu** başlık hiyerarşisiyle birlikte birebir al (H2/H3, paragraf, liste, SSS, iç linkler ve anchor'ları). Bu metin "Mevcut İçerik" olur, **hiç düzeltilmeden** dokümana girer.
- **Ürün adları ürün sitemap'inden alınır; PLP sayfaları ve sitenin tamamı taranmaz** (token tasarrufu). Kaynak: `https://www.flormar.com.tr/sitemaps/sitemap-products-1.xml.gz` (uzantısı .gz ama düz XML; her URL bir renk).
  ```
  S=~/.claude/skills/flormar-kategori-icerik-guncelleme/scripts
  python3 $S/urun_sitemap.py ara "kaş kalemi"            # ilgili ürün grupları + renk sayısı (yalnız sitemap okunur)
  python3 $S/urun_sitemap.py stok GRUP1 GRUP2 GRUP3      # yalnız aday gruplar: resmi ad + stok
  ```
  `stok` ürün sayfasının JSON-LD'sinden resmi adı ve stok durumunu okur, stokta bir renk bulunca o ürünü bırakır. Yalnız metne girmesi düşünülen adaylar (en fazla 4-5 grup) ve mevcut metinde geçen ürünler sorgulanır.
- **Stok kuralı:** **tüm renkleri tükenmiş ürün Yeni İçerik'te anılmaz** (mevcut metinde geçse bile); sadece bazı renkleri tükenmişse kullanılabilir. Tükenmiş ürünleri teslim notunda raporla.
- Mevcut metinde geçip sitemap'te olmayan ürünleri not et (satıştan kalkmış olabilir; Yeni İçerik'te dolaylı atıflarıyla birlikte çıkarılır, kullanıcıya raporlanır).

### Adım 2 - Keyword ve kannibalizasyon (Ahrefs, TR)
Ahrefs tool'larını ilk kullanımdan önce `doc` ile kontrol et.
- `site-explorer-organic-keywords` (URL modu, country=tr): sayfanın sıralandığı KW'ler, pozisyon, hacim.
- `keywords-explorer-matching-terms` + `related-terms` + soru KW'leri (ör. "kaş kalemi nasıl", "… nedir", "… hangisi"): kapsanmayan alt konular ve SSS adayları.
- **Kannibalizasyon:** ana KW'ler için `site-explorer-organic-keywords` domain modunda flormar.com.tr'yi filtrele. Aynı KW'de 2+ Flormar URL'si varsa not et; Yeni İçerik o konuyu derinleştirmek yerine doğru sayfaya iç link verir.
- SeoMonitor'da Flormar kampanyası varsa takip edilen KW'lerin güncel sıralamasını da al (opsiyonel).
- **GSC (ücretsiz tamamlayıcı):** `gsc` MCP `search_analytics` ile sayfanın son 90 günlük sorguları (dimension: query, sayfa filtresi) ve ana KW'yi taşıyan sorgularda sayfa dağılımı (dimensions: query + page). Ahrefs ana kaynak olarak kalır; Ahrefs'e erişilemezse GSC + DataForSEO ile devam edilir ve bu teslim notunda yazılır.
- Çıktı: sayfa başına **1 ana KW + 4-8 ikincil KW + 5-10 soru KW** listesi, hacimleriyle.
- **Kelime sahipliği (bir kelime, bir sahip):** listedeki her kelime dört kovadan birine yazılır:
  - **HEDEF:** bu sayfanın kelimesi.
  - **SERBEST:** kendi sayfası olmayan uzun kuyruk; bu sayfada işlenir.
  - **BAŞKA SAYFA:** sahibi başka bir Flormar sayfası (kategori ya da blog); başlık / SSS / ayrı paragraf açılmaz, o sayfaya link verilir.
  - **KAPSAM DIŞI:** konu ya da ürün gamı dışı (parfüm, cilt bakımı vb.).
- **Kardeş kategoriler** aynı partideyse (ör. likit ruj + tint ruj, kaş kalemi + kaş farı) önce hepsinin kova tablosu çıkarılır, yazıma sonra geçilir.
- **Site düzeyi çakışma içerikle çözülmez** (ör. iki kategori sayfası aynı KW'de dönüşümlü sıralanıyor). Teslim notunda bildirilir.

### Adım 3 - SERP ve PAA
- DataForSEO (`dfs-mcp`, `api_request`): `POST /v3/serp/google/organic/live/advanced`, body `[{"keyword":"…","location_code":2792,"language_code":"tr","device":"desktop","depth":10,"people_also_ask_click_depth":1}]`. Live endpoint istek başına tek task kabul eder. Head KW'de PAA çıkmayabilir; soru KW'siyle ("… nasıl kullanılır") tekrar çek. Bağlı değilse: Ahrefs `serp-overview` (top 10 URL) + rakip sayfaların H2'leri WebFetch ile + PAA için Google TR'yi Playwright ile aç ya da WebSearch kullan.
- **Rakip içeriği en fazla 2 rakibin ilgili kategori sayfasından okunur** (token tasarrufu). SERP listesi ve PAA yeterli veriyi verir; içerik okuması için SERP'te üstte çıkan, kategori sayfası olan en alakalı 2 rakip seçilir. Bloglar ve 3. rakip okunmaz.
- Bu 2 rakibin işlediği ama Flormar sayfasında olmayan alt konuları çıkar (ör. "nasıl çıkarılır", "ne sıklıkla değiştirilir", "ten rengine göre seçim"). Rakip adı yalnız iç çalışmada geçer; içeriğe asla girmez.

### Adım 4 - Bölüm planı (kullanıcı onayı)
Her sayfa için kısa bir tablo hazırla ve **yazmaya başlamadan önce kullanıcıya göster**:

| Mevcut bölüm | Karar (koru / revize / birleştir / sil) | Gerekçe | Hedef KW |
|---|---|---|---|

Altında: **eklenecek yeni başlıklar** (hedef KW / soru ve hacmiyle; rakiplerde olup sayfada olmayan, soru KW'si ya da PAA'dan gelen konular), SSS soru listesi, iç link planı (anchor → URL), kelime kova tablosu (BAŞKA SAYFA ve KAPSAM DIŞI satırları yeterli), metne girecek **en fazla 3 ürün** (resmi ad + stok durumu), kannibalizasyon notu.
Başlık ekleme ölçütü: konunun aranma hacmi / PAA karşılığı var, sayfada başka bölümle örtüşmüyor, başka bir Flormar sayfasının ana konusu değil (değilse o sayfaya iç link verilir).
Bölüm sırası: H1 + giriş → Nedir → Çeşitleri / Nasıl seçilir (ten tonu, alt ton, bitiş tek bölümde) → Nasıl uygulanır → Nasıl çıkarılır / saklanır / değiştirilir → SSS → Kapanış paragrafı (her zaman en son, başlıksız).
5 sayfa birlikte geliyorsa planları tek mesajda topla, tek onay al.

### Adım 5 - Yeni İçerik'i yaz
**Yeni İçerik sayfanın tam halidir**, sadece eklenen kısımlar değil. Korunan eski paragraflar da aynı tona çekilerek yeniden yazılır. Revizelerin çoğu eski-yeni metin kopukluğundan geldi.

Yazım kuralları (ayrıntı referanslarda):
- **Sen dili, özgüvenli ve net.** Ne aşırı temkinli ("tercih edilebilir") ne buyurgan ("-malısın").
- **Emir kipi yok** ("kullan", "yap", "seç", "dene", "tercih et", "unutma", "keşfet"; "kullanın" gibi çoğul hali de). Kapanış/CTA ve SSS cevapları dahil. **Tek istisna:** adım adım anlatım gereken uygulama bölümleri (Nasıl uygulanır / sürülür / temizlenir, adım listeleri): "Öncelikle az miktarda göz farını göz farı fırçasına al." Bu bölümlerin dışında emir cümlesi, geniş zamanlı bilgi cümlesine çevrilir: "Mat ruj seç" → "Mat ruj gün boyu daha az tazeleme ister."
- **"-ebilir" ailesi seyrek:** "-ebilirsin / -abilirsin" ve edilgen "-ilebilir" ("tercih edilebilir", "kullanılabilir", "yardımcı olabilir") aynı hedge'in iki yüzüdür; birini azaltıp diğerine kaçılmaz. Sınırlar: "-bilirsin" sayfada ~350 kelimede 1 (1.400 kelimede en fazla 4) ve paragrafta en fazla 1; "-ilebilir" paragrafta en fazla 1; ikisinin toplamı ~200 kelimede 1. Yerine, cümleye göre:
  - geniş zamanlı net bilgi: "Açılı uç kaş hattını belirler."
  - edilgen geniş zaman: "Far, göz kapağının ortasından dışa doğru dağıtılır." / "Bu adımda ince uçlu fırça kullanılır."
  - koşul + sonuç: "Daha doğal bir görünüm istersen bal tonları uyum sağlar."
  Edilgeni de üst üste dizme; metin kuru ve kılavuz gibi okunmasın. Üç kalıp dönüşümlü kullanılır.
- **Aynı öneri bir kez:** bir tavsiye (ör. "kaş sabitleyici sür") hangi bölüme aitse orada bir kez geçer; başka bölümde ve SSS'te tekrarlanmaz, gerekirse o bölüme atıf yapılır. Aynı kalıp ifade ("gün boyu korur") 3 bölümde geçmez.
- **Kopuk başlık açma:** tek-iki cümlelik içerik için ayrı H3 açılmaz; komşu başlığa birleştirilir (ör. "Kalıcılık" → "Uç Yapısı, Doku ve Kalıcılık").
- **Marka kişiliği:** neşeli, moral veren, yaratıcı, bilinçli (abartı yerine somut bilgi), samimi. Asla abartılı, çocuksu, aşırı süslü, snob.
- Kusursuz / mükemmel / en iyi / ucuz yok. Ünlem en fazla 1.
- Her bilgi sayfada bir kez. Öneriler birbiriyle çelişmez. Terimler tek biçimde.
- **Ürün adları:** sadece Adım 1'de `urun_sitemap.py stok` ile teyit edilen, stokta rengi olan ürünler. **Kategori içeriği başına en fazla 3 ürün adı**, metinde **kalın** (`**Angled Brow Fırça Başlıklı & Asansörlü Ucu Açılı Kaş Kalemi**`) yazılır ve **link verilmez**; ürün linklerini ekip manuel ekler. Ürüne özel iddia yalnızca ürün sayfasında varsa. Kalın biçim yalnız ürün adlarında kullanılır.
- **Giriş paragrafı:** ilk cümle ana KW ile kategoriyi tanımlar; ardından sitedeki gam (alt türler, bitişler) kısaca özetlenir.
- **Net fiyat, fiyat aralığı, indirim oranı, kampanya adı, yıl, "bu sezon" ve ürün sayısı yazılmaz.** "Uygun fiyat", "bütçe dostu" serbest.
- **Rakip marka / perakendeci adı içerikte asla geçmez** (karşılaştırma ya da örnek olarak da).
- **Sağlık / tıbbi iddia kurulmaz** ("tedavi eder", "iyileştirir", "hipoalerjenik", "dermatolojik olarak test edilmiş", "akne yapmaz"); yalnızca ürün sayfasında açıkça yazıyorsa o ürüne bağlanarak geçer. Doğrulanamayan sayı ve iddia yazılmaz.
- **Site hizmetleri** (kargo, iade, mağazadan teslim) yalnızca sitenin kendi sayfasından teyit edilerek ve süre / tutar rakamı verilmeden anılır.
- **Uzun tire (—) ve emoji kullanılmaz.**
- Kanıt noktasından sayfaya uygun 1-2 tane (70 yıl, 80+ ülke, İstanbul'da %100 kalite kontrol, geniş ton yelpazesi). Uygun sayfada sürdürülebilirliğe kısa bir cümleyle yer ver. Hayvan testi konusu asla yazılmaz ("hayvanlar üzerinde test edilmez", "cruelty-free" vb. dahil).
- Akdeniz atfı sayfada en fazla bir kez, doğal oturuyorsa.
- Her H2/H3'ün ilk cümlesi başlığın sorusunu doğrudan yanıtlar.
- Ana KW: H1/ilk paragraf + 1-2 H2'de doğal biçimde. İkincil KW'ler ilgili bölümlerde. KW'yi her başlığa yapıştırma.
- Uzunluk: mevcut metinden kısa olmasın; tipik 800-1.500 kelime (SSS dahil). Doldurmak için tekrar yazma.

**SSS** (Eylül 2026'daki 5 sayfanın hiçbirinde yoktu; soru KW'si / PAA varsa ekle):
- H2 "{Kategori} Hakkında Sıkça Sorulan Sorular", her soru H3. Kapanış paragrafından hemen önce.
- Mevcut SSS varsa geliştir: cevabı zayıf olanı güçlendir, gövdede zaten cevaplanan soruyu at ya da farklı açıdan sor.
- Soru kaynağı Adım 2-3'teki soru KW'leri ve PAA. 4-6 soru.
- Her cevap ilk cümlede doğrudan yanıt verir, toplam 40-70 kelime. AI Overview / snippet'e alıntılanabilir yapıda.
- Soru metni kullanıcının arama diline yakın ("Tint ruj kalıcı mı?"), cevap sen diliyle ama emir kipi olmadan ("nasıl sürülür" tipi adım sorusu hariç).

**İç linkler:**
- Mevcut içerikteki linkler korunur. Düşürülecekse gerekçesi teslim notunda yazılır (Eylül'de Kaş Kalemi'nde "maskara" ve "far paleti" linkleri fark edilmeden düşmüştü).
- Anchor ile hedef sayfa aynı şeyi anlatmalı ("makyaj fırçası" anchor'ı sünger/aplikatör sayfasına gitmemeli).
- **Gerekiyorsa 1-2 yeni kategori linki eklenir.** Metinde geçen ama linklenmemiş ilgili kategori (ör. kaş kalemi sayfasında "kaş farı", "kaş sabitleyici"; ruj sayfasında "dudak kalemi") ya da yeni eklenen bölümün doğal hedefi varsa link verilir. Önce kategori sayfaları, blog ikinci planda. Bağlam yoksa zorla eklenmez; eklenmediyse teslim notunda "uygun hedef yok" diye belirtilir. Bölüm planında (Adım 4) mevcut ve yeni linkler ayrı gösterilir.
- Sadece LP Update Excel'indeki "Tüm Kategori URLleri" havuzundan ya da canlıda 200 dönen Flormar kategori / blog URL'lerine. Yeni eklenen her URL canlıda kontrol edilir (200, yönlendirme yok).
- Sayfa başına toplam 3-8 link (mevcutlar + en fazla 2-3 yeni). Anchor doğal ve hedef sayfanın ana KW'si. Aynı URL'ye tek link.
- Kannibalizasyon notundaki konular için link ver, konuyu bu sayfada derinleştirme. BAŞKA SAYFA kovasındaki kelime metinde bir kez, tanım cümlesinin içinde ve sahibine link veren anchor olarak geçer.
- **Sayfa kendine link vermez;** bu sayfanın ana KW'si başka bir sayfaya anchor olmaz.
- **Link hedefi:** 200 dönen, index'e açık, canonical'ı kendini gösteren ve içinde stokta ürün olan kategori sayfası (blog ikinci planda). **Ürün sayfasına ve kampanya sayfasına link verilmez** (ürün adları kalın yazılır, linki ekip ekler). 301 / 302 dönen hedefin linki son adrese çevrilir; 404, noindex ya da ürünü kalmamış hedef havuzdan yenisiyle değiştirilir.
- **Link metnin içinden çıkar:** anchor silindiğinde cümle anlamlı kalır ("…[kaş farı](url) ile…" evet; "Kaş farı için [tıkla](url)" hayır).
- **Sadece link vermek için konu dışı paragraf yazma.** Parfüm ve cilt bakımına link verme.
- Format: `[anchor](https://www.flormar.com.tr/...)`.

### Adım 6 - Kalite kontrol
1. Yeni ve mevcut içeriği markdown olarak kaydet (`# H1`, `## H2`, `### H3`, linkler `[anchor](url)`) ve çalıştır:
   `python3 ~/.claude/skills/flormar-kategori-icerik-guncelleme/scripts/qc_check.py yeni.md --mevcut mevcut.md --url https://www.flormar.com.tr/kas-kalemi/ --kw "kaş kalemi"`
   (`--url` kendine link, `--kw` ana KW'nin başka sayfaya anchor olmasını yakalar.)
   KRİTİK bulguların hepsi düzeltilir; UYARI'lar tek tek değerlendirilir (bağlamda doğruysa bırakılabilir, raporda belirt). Script temiz çıkana kadar tekrarla. `--fix` görünmez karakter / çift boşluk / çift noktayı temizlenmiş kopyaya yazar.
2. `references/dikkat-listesi.md`'deki `[oto]` olmayan maddeleri elle kontrol et: bölüm sırası, bozuk birleşen cümleler, çelişkiler, ürün adı doğruluğu, kannibalizasyon, konu dışı paragraf, başlık-cevap uyumu.
3. Mevcut İçerik ile Yeni İçerik'i yan yana oku: mevcut metindeki doğru ve değerli bilgi (ürün özelliği, teknik) kaybolmamış olmalı.
4. **Stok son kontrol:** metindeki kalın ürünler `urun_sitemap.py stok` ile bir kez daha sorgulanır (yazım sürerken tükenen ürün metinden çıkar). Yeni eklenen link hedefleri de bu adımda canlıda kontrol edilir (200, canonical, noindex yok).

### Adım 7 - Doküman ve teslim
Her sayfa için:
```
python3 ~/.claude/skills/flormar-kategori-icerik-guncelleme/scripts/build_doc.py \
  --yeni yeni.md --mevcut mevcut.md --out "Yeni_ https___www.flormar.com.tr_kas-kalemi_.docx"
```
Doküman yapısı (Eylül 2026 dokümanlarıyla aynı; yazı tipi Montserrat, tüm başlıklar siyah — `build_doc.py` bunu otomatik yapar): "YENİ İÇERİK" → sayfa H1'i (kalın satır, ör. "Flormar Kaş Kalemi Çeşitleri") → giriş → H2/H3 bölümler (madde listeleri "- " ile başlayan düz paragraf) → SSS → kapanış → "MEVCUT İÇERİK" (Title stili) → yayındaki metin birebir. Dosya adı ajans kalıbında: `Yeni_ <URL'de / ve : yerine _>.docx`. Dosyalar her zaman `~/flormar/Kategori Güncellemeleri/` klasörüne yazılır (tüm dönemler burada). **Yeni İçerik ve Mevcut İçerik her zaman aynı dokümanda olur.** Teslim klasöründe sadece birleşik .docx bulunur. yeni.md / mevcut.md / QC çıktıları scratchpad'de kalır, ayrı dosya olarak teslim edilmez. Drive'a yüklenince Google Doc olarak açılır. Google Drive bağlıysa doğrudan Doc oluşturmayı teklif et.

Kullanıcıya teslim notu (dokümana değil, sohbete), sayfa başına:
- Hedef KW'ler ve hacimleri
- Yapılan ana değişiklikler (eklenen / birleşen / silinen bölümler, SSS değişiklikleri)
- İç linkler: korunan / yeni eklenen (1-2) / düşürülen (gerekçesiyle)
- Metne giren kalın ürün adları (en fazla 3) ve her birinin örnek ürün URL'si: **linkleri ekip manuel ekleyecek**
- Kaynaklar: hangi bilgi nereden alındı (Ahrefs / GSC / DataForSEO / okunan 2 rakip / ürün sayfası)
- Ne yazılmadı ve neden: BAŞKA SAYFA kovasındaki kelimeler, teyit edilemeyen bilgiler, tükenen ürünler
- Teyit gerekenler: satıştan kalkmış görünen ürünler, kannibalizasyon riski, ürün iddiası emin olunamayan yerler, site düzeyi çakışmalar, hedef teyidinde (canonical / noindex) çıkan sorunlar
- QC sonucu (kalan uyarılar ve neden bırakıldıkları)

Son olarak LP Update Excel'inde ilgili satırların güncellenmesi gerektiğini hatırlat (Excel'e kullanıcı onayı olmadan yazma).

## Revizeler kurala dönüşür
Kullanıcıdan ya da markadan gelen her revize, tarihiyle birlikte `references/dikkat-listesi.md`'ye **genel kural cümlesi** olarak işlenir (tek sayfaya özel düzeltme değil, tekrarını önleyen kural). Taranabiliyorsa `qc_check.py`'ye de eklenir ve `[oto]` işaretlenir.

## Test seti
Eylül 2026'da markanın yorum yaptığı 5 doküman `~/Downloads/Yeni_ *flormar*.docx` altında. Skill'de değişiklik yapınca QC'yi bunlarda çalıştır; yorumlardaki hataların (çift nokta, boşluksuz nokta, "Önerisİ", kavsi/kavisi, Adımları/Adımı, U+2060, buyurgan ton, düşen linkler, stilsiz başlık) yakalandığını gör.

## Bilinen eksikler
- Markanın ayrı bir Türkçe ton rehberi olabilir (revize notlarında "dört marka değeri", "sen dili" geçiyor, Ekim 2025 Brand Platform PDF'inde yok). Gelirse `references/marka-rehberi.md` güncellenir.
