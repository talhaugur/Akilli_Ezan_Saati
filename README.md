# 🕋 ESP32 Akıllı Ezan Saati ve Web Kontrollü Müzik Kutusu

ESP32, internetten alınan namaz vakitlerini 16×2 LCD'de gösterir ve seçili vakitlerde DFPlayer Mini üzerinden ezan çalar. Aynı yerel ağdaki telefon veya bilgisayardan açılan web sayfası ile ilahi seçilebilir, ses ayarlanabilir ve yeni yazılım yüklenebilir.

> **Başlamadan önce:** Bu README, paylaşılan Arduino kodundaki `collectApiKey`, `city`, `WiFiManager`, `WebServer` ve `/update` yollarını temel alır. Kodun GitHub sürümünde farklılık varsa dosya ve ayar adlarını o sürüme göre kontrol edin. Paylaşılan örnekteki API anahtarını kullanmayın; herkes kendi anahtarını oluşturmalıdır.

## İçindekiler

- [Özellikler](#özellikler)
- [Gerekenler ve bağlantılar](#gerekenler-ve-bağlantılar)
- [SD kartı hazırlama](#sd-kartı-hazırlama)
- [Arduino IDE ile ilk kurulum](#arduino-ide-ile-ilk-kurulum)
- [CollectAPI anahtarı ve şehir ayarı](#collectapi-anahtarı-ve-şehir-ayarı)
- [WiFi kurulumu ve web arayüzü](#wifi-kurulumu-ve-web-arayüzü)
- [Yazılımı güncelleme](#yazılımı-güncelleme)
- [Sorun giderme](#sorun-giderme)
- [Issue açma ve katkıda bulunma](#issue-açma-ve-katkıda-bulunma)

## Özellikler

- WiFiManager ile ilk bağlantıda `Ezan-Saati-Kurulum` ağı üzerinden WiFi ayarı.
- NTP ile saat, CollectAPI ile il bazında namaz vakitleri; kod saat 03.00'te vakitleri tekrar sorgular.
- LCD'de tarih/saat, şehir, sıradaki vakit, kalan süre ve rastgele seçilen 35 kısa mealden biri. Seçilen metin iki kaydırma turu gösterilir.
- Yerel web arayüzünde dört parça, durdurma ve 0–30 aralığında ses ayarı; düğmeler sayfayı yenilemeden istek gönderir.
- Yerel ağda `/update` sayfasından derlenmiş `.bin` dosyasıyla OTA güncellemesi.

**Bağlantı gereksinimi:** İlk WiFi ayarı için telefon gerekir. Saat ve vakitlerin alınabilmesi için ESP32'nin internete erişmesi gerekir. Web arayüzü ve bu projedeki OTA, ESP32 ile **aynı yerel ağdayken** kullanılır; proje uzaktan bulut güncellemesi yapmaz.

## Gerekenler ve bağlantılar

- ESP32 DevKit V1 ve veri aktarabilen USB kablosu
- DFPlayer Mini, hoparlör ve FAT32 biçimli microSD kart
- 16×2 I2C LCD
- Uygun 5 V besleme ve ortak GND
- Arduino IDE, ESP32 kart paketi ve kodun kullandığı kütüphaneler

| ESP32 | Bileşen | Bağlantı |
| --- | --- | --- |
| VIN / 5V | DFPlayer Mini | VCC |
| VIN / 5V | LCD I2C modülü | VCC |
| GND | DFPlayer Mini ve LCD | GND |
| GPIO 21 | LCD | SDA |
| GPIO 22 | LCD | SCL |
| GPIO 16 (RX2) | DFPlayer Mini | TX |
| GPIO 17 (TX2) | DFPlayer Mini | RX; seri hatta 1 kΩ direnç kullanılabilir |
| — | Hoparlör | DFPlayer SPK+ ve SPK− |

> **Donanım notu:** ESP32 GPIO pinleri 3,3 V seviyesindedir. 5 V ile beslenen bazı I2C LCD modülleri SDA/SCL hatlarını 5 V'a çeker; modülünüzü kontrol edin ve gerekiyorsa iki yönlü I2C seviye dönüştürücü kullanın. LCD adresi farklıysa kodda `LiquidCrystal_I2C lcd(0x27, 16, 2)` satırındaki `0x27` değerini değiştirin.

## SD kartı hazırlama

Kartı FAT32 olarak biçimlendirin, kök dizine `mp3` klasörü açın ve dosyaları aşağıdaki gibi adlandırın. Ses dosyalarının kullanım hakkını gözetin.

```text
SD_KART/
└── mp3/
    ├── 0001.mp3  İmsak/Sabah ezanı
    ├── 0002.mp3  Öğle ezanı
    ├── 0003.mp3  İkindi ezanı
    ├── 0004.mp3  Akşam ezanı
    ├── 0005.mp3  Yatsı ezanı
    ├── 0006.mp3  Kabede Hacılar Hu Der Allah
    ├── 0007.mp3  Giremem o Cennetine
    ├── 0008.mp3  Ey Sevgili
    └── 0009.mp3  Fairouz & Biggie
```

**Kodla eşleştirme düzeltmesi:** Paylaşılan kod `myDFPlayer.play(n)` çağırıyor. DFRobot kütüphanesinde `/mp3/0001.mp3` gibi **dosya adına göre** seçim için `myDFPlayer.playMp3Folder(n)` kullanılır. Bu klasör düzenini kullanıyorsanız `playTrack()` içindeki ve `checkAndPlay()` içindeki bütün `myDFPlayer.play(...)` çağrılarını `myDFPlayer.playMp3Folder(...)` olarak değiştirin. Bu düzeltme yapıldıktan sonra dosyaları karta sırayla tek tek kopyalamak zorunlu değildir. Ezan/vakit eşleştirmesini kullandığınız API yanıtının sırasına göre kontrol edin; kod altı vakit girdisi okurken ikinci girdi için ses çalmıyor.

## Arduino IDE ile ilk kurulum

1. [Arduino IDE'yi](https://www.arduino.cc/en/software) kurun. **Dosya → Tercihler → Ek Kart Yöneticisi URL'leri** alanına ESP32'nin kararlı paket adresini ekleyin: `https://espressif.github.io/arduino-esp32/package_esp32_index.json`.
2. **Araçlar → Kart → Kart Yöneticisi** bölümünden Espressif'in `esp32` paketini kurun. Kartınız listede yoksa **ESP32 Dev Module** seçin. [Resmî ESP32 kurulum kılavuzu](https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html).
3. **Kütüphane Yöneticisi** üzerinden `WiFiManager` (tzapu), `LiquidCrystal_I2C`, `DFRobotDFPlayerMini` ve `ArduinoJson` kütüphanelerini kurun. Aynı isimli LCD kütüphanelerinin yöntemleri değişebilir; derleme hatasında kodun kullandığı `init()` ve `backlight()` yöntemlerini destekleyen sürümü seçin. `WiFi.h`, `WebServer.h`, `HTTPClient.h`, `Update.h`, `Wire.h` ve `time.h` ESP32 kart paketiyle gelir.
4. Proje `.ino` dosyasını açın; aşağıdaki API anahtarı ve şehir ayarını yapın. Kodun başında gereken `#include` satırlarının bulunduğunu kontrol edin.
5. **Araçlar → Kart** ve **Araçlar → Port** üzerinden ESP32'yi seçin. **Araçlar → Partition Scheme** altında OTA bölümleri içeren bir düzen kullanın; `No OTA` seçmeyin. Kodunuz iki uygulama bölümüne sığmalıdır.
6. USB kablosunu bağlayın, **Doğrula** ile derleyin ve **Yükle** ile ilk yazılımı karta gönderin. Yükleme başlamazsa kartınızın **BOOT** düğmesini yükleme başlarken kısa süre basılı tutmanız gerekebilir.

> **Kopyalama uyarısı:** Sohbet veya Markdown'dan alınan kodda `\*`, `\<` gibi kaçışlar ya da `h.begin("[https://...](https://...)")` biçiminde Markdown bağlantısı varsa bunlar gerçek C++ kodu değildir. API çağrısındaki adresin düz metin URL olması gerekir: `"https://api.collectapi.com/pray/all?city=" + String(city)`. Kodun derlenebilir proje dosyasını temel alın.

## CollectAPI anahtarı ve şehir ayarı

1. [CollectAPI Namaz Vakitleri API](https://collectapi.com/tr/api/pray/namaz-vakitleri-api) sayfasından hesap açın veya oturum açın. Kullanım hakkı ve limitlerini hesabınızdan kontrol edin.
2. Hesabınızın **Profil → Token** bölümünden kendi anahtarınızı alın. İsteklerde `Authorization` başlığına gönderilir. [CollectAPI anahtar kılavuzu](https://docs.collectapi.com/docs/general.html).
3. Yerel proje klasörünüzde `secrets.h` adlı dosya oluşturun:

   ```cpp
   #pragma once
   const char* collectApiKey = "apikey BURAYA_KENDI_ANAHTARINIZ";
   ```

   Hesabınızda gösterilen yetkilendirme değerini, gerekli `apikey ` önekiyle birlikte ve boşluk eklemeden yazın. Gerçek değeri README'ye, issue'ya veya ekran görüntüsüne koymayın.
4. Ana `.ino` dosyasındaki mevcut `const char* collectApiKey = ...;` satırını kaldırıp başa `#include "secrets.h"` ekleyin. `const char* city = "istanbul";` satırındaki şehri kendi ilinizle değiştirin. Şehir adı API'nin kabul ettiği biçimde olmalıdır.
5. Proje kökündeki `.gitignore` dosyasına `secrets.h` satırı ekleyin. Ardından `git status` ile dosyanın gönderilecekler arasında olmadığını kontrol edin. Dosya daha önce Git tarafından izleniyorsa yalnızca `.gitignore` yeterli olmaz; `git rm --cached secrets.h` ile indeks kaydını kaldırın. Git geçmişinde yer almış anahtarı ayrıca iptal edip yenileyin.

Kod, anahtarı `h.addHeader("authorization", collectApiKey)` ile kullanır. WiFi kurulum ekranı şu an yalnızca ağ adını ve şifresini alır; **API anahtarı web sayfasından girilmiyor**. Şehri veya anahtarı değiştirdikten sonra yeni yazılımı USB veya OTA ile yüklemek gerekir.

**Önemli:** Bu proje için paylaşılan kodda gerçek bir API anahtarı görünüyordu. Anahtar sahibiyseniz CollectAPI panelinden onu iptal edip yeni anahtar oluşturun. Daha önce GitHub'a gönderildiyse son commit'ten silmek yeterli değildir; geçmişte kalmış olabileceği için anahtarı mutlaka yenileyin.

## WiFi kurulumu ve web arayüzü

1. İlk açılışta cihaz kayıtlı ağa bağlanamazsa `Ezan-Saati-Kurulum` adlı bir erişim noktası açar. Telefonu bu ağa bağlayın. Kurulum sayfası kendiliğinden açılmazsa tarayıcıda `http://192.168.4.1` adresini deneyin.
2. WiFiManager sayfasından ev ağınızı seçip şifresini girin. ESP32 bu ağa bağlanır ve sonraki açılışlarda kayıtlı bilgileri kullanır. Telefon ve ESP32 aynı yerel ağda olmalıdır.
3. ESP32'nin IP adresini modeminizin bağlı cihazlar listesinden bulun; `http://ESP32_IP_ADRESI/` adresine gidin (örnek: `http://192.168.1.42/`). **Bu kod IP adresini Seri Monitör'e yazdırmıyor.** İsterseniz WiFi bağlandıktan sonra `Serial.println(WiFi.localIP());` satırını ekleyip 115200 baud ile görebilirsiniz.
4. Sayfadan dört ilahiden birini başlatın, durdurun veya ses seviyesini ayarlayın.

WiFi şifresi ya da modem değiştiğinde ESP32 önce kayıtlı ağa bağlanmayı dener. Bağlanamazsa kurulum ağı yeniden açılır; yeni ağı oradan seçebilirsiniz. Eski ağa hâlâ bağlanabiliyorsa ve farklı ağa geçmek istiyorsanız mevcut kodda ayrıca bir **WiFi sıfırla** düğmesi bulunmadığını unutmayın; WiFiManager'ın ayar silme işlevi eklenmeli veya uygun yöntemle kayıtlı WiFi ayarları temizlenmelidir. Tüm flash'ı silmek, yazılımı da etkileyebileceğinden gelişigüzel kullanmayın.

## Yazılımı güncelleme

### A) USB ile normal yükleme

Kodda değişiklik yapın, aynı kart ve uygun partition seçimiyle tekrar **Doğrula → Yükle** adımlarını uygulayın. İlk kurulum, OTA çalışmadığında kurtarma ve partition düzeni değişikliği için bu yolu kullanın.

### B) Yerel ağ üzerinden tarayıcıyla OTA

1. Arduino IDE'de kodu derleyin; **Sketch → Export Compiled Binary** (Türkçe arayüzde **Derlenmiş İkiliyi Dışa Aktar**) komutuyla uygulama `.bin` dosyasını oluşturun. Çıktı dosyasının yerini proje klasöründe kontrol edin. `bootloader.bin` veya `partitions.bin` yerine **uygulama/sketch `.bin` dosyasını** seçin.
2. ESP32 ve bilgisayar aynı yerel ağdayken `http://ESP32_IP_ADRESI/update` adresini açın. Ana sayfadaki **Yazılım Güncelle (OTA)** düğmesi de buraya gider.
3. Derlenmiş uygulama `.bin` dosyasını seçip **Yazılımı Yükle** düğmesine basın. Cihaza güç vermeye ve ağa bağlı tutmaya devam edin; işlem sırasında sayfayı kapatmayın.
4. `Guncelleme Basarili!` yanıtından ve yeniden başlamasından sonra ana sayfayı tekrar açıp işlevleri kontrol edin. IP adresi yeniden bağlantıda değişebilir.

**OTA sınırları:** Bu, kaynak kodunu GitHub'dan kendi kendine indirmez. Her değişiklik önce derlenmeli, ardından `.bin` dosyası cihaza yüklenmelidir. OTA için kartta uygun bölümleme ve yeterli boş uygulama bölümü gerekir. Mevcut `/update` yolunda kimlik doğrulaması bulunmuyor; aynı ağa erişebilen biri firmware yüklemeyi deneyebilir. Yalnızca güvendiğiniz yerel ağda kullanın; internete port yönlendirmesiyle açmayın.

## Sorun giderme

| Belirti | Kontrol edilecekler |
| --- | --- |
| Kod derlenmiyor | Eksik `#include` veya kütüphaneler, Markdown'dan kopyalanan `\` karakterleri ve API URL'sinin düz metin olup olmadığı. ArduinoJson sürümündeki `JsonDocument` uyumunu da kontrol edin. |
| Kurulum ağı görünmüyor | Cihaz kayıtlı WiFi'ye bağlanmış olabilir. Modem listesinden IP'yi kontrol edin; güç ve USB kablosunu da kontrol edin. |
| LCD boş veya yanlış karakter gösteriyor | Güç/GND, SDA 21, SCL 22, I2C adresi (`0x27`), kontrast ayarı ve I2C voltaj seviyeleri. |
| Saat veya vakitler gelmiyor | İnternet ve NTP erişimi, şehir yazımı, CollectAPI anahtarı/limitleri, API URL'si ve JSON yanıtı. Kod API hatalarını LCD'de ayrıntılı göstermiyor. |
| Ses yok veya yanlış parça çalıyor | FAT32 kart, `/mp3/0001.mp3` adları, DFPlayer TX/RX çapraz bağlantısı, hoparlör ve `playMp3Folder()` düzeltmesi. Kod DFPlayer başlatma hatasında ayrıntılı uyarı vermiyor. |
| `/update` açılamıyor | Telefon/bilgisayar ile ESP32'nin aynı ağda olduğundan, IP'nin güncel olduğundan ve web sunucusunun çalıştığından emin olun. |
| OTA başarısız | Uygulama `.bin` dosyasını, OTA destekli partition düzenini, firmware boyutunu ve kesintisiz beslemeyi kontrol edin. Gerekirse USB ile yeniden yükleyin. |

**Kodun bilinen davranışları:** İkinci API girdisinde ses çalınmıyor; bu, güneş vaktini atlamak için düşünülmüş olabilir ama gerçek API sırasıyla doğrulanmalıdır. `sonOkunanVakit` değeri gün değişiminde temizlenmediği için bazı koşullarda aynı adlı vaktin ertesi gün tetiklenmesi engellenebilir. `checkAndPlay()` içinde `delay(5000)` vardır; ezan tetiklenince web sunucusu bu süre boyunca yanıt vermez. Arayüzdeki ses çubuğu sunucudan gerçek değeri tekrar okumaz; istek başarısızsa görünen değer sapabilir. Bunlar sonraki kod iyileştirmeleri için uygun issue konularıdır.

## Issue açma ve katkıda bulunma

1. Bu projenin GitHub deposunda **Issues → New issue** bölümünü açın. Önce aynı konunun mevcut olup olmadığını arayın.
2. Kısa, açıklayıcı bir başlık yazın: örneğin `OTA yüklemesi %60'ta başarısız oluyor`.
3. ESP32 kart modeli, Arduino ESP32 paket sürümü, ilgili kütüphane sürümleri, yükleme yöntemi (USB/OTA), beklenen ve gerçekleşen davranış ile sorunu yeniden oluşturma adımlarını ekleyin.
4. Varsa derleyici hata metnini, 115200 baud Seri Monitör çıktısını ve bağlantı/ekran fotoğraflarını ekleyin. API anahtarlarını, WiFi şifrelerini ve kişisel bilgileri paylaşmadan önce çıkarın.
5. Düzeltme önermek için depoyu fork'layıp ayrı bir dalda değişiklik yapın; neyi değiştirdiğinizi ve nasıl denediğinizi açıklayan bir **Pull Request** açın. Depoda ayrıca katkı kuralları varsa onları izleyin.

## Kaynaklar

- [Espressif Arduino ESP32 kurulum kılavuzu](https://docs.espressif.com/projects/arduino-esp32/en/latest/installing.html)
- [Espressif partition düzenleri](https://docs.espressif.com/projects/arduino-esp32/en/latest/tutorials/partition_table.html)
- [CollectAPI token kullanımı](https://docs.collectapi.com/docs/general.html)
- [DFRobot DFPlayer Mini örnekleri](https://github.com/DFRobot/DFRobotDFPlayerMini/blob/master/examples/FullFunction/FullFunction.ino)
- [WiFiManager deposu](https://github.com/tzapu/WiFiManager)
