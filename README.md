# PaliGemma 2: Mimari Analiz, İnteraktif Görselleştirme ve Deneysel Çıkarım

Bu repo Google DeepMind tarafından yayımlanan **PaliGemma 2** (*A Versatile Family of Vision-Language Models for Transfer*) çalışmasının teknik analizini, modelin iç mekanizmasını açıklayan interaktif bir web görselleştirmesini ve Google Colab üzerinde gerçekleştirilen yerel çıkarım (inference) deneylerini içermektedir.

---

## Proje Bileşenleri

1. **Deneysel Çıkarım Testleri (`paligemma2_3b_mix_224.ipynb`)**:
   - `google/paligemma2-3b-mix-224` modeli kullanılarak Google Colab (Tesla T4 GPU, FP16) ortamında testler gerçekleştirilmiştir.
   - **Makro Metin Testi (Sokak Panosu: `aci.jpg`):** Büyük puntolu Türkçe metin okuma (OCR), altyazılama (`caption tr`) ve görsel soru-cevap (`VQA`) performansı doğrulanmıştır.
   - **Mekânsal ve Geometrik Algı Testi (Trafik Tabelası: `yol-tabelasi.png`):** Türkçe özel isimlerin tanınması, sahne ayrıştırma ve `<locDDDD>` formatındaki nesne sınır kutusu (bounding box) tespiti sınanmıştır.
   - **Mikro Metin Testi (Market Fişi: `fis.jpg`):** $224\text{px}^2$ giriş çözünürlüğünde yoğun ve küçük fontlu metinlerin okunabilirliği sınanmış; makalede vurgulanan "doküman görevlerinde yüksek çözünürlük gereksinimi" ampirik olarak gözlemlenmiştir.

2. **İnteraktif Mimari Simülasyonu**:
   - Görüntünün model içerisindeki akışını (SigLIP $\to$ Lineer Projeksiyon $\to$ Gemma 2) adım adım modeller.
   - Çözünürlük ölçeklemesinin ($224\text{px}^2$, $448\text{px}^2$, $896\text{px}^2$) yama (patch) sayısı ve hesaplama maliyeti üzerindeki etkisini görselleştirir.
   - Farklı akademik görevlerin (DocVQA, ScienceQA, NLVR2 vb.) model boyutu ve çözünürlük duyarlılıklarını karşılaştırır.
   - Nesne tespiti için kullanılan konum belirteçlerini (`<locDDDD>`) simüle eder.

---

## Deneysel Bulgular Özeti

### 1. `paligemma2-3b-mix-224` Çıktıları ve Gözlemler

| Görsel | Görev / Prompt | Çıktı Durumu | Model Çıktısı & Gözlem |
| :--- | :--- | :---: | :--- |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "ocr"` | Başarılı | `BURSLULUK SINAVI 12 OCAK 2022...`<br>Pano üzerindeki büyük puntolu başlıkları eksiksiz yakaladı; sonuna ön eğitim kaynaklı saat ve yıl ekledi. |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "caption tr"` | Başarılı | `a billboard with the words "bursluluk sinavi" on it.`<br>İstem Türkçe olmasına karşın açıklama İngilizce üretildi; görsel semantiği tam doğru kavradı. |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "answer tr sınav hangi tarihte yapılacak?"` | Kısmen Başarılı | `14.05.2015`<br>224px çözünürlük kısıtı sebebiyle küçük tarih yazısını tam seçemeyip tahmini bir tarih üretti. |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "answer tr hangi sınıflar için sınav var?"` | Kısmen Başarılı | `12, 11 ve 10`<br>Panodaki tüm sınıf listesi yerine genel lise düzeylerini tahmin etti. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "ocr"` | Başarılı | `Çatalkaya Havalimanı Kayseri Edirne İstanbul...`<br>Türkçe özel karakterler (`Ç`, `ı`) ve şehir adları eksiksiz tanındı; yön levhalarına göre tekrarlar oluştu. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "caption tr"` | Başarılı | `In the image we can see there are vehicles on the road. This is a sign board, light pole, trees...`<br>Açıklama İngilizce geldi; araçlar, tabela, aydınlatma direği ve çimler gibi sahne elemanlarını doğru ayrıştırdı. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "answer tr tabela nereyi veya hangi yönü gösteriyor?"` | Başarılı | `tablo, doğrudan ve sağa dönük bir şehirye yön veriyor.`<br>Tabelanın geometrik yön oklarını (düz ve sağa) doğru anlayıp Türkçe yanıtladı. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "detect road sign"` | Başarılı | `<loc0212><loc0717><loc0485><loc0880> road sign ; ...`<br>Model pix2seq 0–1023 normalize konum belirteçlerini kullanarak tabelaları kutu içine aldı. |
| **Market Fişi** (`fis.jpg`) | `prompt = "ocr"` | Başarısız | `SOK. MARKETLER TICA.S MARKEZ GARCE ÖZDEHİR...`<br>En üstteki büyük puntolu market adını yakaladı; alt satırlardaki ürün ve adres harflerinde pikseller dağıldı. |
| **Market Fişi** (`fis.jpg`) | `prompt = "answer tr marketin adı nedir?"` | Kısmen Başarılı | `söz marketler ticaret`<br>Market adındaki harf yapısını kavradı ancak görsel çözünürlük kaybından dolayı `Şok` yerine `söz` üretti. |
| **Market Fişi** (`fis.jpg`) | `prompt = "answer tr fişin tarihi nedir?"` | Kısmen Başarılı | `01/02/2014`<br>Gün ve ay bilgisini (01/02) doğru okudu; mikro puntodan ötürü 2024 yılını 2014 olarak ayrıştırdı. |
| **Market Fişi** (`fis.jpg`) | `prompt = "answer tr toplam tutar ne kadar?"` | Başarısız | `46.60`<br>Gerçek genel toplam yerine hemen üst satırdaki KDV tutarını okudu; 224px kısıtının dikey satır ayrımını imkansız kıldığını kanıtladı. |

