# Plaka Tanima Sistemi (MATLAB)
Bu proje, araç görselleri üzerinden **plaka tespiti ve OCR ile plaka karakterlerinin okunmasını sağlayan** bir MATLAB tabanlı araç plaka tanıma sistemidir. Grafik arayüz (GUI) ile çalışır ve görselleri otomatik olarak analiz eder. Proje OCR tabanlıdır ve farklı yöntemlerle plaka bölgesini tespit etmeye çalışır.

## 🚀 Proje Özellikleri

- Kullanıcı dostu grafik arayüz (GUI)
- 3 farklı plaka tespit yöntemi (standart, agresif, ekstra)
- Gelişmiş OCR iyileştirmeleri
- Türk plaka formatına uygun analiz
- Yanlış OCR sonuçlarını otomatik düzeltme
- Görsel yükleme ve analiz butonu

## 🧩 Gerekli Toolboxlar

Bu projeyi çalıştırmak için aşağıdaki MATLAB toolboxlarının yüklü olması gerekir:

- **Image Processing Toolbox**
- **Computer Vision Toolbox**

Bu toolboxlar MATLAB kurulumuna dahil değilse, MATLAB Add-On Manager'dan yükleyebilirsiniz.

## 📂 Proje Dosya Yapısı
PlakaTanimaSistemi/
│
├── PlakaTanimaProjesi.m # Tüm sistemin çalıştığı tek script dosyası
├── Proje Kodu.txt # Kodun metin hali (yedek olarak)
├── Plaka Gorselleri/ # Denemeler için örnek araç görselleri klasörü
└── README.md # Bu döküman

## ⚙️ Kurulum

1. MATLAB'i açın.
2. Bu repoyu bilgisayarınıza indirin veya zip olarak çıkarın.
3. MATLAB'de **PlakaTanimaProjesi.m** dosyasının bulunduğu klasörü `Current Folder` olarak ayarlayın.
4. `PlakaTanimaProjesi.m` dosyasını açın ve çalıştırın (Run tuşuna basın veya komut penceresine `plakaMenusu()` yazın).

## 🧪 Kullanım

1. Program çalıştırıldığında GUI ekranı açılır.
2. Ana menüde **"Fotoğraf Yükle"** butonuna tıklayın.
3. Açılan pencereden bir araç görseli seçin (örnekler `Plaka Gorselleri` klasöründe yer alıyor).
4. Seçilen görsel analiz edilir, tespit edilen plaka görüntülenir ve OCR sonucu ekranda gösterilir.

## ❗️ Hatalar veya Uyarılar

- Eğer **OCR sonucu boş** veya **plaka tespit edilemedi** uyarısı alırsanız, farklı bir görsel deneyin.
- Görsel çok düşük kaliteliyse, algoritma başarılı tespit yapamayabilir.
- Toolbox yüklü değilse çalışmaz.

## 👨‍💻 Geliştirici

 **Recep POLAT GitHub: @recepolat63**

## 📝 Lisans

Bu proje açık kaynaklıdır ve kişisel, akademik amaçlarla serbestçe kullanılabilir. Ticari kullanım için geliştiriciyle iletişime geçilmesi tavsiye edilir.
