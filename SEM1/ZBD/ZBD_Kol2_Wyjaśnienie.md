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
