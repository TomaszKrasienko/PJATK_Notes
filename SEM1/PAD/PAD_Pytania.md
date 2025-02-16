# Rodzaje danych i metody analizy

## Dane ciągłe

### Jakie to dane?

Dane numeryczne, które mogą przyjmować dowolną wartość a określonym przedziale.

### Przykłady metod

- Analiza korelacji: Badanie siły i kierunku związku między zmiennymi (np. współczynnik korelacji Pearsona)
- Regresja liniowa: Modelowanie relacji między zmienną zależną, a niezależnymi.
    - Technika, która pozwala opisać zależność między zmienną zależną Y, a jedną lub wieloma zmiennymi niezależnymi (X)
    - Pozwala na przewidywanie przyszłych wartości
    - Analizy związków - ocenia siłę i kierunek zależności między zmiennymi

## Szeregi czasowe

### Jakie to dane?

Dane zorganizowane w sposób chronologiczny

### Przykłady metod

- Dekompozycja - rozkład szeregu czasowego na komponenty - trend, sezonowość, reszty
    - Trend - długoterminowy kierunek zmian w danych, np. sprzedaż lodów w trakcie dekady → ignoruje krótkie wahania, ważna ogólna tendencja
    - Sezonowość - regularne, cykliczne wzorce powtarzające się w określonych interwałach, np. co rok → jest przewidywalna, może wynikać z czynników kulturowych, pogodowych czy gospodarczych
    - Reszty - nie da się wyjaśnić przez trend ani sezonowość, np. niespodziewane spadki sprzedaży lodów z powodu jednorazowego wydarzenia → pozwala ocenić, jak dobrze model (trend + sezonowość) opisuje dane, reszty powinny przypominać szum losowy

## Dane kategoryczne

### Jakie to dane?

Dane podzielone na kategorie (np. płeć, kolory)

### Przykłady metod

- Tabele krzyżowe - prezentacja relacji między zmiennymi kategoriami
- Test chi-kwadrat - ocena, czy istnieje związek między zmiennymi kategorycznymi

# *Hipotezy statystyczne*

Hipoteza statystyczna to założenie lub stwierdzenie dotyczące właściwości populacji, które jest weryfikowane za pomocą danych próbkowych.

### Jaki jest cel?

Umożliwienie uzyskania zgodnych wniosków z tych samych danych.

### Kluczowe aspekty testów statystycznych

$H_0$ → Hipoteza zerowa - zwykle oznacza brak efektu lub różnicy, np. średnia wynosi 0

$H_1$ → Hipoteza alternatywna - zwykle oznacza istnienie efektu lub różnicy, np. średnia
 różna od 0

### Poziom istotności

