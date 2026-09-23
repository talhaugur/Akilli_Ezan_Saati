# 🕋 ESP32 Akıllı Ezan Saati ve Web Kontrollü Müzik Kutusu

Bu proje, ESP32 mikrokontrolcüsü kullanılarak geliştirilmiş, WiFi bağlantılı, web arayüzünden kontrol edilebilen tam donanımlı bir **Akıllı Ezan Saati ve İlahi Çalar** sistemidir. 

Sistem, internet üzerinden güncel namaz vakitlerini çeker, vakit girdiğinde otomatik ezan okur ve 16x2 LCD ekran üzerinde dinamik bilgiler sunar. Aynı zamanda yerel ağ üzerinden erişilebilen web arayüzü ile cihaza dokunmadan ses kontrolü, ilahi çalma ve yazılım güncelleme (OTA) işlemleri yapılabilir.

## 🌟 Öne Çıkan Özellikler

- **WiFiManager Entegrasyonu:** Kod içerisine WiFi SSID ve şifre gömmeye gerek yoktur. Cihaz ağ bulamazsa kendi kurulum ağını (`Ezan-Saati-Kurulum`) açar ve telefondan modem seçilmesine olanak tanır.
- **Kesintisiz Web Arayüzü (AJAX):** Sayfa yenilenmesine gerek kalmadan ses açma/kısma, ilahi başlatma ve durdurma işlemleri arka planda gerçekleşir. Dinamik bir ses seviyesi barı (Progress Bar) bulunur.
- **Kayan Ayet Ekranı (Ghosting Korumalı):** 16x2 I2C LCD ekranda Şehir, Vakit ve Kalan Süre bilgilerinin yanı sıra, 35 farklı ayet meali sırayla ve 2 tur atarak (marquee) gösterilir. Ekran titremesi ve harf kalıntısı (ghosting) problemleri tam 16 karaktere tamamlama yöntemiyle çözülmüştür.
- **Kablosuz Güncelleme (OTA):** Cihazı kutusundan çıkarmadan, `/update` adresi üzerinden `.bin` dosyası ile tarayıcıdan yeni yazılım yüklenebilir.
- **NTP Saat & API Senkronizasyonu:** İnternet üzerinden gerçek zamanlı saat (NTP) ve CollectAPI üzerinden il bazlı anlık namaz vakitleri senkronizasyonu.

## 🛠️ Kullanılan Donanımlar

* **ESP32 DevKit V1** (Mikrokontrolcü)
* **DFRobot DFPlayer Mini** (MP3 Çalar Modülü)
* **16x2 LCD Ekran + I2C Modülü**
* **Mini Hoparlör** (DFPlayer'ın SPK+ ve SPK- pinlerine bağlı)
* **MicroSD Kart** (FAT32 formatlı, maks 32GB)

## 📌 Pin Bağlantı Şeması

| ESP32 Pini | Bileşen | Bileşen Pini |
| :--- | :--- | :--- |
| **VIN / 5V** | DFPlayer & LCD | VCC |
| **GND** | DFPlayer & LCD | GND |
| **GPIO 21 (SDA)**| LCD I2C | SDA |
| **GPIO 22 (SCL)**| LCD I2C | SCL |
| **GPIO 16 (RX2)**| DFPlayer Mini | TX |
| **GPIO 17 (TX2)**| DFPlayer Mini | RX |

*(Not: DFPlayer RX pini ile ESP32 arasına 1K ohm direnç konulması ses parazitini azaltır.)*

## 📁 SD Kart Dosya Yapısı

DFPlayer Mini'nin doğru çalışması için SD kart içerisine `mp3` adında bir klasör açılmalı ve dosyalar 4 haneli numaralarla isimlendirilmelidir. **Dosyalar kopyalanırken numara sırasına göre tek tek karta atılmalıdır.**

```text
SD_KART_KOK_DIZIN/
└── mp3/
    ├── 0001.mp3   (İmsak/Sabah Ezanı)
    ├── 0002.mp3   (Öğle)
    ├── 0003.mp3   (İkindi)
    ├── 0004.mp3   (Akşam)
    ├── 0005.mp3   (Yatsı)
    ├── 0006.mp3   (Web Buton 1: Kabede Hacılar Hu Der Allah)
    ├── 0007.mp3   (Web Buton 2: Giremem o Cennetine)
    ├── 0008.mp3   (Web Buton 3: Ey Sevgili)
    └── 0009.mp3   (Web Buton 4: Fairouz & Biggie)
