# PaliGemma 2: Mimari Analiz, İnteraktif Görselleştirme ve Deneysel Çıkarım

Bu repo Google DeepMind tarafından yayımlanan **PaliGemma 2** (*A Versatile Family of Vision-Language Models for Transfer*) çalışmasının teknik analizini, modelin iç mekanizmasını açıklayan interaktif bir web görselleştirmesini ve Google Colab üzerinde gerçekleştirilen yerel çıkarım (inference) deneylerini içermektedir.

## İnteraktif Görselleştirme

[![Canlı Simülasyon](https://img.shields.io/badge/Canlı%20Simülasyon-brightgreen?style=flat-square&logo=github)](https://sevval12345.github.io/paligemma2-analysis-and-experiments/)
---

## Proje Bileşenleri

1. **Notebook'lar**
   - `paligemma2_3b_mix_224.ipynb` → `google/paligemma2-3b-mix-224`
   - `paligemma2_3b_mix_448.ipynb` → `google/paligemma2-3b-mix-448`
   - Ortam: Google Colab, Tesla T4, FP16. Tüm çıktılar tek görsel üzerinde, tek denemede, greedy decoding (`do_sample=False`) ve `max_new_tokens=64` ile alındı. Uzun OCR çıktıları bu limitte kesilir.
2. **İnteraktif sayfa**
   - Görüntünün model içindeki akışı (SigLIP → doğrusal projeksiyon → Gemma 2).
   - Çözünürlüğün (224 / 448 / 896 px) yama sayısına ve eğitim maliyetine etkisi.
   - Paper'daki bazı görevlerin (DocVQA, AI2D, NLVR2 vb.) model boyutuna ve çözünürlüğe duyarlılığı.
   - Konum token'larının (`<locDDDD>`) kutuya dönüşümü.

## Deneylerin Kapsamı ve Sınırları

- Görseller: bir bursluluk afişi (`aci.jpg`), bir yol tabelası (`yol-tabelasi.png`) ve bir market fişi (`fis.jpg`).
- `mix-224` ve `mix-448`, paper'daki pt checkpoint transfer çalışması değil, **ayrı ince ayarlanmış "mix" checkpoint'leridir**. İkisi arasındaki fark yalnızca çözünürlükten kaynaklanmaz (farklı eğitim karışımı ve aşama ağırlıkları olabilir).
- Görsel başına tek deneme var. Aşağıdaki tablolar **gözlemdir, istatistiksel kanıt değildir.**
- Referans doğrular görsellerden elle okundu: afişte sınıflar `3-4-5-6-7-9-10-11`, tarih `12 Ocak Cumartesi`; fişte `TOPKDV 46,60`, `TOPLAM 427,42`, tarih `01/02/2024`; tabela görselinde 4 tabela.
---
## Sonuçlar: `paligemma2-3b-mix-224`

| Görsel | Prompt | Model çıktısı (kısaltılmamış) | Değerlendirme |
| :--- | :--- | :--- | :--- |
| Afiş | `ocr` | `BURSLULUK BURSLULUK Yükseköğretim Yükseköğretim SINAVI SINAVI 12 12 OCAK OCAK 24/04/2020 Salı 24/04/2020 Sal` | Kısmen: başlık, `12 OCAK` okundu; `Yükseköğretim` ve `24/04/2020 Salı` hatalı/uydurma, satırlar tekrarlı, 64 token'da kesildi |
| Afiş | `caption tr` | `a billboard that says 'bursluluk sinavi' on it` | Doğru içerik, ama İngilizce |
| Afiş | `answer tr sınav hangi tarihte yapılacak?` | `14.06.2015` | Yanlış (doğru: 12 Ocak) |
| Afiş | `answer tr hangi sınıflar için sınav var?` | `12, 11 ve 10` | Yanlış/eksik (doğru: 3-4-5-6-7-9-10-11) |
| Yol tabelası | `ocr` | `Çatalkaya Çatalkaya Havalimanı Çatalkaya Silindiri Havalimanı Havalimanı Kayseri Kayseri Edirne İstanbul İstanbul` | Kısmen: `Kayseri` tabelada yok (uydurma), `Silindiri` hatalı, çok tekrar |
| Yol tabelası | `caption tr` | `In the foreground of the picture there are sign boards, road, vehicles, trees, street lights and other objects. In the center of the picture there are street lights, trees, road and other objects. Sky is cloudy.` | Sahne doğru, ama İngilizce |
| Yol tabelası | `answer tr tabela nereyi veya hangi yönü gösteriyor?` | `tablo, doğrudan ve sağa dönük bir şehirye yön veriyor.` | Kısmen: yön fikri var, hedef adı yok, dil bozuk |
| Yol tabelası | `detect road sign` | `<loc0212><loc0717><loc0485><loc0880> road sign ; <loc0200><loc0155><loc0459><loc0345> road sign ; <loc0210><loc0357><loc0464><loc0563> road sign ; <loc0558><loc0547><loc0652><loc0691> road sign` | 4 kutu (4 tabelayla uyumlu) |
| Fiş | `ocr` | `SOK. MARKETLER TICA.S 1.2401. MARKE GARCE ÖZDEHİR 05.02.2014 15:00 HATAY - ANKARA 05.02.2014 15` | Kısmen: market adı okundu; adres/tarih kısmı güvenilmez (`05.02.2014`, `HATAY`) |
| Fiş | `answer tr marketin adı nedir?` | `söz marketler ticaret` | Yanlış (`Şok Marketler`) |
| Fiş | `answer tr fişin tarihi nedir?` | `01/02/2014` | Kısmen: gün/ay doğru, yıl yanlış (2024) |
| Fiş | `answer tr toplam tutar ne kadar?` | `46.60` | Yanlış: bu tutar KDV satırı, toplam 427.42 |

## Sonuçlar: `paligemma2-3b-mix-448`

| Görsel | Prompt | Model çıktısı (kısaltılmamış) | Değerlendirme |
| :--- | :--- | :--- | :--- |
| Afiş | `ocr` | `BURSLULUK 2019 New'll Feel Ap't's Difference BURSLULUK New'll Feel Ap't's Difference 12 SINAVI 12 SINAVI OCAK OCAK Cumartesi Cumartesi 3-4-` | Kısmen: `12 OCAK Cumartesi` ve `3-4-` geldi; `2019` ve İngilizce parça afişte yoksa uydurmadır, tekrarlar var, 64 token'da kesildi |
| Afiş | `caption tr` | `a billboard with the words "bursluluk sinavi" on it.` | Doğru içerik, ama İngilizce |
| Afiş | `answer tr sınav hangi tarihte yapılacak?` | `12 ocak` | Doğru (hafta günü yok) |
| Afiş | `answer tr hangi sınıflar için sınav var?` | `12` | Yanlış/eksik. Tarihteki `12` ile karışmış olabilir (hipotez) |
| Yol tabelası | `ocr` | `Catalca İstanbul Havaalanı Çerkezköy Silivri Selimpasa Corlu Atatürk Havaalanı Yavru Sultan Selim Köprüsü İstanbul Edirne İstanbul` | İyi: 224'teki `Kayseri` yok. Küçük hatalar: `Catalca`, `Yavru` (Yavuz), `Selimpasa`, `Corlu` (diakritik kaybı) |
| Yol tabelası | `caption tr` | `Aşağıda bir tabelanın altında bir otoyolda giden bir minibüs.` | Türkçe, akıcı ama "minibüs" ve ifade tartışmalı |
| Yol tabelası | `answer tr tabela nereyi veya hangi yönü gösteriyor?` | `istanbul` | Makul (tek hedef verdi) |
| Yol tabelası | `detect road sign` | `<loc0204><loc0366><loc0469><loc0558> road sign ; <loc0204><loc0159><loc0459><loc0357> road sign ; <loc0215><loc0710><loc0476><loc0879> road sign ; <loc0558><loc0547><loc0647><loc0691> road sign` | 4 kutu (4 tabelayla uyumlu) |
| Fiş | `ocr` | `SOK MARKETLER TİC.A.S 13462 MAMAK CANZE ÖZDEMİR ŞAHINTEPE NH 637.5K NO 2 MAMAK ANKARA 8140131899 ANADOLU KUR` | Market adı/şube bilgisi okundu, doğruluk satır satır doğrulanmadı; 64 token'da kesildi |
| Fiş | `answer tr marketin adı nedir?` | `sok marketler tic.a.s` | Doğru |
| Fiş | `answer tr fişin tarihi nedir?` | `01/02/2024` | Doğru |
| Fiş | `answer tr toplam tutar ne kadar?` | `427.42` | Doğru |

### Kısa karşılaştırma

| Soru | 224 | 448 |
| :--- | :--- | :--- |
| Afiş tarihi | `14.06.2015` (yanlış) | `12 ocak` (doğru) |
| Afiş sınıfları | `12, 11 ve 10` (yanlış) | `12` (yanlış) |
| Tabela OCR | `Kayseri` uydurdu | `Kayseri` yok |
| Tabela kutu sayısı | 4 | 4 |
| Fiş market adı | `söz marketler ticaret` | `sok marketler tic.a.s` |
| Fiş tarihi | `01/02/2014` | `01/02/2024` |
| Fiş toplam | `46.60` (KDV) | `427.42` (doğru) |

Bu tek denemelik tablo, belge/OCR görevlerinde yüksek çözünürlüğün yardımcı olduğu yönündeki paper bulgusuyla **tutarlıdır**; fakat tek görsel, tek deneme ve ayrı checkpoint olduğu için bunu doğrulamaz. Afiş sınıf sorusu iki çözünürlükte de başarısız oldu, yani çözünürlük her hatayı çözmedi.

---

## Ek Deney: En-Boy Oranı (squash vs padding)

`fis.jpg` yaklaşık 3:4 oranındadır. Processor görseli varsayılan olarak kareye yeniden boyutlandırır (squash). Karşılaştırma için görsel ayrıca beyaz dolguyla kareye tamamlandı (padding).

| Ön işleme | Model | Prompt | Çıktı |
| :--- | :--- | :--- | :--- |
| Varsayılan yeniden boyutlandırma (squash) | `mix-448` | `answer tr toplam tutar ne kadar?` | `427.42` |
| Beyaz dolgu (padding) | `mix-448` | `answer tr toplam tutar ne kadar?` | `427.42` |

Her iki yöntemde de cevap doğrudur. Padding'de görüntü alanının bir kısmı (fiş 3:4 ise yaklaşık dörtte biri) boş piksellerden oluşur. Tek örnekle bu iki yöntemin başarı farkı hakkında sonuç çıkarılamaz.

---

## Yeniden Üretmek İçin

1. Hugging Face'te ilgili modellerin lisansını kabul edip `login()` yapın.
2. Notebook'u Colab'de (T4 GPU) açın, görselleri aynı klasöre koyun.
3. Prompt'u değiştirerek hücreyi çalıştırın.

---

## Kaynakça

1. Steiner, A., et al. (2024). *PaliGemma 2: A Versatile Family of Vision-Language Models for Transfer*. arXiv preprint [arXiv:2412.03555](https://arxiv.org/abs/2412.03555).

2. Hugging Face Model Kartı (mix-224): [`google/paligemma2-3b-mix-224`](https://huggingface.co/google/paligemma2-3b-mix-224).

3. Hugging Face Model Kartı (mix-448): [`google/paligemma2-3b-mix-224`](https://huggingface.co/google/paligemma2-3b-mix-448)
