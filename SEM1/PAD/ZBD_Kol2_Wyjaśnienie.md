# SQL Server

## Indeks pogrupowany

- B+drzewo
- Na poziomie liści całe wiersze (uporządkowane)
- Używać przy:
    - Zapytaniach z zakresami w WHERE
    - Zapytaniach z sortowaniem, agregowaniem (ORDER BY, GROUP BY)
    - Zapytania z WHERE z wartością klucza
- Dostępne również “included columns” dla indeksu pogrupowanego
    - Używać przy:
        - zapytania wymagające unikalności na kombinacji kolumn
    - Baza danych sortuje według kombinacji kolumn (najpierw pierwsza kolumna i potem reszta)
- Nie używać przy:
    - Częstych modyfikacja i aktualizacją danych (patrz punkt niżej)
    - Tabele z operacjami na różnych kolumnach (indeks na jednej kolumnie nic nie da)
    - Gdy dane muszą być dostępne w różnych porządkach
- Dodawanie wierszy jest wtedy wydłużone gdy dodajemy je niezgodnie z IDENTITY(1,1),
natomiast przy zachowaniu kolejności nie jest to operacja różniąca się od zwykłego dodania wierszy bez indeksu - rekord jest dodawany na końcu

```sql
CREATE TABLE (..) 
CREATE CLUSTERED INDEX nazwa ON tabela (kolumny)
```

## Indeks niepogrupowany

- B+drzewo
- Używać przy:
    - Zapytaniach punktowych
    - Zapytań z filtrami na różnych kolumnach
    - Indeksy kolumn, które nie są kluczem głównym
    - Do pokrycia zapytań (Strategia tylko index)
    - Zapytania z JOIN
- Nie używać gdy
    - Zapytania powinny być posortowane fizycznie
    - Duża ilość dodawanych rekordów i modyfikacji

```sql
CREATE NONCLUSTERED INDEX [nazwa_index'u] ON [tabela_nazwa]([kolumna_nazwa]);
```
## Partycjonowanie tabel

Podzielenie tabeli fizycznie na kilka części. Podział horyzontalny (wiersze). 

- Szybszy dostęp do danych, ponieważ zbiory są mniejsze
- Przyspieszenie wykonywania zapytań odwołujących się do podzbiorów rekordów
- Zalecane dla bardzo dużych tabel
- Wymaga utworzenia funkcji partycjonującej i schematu partycjonowania
- Wyłącznie zakresowe

## Indeksowane widoki

- **Indeksowane widoki** w SQL Server pozwalają na zapisanie wyników zapytania w bazie danych. Aby widok stał się indeksowany, należy założyć na nim **indeks pogrupowany** (clustered index). Indeksowany widok przechowuje dane fizycznie i pozwala na ich szybszy dostęp. Jednak w przypadku wstawiania, aktualizowania lub usuwania danych w tabelach, które są powiązane z widokiem, SQL Server automatycznie utrzymuje dane widoku.
- **Ograniczenia**: Aby stworzyć indeksowany widok, widok musi spełniać określone wymagania, takie jak brak funkcji agregujących (np. `COUNT`, `SUM`), brak `DISTINCT`, oraz brak podzapytań.

```sql
-- Tworzenie widoku
CREATE VIEW MyView AS
SELECT Dzial.Nazwa, COUNT(*) 
FROM Dzial
INNER JOIN Pracownik ON Pracownik.IdDzial = Dzial.IdDzial
GROUP BY Dzial.Nazwa;

-- Tworzenie indeksu na widoku
CREATE UNIQUE CLUSTERED INDEX idx_MyView ON MyView(Nazwa);

```
# ORACLE

## Tabela organizowana indeksem

- B+drzewo
- Na poziomie liści całe wiersze (uporządkowane)
- Używać:
    - Gdy tabela ma być uporządkowana według klucza głównego
    - Dla dużej ilości zapytań SELECT z użyciem klucza głównego
    - Gdy zapytania bazują na podstawie klucza głównego z klauzulami WHERE i ORDER BY
    - Gdy tabela posiada unikalne identyfikatory, które są często używane
- Nie używać
    - gdy ilość kolumn jest duża
    - częsta modyfikacja tabel
    - częste zapytania wykorzystujące inne kolumny niż klucz główny
- Tylko dla klucza głównego

```sql
CREATE TABLE table (..) ORGANIZATION INDEX
```

## Index bitmapowy

- do mało selektywnej kolumny - powtarzają się dane

```sql
CREATE BITMAP INDEX nazwa ON tabela (kolumna);
```

## Index funkcyjny (ORACLE)

- przydatny gdy WHERE lub ORDER BY często będzie wyliczany za pomocą deterministycznej funkcji

```sql
CREATE INDEX t ON test (wartosc2 + id);
```

## Klastry indeksowe

- Wiele tabel przechowywanych w jednym segmencie
- Przydatne, gdy tabele są odczytywane razem
- Znaczne pogorszenie wydajności aktualizacji

![image.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e5d54cb1-c459-4ba8-9baa-5376fe0ae6db/14df398e-cdbc-434b-9aa9-dc27ecb4584f/image.png)

```sql
CREATE CLUSTER testowy (IdDzial INTEGER);
CREATE INDEX t ON CLUSTER testowy;
CREATE TABLE dzial (
	IdDzial INTEGER PRIMARY KEY,
	Nazwa VARCHAR2(10))
CLUSTER testowy(IdDzial);
CREATE TABLE pracownik (
	IdPracownik INTEGER PRIMARY KEY,
	Nazwisko VARCHAR2(20),
	IdDzial INTEGER REFERENCES dzial)
CLUSTER testowy(IdDzial);
```

## Klastry hashowane (ORACLE)

- zamiast indeksu wyszukuje funkcję hashującą
- dane w klastrze są przechowywane na podstwie jej wyniku
- funkcja hashująca zwraca liczbę, która jednoznacznie wyznacza miejsce, gdzie przechowywane są dane
- **dobre dla warunków równościowych**

## Partycjonowanie tabel

Podzielenie tabeli fizycznie na kilka części. Podział horyzontalny (wiersze). 

- Szybszy dostęp do danych, ponieważ zbiory są mniejsze
- Przyspieszenie wykonywania zapytań odwołujących się do podzbiorów rekordów
- Zalecane dla bardzo dużych tabel
- Definiujemy przy tworzeniu tabeli
- Zakresowe, Listowe, Hashowane, Mieszane

## Perspektywy zmaterializowane

W ORACLE perspektywy zmaterializowane są dostępne.

```sql
CREATE MATERIALIZED VIEW pm
BUILD IMMEDIATE
REFRESH COMPLETE
ON COMMIT
WITH PRIMARY KEY
AS
SELECT Dzial.Nazwa, COUNT(*) FROM Dzial
INNER JOIN Pracownik ON Pracownik.IdDzial = Dzial.IdDzial
GROUP BY Dzial.Nazwa
ORDER BY 2

```
