# Tesseract jako OCR

## Dlaczego Tesseract?

Posiadam licencję na Adobe Acrobat, ale to rozwiązanie ma wiele wad:

- jest to stara wersja, nie wiadomo co się stanie przy nowym kompie
- użycie pełnego programu blokuje przeglądarkę PDF nawet na wiele godzin
- rozpoznawanie tekstu w porównaniu z Tesseract jest na bardzo podobnym
  poziomie, Adobe mimo wieku ma przewagę, ale nie jest ona znacząca
- Tesseract jest wolniejszy, ale przy *headless* i przewidywanym obciążeniu
  nie ma to większego znaczenia

## Wymagania programowe

- Tesseract (4.1.1): z dystrybucji Cygwin

- Tesseract pol: https://github.com/tesseract-ocr/tessdata/blob/master/pol.traineddata

  Plik danych do języka polskiego.

  Plik umieszczony w `/usr/share/tessdata` (razem z innymi plikami językowymi).

- Ghostscript (9.50): https://www.ghostscript.com/download/gsdnld.html (AGPL v. 3)

  Konieczny do konwersji plików PDF na pliki graficzne czytane przez Tesseract.

- pdftk (2.02): https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/

  Program do manipulacji plików PDF, konieczny by zredukować rozmiar pliku
  wyjściowego jeśli wejściowym jest PDF, inaczej pliki puchną co najmniej
  8-krotnie. Program nie rozwijany od 2013, mogą być problemy na nowszych
  dystrybucjach Linuksa, na Windows działa bez problemu.

## Polecenia podstawowe

### Z PDF do obrazka

Konwersja PDF do TIF lub JPG:

    gswin64c -q -dQUIET -dSAFER -dBATCH -dNOPROMPT -dNOPAUSE -sDEVICE=tifflzw -r360 -sOutputFile="in%d.tif" in.pdf

    gswin64c -q -dQUIET -dSAFER -dBATCH -dNOPROMPT -dNOPAUSE -sDEVICE=jpeg -r360 -sOutputFile="in%d.jpg" in.pdf

- seria opcji `-d` dla zminimalizowania interakcji.
- `-sDEVICE` - format wyjściowy.
- `-r` - rozdzielczość pliku, taka wysoka powinna nas zabezpieczyć przed niespodziankami
- `-sOutputFile` - nazwa pliku tymczasowego dla Tesseract, %d to kolejne numery (MM: co z > 10)?
- na końcu plik wejściowy

### OCR

OCR pliku:

    tesseract in.jpg out_no_ext --oem 1 -l pol pdf

- plik wejściowy
- nazwa pliku wyjściowego bez rozszerzenia
- `--oem 1` - metoda OCR, najskuteczniejsza wg testów
- `-l pol` - język do OCR, dodawania innych np. przez `pol+eng` zmniejsza
  skuteczność rozpoznania, jakiś mechanizm zmiany/rozpoznawania języka?
- `pdf` format wyjściowy

Polecenie uruchamiane w pętli by przerobić wszystkie strony.

### Z powrotem do PDF

    gswin64c -q -dQUIET -dSAFER -dBATCH -dNOPROMPT -dNOPAUSE -sDEVICE=pdfwrite -r96 -sOutputFile="out-im.pdf" in?.pdf

Znaczące zmiany w porównaniu z pierwszym poleceniem to inne `DEVICE` - pdfwrite
oraz ostateczna rozdzielczość 96dpi.

Jeśli plikiem wejściowym był PDF to jako plik pomocniczy tworzy się bardzo
zubożony plik PDF o minimalnej wielkości (m.in. obrazki cz-b w 1dpi), ale z zachowanym OCR.

### pdftk

    pdftk in.pdf background back.pdf output out.pdf

Wstawia zubożoną stronę z OCR jako tło wizerunku oryginalnego. W ten sposób mamy:

- mały plik PDF (ghostcript na podstawie JPG tworzy pliki nawet 8 razy większe
  niż oryginały, dzięki temu pliki są tylko 20% większe)
- OCR
