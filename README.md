## -TR-
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

**Recep POLAT  
GitHub: [@recepolat63](https://github.com/recepolat63)**


## 📝 Lisans

Bu proje açık kaynaklıdır ve kişisel, akademik amaçlarla serbestçe kullanılabilir. Ticari kullanım için geliştiriciyle iletişime geçilmesi tavsiye edilir.

## -EN-
# License Plate Recognition System (MATLAB)

This project is a **MATLAB-based license plate recognition system** that detects license plates in vehicle images and reads plate characters using OCR. It operates through a graphical user interface (GUI) and analyzes images automatically. The project uses OCR as its core and applies multiple methods to locate the plate region.

## 🚀 Project Features

- User-friendly graphical interface (GUI)
- 3 different plate detection methods (standard, aggressive, extra)
- Advanced OCR improvements
- Analysis optimized for Turkish plate formats
- Automatic correction of incorrect OCR results
- Image upload and analysis button

## 🧩 Required Toolboxes

To run this project, the following MATLAB toolboxes must be installed:

- **Image Processing Toolbox**
- **Computer Vision Toolbox**

If these toolboxes are not included in your MATLAB installation, you can add them via the MATLAB Add-On Manager.

## 📂 Project File Structure
LicensePlateRecognitionSystem/
│
├── PlakaTanimaProjesi.m # Main script file containing the entire system
├── Project Code.txt # Code as plain text (backup)
├── Plate Images/ # Sample vehicle images for testing
└── README.md # This document

## ⚙️ Setup

1. Open MATLAB.
2. Download or unzip this repository to your computer.
3. In MATLAB, set the folder containing `PlakaTanimaProjesi.m` as the **Current Folder**.
4. Open `PlakaTanimaProjesi.m` and run it (either click **Run** or type `plakaMenusu()` in the command window).

## 🧪 Usage

1. When the program runs, the GUI screen will open.
2. In the main menu, click **"Upload Photo"**.
3. Select a vehicle image (samples are located in the `Plate Images` folder).
4. The selected image will be analyzed, the detected plate will be shown, and the OCR result will be displayed on the screen.

## ❗️ Errors or Warnings

- If you see **"No OCR result"** or **"Plate could not be detected"**, try using a different image.
- The algorithm may fail on very low-quality images.
- The program won't run without the required toolboxes.

## 👨‍💻 Developer

**Recep POLAT  
GitHub: [@recepolat63](https://github.com/recepolat63)**

## 📝 License

This project is open-source and may be freely used for personal or academic purposes.  
For commercial use, please contact the developer.

