*[English version](README.en.md)*

# Türkiye Hava Sahası Uçuş Trafiği Analizi

OpenSky Network verileriyle Türkiye hava sahasındaki uçuş trafiğini inceleyen bir veri analizi projesi. Atatürk Üniversitesi Veri Bilimi ve Analitiği bölümünde 1. sınıf öğrencisiyim ve bu, veri bilimi eğitimim kapsamında geliştirdiğim ilk bağımsız projedir — canlı API'lerle çalışma, kimlik doğrulama, veri temizleme ve görselleştirme adımlarını uçtan uca kendi başıma deneyimlemek için tasarladım.

## Projenin Amacı

- Günün hangi saatlerinde Türkiye hava sahasının en yoğun/en sakin olduğunu tespit etmek
- Farklı havayollarının (THY, Pegasus, SunExpress) operasyonel trafik örüntülerini karşılaştırmak
- Elde edilen verinin coğrafi kapsama sınırlarını ve güvenilirliğini değerlendirmek

## Kullanılan Veri

OpenSky Network'ün ["Weekly 24 Hours of State Vector Data"](https://opensky-network.org/data/scientific) veri setinden alınan, 1 günlük (24 saatlik, saatlik dosyalar halinde) tam state vector verisi. Türkiye hava sahasını kapsayan bir dikdörtgen bölgeye (enlem 36-42°, boylam 26-45°) göre filtrelenmiştir.

- **Ham veri:** ~61 milyon satır (dünya geneli)
- **Filtrelenmiş veri:** 897.996 satır (Türkiye bölgesi)
- **Sütunlar:** zaman damgası, uçak kimliği (icao24), çağrı işareti (callsign), konum (enlem/boylam), irtifa, hız, yön, transponder kodu (squawk) ve diğer teknik alanlar

## Yöntem

1. **Veri toplama:** OpenSky'ın canlı REST API'si (OAuth2 kimlik doğrulamalı) ve hazır haftalık arşiv verisi denenmiş; kapsamlı ve tutarlı sonuç için arşiv verisi tercih edilmiştir.
2. **Temizleme:** Zaman damgaları okunabilir formata çevrildi, yerdeki uçaklar filtrelendi, gereksiz teknik sütunlar (sensors, spi) çıkarıldı.
3. **Analiz:** Saatlik gruplama, havayolu bazlı kırılım (callsign öneki üzerinden), coğrafi dağılım incelemesi, acil durum sinyali (squawk kodu) taraması.
4. **Görselleştirme:** Folium ile interaktif haritalar, Matplotlib ile saatlik/havayolu bazlı grafikler.

## Bulgular

### 1. Saatlik Trafik Yoğunluğu
Trafik gece yarısından sonra (00:00-04:00) en düşük seviyede seyrediyor, sabah 05:00'ten itibaren hızla artıyor ve **11:00'de günün zirvesine** (399 farklı uçak) ulaşıyor. Akşam 19:00 civarında ikinci, daha küçük bir yoğunluk artışı gözlemleniyor.

![Saatlik Uçuş Trafiği](images/saatlik_trafik.png)

### 2. Havayolu Bazında Farklı Operasyonel Modeller
- **THY:** Belirgin "dalga" yapısı sergiliyor — 05:00 ve 19:00'da keskin zirveler var. Bu, İstanbul merkezli hub-and-spoke operasyon modeliyle uyumlu.
- **Pegasus (PGT):** Gün boyu nispeten dengeli, dar bir bantta (25-46 uçak) seyrediyor — nokta-nokta uçuş ağı yapısını işaret ediyor.
- **SunExpress (SXS):** En düşük hacimli ama yine dengeli bir seyir izliyor, İzmir/Antalya ağırlıklı bölgesel bir profil sergiliyor.

![Havayolu Bazında Saatlik Trafik](images/havayolu_saatlik_trafik.png)

### 3. Coğrafi Kapsama Sınırları
Veri, Türkiye'nin batı yarısında (İstanbul, Ege, Akdeniz'in batısı) yoğunken, doğuya doğru azalıyor ve yaklaşık **36.5°E boylamından sonra neredeyse hiç veri bulunmuyor.** Bu sınır simetrik değil — güney kıyısında (Adana-Mersin civarı) kapsama, kuzeye göre biraz daha doğuya uzanıyor. Bu durum, OpenSky'ın gönüllü ADS-B alıcı ağının coğrafi dağılımından kaynaklanıyor; Doğu Anadolu'da trafik olmadığı anlamına gelmiyor, o bölgede yeterli alıcı istasyonu olmadığını gösteriyor. Görselleştirmede beliren belirgin çizgiler, ülkenin ana hava trafiği koridorlarını yansıtıyor.

![Veri Kapsama Alanı](images/kapsama_alani.png)

### 4. Anomali Kontrolü
Squawk kodları üzerinden acil durum sinyali (7500/7600/7700) taraması yapıldı; incelenen 1 günlük veride herhangi bir acil durum sinyaline rastlanmadı — örneklem büyüklüğü ve bu tür olayların istatistiksel nadirliği göz önüne alındığında beklenen bir sonuç.

## Sınırlılıklar

- Analiz **tek bir günü** (Pazartesi) kapsıyor; mevsimsel veya haftalık örüntüler hakkında genelleme yapılamaz.
- Veri, OpenSky'ın gönüllü alıcı ağına bağlı olduğundan Doğu Anadolu'da kapsama zayıf/yoktur.
- `origin_country` alanı uçağın tescil ülkesini gösterir, kalkış ülkesini değil.

## Kullanılan Araçlar

Python, pandas, Matplotlib, Folium · Google Colab & VS Code (Jupyter) · OpenSky Network API/veri arşivi

## Geliştirme Fikirleri

- Hafta içi / hafta sonu karşılaştırması için ek gün verisi eklemek
- Uçak metadata veritabanı ile birleştirip uçak tipine göre kırılım yapmak
- OpenSky Trino erişimi onaylanırsa çok aylık zaman serisi analizine geçmek

## İlgili Kaynaklar

Bu projeyi geliştirirken faydalandığım/incelediğim bazı kaynaklar:

- [OpenSky Network](https://opensky-network.org/) — projenin temel veri kaynağı
- [thomasdubdub/opensky-traffic-viz](https://github.com/thomasdubdub/opensky-traffic-viz) — OpenSky verisiyle ülke bazlı uçuş görselleştirmesi yapan, benzer amaçlı bir açık kaynak proje
- [xoolive/traffic](https://github.com/xoolive/traffic) — hava trafiği verisi analizi için daha kapsamlı bir Python kütüphanesi; ileride projeyi büyütmek için incelemeyi planladığım bir kaynak

## Hakkında

**Ömer Bolat**
Data Science and Analytics Student, Atatürk Üniversitesi

Bu proje sürecinde OpenSky'ın canlı API'sinde yaşanan bağlantı/kimlik doğrulama sorunları, farklı platformlar (Google Colab, Kaggle, VS Code) arası geçişler ve büyük veri işleme gibi gerçek dünya problemleriyle karşılaştım; her birini adım adım çözerek ilerledim.

- LinkedIn: [linkedin.com/in/ömer-bolat](https://www.linkedin.com/in/ömer-bolat-604b1932b/)
- GitHub: [https://github.com/omer-bolat](https://github.com/omer-bolat)
- Kaggle: [https://www.kaggle.com/bolatomer](https://www.kaggle.com/bolatomer)

