% Ana menu icin GUI olusturma ve kullanici arabirimini hazirlama
function plakaMenusu()
fig = figure('Name', 'Plaka Tanima Sistemi', 'Position', [200, 150, 1000, 700], 'MenuBar', 'none', 'ToolBar', 'none', 'Resize', 'on');
yaziBasligi = uicontrol('Style', 'text', 'Position', [350, 620, 300, 50], 'String', 'PLAKA TANIMA SISTEMI', 'FontSize', 18, 'FontWeight', 'bold');
yuklemeButonu = uicontrol('Style', 'pushbutton', 'Position', [400, 550, 200, 50], 'String', 'Fotograf Yukle', 'FontSize', 14, 'FontWeight', 'bold', 'Callback', @(~,~) fotografYukle(fig));
sonucPaneli = uipanel('Parent', fig, 'Position', [0.05, 0.1, 0.9, 0.65], 'Title', 'Sonuclar', 'FontSize', 12, 'Visible', 'off');
setappdata(fig, 'sonucPaneli', sonucPaneli);
setappdata(fig, 'yaziBasligi', yaziBasligi);
setappdata(fig, 'yuklemeButonu', yuklemeButonu);
set(fig, 'ResizeFcn', @(src,evt) boyutlandirmaCallback(src));
end

%==========================================================================
%                           DOSYA YUKLEME VE BASLANGIC
%==========================================================================

% Dosya secme ve islem baslatma fonksiyonu - kullanicidan resim alir ve plaka tanima islemini baslatir
function fotografYukle(anaFigur)
[dosyaAdi, dosyaYolu] = uigetfile({'*.jpg;*.jpeg;*.png;*.bmp', 'Gorsel Dosyalari (*.jpg,*.jpeg,*.png,*.bmp)'}, 'Araba gorseli secin');
if isequal(dosyaAdi,0)
msgbox('Dosya secilmedi.', 'Uyari', 'warn');
return;
end
resim = imread(fullfile(dosyaYolu, dosyaAdi));
sonucPaneli = getappdata(anaFigur, 'sonucPaneli');
set(sonucPaneli, 'Visible', 'on');
delete(get(sonucPaneli, 'Children'));
ekseni1 = axes('Parent', sonucPaneli, 'Position', [0.05, 0.1, 0.28, 0.8]);
ekseni2 = axes('Parent', sonucPaneli, 'Position', [0.36, 0.1, 0.28, 0.8]);
ekseni3 = axes('Parent', sonucPaneli, 'Position', [0.67, 0.1, 0.28, 0.8]);
imshow(resim, 'Parent', ekseni1);
title(ekseni1, 'Yuklenen Araba Gorseli', 'FontSize', 10, 'FontWeight', 'bold');
try
plakaMetni = plakaTespitVeTanima(resim, ekseni2, ekseni3);
if ~isempty(plakaMetni) && ~strcmp(plakaMetni, 'TESPIT_EDILEMEDI')
uicontrol('Style', 'text', 'Parent', sonucPaneli, 'Position', [20, 20, 600, 35], 'String', sprintf('Tespit Edilen Plaka: %s', plakaMetni), 'FontSize', 14, 'FontWeight', 'bold', 'ForegroundColor', 'blue', 'BackgroundColor', get(sonucPaneli, 'BackgroundColor'));
else
uicontrol('Style', 'text', 'Parent', sonucPaneli, 'Position', [20, 20, 600, 35], 'String', 'Plaka tespit edilemedi.', 'FontSize', 14, 'FontWeight', 'bold', 'ForegroundColor', 'red', 'BackgroundColor', get(sonucPaneli, 'BackgroundColor'));
end
catch hataDetayi
uicontrol('Style', 'text', 'Parent', sonucPaneli, 'Position', [20, 20, 600, 35], 'String', sprintf('Hata: %s', hataDetayi.message), 'FontSize', 14, 'FontWeight', 'bold', 'ForegroundColor', 'red', 'BackgroundColor', get(sonucPaneli, 'BackgroundColor'));
end
end

%==========================================================================
%                        ANA PLAKA TANIMA KOORDINASYONU
%==========================================================================

