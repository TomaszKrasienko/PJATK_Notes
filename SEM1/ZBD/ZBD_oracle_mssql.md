# Oracle

## Utworzenie tabeli

```sql
CREATE TABLE test (Id INTEGER, Wartosc INTEGER, Wartosc2 INTEGER);
```

## Zasilenie danymi

```sql
DECLARE
  licz INTEGER;
  wartosc INTEGER;
  wartosc2 INTEGER;
BEGIN
  FOR licz IN 1..100000
  LOOP
    SELECT ROUND(dbms_random.value(1,100000),0) INTO wartosc FROM dual;
    SELECT ROUND(dbms_random.value(1,10),0) INTO wartosc2 FROM dual;
    INSERT INTO test (Id, Wartosc, Wartosc2) VALUES (licz, wartosc, wartosc2);
  END LOOP;
END;

```

## Prosty select

```sql
SELECT * FROM test WHERE wartosc = 12345;
```

## Tworzenie prostego indeksu

```sql
CREATE INDEX nazwa ON tabela (kolumna);
```

Prosty indeks jest dobry do mocno selektywnej kolumny (czyli jak dane są zróżnicowane).

## Tworzenie indeksu bitmapowego

```sql
CREATE BITMAP INDEX nazwa ON tabela (kolumna);
```

Bitmapowy indeks jest dobry do mało selektywnej kolumny (czyli jak dane nie są mocno zróżnicowane).

## Tworzenie indeksu funkcyjnego

```sql
CREATE INDEX t ON test (wartosc2 + id);
```

Sprawdza się gdy mamy w klauzuli `where` lub `order by` konkretne działanie. Na przykład: `wartosc2 + id` , `UPPER(Nazwisko)` , `ROUND(Salary, 2)` 

Przykład zapytania:

```sql
SELECT * FROM test WHERE wartosc2 + id = 1234;
```

## Tworzenie tabeli index-organized

```sql
CREATE TABLE test (Id INTEGER PRIMARY KEY, Wartosc INTEGER, Wartosc2 INTEGER) 
ORGANIZATION INDEX;
```

Jest to dobre do: wyszukiwania zakresowego, odczytywania na podstawie klucza głównego, 

Przykład

```sql
SELECT * FROM test WHERE id BETWEEN 10000 AND 30000;
```

## Klaster indeksowy

Pozwala na grupowanie danych z różnych tabel w jednym fizycznym segmencie danych na podstawie wspólnej kolumny (lub kolumn). Przyspiesza odczyt dotyczących tabel po kolumnie klastrowej, mniejsze obciążenie dysku przy odczycie powiązanych danych. Większy narzut wstawiania danych, klaster może być mniej efektywny przy kolumnach o dużej liczbie unikalnych wartości.

### Tworzenie klastra

```sql
CREATE CLUSTER testowy (IdDzial INTEGER);
```

Najpierw definiujemy klaster.

### Tworzenie indeksu na klastrze

```sql
CREATE INDEX t ON CLUSTER testowy;
```

Następnie tworzymy indeks dla klastra.

### Tworzenie tabel w klastrze

```sql
CREATE TABLE dzial (
    IdDzial INTEGER PRIMARY KEY,
    Nazwa VARCHAR2(10)
)
CLUSTER testowy(IdDzial);

CREATE TABLE pracownik (
    IdPracownik INTEGER PRIMARY KEY,
    Nazwisko VARCHAR2(20),
    IdDzial INTEGER REFERENCES dzial
)
CLUSTER testowy(IdDzial);
```

Tworząc tabele musimy dodać słowo kluczowe `CLUSTER` podając nazwę i kolumnę oraz w tabeli pracownik wskazać `REFERENCES` do tabeli dzial.

## Materialized View

Przechowuje dane fizycznie na dysku. W przeciwieństwie do zwykłego widoku, materialized view nie odświeża się dynamicznie w czasie rzeczywistym - dane przechowywane są w formie zdenormalizowanej i mogą być okresowo odświeżane.

### Tworzenie materialized view

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

Jest on przydatny do: zapytań złożonych i czasochłonnych, szybkich odpowiedzi na zapytania analityczne, rzadko zmieniających się danych, przy replikacji danych.

