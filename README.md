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

| Görsel | Görev / İstem | Çıktı Durumu | Gözlem |
| :--- | :--- | :---: | :--- |
| **Sokak Panosu** (`aci.jpg`) | `ocr` | Başarılı | "BURSLULUK SINAVI", "12 OCAK" gibi büyük fontlar eksiksiz okundu; sonuna eğitim verisi kaynaklı eklemeler yapıldı. |
| **Sokak Panosu** (`aci.jpg`) | `caption tr` | Başarılı | Komut Türkçe olmasına rağmen açıklama İngilizce üretildi; semantik sahne doğruluğu tam. |
| **Sokak Panosu** (`aci.jpg`) | `answer tr ...` | Kısmen Başarılı | 224px kısıtı nedeniyle küçük puntolu alt sınıf aralıklarında tahmine dayalı yanıt verildi. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `ocr` | Başarılı | "Çatalkaya", "Havalimanı", "Kayseri", "İstanbul" gibi Türkçe karakterli yer adları hatasız tanındı. |
| **Yol Tabelası** (`yol-tabelasi.png`) | `detect road sign` | Başarılı | Levhalar `<loc0212><loc0717><loc0485><loc0880>` normalize koordinatlarıyla doğru kutulandı. |
| **Market Fişi** (`fis.jpg`) | `ocr` | Başarısız | "SOK. MARKETLER" başlığı tanındı; alt satırlardaki ürün ve adres harflerinde çözünürlük kaybı nedeniyle bozulmalar oldu. |
| **Market Fişi** (`fis.jpg`) | `answer tr (tutar)` | Başarısız | 427,42 TL olan genel toplam yerine hemen üstteki 46,60 TL KDV tutarı okundu (dikey satır ayrımı yapılamadı). |

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
1. Steiner, A., et al. (2024). PaliGemma 2: A Versatile Family of Vision-Language Models for Transfer. arXiv preprint arXiv:2412.03555.

2. Hugging Face Transformers: google/paligemma2-3b-mix-224.