% Ana plaka tanima fonksiyonu - resim onisleme ve farkli yontemlerle plaka tespiti
function plakaMetni = plakaTespitVeTanima(resim, ekseni2, ekseni3)
% ADIM 1: Resim onisleme - gri donusum ve boyut optimizasyonu
if size(resim, 3) == 3
griResim = rgb2gray(resim);
else
griResim = resim;
end
[yukseklik, genislik] = size(griResim);
if max(yukseklik, genislik) > 1000
olcekFaktoru = 1000 / max(yukseklik, genislik);
griResim = imresize(griResim, olcekFaktoru);
end
% ADIM 2: Yontem denemeleri - standart, agresif ve ekstra yontemler sirasiyla denenir
plakaMetni = birinciyontem_klasik(griResim, ekseni2, ekseni3);
if strcmp(plakaMetni, 'TANINAMADI')
plakaMetni = ikinciyontem_agresif(griResim, ekseni2, ekseni3);
end
if strcmp(plakaMetni, 'TANINAMADI')
plakaMetni = ucuncuyontem_ekstra(griResim, ekseni2, ekseni3);
end
if strcmp(plakaMetni, 'TANINAMADI')
plakaMetni = 'TESPIT_EDILEMEDI';
end
end

%==========================================================================
%                          PLAKA TANIMA YONTEMLERI
%==========================================================================

% Birinci yontem - standart goruntu isleme teknikleri kullanarak plaka tanima
function plakaMetni = birinciyontem_klasik(griResim, ekseni2, ekseni3)
try
% ADIM 1: Goruntu kalitesi artirma - kontrast iyilestirme ve gurultu azaltma
islenmisResim = adapthisteq(griResim, 'ClipLimit', 0.02, 'NumTiles', [8 8]);
islenmisResim = medfilt2(islenmisResim, [3 3]);
% ADIM 2: Kenar tespiti - plaka sinirlarini belirginlestirme
kenarlar = edge(islenmisResim, 'Sobel');
% ADIM 3: Morfolojik islemler - dikdortgen plaka yapilarini vurgulama
yapiElemani1 = strel('rectangle', [2, 8]);
yapiElemani2 = strel('rectangle', [3, 3]);
islenmisResim = imclose(kenarlar, yapiElemani1);
islenmisResim = imclose(islenmisResim, yapiElemani2);
islenmisResim = imfill(islenmisResim, 'holes');
% ADIM 4: PLAKA ALANI TESPİTİ
plakaBolgeleri = basitPlakaAlaniTespit(islenmisResim, griResim);
% ADIM 5: PLAKA KIRPMA VE OCR İŞLEMİ
if ~isempty(plakaBolgeleri)
maksimumAday = min(3, length(plakaBolgeleri));
for indeks = 1:maksimumAday
plakaResmi = basitPlakaAlaniKirp(griResim, plakaBolgeleri(indeks));
if ~isempty(plakaResmi)
imshow(plakaResmi, 'Parent', ekseni2);
title(ekseni2, 'Tespit Edilen Plaka');
plakaMetni = gelismisOCROku(plakaResmi, ekseni3);
if ~strcmp(plakaMetni, 'TANINAMADI') && length(plakaMetni) >= 4
return;
end
end
end
end
plakaMetni = 'TANINAMADI';
catch
plakaMetni = 'TANINAMADI';
end
end