### Przykład zapytania

```sql
SELECT * FROM pm;
```

Zapytanie jest szybsze ponieważ odczytuje dane bez wykonywania dodatkowych obliczeń.

## Partycjonowanie tabeli

Dzieli tabelę na mniejsze, bardziej zarządzalne części zwane partycjami. Każda partycja przechowywana jest oddzielnie, nie ma to wpływu na to jak użytkownicy i aplikacje widzą dane.

### Tworzenie tabeli dzial

```sql
CREATE TABLE dzial (
    IdDzial INTEGER PRIMARY KEY,
    Nazwa VARCHAR2(10)
);
```

### Tworzenie tabeli pracownik i jej partycjonowanie

```sql
CREATE TABLE pracownik (
    IdPracownik INTEGER PRIMARY KEY,
    Nazwisko VARCHAR2(20),
    IdDzial INTEGER REFERENCES dzial
)
PARTITION BY LIST (IdDzial) (
    PARTITION p1 VALUES (1, 2, 3) TABLESPACE studenci,
    PARTITION p2 VALUES (4, 5, 6) TABLESPACE studenci,
    PARTITION p3 VALUES (7, 8, 9, 10) TABLESPACE studenci
);
```

Partycjonowanie jest przydatne gdy: tabela jest duża, często filtrowane są dane według określonych wartości, rzadko używane są pewne fragmenty danych. Dodatkowo pozwala poprawić wydajność operacji administracyjnych - aktualizujemy dane czy indeksy tylko na określonych partycjach.

### Przykład zapytania

```sql
SELECT * FROM Pracownik WHERE IdDzial IN (1, 2, 3);
```

Bez partycjonowania przeskanujemy całą tabelę. Z partycjonowaniem odczytujemy dane tylko z partycji p1.

# SQL Server

Podstawowa tabela z danymi

```sql
CREATE TABLE test (
    Id INT IDENTITY, 
    Zawartosc INT, 
    Zawartosc2 INT
);
GO

DECLARE @a INT;
SET @a = 1;

WHILE @a < 100000 
BEGIN
    INSERT INTO test (Zawartosc, Zawartosc2) 
    VALUES (CONVERT(INT, RAND() * 100000), CONVERT(INT, RAND() * 100000));
    SET @a = @a + 1;
END;
GO
```

Bez indeksu jest pełny table scan.

## Indeks nieklastrowany

```sql
CREATE NONCLUSTERED INDEX idx_Zawartosc ON test(Zawartosc);
```

Jest szybciej niż bez indeksu - następuje index seek.

## Tylko indeks

```sql
CREATE NONCLUSTERED INDEX idx_Zawartosc_Zawartosc2 
ON test(Zawartosc, Zawartosc2);
```

### Przykładowe zapytanie

```sql
SELECT Zawartosc, Zawartosc2 FROM test WHERE Zawartosc = 12345;
```

### Wyszukiwanie zakresowe

### Zapytanie

```sql
SELECT * FROM test WHERE Zawartosc BETWEEN 10000 and 20000;
```

Stworzenie indeksu nieklastrowanego nic nie zmieni, sql z niego nie skorzysta. Należy stworzyć indeks klastrowany.

```sql
--CREATE NONCLUSTERED INDEX idx_Zawartosc ON test(Zawartosc);
CREATE CLUSTERED INDEX idx_Zawartosc ON test(Zawartosc);
```

## Indeksy przy order by

### Zapytanie

```sql
SELECT * FROM test ORDER BY Zawartosc;
```

Stworzenie indeksu nieklastrowanego nie pomoże, sql z niego nie skorzysta. Tak jak w zakresowym należy stworzyć indeks klastrowany.

```sql
--CREATE NONCLUSTERED INDEX idx_Zawartosc ON test(Zawartosc);
CREATE CLUSTERED INDEX idx_Zawartosc ON test(Zawartosc);
```

### Depth indeksu

Table > Indexes > Index > Properties > Fragmentation

Dla inta - 2

Dla varchar(500) - 5

## Indeksy na kluczach obcych

Tabele

```sql
CREATE TABLE test (
    Id INT PRIMARY KEY,
    Zawartosc VARCHAR(10)
);

CREATE TABLE test2 (
    Id INT IDENTITY PRIMARY KEY,
    Zawartosc INT,
    IdTest INT REFERENCES test(Id)
);
```

