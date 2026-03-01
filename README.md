## 🧹 Veri Ön İşleme ve Gürültü Temizleme (Data Preprocessing & Noise Reduction)

Veri madenciliği ve duygu analizi süreçlerinde ham verinin kalitesi, modelin başarısını doğrudan etkileyen en temel faktördür. Bu aşamada, sosyal medyadan ve çeşitli kaynaklardan toplanan ham metinler, derin öğrenme modelinin (**CNN-LSTM**) odaklanması gereken anlamsal ögeleri (duygu yüklü kelimeler) gölgeleyen "gürültü" bileşenlerinden arındırılmıştır. Temizlik süreci iki ana fazda gerçekleştirilmiştir:

* **Sosyal Medya Kalıntılarının Temizlenmesi:** Metinler içerisinde yer alan ancak duygu taşımayan teknik bileşenler Regex (Düzenli İfadeler) yöntemiyle filtrelenmiştir. Bu kapsamda `https?://\S+|www\.\S+` kalıbı ile URL bağlantıları, `@\w+` kalıbı ile kullanıcı etiketleri (mention) ve `\bRT\b` kalıbı ile Retweet ibareleri metinden tamamen çıkarılmıştır.
* **Karakter Standardizasyonu:** Analiz birliğini sağlamak amacıyla tüm metinler küçük harfe dönüştürülmüştür. Türkçe dil yapısına uygun olarak "I" karakterinin "ı", "İ" karakterinin "i" olarak dönüşümü manuel olarak kontrol edilmiştir. Ayrıca, duygu analizinde istatistiksel karmaşaya yol açan noktalama işaretleri, emojiler ve rakamlar `[^a-zçğıöşü\s]` Regex filtresiyle ayıklanarak sadece alfabetik karakterlerin kalması sağlanmıştır.

| Ham Metin | İşlem Sonrası | Açıklama |
| :--- | :--- | :--- |
| `@BanuGltekin6 İstanbul'da soğuk üstelik yağmurlu🙂` | `istanbul soğuk üstelik yağmur` | Mention, Emoji ve Noktalama içerir. |
| `"RT @EgePolitik: Hüseyin Küçükdemirci'nin kaleminden ""2021 Yılı Asgari Ücret Zammı Üzerine"" https://t.co/OnR0MyMu26"` | `hüseyin küçükdemirci nin kalem yıl asgari ücret zam üzeri` | Noktalama, link, Mention, RT ve hashtag sembolü içerir. |

*Tablo 1: Ham ve Temizlenmiş Metin Karşılaştırması*

---

### Etkisiz Kelimeler (Stop-words) Filtreleme

Metin madenciliğinde "Stop-words", dilde çok sık kullanılan ancak tek başlarına anlamsal bir ağırlığı olmayan, modelin odaklanması gereken anahtar kelimeler üzerinde istatistiksel gürültü yaratan kelimelerdir. Duygu analizi sürecinde, bu kelimelerin temizlenmesi modelin hesaplama karmaşıklığını azaltmakta ve öznitelik (feature) seçimini daha tutarlı hale getirmektedir.

Bu çalışma kapsamında, projenin dil yapısına özel olarak hazırlanan geniş kapsamlı bir stop-words sözlüğü oluşturulmuştur. Sözlük; bağlaçlar, edatlar, miktar belirteçleri ve zamirleri içermektedir. Hazırlanan liste (acaba, ama, ancak, artık…) `.txt` formatında harici bir dosyadan okunarak Python ortamına aktarılmış ve her bir metin örneği üzerinden kelime bazlı filtreleme yapılmıştır.

**Stratejik Karar ve İstisnalar:** Klasik stop-words listelerinin aksine, bu projede duygu analizinin hassasiyeti göz önünde bulundurularak "duygu yönünü tersine çeviren" kelimeler üzerinde özel bir istisna uygulanmıştır.

* **Olumsuzluk Belirteçleri:** "Değil", "Yok", "Asla", "Hiç" gibi kelimeler normalde stop-word kabul edilse de cümledeki duygu kutbunu (pozitiften negatife veya tam tersi) doğrudan belirledikleri için temizleme listesinden çıkarılmış ve veri setinde korunmuştur. *(Örn: "Güzel değil" ifadesindeki "değil" kelimesi atıldığında modelin cümleyi "Güzel" yani pozitif olarak algılamasının önüne geçilmiştir.)*

| Ham Cümle (Gürültüsüz) | Çıkarılan Kelimeler (Stop-words) | Filtrelenmiş Metin (Final) |
| :--- | :--- | :--- |
| `ve sonunda ama çok güzel bir gün oldu` | `ve, ama, çok, bir` | `sonunda güzel gün oldu` |
| `seninle veya onlarla gitmek bana göre değil` | `seninle, veya, onlarla, bana, göre` | `gitmek değil` |
| `herkes geldi ama o gelmedi` | `herkes, ama, o` | `geldi gelmedi` |

*Tablo 2: Stop-words Filtreleme Öncesi ve Sonrası Durum Analizi*