% Ikinci yontem - agresif parametreler ile zorlu goruntuler icin plaka tanima
function plakaMetni = ikinciyontem_agresif(griResim, ekseni2, ekseni3)
try
% ADIM 1: Guclu kontrast iyilestirmesi - dusuk kaliteli goruntuler icin
gelistirilmisResim = imadjust(griResim, stretchlim(griResim), [0 1]);
gelistirilmisResim = adapthisteq(gelistirilmisResim, 'ClipLimit', 0.01);
% ADIM 2: Coklu kenar tespiti - farkli algoritmalari birlestirme
kenarlar1 = edge(gelistirilmisResim, 'Canny', [0.05, 0.2]);
kenarlar2 = edge(gelistirilmisResim, 'Sobel');
birlesikKenarlar = kenarlar1 | kenarlar2;
% ADIM 3: Morfolojik islemler - kenar yapilarini gucendirme
yapiElemani1 = strel('rectangle', [2, 8]);
yapiElemani2 = strel('rectangle', [3, 3]);
islenmisResim = imclose(birlesikKenarlar, yapiElemani1);
islenmisResim = imclose(islenmisResim, yapiElemani2);
islenmisResim = imfill(islenmisResim, 'holes');
% ADIM 4: PLAKA ALANI TESPİTİ
plakaBolgeleri = agresifPlakaAlaniTespit(islenmisResim, griResim);
% ADIM 5: PLAKA KIRPMA VE OCR İŞLEMİ
if ~isempty(plakaBolgeleri)
maksimumAday = min(5, length(plakaBolgeleri));
for indeks = 1:maksimumAday
plakaResmi = basitPlakaAlaniKirp(griResim, plakaBolgeleri(indeks));
if ~isempty(plakaResmi)
imshow(plakaResmi, 'Parent', ekseni2);
title(ekseni2, 'Tespit Edilen Plaka (Agresif)');
plakaMetni = gelismisOCROku(plakaResmi, ekseni3);
if ~strcmp(plakaMetni, 'TANINAMADI') && length(plakaMetni) >= 4
return;
end
end
end
end
plakaMetni = 'TANINAMADI';
catch
plakaMetni = 'TANINAMADI';
end
end

% Ucuncu yontem - son care olarak farkli parametre kombinasyonlari ile plaka tanima
function plakaMetni = ucuncuyontem_ekstra(griResim, ekseni2, ekseni3)
try
% ADIM 1: Coklu kontrast denemesi - farkli histogram ayarlari
kontrastCesitleri = {
imadjust(griResim, [0.2 0.8], [0 1]),
imadjust(griResim, [0.3 0.7], [0 1]),
histeq(griResim)
};
% ADIM 2: Her kontrast icin farkli threshold degerlerini deneme
for kontrastIndeksi = 1:length(kontrastCesitleri)
try
geciciResim = kontrastCesitleri{kontrastIndeksi};
for esikDegeri = [0.3, 0.5, 0.7]
try
% ADIM 3: Binary donusum ve morfolojik temizleme
binaryResim = imbinarize(geciciResim, esikDegeri);
yapiElemani = strel('rectangle', [2, 6]);
binaryResim = imclose(binaryResim, yapiElemani);
binaryResim = imfill(binaryResim, 'holes');
binaryResim = bwareaopen(binaryResim, 30);
% ADIM 4: PLAKA ALANI TESPİTİ
plakaBolgeleri = ekstraPlakaAlaniTespit(binaryResim, griResim);
% ADIM 5: PLAKA KIRPMA VE OCR İŞLEMİ
if ~isempty(plakaBolgeleri)
maksimumAday = min(3, length(plakaBolgeleri));
for indeks = 1:maksimumAday
plakaResmi = basitPlakaAlaniKirp(griResim, plakaBolgeleri(indeks));
if ~isempty(plakaResmi)
imshow(plakaResmi, 'Parent', ekseni2);
title(ekseni2, 'Tespit Edilen Plaka (Ekstra)');
plakaMetni = gelismisOCROku(plakaResmi, ekseni3);
if ~strcmp(plakaMetni, 'TANINAMADI') && length(plakaMetni) >= 4
return;
end
end
end
end
catch
continue;
end
end
catch
continue;
end
end
plakaMetni = 'TANINAMADI';
catch
plakaMetni = 'TANINAMADI';
end
end

%==========================================================================
%                          PLAKA ALANI TESPİT FONKSİYONLARI
%==========================================================================

