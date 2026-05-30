# 🚷 Cin Ali Asmaca (Hangman) Oyun Projesi

## 📘Projenin Amacı ve Genel Tanımı
Bu proje; kullanıcı güvenliği, dinamik dosya yönetimi ve gerçek zamanlı arayüz güncellemelerini tek bir masaüstü uygulamasında birleştiren nesne yönelimli bir **Adam Asmaca** oyunudur. 

Projenin temel amacı; dinamik kullanıcı giriş kontrolü (şifreleme ve loglama), harici dosyalardan veri çekerek rastgele kelime üretimi, oyun istatistiklerinin takibi ve güvenli veri sıfırlama mekanizmaları gibi temel yazılım mühendisliği konseptlerini görsel bir kullanıcı arayüzü (GUI) üzerinde başarıyla simüle etmektir.

---

### ⚠️BAŞLAMADAN ÖNCE
Uygulamanın yerel bilgisayarda hatasız ve kararlı bir şekilde çalışabilmesi için aşağıda belirtilen dizin mimarisinin ve dosyaların önceden yapılandırılmış olması gerekmektedir:

```
C:/P2OYUN
├── TXTDosyalar/
│   ├── kelimeler.txt
│   ├── sifre.txt
│   ├── log.txt
│   └── oyunlar.txt
└── Resimler/
    ├── 1.jpg
    ├── 2.jpg
    ├── ...
    ├── 11.jpg
    └── 271.jpg
```

## 🏗️ Mimari Yapı ve Ekran Yönetimi (`CardLayout`)
Uygulama, tek bir pencere `JFrame` içerisinde farklı paneller arasında pürüzsüz ve dinamik geçişler sağlamaktadır

Program ilk başlatıldığında otomatik olarak arka planda bir yapılandırma kontrolü gerçekleştirir:
* **Şifre Kontrolü** Sistem, önceden kaydedilmiş bir kullanıcı şifresinin varlığını sorgular.
* **KAYIT Paneli:** Eğer sisteme kayıtlı bir şifre bulunamadıysa, kullanıcı doğrudan şifre oluşturma ekranına yönlendirilir.
* **GIRIS Paneli:** Eğer sistemde aktif bir şifre mevcutsa, kullanıcıyı karşılayan ve güvenlik doğrulamasından sorumlu tutan giriş paneli yüklenir.
* **ANAPANEL Paneli:** Giriş işlemi başarıyla doğrulandığında, kullanıcının asıl oyuna ve istatistik sekmelerine erişebileceği ana yönetim paneli aktif hale getirilir.

---

## 🔒 Güvenlik ve Hatalı Giriş Takip Sistemi
Kullanıcı güvenliğini artırmak ve kaba kuvvet (brute-force) saldırılarını engellemek adına gelişmiş bir kilitlenme mekanizması kurgulanmıştır:
* **Görsel Takip `JProgressBar` :** Giriş ekranında yer alan `sifreBar` bileşeni, kullanıcının yaptığı hatalı giriş denemelerini görsel olarak takip eder.
* **Maksimum Hak Sınırı:** Kullanıcıya en fazla **3 hatalı giriş hakkı** tanınmıştır.
* **Sistem Kilitleme:** Her hatalı tahminde ilerleme çubuğu bir kademe artar. 3. hatalı girişin ardından giriş butonu `setEnabled(false)` yapılarak tamamen pasif hale getirilir ve yazılımsal olarak sistem kilitlenir.

---

## 🎲 Oyun Mekanikleri ve Dinamik Bileşen Yönetimi
Oyun döngüsü, statik veriler yerine tamamen harici dosya bağımlılıkları ve dinamik arayüz nesneleriyle yönetilmektedir:

* **Dinamik Kelime Üretimi:** Oyun başladığında `C:\P2Oyun\TXTDosyalar\kelimeler.txt` dosyasındaki kelimeler bir `List<String>` yapısına okunur. **Random** sınıfı vesilesiyle bu listeden rastgele bir kelime seçilir.
* **Çalışma Zamanı Arayüz Oluşturumu:** Seçilen kelimenin harf uzunluğu hesaplanarak, arayüze dinamik olarak o uzunlukta `*` işaretli `JLabel` dizisi (`harfEtiketleri`) yerleştirilir.
* **Esnek Tahmin Modeli:** Kullanıcı ister tekil harf tahmini yapabilir, isterse kelimenin tamamını tahmin ederek oyunu tek hamlede bitirebilir. Doğru bilinen harfler, ilgili indislerdeki yıldız işaretlerini kaldırarak harfin kendisini görünür kılar.
***NOT:Kelime Tahmini yanlış olursa sistem bunu da harf tahmininde olduğu gibi bir hak sayar.Yani 1 tane değil hak sayısı dolana kadar kelime tahmini de yapılabilir***

* **Görsel Dar Ağacı İlerlemesi:** Hatalı yapılan her tahminde `tahminSayisi` sayacı artırılır ve `C:\P2Oyun\Resimler\` dizininden hata sırasına uygun görsel `1.jpg`, `2.jpg`, ..., `11.jpg` dinamik olarak ölçeklendirilip ekrana basılır. Maksimum hata limiti **11** olarak belirlenmiştir. Bunlara binaen oyunu kazanınca `271.jpg` ekrana basılır .
* **Zamanlayıcı Kontrolü (`javax.swing.Timer`):** Oyunun başlangıcından itibaren saniyede bir tetiklenen asenkron bir zamanlayıcı, kullanıcının oyunu ne kadar sürede tamamladığını anlık olarak hesaplar ve ekrandaki süre etiketini günceller.

---

## 📎Sekmeli Yönetim `JTabbedPane` ve Veri Entegrasyonu
Ana oyun paneli, kullanıcı deneyimini optimize etmek adına **3 farklı sekmeden** oluşur. `tabPanelStateChanged` olay yakalayıcısı sayesinde sekmeler arasında her geçiş yapıldığında ilgili dosyalar baştan okunarak tablolar anlık güncellenir:

1. **Oyun Sekmesi:** Aktif olarak kelime tahminlerinin yapıldığı, dar ağacı görselinin ve zamanlayıcının yer aldığı ana oyun alanıdır.
2. **Skor Tablosu Sekmesi (`model1`):** `oyunlar.txt` dosyasından beslenir. Oyuncunun geçmişte tamamladığı oyunların listesini, "GALIBIYET" veya "MAGLUBIYET" durumlarını ve harcanan süreleri `JTable` üzerinde listeler.
3. **Geçmiş Log Kayıtları Sekmesi (`model2`):** `log.txt` dosyasından beslenir. Uygulamaya yönelik gerçekleştirilen tüm başarılı ve başarısız giriş denemelerini, deneme yapılan hatalı şifre metinlerini ve işlem zamanını rapor eder.

---

## 🖲️ Güvenli Veri Sıfırlama Mekanizması
Skor Tablosu ve Log Geçmişi pencerelerinin altında bulunan "Temizle" butonları, kritik verilerin kazara veya yetkisiz kişilerce silinmesini önlemek amacıyla koruma altına alınmıştır:
* Butona tıklandığında doğrudan silme işlemi yapmak yerine, `JOptionPane` aracılığıyla arayüz üzerinde dinamik bir `JPasswordField` (Şifre Giriş Kutusu) açılır.
* Kullanıcıdan uygulamanın ana giriş şifresini doğrulaması istenir.
* Yalnızca şifrenin doğru girilmesi durumunda ilgili `.txt` dosyalarının içeriği sıfırlanır ve bağlı bulunan `DefaultTableModel` temizlenerek arayüz güncellenir.

---