### 2. `paligemma2-3b-mix-448` Çıktıları ve Karşılaştırmalı Gözlemler

| Görsel | Görev / Prompt | Çıktı Durumu | Model Çıktısı & Gözlem |
| :--- | :--- | :---: | :--- |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "ocr"` | Başarılı | `BURSLULUK 2019 New'll Feel Ap't's Difference BURSLULUK New'll Feel Ap't's Difference 12 SINAVI 12 SINAVI OCAK OCAK Cumartesi Cumartesi 3-4-...`<br>Çözünürlük artışıyla afişteki ikincil ve küçük metin blokları da okundu; tekrarlı yapılar görüldü. |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "caption tr"` | Başarılı | `a billboard with the words "bursluluk sinavi" on it.`<br>İstem Türkçe olmasına karşın model 448px'de de açıklamayı İngilizce üreterek dil kaymasını (language drift) sürdürdü. |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "answer tr sınav hangi tarihte yapılacak?"` | Başarılı | `12 ocak`<br>224px modelindeki uydurma tarih (halüsinasyon) ortadan kalktı; 1024 yama sayesinde panodaki gerçek sınav tarihi eksiksiz okundu. |
| **Sokak Panosu** (`aci.jpg`) | `prompt = "answer tr hangi sınıflar için sınav var?"` | Kısmen Başarılı | `12`<br>Tarihte geçen `12 OCAK` ifadesi sınıf seviyesiyle karıştı; görsel-metin akıl yürütmesinde bağlamsal kısıt sergiledi. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "ocr"` | Başarılı | `Catalca İstanbul Havaalanı Çerkezköy Silivri Selimpasa Corlu Atatürk Havaalanı Yavuz Sultan Selim Köprüsü İstanbul Edirne İstanbul`<br>Tüm yön levhaları, havalimanı ve köprü isimleri doğru hiyerarşiyle ve eksiksiz metne döküldü. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "caption tr"` | Başarılı | `Aşağıda bir tabelanın altında bir otoyolda giden bir grup araba.`<br>`caption tr` istemine bu görselde doğrudan ve doğal bir Türkçe cümleyle doğru yanıt verdi. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "answer tr tabela nereyi veya hangi yönü gösteriyor?"` | Başarılı | `istanbul`<br>Levhadaki ana varış noktasını doğrudan ve doğru bir şekilde tespit etti. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `prompt = "detect road sign"` | Başarılı | `<loc0204><loc0366><loc0469><loc0558> road sign ; <loc0204><loc0162><loc0459><loc0357> road sign ; <loc0215><loc0710><loc0477><loc0879> road sign ; <loc0558><loc0551><loc0648><loc0692> road sign`<br>Görseldeki 4 farklı tabelayı ayrı ayrı algılayıp koordinatlarını tam olarak sınırlayıcı kutulara (bounding box) dönüştürdü. |
| **Market Fişi** (`fis.jpg`) | `prompt = "ocr"` | Başarılı | `SOK MARKETLER TİC.A.S 13462 MAMAK CANZE ÖZDEMİR ŞAHINTEPE NH 637.5K NO 2 MAMAK ANKARA 8140131899 ANADOLU KUR...`<br>224px'de dağılan küçük adres ve şube harfleri satır satır ve yüksek doğrulukla okundu (token limiti nedeniyle sonda durdu). |
| **Market Fişi** (`fis.jpg`) | `prompt = "answer tr marketin adı nedir?"` | Başarılı | `sok marketler tic.a.s`<br>224px'deki `söz` hatası düzeldi; ticari unvan eksiksiz okundu. |
| **Market Fişi** (`fis.jpg`) | `prompt = "answer tr fişin tarihi nedir?"` | Başarılı | `01/02/2024`<br>Yıl hanesindeki piksel netleştiği için 2014 hatası ortadan kalktı ve gerçek tarih olan 2024 eksiksiz okundu. |
| **Market Fişi** (`fis.jpg`) | `prompt = "answer tr toplam tutar ne kadar?"` | Başarılı | `427.42`<br>224px'deki KDV satırına kayma sorunu çözüldü; model fişin en altındaki gerçek dip toplam tutarını kuruşu kuruşuna tespit etti. |

---
### En-Boy Oranı ve Ön İşleme Deneyi: Kareye Sıkıştırma (Squash) vs. Beyaz Dolgu (Padding)

Dikey ve dar market fişi (`fis.jpg`), `mix-448` modeline iki farklı ön işleme yöntemiyle verilerek en-boy oranı hassasiyeti test edildi:

| Ön İşleme Yöntemi | Görsel Durumu | İstem (Prompt) | Model Çıktısı | Gözlem ve Çıkarım |
| :--- | :--- | :--- | :--- | :--- |
| **Kareye Sıkıştırma (Squash)** | En-boy oranı bozuldu; harfler yatayda basıklaştırılıp uzatıldı. | `"answer tr toplam tutar ne kadar?"` | `427.42` | **Geometrik Dayanıklılık:** 448px'deki 1024 yama, basıklaşan font deformasyonunu tolere edebilecek kadar yüksek temsil kapasitesi sundu. |
| **Beyaz Dolgu (Letterbox Padding)** | Orijinal oran korundu; sağ ve soldaki boşluklar beyaz piksellerle dolduruldu. | `"answer tr toplam tutar ne kadar?"` | `427.42` | **Kaynak İsrafı:** Karakter yapısı doğal kaldı ve sonuç doğru çıktı; ancak 1024 yamanın yaklaşık yarısı anlamsız beyaz arka planı işlemeye harcandı. |

> **Sonuç:** `mix-448` her iki durumda da doğru cevabı üretse de, beyaz dolgu yöntemi işlem bütçesini boş piksellere harcar. Bu ölçüm, LLaVA-NeXT gibi görselin oranını koruyarak dinamik ızgara (any-res) tahsis eden mimarilerin hesaplama verimliliği açısından neden daha üstün olduğunu deneysel olarak göstermektedir.

---
## İnteraktif Görselleştirme

[![Canlı Simülasyon](https://img.shields.io/badge/Canlı%20Simülasyon-brightgreen?style=flat-square&logo=github)](https://sevval12345.github.io/paligemma2-analysis-and-experiments/)
---

## Repo Yapısı
```
├── paligemma2_3b_mix_224.ipynb  # Colab ortamında yürütülen çıkarım defteri
├── aci.jpg                      # Makro metin test görseli
├── fis.jpg                      # Mikro metin / fiş test görseli
├── yol-tabelasi.png             # Yol yön levhası ve nesne tespiti görseli
└── README.md                    # Proje dokümantasyonu
```
---

## Kaynakça

1. Steiner, A., et al. (2024). *PaliGemma 2: A Versatile Family of Vision-Language Models for Transfer*. arXiv preprint [arXiv:2412.03555](https://arxiv.org/abs/2412.03555).

2. Hugging Face Model Kartı: [`google/paligemma2-3b-mix-224`](https://huggingface.co/google/paligemma2-3b-mix-224).