% Standart plaka alani tespiti - normal kriterlerle dikdortgen plaka benzeri alanlari bulma
function plakaBolgeleri = basitPlakaAlaniTespit(binaryResim, griResim)
bagliBileskenler = bwconncomp(binaryResim);
if bagliBileskenler.NumObjects == 0
plakaBolgeleri = [];
return;
end
ozellikler = regionprops(bagliBileskenler, griResim, 'Area', 'BoundingBox', 'Solidity', 'Extent');
resimAlani = size(binaryResim, 1) * size(binaryResim, 2);
minimumAlan = max(200, resimAlani * 0.0005);
maksimumAlan = resimAlani * 0.4;
plakaBolgeleri = [];
for indeks = 1:length(ozellikler)
sinirKutusu = ozellikler(indeks).BoundingBox;
en_boy_orani = sinirKutusu(3) / sinirKutusu(4);
if ozellikler(indeks).Area >= minimumAlan && ozellikler(indeks).Area <= maksimumAlan && ozellikler(indeks).Solidity >= 0.15 && ozellikler(indeks).Extent >= 0.1 && en_boy_orani >= 1.8 && en_boy_orani <= 15.0 && sinirKutusu(3) > 15 && sinirKutusu(4) > 5
plakaBolgeleri = [plakaBolgeleri, ozellikler(indeks)];
end
end
if ~isempty(plakaBolgeleri)
enBoyOranlari = arrayfun(@(x) x.BoundingBox(3)/x.BoundingBox(4), plakaBolgeleri);
alanlar = [plakaBolgeleri.Area];
idealSkor = zeros(size(enBoyOranlari));
for sayac = 1:length(enBoyOranlari)
if enBoyOranlari(sayac) >= 2.5 && enBoyOranlari(sayac) <= 4.5
idealSkor(sayac) = alanlar(sayac) * 2;
else
idealSkor(sayac) = alanlar(sayac);
end
end
[~, siralamaIndeksi] = sort(idealSkor, 'descend');
plakaBolgeleri = plakaBolgeleri(siralamaIndeksi);
end
end

% Agresif plaka alani tespiti - daha esnek kriterlerle kucuk veya bozuk plakalari yakalama
function plakaBolgeleri = agresifPlakaAlaniTespit(binaryResim, griResim)
bagliBileskenler = bwconncomp(binaryResim);
if bagliBileskenler.NumObjects == 0
plakaBolgeleri = [];
return;
end
ozellikler = regionprops(bagliBileskenler, griResim, 'Area', 'BoundingBox', 'Solidity', 'Extent');
resimAlani = size(binaryResim, 1) * size(binaryResim, 2);
minimumAlan = max(120, resimAlani * 0.0003);
maksimumAlan = resimAlani * 0.5;
plakaBolgeleri = [];
for indeks = 1:length(ozellikler)
sinirKutusu = ozellikler(indeks).BoundingBox;
en_boy_orani = sinirKutusu(3) / sinirKutusu(4);
if ozellikler(indeks).Area >= minimumAlan && ozellikler(indeks).Area <= maksimumAlan && ozellikler(indeks).Solidity >= 0.1 && ozellikler(indeks).Extent >= 0.08 && en_boy_orani >= 1.5 && en_boy_orani <= 20.0 && sinirKutusu(3) > 12 && sinirKutusu(4) > 4
plakaBolgeleri = [plakaBolgeleri, ozellikler(indeks)];
end
end
if ~isempty(plakaBolgeleri)
enBoyOranlari = arrayfun(@(x) x.BoundingBox(3)/x.BoundingBox(4), plakaBolgeleri);
alanlar = [plakaBolgeleri.Area];
idealSkor = zeros(size(enBoyOranlari));
for sayac = 1:length(enBoyOranlari)
if enBoyOranlari(sayac) >= 2.0 && enBoyOranlari(sayac) <= 6.0
idealSkor(sayac) = alanlar(sayac) * 1.5;
else
idealSkor(sayac) = alanlar(sayac);
end
end
[~, siralamaIndeksi] = sort(idealSkor, 'descend');
plakaBolgeleri = plakaBolgeleri(siralamaIndeksi);
end
end