Zapytanie

```sql
SELECT * 
FROM test
INNER JOIN test2 ON test2.IdTest = test.Id;
```

Indeks nieklastrowany nie zmienił nic. Trzeba zamienić indeks klastrowany z id na idTest.

```sql
--CREATE NONCLUSTERED INDEX idx_test2_IdTest ON test2(IdTest);

ALTER TABLE [dbo].[test2] DROP CONSTRAINT [PK__test2__3214EC0732409DC6] WITH ( ONLINE = OFF )
ALTER TABLE [dbo].[test2] DROP CONSTRAINT [FK__test2__IdTest__4E88ABD4]

ALTER TABLE test2
ADD CONSTRAINT PK_test2 PRIMARY KEY NONCLUSTERED (Id);
CREATE CLUSTERED INDEX idx_test2_IdTest ON test2(IdTest);
```

## Współczynnik fillfactor

Tworzymy tabelę, wypełniamy, tworzymy indeks z fillfactorem ustawionym na 100% (nie zostanie zachowane miejsce na przyszłe inserty).

```sql
CREATE NONCLUSTERED INDEX idx_zawartosc_100
ON test (Zawartosc)
WITH (FILLFACTOR = 100);
```

Sprawdzenie rozmiaru indeksu i liczby stron

```sql
SELECT 
    object_name(i.object_id) AS table_name,
    i.name AS index_name,
    ps.index_type_desc,
    ps.page_count * 8 AS index_size_kb,
    ps.page_count AS page_count
FROM 
    sys.indexes i
    INNER JOIN sys.dm_db_index_physical_stats(NULL, NULL, NULL, NULL, 'DETAILED') ps
        ON i.object_id = ps.object_id
        AND i.index_id = ps.index_id
WHERE 
    i.name = 'idx_zawartosc_100';
```

Sprawdzenie fragmentacji - Lub index > properties > Fragmentation

```sql
DBCC SHOWCONTIG('test', 'idx_zawartosc_100');
```

Fragmentacja po insercie kolejnych 100k rekordów wzrosła z 4% na 99%. Dane stały się bardziej rozproszone i nierówno rozmieszczone po stronach indeksu.

Tworzymy nowy indeks z mniejszą fragmentacją

```sql
CREATE NONCLUSTERED INDEX idx_zawartosc_50
ON test (Zawartosc)
WITH (FILLFACTOR = 50);
```

Fragmentacja pozostała na poziomie 1% pomimo zrobienia inserta kolejnych 100k rekordów. 

Wniosek: duży fillfactor pozwala zaoszczędzić miejsce ale zwiększa fragmentację przy insertach, mały fillfactor zabiera więcej miejsca ale fragmentacja nie zwiększa się aż tak przy insertach.

### Jawne wskazanie ręcznych transakcji (każde okno oddzielnie)

```sql
SET IMPLICIT_TRANSACTIONS ON
```

### Przy ręcznych transakcjach trzeba pamiętać żeby na koniec wykonać (każde okno oddzielnie)

```sql
COMMIT
```

### Ustawianie poziomów izolacji (przed ustawieniem najlepiej zrobić commit albo rollback)

```sql
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

SET TRANSACTION ISOLATION LEVEL SNAPSHOT;
-- jak nie działa snapshot to trzeba zrobić
ALTER DATABASE DbName SET ALLOW_SNAPSHOT_ISOLATION ON;

SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

```

|  | Komenda | Ochrona przed: |
| --- | --- | --- |
| **READ UNCOMMITTED** | `SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;` | Niczym, brudne dane |
| **READ COMMITTED** | `SET TRANSACTION ISOLATION LEVEL READ COMMITTED;` | Brudne dane |
| **REPEATABLE READ** | `SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;` | Brudne i niepowtarzalne odczyty |
| **SNAPSHOT** | `SET TRANSACTION ISOLATION LEVEL SNAPSHOT;` | Brudne i niepowtarzalne odczyty, wymaga wersjonowania danych |
| **SERIALIZABLE** | `SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;` | Brudne dane, niepowtarzalne odczyty, phantom reads |