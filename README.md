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

---

## Mimari Analiz, İnteraktif Görselleştirme ve Deneysel Çıkarım

[![Canlı Simülasyon](https://img.shields.io/badge/Demo-Canlı%20Simülasyon-brightgreen?style=flat-square&logo=github)](https://sevval12345.github.io/paligemma2-analysis-and-experiments/)
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Sevval12345/paligemma2-analysis-and-experiments/blob/main/paligemma2_3b_mix_224.ipynb)
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