- zwykle 0.05 reprezentuje tolerancje dla błędu pierwszego rodzaju (odrzucenie prawdziwej hipotezy zerowej

### P-wartość
- prawdopodobieństwo popełnienia błędu pierwszego rodzaju
- wartość, która mówi nam czy możemy odrzucić hipotezę zerową czy nie. 

**P-wartość** → To liczba, która mówi „jak rzadki” jest twój wynik

**α** → To twój próg cierpliwości wobec błędów: jak często możesz zaakceptować, że się mylisz, odrzucając

## Omówienie Testów

### Test Z:

**Kiedy używamy?**

Używamy, gdy chcemy porównać **średnią próby** (np. grupa ludzi - klasa) ze **średnią populacyjną**, ale działa najlepiej gdy znamy odchylenie w populacji (**odchylenie standardow**).

**Przykład**:
- sprawdzenie średniego wzrostu uczniów w klasie vs. średni wzrost wszystkich ludzi, zakładając, że znamy odchylenie standardowe, czyli jak bardzo ten wzrost się różni


### Test T (T-student):

**Kiedy używamy?**

Gdy mamy małą próbkę lub nie znamy rozrzutu w populacji.
- **Rodzaje testów T**:
    - **Test T dla jednej próbki**: Porównujemy średnią próbki z konkretną wartością (np. czy średnia temperatura grupy jest równa 36,6°C).
    - **Test T dla dwóch niezależnych próbek**: Sprawdzamy, czy średnie w dwóch niezależnych grupach się różnią (np. średni wzrost kobiet vs mężczyzn).
    - **Test T dla próbek sparowanych**: Porównujemy te same jednostki przed i po jakimś działaniu (np. wyniki ucznia przed i po korepetycjach).

### Test chi-kwadrat

**Kiedy używamy?**

Do analizy zmiennych jakościowych (np. płeć, kolor włosów).
- **Rodzaje zastosowań**:
    - **Test zgodności**: Sprawdzamy, czy rozkład danych w próbce pasuje do oczekiwanego (np. czy liczba osób o różnych grupach krwi odpowiada teoretycznym proporcjom).
    - **Test niezależności**: Sprawdzamy, czy dwie zmienne jakościowe są ze sobą powiązane (np. czy płeć wpływa na preferencję ulubionego koloru).

### Test ANOVA (wariancji)

**Kiedy używamy?**

Służy do porównania średnich w więcej niż dwóch grupach.Zakłada, że dane są normalnie rozłożone, a wariancje są jednorodne.

**Przykład:**

Porównanie średniego czasu reakcji kierowców, którzy spożywali trzy różne rodzaje napojów energetycznych

### Test F

- **Kiedy używamy?**
Gdy chcemy porównać wariancję w więcej niż dwóch grupach. Często stosowany jako część ANOVA, aby sprawdzić jednorodność wariancji.

**Przykład:**

Użycie testu F w ANOVIE do sprawdzenia czy wariancje są jednorodne.

# Techniki modelowania statystycznego

### Czym jest modelowanie?

To proces budowania i oceny modeli statystycznych lub uczenia maszynowego w celu przewidywania wartości lub klasyfikacji danych?

## Rodzaje problemów

### Klasyfikacja

- Celem jest przypisanie obserwacji do jednej z predefiniowanych kategorii.
- Przykłady modeli
    - **Drzewa decyzyjne**
    Tworzenie reguł decyzyjnych na podstawie danych
    - **Regresja logistyczna
    M**odelowanie prawdopodobieństwa przynależności do danej klasy
- Miara oceny
    - **Dokładność (accuracy)**
    Precent poprawnych klasyfikacji
    - **F1-score**
    Średnia harmoniczna precyzji i czułości

### Regresja

- Celem jest przewidywanie wartości ciągłej
- Przykłady modeli:
    - **Regresja liniowa**
    Modelowanie zależności liniowej między zmiennymi
    - **Lasy losowe**
    Złożony model bazujący na wielu drzewach decyzyjnych
- Miary oceny:
    - **R-kwadrat**
    Wyjaśniona wariancja zmiennej zależnej przez model
    - **Średni błąd kwadratowy**
    Średnia różnica między przewidywanymi a rzeczywistymi wartościami

Pytanie dodatkowe:

### 1. Czym jest współczynnik korelacji Pearsona?

Pokazuje siłę i kierunek liniowej zależności między dwiema zmiennymi. Przyjmuje wartości od -1 do 1
**Gdzie**:

- 1 - oznacza idealną korelację dodatnią
- -1 - oznacza idealną korelację ujemną
- 0 - oznacza brak korelacji liniowej

### 2. Homoskedastyczność i Heteroskedastyczność

To zjawisko, gdy wariancja reszt (różnic między wartościami przewidywanymi a rzeczywistymi) jest stała dla wszystkich obserwacji. Oznacza równomierne rozłożenie punktów wokół linii regresji.

Przeciwieństwem jest heteroskedastyczność - sytuacja gdy wariancja reszt zmienia się systematycznie wraz ze zmianą wartości zmiennej niezależnej.

<img src="Assets/homo_hetero_skedastycznosc.png"/>

### 3. Co to jest rozkład normalny danych?

Zwany też rozkładem Gaussa. Charakteryzuje się symetrycznym rozkładem wokół średniej, 
co oznacza, że wartości powyżej i poniżej średniej mają równą częstość występowania.
Ma kształt dzwonu.

<img src="Assets/rozkladnormalny.png"/>

### 4. Wariancja

Opisuje jak bardzo dane różnią się od średniej wartości zbioru.
np 

- 50, 50, 50, 50 - średnia 50, wariancja 0
- 20, 40, 60, 80 - średnia 50, wariancja duża

## Podsumowanie

<img src="Assets/podsumowanie.png"/>

## Przykłady testów
### Porównywanie średnich dwóch grup (test t-Studenta)

**Cel**: Sprawdzenie, czy dwie grupy różnią się pod względem średniej jakiejś zmiennej.

**Przykład**: Analiza wyników sprzedaży przed i po wprowadzeniu nowej strategii marketingowej.

- Hipoteza zerowa ($H_0$): Średnia sprzedaż przed i po wprowadzeniu strategii jest taka sama.
- Hipoteza alternatywna($H_1$): Średnia sprzedaż po wprowadzeniu strategii różni się od sprzedaży wcześniejszej.

### Testowanie proporcji (test Z dla proporcji)
**Cel**: Sprawdzenie, czy udział jakiejś kategorii w populacji zmienia się w czasie.

**Przykład**: Analiza udziału klientów korzystających z danej promocji.

- $H_0$: Proporcja klientów korzystających z promocji wynosi 30%.
- $H_1$: Proporcja klientów korzystających z promocji różni się od 30%.

### Analiza wariancji (ANOVA)
**Cel**: Porównanie średnich więcej niż dwóch grup.

**Przykład**: Sprawdzenie, czy różne kanały reklamowe (TV, social media, radio) mają różny wpływ na wyniki sprzedaży.

- $H_0$: Średnie sprzedaży dla różnych kanałów reklamowych są takie same.
- $H_1$: Średnie sprzedaży różnią się między kanałami.

### Test Chi-kwadrat dla niezależności
**Cel**: Sprawdzenie, czy dwie cechy są ze sobą powiązane.

**Przykład**: Analiza związku między płcią klienta a preferencją zakupu (np. wybór kategorii produktu).

- $H_0$: Płeć klienta i preferencja zakupu są niezależne.
- $H_1$: Płeć klienta i preferencja zakupu są powiązane.