% Ekstra plaka alani tespiti - en esnek kriterlerle son umut plaka yakalama denemesi
function plakaBolgeleri = ekstraPlakaAlaniTespit(binaryResim, griResim)
bagliBileskenler = bwconncomp(binaryResim);
if bagliBileskenler.NumObjects == 0
plakaBolgeleri = [];
return;
end
ozellikler = regionprops(bagliBileskenler, griResim, 'Area', 'BoundingBox', 'Solidity', 'Extent');
resimAlani = size(binaryResim, 1) * size(binaryResim, 2);
minimumAlan = max(80, resimAlani * 0.0002);
maksimumAlan = resimAlani * 0.6;
plakaBolgeleri = [];
for indeks = 1:length(ozellikler)
sinirKutusu = ozellikler(indeks).BoundingBox;
en_boy_orani = sinirKutusu(3) / sinirKutusu(4);
if ozellikler(indeks).Area >= minimumAlan && ozellikler(indeks).Area <= maksimumAlan && ozellikler(indeks).Solidity >= 0.05 && ozellikler(indeks).Extent >= 0.05 && en_boy_orani >= 1.2 && en_boy_orani <= 25.0 && sinirKutusu(3) > 10 && sinirKutusu(4) > 3
plakaBolgeleri = [plakaBolgeleri, ozellikler(indeks)];
end
end
if ~isempty(plakaBolgeleri)
[~, siralamaIndeksi] = sort([plakaBolgeleri.Area], 'descend');
plakaBolgeleri = plakaBolgeleri(siralamaIndeksi);
end
end

%==========================================================================
%                          PLAKA KIRPMA VE OCR İŞLEMLERİ
%==========================================================================

% Plaka alani kirpma fonksiyonu - tespit edilen plaka bolgesini keserek OCR icin hazirlar
function plakaResmi = basitPlakaAlaniKirp(griResim, plakaBolgesi)
sinirKutusu = plakaBolgesi.BoundingBox;
kenarBoslugu = 0.1;
x = max(1, round(sinirKutusu(1) - sinirKutusu(3) * kenarBoslugu));
y = max(1, round(sinirKutusu(2) - sinirKutusu(4) * kenarBoslugu));
genislik = min(size(griResim, 2) - x, round(sinirKutusu(3) * (1 + 2 * kenarBoslugu)));
yukseklik = min(size(griResim, 1) - y, round(sinirKutusu(4) * (1 + 2 * kenarBoslugu)));
if genislik <= 0 || yukseklik <= 0
plakaResmi = [];
return;
end
plakaResmi = imcrop(griResim, [x, y, genislik, yukseklik]);
if size(plakaResmi, 1) < 6 || size(plakaResmi, 2) < 18
plakaResmi = [];
return;
end
plakaResmi = imadjust(plakaResmi, stretchlim(plakaResmi));
if size(plakaResmi, 1) < 35
buyutmeOrani = 35 / size(plakaResmi, 1);
plakaResmi = imresize(plakaResmi, buyutmeOrani);
end
end

