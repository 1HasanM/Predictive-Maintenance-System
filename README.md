Proje Amacı
Bu projenin temel amacı, endüstriyel cihazlardan gelen sensör verilerini (metric1'den metric9'a kadar olan metrikler) kullanarak cihazların potansiyel arızalarını (failure sütunu) makine öğrenmesi modelleri aracılığıyla önceden tahmin etmektir. Projenin başarısı, özellikle arızalı cihazları doğru bir şekilde tespit etme yeteneğine (recall) bağlıdır.

Aşamalar ve Uygulanan Yöntemler
1. Aşama: Veri Keşfi ve Temel Model Denemesi

Uygulanan Yöntem: pandas kütüphanesi kullanılarak predictive_maintenance_dataset.csv adlı veri seti yüklendi ve ilk inceleme (.head()) yapıldı.

Kod 1 Yorumu: Veri setinin cihaz bazlı metrikler ve arıza bilgisi içerdiği tespit edildi. Sütun isimleri date, device, failure ve metric1'den metric9'a kadar olan değerlerden oluşuyordu.

Görsel Analiz (Kod 2 ile):

Arıza Dağılımı: Veri setinin aşırı dengesiz (highly imbalanced) olduğu görüldü. Arıza (failure=1) örnekleri, arızasız (failure=0) örneklere göre çok az sayıdaydı.

Korelasyon Matrisi: metric1, metric6 ve metric5 gibi bazı metriklerin arızayla pozitif korelasyonu olduğu tespit edildi.

Temel Model Denemesi (Kod 2):

Model: RandomForestClassifier kullanıldı.

Sonuçlar: Model, yüksek doğruluk (accuracy=0.999) skoruna ulaşsa da, bu skorun yanıltıcı olduğu anlaşıldı. Arızalı cihazların hiçbirini tespit edemedi (recall=0.00, True Positives=0). Bu durum, modelin dengesiz verilerde baskın olan arızasız sınıfı ezberlemesinden kaynaklanıyordu.

2. Aşama: Veri Dengesizliği Sorununu Giderme Girişimi

Uygulanan Yöntem: Veri dengesizliği sorununu çözmek için RandomForestClassifier modeline class_weight='balanced' parametresi eklendi. Bu parametre, modelin azınlık sınıfına daha fazla önem vermesini sağlamayı amaçlıyordu.

Kod 3 Yorumu:

Özellik Önemi: class_weight parametresinin modelin hangi özellikleri daha önemli gördüğünü değiştirdiği gözlemlendi (metric4 öne çıktı).

Sonuçlar: Ne yazık ki bu yaklaşım yeterli olmadı. Modelin performansı neredeyse hiç değişmedi. Confusion Matrix'e göre yine hiçbir arızalı cihaz doğru tahmin edilemedi (recall=0.00, True Positives=0).

3. Aşama: Geliştirilmiş Dengesizlik Giderme ve Gelişmiş Modelleme

Uygulanan Yöntem: Önceki başarısız denemelerden ders alınarak daha kapsamlı bir strateji belirlendi:

Özellik Mühendisliği: Cihazların zaman içerisindeki davranışlarını yakalamak amacıyla, metriklerin son 30 günlük ortalamalarını (avg_30d_metricX) içeren yeni özellikler oluşturuldu.

SMOTE Kullanımı: imblearn kütüphanesinden SMOTE (Synthetic Minority Over-sampling Technique) kullanılarak eğitim verisindeki arızalı örnekler sentetik olarak çoğaltıldı ve veri dengeli hale getirildi.

Yeni Model: Daha önce denenen Random Forest yerine, dengesiz verilerle daha başarılı olabilen GradientBoostingClassifier modeli seçildi.

Kod 4 Yorumu:

SMOTE Uygulaması: Eğitim verisindeki arıza örnek sayısı 85'ten 99510'a çıkarılarak dengesizlik tamamen giderildi.

Sonuçlar: Bu yeni yaklaşım, projenin en büyük başarısını getirdi:

ROC AUC Skoru: 0.69'dan 0.88'e yükseldi. Bu, modelin arızaları ayırt etme yeteneğinin ciddi oranda arttığını gösterdi.

Confusion Matrix ve Classification Report: recall değeri, arıza sınıfı (failure=1) için 0.00'dan 0.57'ye yükseldi. Model, artık 21 arızalı cihazdan 12'sini doğru bir şekilde tespit edebiliyordu. Bu, modelin pratik bir fayda sağladığını kanıtladı. precision değeri (0.02) düşük kalsa da, bu birçok bakım senaryosunda kritik olan arıza kaçırma (False Negatives) riskini azaltmıştır.

Genel Sonuç ve Öneriler
Bu proje, bir makine öğrenmesi modelinin accuracy gibi yanıltıcı metriklerle değil, projenin asıl amacına uygun metriklerle (recall) değerlendirilmesinin önemini göstermiştir. Veri dengesizliği gibi zorlu bir sorun, standart modelleme yöntemleriyle çözülememiş, ancak SMOTE, özellik mühendisliği ve uygun algoritma seçimi gibi gelişmiş tekniklerin kombinasyonuyla başarıyla aşılmıştır.
