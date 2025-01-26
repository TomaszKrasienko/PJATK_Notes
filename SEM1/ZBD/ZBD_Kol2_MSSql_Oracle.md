# ORACLE
### Utworzenie indeksu
```sql
CREATE INDEX nazwa ON tabela (kolumna);
```
### Utworzenie bitmapowego indeksu
```sql
CREATE BITMAP INDEX nazwa ON tabela (kolumna);
```
### Utworzenie indeksu funkcyjnego 
```sql
CREATE INDEX t ON test (wartosc2 + id);
```
### Utworzenie tabeli “index-organised table”
```sql
CREATE TABLE test (Id INTEGER PRIMARY KEY, Wartosc INTEGER, Wartosc2 INTEGER) 
ORGANIZATION INDEX;
```
### Utworzenie klastra hashowanego
```sql
CREATE CLUSTER empt_depnto (deptno INTEGER) HASHKEYS71;
```
### Utworzenie klastra indeksowego dwóch tabel ##
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
### Utworzenie perspektywy zmaterializowanej dla tabeli
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
### Partycjonowanie utworzenia tabel
```sql
CREATE TABLE dzial (
	IdDzial INTEGER PRIMARY KEY,
	Nazwa VARCHAR2(10));
CREATE TABLE pracownik (
	IdPracownik INTEGER PRIMARY KEY,
	Nazwisko VARCHAR2(20),
	IdDzial INTEGER REFERENCES dzial)
PARTITION BY LIST (IdDzial) (
  PARTITION p1 VALUES (1, 2, 3) TABLESPACE studenci,
  PARTITION p2 VALUES (4, 5, 6) TABLESPACE studenci,
  PARTITION p3 VALUES (7, 8, 9, 10) TABLESPACE studenci
);
```