% OCR islemi fonksiyonu - farkli goruntu isleme teknikleriyle plaka metnini cikartir
function plakaMetni = gelismisOCROku(plakaResmi, ekseni3)
try
% ADIM 1: Coklu OCR denemesi - farkli goruntu isleme yontemleriyle karakterleri tanir
ocrSonuclari = {};
try
sonuc1 = ocr(plakaResmi, 'CharacterSet', '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ');
ocrSonuclari{end+1} = sonuc1.Text;
catch
ocrSonuclari{end+1} = '';
end
try
kontrastResim = imadjust(plakaResmi, stretchlim(plakaResmi, [0.01, 0.99]));
sonuc2 = ocr(kontrastResim, 'CharacterSet', '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ');
ocrSonuclari{end+1} = sonuc2.Text;
catch
ocrSonuclari{end+1} = '';
end
try
binaryResim = imbinarize(plakaResmi, 'adaptive', 'Sensitivity', 0.4);
binaryResim = bwareaopen(binaryResim, 3);
imshow(binaryResim, 'Parent', ekseni3);
sonuc3 = ocr(binaryResim, 'CharacterSet', '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ');
ocrSonuclari{end+1} = sonuc3.Text;
catch
ocrSonuclari{end+1} = '';
end
try
tersBinary = ~imbinarize(plakaResmi, 'adaptive', 'Sensitivity', 0.6);
tersBinary = bwareaopen(tersBinary, 3);
sonuc4 = ocr(tersBinary, 'CharacterSet', '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ');
ocrSonuclari{end+1} = sonuc4.Text;
catch
ocrSonuclari{end+1} = '';
end
try
histogramEsitResim = histeq(plakaResmi);
histogramEsitResim = imbinarize(histogramEsitResim);
sonuc5 = ocr(histogramEsitResim, 'CharacterSet', '0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ');
ocrSonuclari{end+1} = sonuc5.Text;
catch
ocrSonuclari{end+1} = '';
end
% ADIM 2: En iyi sonuc secimi - Turk plaka formatina en uygun metni bulma
enIyiPlaka = 'TANINAMADI';
enUzunluk = 0;
for indeks = 1:length(ocrSonuclari)
temizlenmisPlaka = plakatemizle(ocrSonuclari{indeks});
if ~strcmp(temizlenmisPlaka, 'TANINAMADI')
gecerliKarakterSayisi = sum(isstrprop(temizlenmisPlaka, 'alphanum'));
if gecerliKarakterSayisi == length(temizlenmisPlaka) && length(temizlenmisPlaka) >= 4 && length(temizlenmisPlaka) <= 8
rakamSayisi = sum(isstrprop(temizlenmisPlaka, 'digit'));
harfSayisi = sum(isstrprop(temizlenmisPlaka, 'alpha'));
if rakamSayisi >= 2 && harfSayisi >= 1 && length(temizlenmisPlaka) > enUzunluk
enUzunluk = length(temizlenmisPlaka);
enIyiPlaka = temizlenmisPlaka;
elseif enUzunluk == 0 && length(temizlenmisPlaka) >= 4
enUzunluk = length(temizlenmisPlaka);
enIyiPlaka = temizlenmisPlaka;
end
end
end
end
plakaMetni = enIyiPlaka;
if ~strcmp(plakaMetni, 'TANINAMADI')
title(ekseni3, sprintf('OCR: %s', plakaMetni));
else
title(ekseni3, 'OCR: TANINAMADI');
end
catch
imshow(plakaResmi, 'Parent', ekseni3);
title(ekseni3, 'OCR: HATA');
plakaMetni = 'TANINAMADI';
end
end

%==========================================================================
%                          METİN TEMİZLEME FONKSİYONU
%==========================================================================

% Plaka metni temizleme fonksiyonu - OCR sonucunu Turk plaka formatina uygun hale getirir
function temizPlaka = plakatemizle(hamMetin)
if isempty(hamMetin)
temizPlaka = 'TANINAMADI';
return;
end
% ADIM 1: Temel temizleme - sadece harf ve rakamlari birakir
temizMetin = upper(regexprep(hamMetin, '[^A-Z0-9]', ''));
% ADIM 2: TR ibare temizligi - plakada bulunmayan TR yazilarini siler
temizMetin = regexprep(temizMetin, '^TR', '');
temizMetin = regexprep(temizMetin, 'TR$', '');
temizMetin = regexprep(temizMetin, 'TR', '');
temizMetin = regexprep(temizMetin, '^T[0-9]', '');
temizMetin = regexprep(temizMetin, '^[0-9]T', '');
temizMetin = regexprep(temizMetin, '^R[0-9]', '');
temizMetin = regexprep(temizMetin, '^[0-9]R', '');
% ADIM 3: Turk plaka format duzeltmesi - fazla rakam baslangiclarini siler
if length(temizMetin) >= 3
if isstrprop(temizMetin(1), 'digit') && isstrprop(temizMetin(2), 'digit') && isstrprop(temizMetin(3), 'digit')
temizMetin = temizMetin(2:end);
elseif length(temizMetin) >= 4 && isstrprop(temizMetin(1), 'digit') && isstrprop(temizMetin(2), 'digit') && isstrprop(temizMetin(3), 'digit') && isstrprop(temizMetin(4), 'alpha')
temizMetin = temizMetin(2:end);
elseif length(temizMetin) >= 5
harfPozisyonlari = find(isstrprop(temizMetin, 'alpha'));
if ~isempty(harfPozisyonlari)
ilkHarfPozisyonu = harfPozisyonlari(1);
if ilkHarfPozisyonu > 3
oncekiRakamlar = temizMetin(1:ilkHarfPozisyonu-1);
if all(isstrprop(oncekiRakamlar, 'digit'))
temizMetin = [temizMetin(2:ilkHarfPozisyonu-1), temizMetin(ilkHarfPozisyonu:end)];
end
end
end
end
end
% ADIM 4: Ek TR kalinti temizligi - yanlis taninan TR kombinasyonlarini duzeltir
temizMetin = regexprep(temizMetin, '^1R', '1');
temizMetin = regexprep(temizMetin, '^2R', '2');
temizMetin = regexprep(temizMetin, '^3R', '3');
temizMetin = regexprep(temizMetin, '^4R', '4');
temizMetin = regexprep(temizMetin, '^5R', '5');
temizMetin = regexprep(temizMetin, '^6R', '6');
temizMetin = regexprep(temizMetin, '^7R', '7');
temizMetin = regexprep(temizMetin, '^8R', '8');
% ADIM 5: Yanlis baslangic duzeltmesi - harf ile baslayan yanlis tanimalari duzeltir
if length(temizMetin) > 5 && isstrprop(temizMetin(1), 'alpha') && isstrprop(temizMetin(2), 'digit')
temizMetin = temizMetin(2:end);
end
% ADIM 6: Son format kontrolu - hala 3 rakamla baslayanlar duzeltilir
if length(temizMetin) >= 3
if isstrprop(temizMetin(1), 'digit') && isstrprop(temizMetin(2), 'digit') && isstrprop(temizMetin(3), 'digit')
temizMetin = temizMetin(2:end);
end
end
% ADIM 7: Format gecerlilik kontrolu - Turk plaka formatlarini kontrol eder
if ~isempty(temizMetin)
gecerliFormat = false;
formatlar = {'^[0-9]{1,2}[A-Z]{1,3}[0-9]{1,4}$', '^[0-9]{1,2}[A-Z]{1,3}[0-9]{1,2}$', '^[0-9]{1,2}[A-Z]{1,2}[0-9]{1,4}$', '^[0-9]{1,2}[A-Z]{1,1}[0-9]{1,4}$'};
for i = 1:length(formatlar)
if ~isempty(regexp(temizMetin, formatlar{i}, 'once'))
gecerliFormat = true;
break;
end
end
if ~gecerliFormat && length(temizMetin) >= 5
while length(temizMetin) > 3 && isstrprop(temizMetin(1), 'digit') && isstrprop(temizMetin(2), 'digit') && isstrprop(temizMetin(3), 'digit')
temizMetin = temizMetin(2:end);
end
end
end
% ADIM 8: Son kontrol ve sonuc dondurmesi - uzunluk ve karakter sayisi kontrolu
if length(temizMetin) < 4 || length(temizMetin) > 8
temizPlaka = 'TANINAMADI';
else
rakamSayisi = sum(isstrprop(temizMetin, 'digit'));
harfSayisi = sum(isstrprop(temizMetin, 'alpha'));
if rakamSayisi >= 2 && harfSayisi >= 1
temizPlaka = temizMetin;
else
temizPlaka = 'TANINAMADI';
end
end
end

%==========================================================================
%                          ARAYÜZ BOYUTLANDIRMA
%==========================================================================

% Pencere boyutlandirma fonksiyonu - arayuz elemanlarini yeni pencere boyutuna gore duzenler
function boyutlandirmaCallback(figur)
pozisyon = get(figur, 'Position');
genislik = pozisyon(3);
yukseklik = pozisyon(4);
yaziBasligi = getappdata(figur, 'yaziBasligi');
ogrenciAdi = getappdata(figur, 'ogrenciAdi');
yuklemeButonu = getappdata(figur, 'yuklemeButonu');
sonucPaneli = getappdata(figur, 'sonucPaneli');
set(yaziBasligi, 'Position', [(genislik-300)/2, yukseklik-80, 300, 50]);
set(yuklemeButonu, 'Position', [(genislik-200)/2, yukseklik-150, 200, 50]);
set(ogrenciAdi, 'Position', [genislik-320, 30, 300, 30]);
panelYuksekligi = yukseklik - 200;
if panelYuksekligi > 100
set(sonucPaneli, 'Position', [0.05, 80/yukseklik, 0.9, panelYuksekligi/yukseklik]);
end
end