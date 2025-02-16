## **1. Trzy poziomy abstrakcji schematu bazy danych i niezależność danych**

### Poziomy abstrakcji

- schemat logiczny - definiuje strukturę logiczną bazy danych
- schemat fizyczny - opisuje używane dyski, katalogi, pliki i rekordy
- schematy zewnętrzne - opisują jak aplikacje i użytkownicy widzą i używają dane

### Niezależność danych

Dane w aplikacji powinny być odizolowane od tego, jaką logiczną strukturę mają dane w bazie danych i jak są składowane na dysku 

- Logiczna niezależność - schematy danych w aplikacji są niezależne od zmian w logicznym schemacie bazy danych
- Fizyczna niezależność - schematy danych w aplikacji są niezależne od zmian w fizycznym schemacie bazy danych

---

## **2. Użycie perspektyw do realizacji zewnętrznego poziomu w bazie danych**

Na poziomach zewnętrznych należy używać perspektyw i funkcji/procedur składowanych - nie tabel.

Ma to na celu
- zapewnienie spójnego, bezpiecznego i elastycznego dostępu do danych
- Ukrycie szczegółów implementacyjnych i struktury bazy

---

## **3. Instrukcje SQL z klauzulą WITH**

Opcja “WITH CHECK OPTION” przy **INSERT** i **UPDATE** - następuje sprawdzenie, czy wstawiany bądź modyfikowany wiersz spełnia warunek w klauzuli **WHERE**

- Jeśli spełnia, operacja jest wykonywana
- Jeśli nie spełnia, operacja nie zostanie wykonana

*Perspektywa pracowników będących na urlopie bezpłatnym*

```sql
CREATE VIEW Emp_na_urlopie_bezpłatnym
AS SELECT *
FROM Emp
WHERE **sal = 0 OR sal IS NULL**
WITH CHECK OPTION;
```

Poprzez tę perspektywę nie uda się zmienić wysokości zarobków pracownika o nazwisku ‘Kowalski’, którego zarobki są równe 0 lub *Null*

```sql
UPDATE Emp_na_urlopie_bezpłatnym
SET sal = 10000
WHERE ename = 'KOWALSKI';
```

*Lokalne perspektywy (inline views) (albo CTE)*
Lokalne perspektywy to tymczasowe widoki definiowane za pomocą klauzuli WITH, widoczne tylko w obrębie jednej instrukcji SQL. Umożliwiają modularne programowanie w SQL, upraszczając złożone zapytania.

```sql
WITH SredniePensje AS (
    SELECT Dzial, AVG(Pensja) AS SredniaPensja
    FROM Pracownicy
    GROUP BY Dzial
)
SELECT p.Imie, p.Dzial, p.Pensja
FROM Pracownicy p
JOIN SredniePensje sp ON p.Dzial = sp.Dzial
WHERE p.Pensja > sp.SredniaPensja;
```

With można użyć też w zapytaniu **SELECT**

```sql
SELECT ID, Imie, Pensja 
FROM Pracownicy WITH (NOLOCK) 
WHERE Pensja > 5000;

```
Dostępne opcje:
- NOLOCK – pozwala czytać dane bez blokowania (mogą być niezatwierdzone zmiany).
- HOLDLOCK – działa jak SERIALIZABLE, blokuje cały zakres danych do końca transakcji.
- UPDLOCK – blokada aktualizacyjna (przydatne w SELECT ... FOR UPDATE).
- TABLOCK – blokada całej tabeli.
- READPAST – pomija zablokowane wiersze.
- ROWLOCK – blokuje tylko pojedyncze wiersze.

---

## 4. Katalog (słownik) systemowy (danych)

Metadane czyli dane o schematach obiektów bazy danych.

- Ma postać zbioru tabel i perspektyw
- Z przedrostkiem **User** - informacje o schematach wszystkich obiektów, których dany użytkownik jest właścicielem
- z przedrostkiem All - dotyczy wszystkich obiektów, do których użytkownik ma uprawnienia
- z przedrostkiem Dba - informacje o obiektach dostępne tylko dla administratorów.

---

## 5. Wyzwalacze w bazie danych

Wiązane z użyciem jednego z obiektów: tabeli, perspektywy, schematu, bazy danych.

Wywoływane przez system przy zajściu odpowiedniego zdarzenia, które może być zdarzeniem systemowym abo realizacją na obiekcie jednej z instrukcji SQL.

---

## 6. Obiektowo-relacyjny model danych w bazie danych Oracle

*(Z chata)* 
- Możliwość utworzenia kolumny z typem obiektowym, który umożliwia tworzenie złożonych struktur.

- Chodzi o to że możemy tworzyć własne typy danych. Jest tu też dziedziczenie i inne dziwne opcje ale nie ma co się zagłębiać:
``` sql
CREATE TYPE Adres AS OBJECT (
    Ulica VARCHAR2(100),
    Miasto VARCHAR2(50),
    Kod_Pocztowy VARCHAR2(10)
);

CREATE TABLE Klienci (
    ID NUMBER PRIMARY KEY,
    Imie VARCHAR2(50),
    DaneAdresowe Adres  -- Kolumna przechowująca obiekt typu Adres
);



```
- W mssql tego nie ma.

---

## 7. Duże obiekty LOB i dokumenty XML w relacyjnej bazie danych

### Obiekty LOB

- Mogą wystąpić jako:
    - atrybut typu obiektowego
    - kolumna w tabeli
- Typ BFILE w Oracle
- Typ FILESTREAM w SQL Server
- Przechowywane w samej bazie danych lub na zewnątrz bazy danych.
- Na ogół w wierszu tabeli jest zapisywany tylko wskaźnik do dużego obiektu lob.
- Mogą być trzymane w osobnych obszarach dysku jako ciąg sąsiednich stron

### Typ obiektowy XML

Wbudowany typ obiektowy - najczęstsze zastosowania “obiektów” w bazie danych. Są dla niego specjalne indeksy “ścieżkowe” - umożliwiające wyszukiwanie danych wewnątrz dokumentu XML.

Zastosowania: 

- Zapisanie w bazie danych obok siebie na dysku powiązanych danych pozwala na szybszy dostęp do danych w postaci paczki danych.
- Integracja danych pochodzących z różnych baz danych i aplikacji.
- Ułatwienie wymiany danych biznesowych między aplikacjami i bazami danych, w szczególności między bazą danych i przeglądarką.

---

## **8. Dyskowy model fizyczny bazy danych**

### Dyskowy model fizyczny w ORACLE

- gdy rozmiar rekordu jest większy niż rozmiar strony to rekord dzielony jest na części przechowywane na osobnych stronach
- gdy schemat dostępu do danych polega na użyciu powiązanych danych z dwóch lub więcej tabel to w jednym miejscu na dysku są zbierane dane z kilku tabel w oparciu o wspólny klucz - jak w bazach hierarchicznych

### Dyskowy model fizyczny w MSSQL Server

- ograniczenie rozmiaru każdego rekordu do 8kB
- system stara się trzymać cały rekord na jednej stronie
- jeśli po modyfikacji rekord przestaje mieścić się na stronie, nie mieszcząca się część rekordu zostaje zapisana na liście stron nadmiarowych, a wskaźnik do strony nadmiarowej zostaje zapisany w rekordzie

---

## **9. Zarządzanie miejscem na dysku**

Plik rekordu składa się z obszaru z używanymi ekstentami oraz z dostępem do wolnego obszaru na dysku, z którego można pobierać i alokować nowe ekstenty do pliku rekordów.

Podstawowe realizowane funkcje:

- Alokacja/dealokacja ekstentu
- Odczyt/zapis strony o podanym adresie
- Wyznaczenie strony do zapisu nowego rekordu

Strukturą danych organizującą dostęp do stron i ekstentów w pliku są bitmapy (wektory bitów) zapisywane w pliku razem z danymi. Dotyczy to:

- alokowanych i wolnych ekstentów
- stron zajętych przez dany obiekt bazodanowy

## **10. Zarządzanie buforami (w RAM)**

Proces serwera realizujący zlecenie użytkownika chce uzyskać dostęp do strony z danymi.

- Dane muszą być w RAM, aby serwer mógł na nich operować
- Pomocnicza struktura danych: tablica hashowana po identyfikatorze strony do sprawdzania czy i gdzie strona znajduje się w puli buforów

Dla każdego bufora 

- licznik odwołań - ile transakcji używa strony zapisanej w buforze w danej chwili
    - na początku po umieszczeniu strony w buforze
        - **licznik odwołań := 1,**
    - gdy kolejna transakcja uzyskuje dostęp do strony:
        - **licznik odwołań + 1**
    - gry transakcja zwalnia dostęp do strony:
        - **licznik odwołań - 1**
- bit modyfikacji - czy po sprowadzeniu strony do pamięci zawartość bufora została zmodyfikowana i do tej pory nie zapisana na dysk
    - na początku po umieszczeniu strony w buforze
        - **bit modyfikacji - false**
    - gdy zmienia się zawartość bufora
        - **bit modyfikacji - true**

Strona w buforze może być używana przez wiele procesów:

- gdy proces zwalnia stronę, jej licznik odwołań zmniejsza się o jeden
- strona staje się kandydatem do zastąpienia, gdy jej licznik odwołań = 0
- zapisuje się też informację o rodzaju blokady:
    - do zapisu - może tylko jeden proces
    - do odczytu - może wiele procesów
- gdy strona pozostaje długo nieużywana, system odwołań automatycznie ją zwalnia, ustawiając licznik odwołań na 0
    - potrzebny dodatkowy atrybut dla strony w buforze - kiedy ostatni raz była używana
    - przy czym jeśli strona została zmieniona, system zapisuje zmienioną zawartość na dysk
- przy odczytywaniu wierszy z dużej tabeli bez ich zmieniania, dla sprowadzonych stron (po ich odczytaniu) system ustawia ich licznik odwołań od razu na 0 (zamiast na 1)
- często używane strony są przechowywane w buforach
- zarówno skomplikowane instrukcje SQL jak i wyniki zapytań też są zapisywane w buforach i używane wielokrotnie
- punkt kontrolny (checkpoint) - dodatkowy proces, zmienia bit modyfikacji na false z puli buforów co pewien czas

---

## 11. Formaty rekordów i stron

### Rekordy

**Stała długość pół** - 
Rozmiary pól takie same dla wszystkich rekordów w pliku - zapisane w katalogu systemowym

<img src="Assets/rekord_stala_dlugosc.png"/>

**Zmienna długość pól** - 
Dwa alternatywne formaty (#pól jest stała)

<img src="Assets/rekord_zmienna_dlugosc.png"/>

- W drugim przypadku bezpośredni dostęp do wartości i-tego pola
- Format mieszany - każdy rekord składa się z dwóch części - stałego i zmiennego rozmiaru (SQL Server)

### Strony

<img src="Assets/strony.png"/>

- Identyfikator rekordu rid = id strony, nr rekordu w tablicy pozycji
- Można przesuwać rekordy na stronie aktualizując wartości w tablicy pozycji bez konieczności zmiany identyfikatorów rekordów

---

## 12. Podstawowe organizacje plików bazy danych

### Plik rekordów (segment obiektu bazy danych)

- kolekcja strong, zawierająca zbiór rekordów jednego obiektu bazy danych np. tabeli
- operacje:
    - wstawienie/usunięcie/zmodyfikowanie rekordu
    - odczytanie konkretnego rekordu
    - wyszukanie wszystkich rekordów spełniających podane warunki

### Plik nieuporządkowany (sterta, heap)

- rekordy są przechowywane na stronach bez konkretnego porządku
- nowy rekord jest wstawiany do pierwszej strony, na której jest wolne miejsce
- efektywne wyszukiwanie rekordów gdy dotyczy
    - małej tabeli
    - sporej części większej tabli, np. co najmniej 50% rekordów
- przy wyszukiwaniu rekordów po wartości klucza wyszukiwania stosowany łącznie z indeksami

### Plik uporządkowany

- rekordy zawsze zapisywane na kolejnych stronach zgodnie z porządkiem względem klucza rekordu albo klucza wyszukiwania
- taka reprezentacja jest wygodna, gdy rekordy są przetwarzane zwykle w tym samym, ustalonym porządku lub tylko pewne ich zakresy względem tego porządku
- bardziej skomplikowane stają się operacje wstawienia nowego rekordu do pliku jak i usunięcie rekordu z pliku
- implementacje:
    - spójny obszar stron - sąsiadujących ze sobą na dysku - rekordy uporządkowane wg wartości klucza
        - problem z wstawieniem nowego rekordu
        - możliwość wyszukiwania binarnego
    - lista stron = rekordy uporządkowane wg wartości klucza
        - łatwe wstawienie nowego rekordu do pliku (gdy znane miejsce)
        - nie można bezpośrednio zastosować wyszukiwania binarnego
    - lista stron z indeksem - kompromis - rekordy uporządkowane wg wartości klucza. W indeksie dla każdej strony zapisany jest najmniejszy klucz rekordu i jej adres
        - ułatwione wyszukiwanie.
        - łatwe wstawienie nowego rekordu i usunięcie

### Plik hashowany

- plik jest kolekcją segmentów (ang. buckets), gdzie segment = strona główna oraz zero lub więcej stron nadmiarowych

### Wielowymiarowa tablica (IBM DB2)

- klasyfikacja rekordów względem wartości pól nazywanych wymiarami
- zastosowania - hurtownia danych, dane przestrzenne

---

## 13. Realizacja wyszukiwania rekordów po kluczu wyszukiwania

*z chata*

- Realizacja wyszukiwania rekordów po kluczu wyszukiwania w SQL może być wykonana za pomocą zapytań wykorzystujących indeksowane kolumny, zwłaszcza klucze główne (PRIMARY KEY) i indeksy unikalne (UNIQUE). Najczęściej stosowaną instrukcją jest `SELECT` z warunkiem `WHERE`. Wyszukiwanie może również odbywać się na podstawie kluczy obcych (FOREIGN KEY), jeśli zapytanie łączy dane z różnych tabel

---

## 14. Indeksy pogrupowane i niepogrupowane

### Indeks pogrupowany

- indeks na B+ drzewie dla uporządkowanego pliku rekordów - w liściach znajdują się strony zawierające rekordy danych:
    - rekordy są posortowane wg wartości klucza indeksu
    - dla jednej tabeli - tylko jeden indeks pogrupowany
    - pozostałe indeksy na B+ drzewach - niepogrupowane - liście drzewa nie zawierają stron z rekordami danych - tylko wskaźniki do nich

### Indeks niepogrupowany

- w liściach indeksu niepogrupowanego znajdują się
    - gdy na tabeli nie ma indeksu pogrupowanego - adresy do rekordów
    - gdy na tabeli jest indeks pogrupowany - wartości klucza pogrupowanego

---

## 15. Składowanie danych kolumnami, indeks kolumnowy, indeks bitmapowy

### Składowanie danych kolumnami

- dane są czytane tylko z wybranych kolumn, co zmniejsza liczbę operacji we/wy
- gdy dane powtarzają się w kolumnie, możliwość kompresji i zmniejszenia ilości miejsca do zapisu
- podejście hybrydowe - zapis całej tabeli wierszami na dysku, a w pamięci RAM zapis kilku kolumn do analizy wartości w nich.
- taki zapis kolumnowy jest nazywany indeksem kolumnowym

### Indeks bitmapowy (ORACLE)

- przykład:

<img src="Assets/indeks_bitmapowy.png"/>
    
- zapytania z WHERE sprowadzają się do operacji na wektorach bitowych ( WHERE 10010 AND 10001)

---

## 16. Realizacja selekcji

- Proces wyszukiwania danych, które spełniają określone kryteria w tabeli
- Bez indeksu
    - Przechodzimy przez wszystkie rekordy z tabeli, aby sprawdzić czy wartość w kolumnie jest równa kryteriom
    - Jeśli jest równa to zwracamy wiersz
- Z indeksem
    - Używamy indeksu, aby znaleźć rekordy, które odpowiadają zapytaniu
    - Gdy znajdziemy odpowiednią pozycję w indeksie, przechodzimy do rekordu, aby uzyskać dane z tabel
    - Gdy indeks obejmuje wszystkie atrybuty, których potrzebujemy nie przechodzimy do danych po znalezieniu odpowiednich wierszy, tylko korzystamy z danych w indeksie

---

## 17. Realizacja projekcji

- Odnosi się do wybierania określonych kolumn z tabeli
- Bez indeksu
    - Baza danych musi odczytać wszystkie rekordy w pliku, aby uzyskać wartość kolumn
    - Przejście przez każdy rekord
    - Utworzenie wyniku z odczytanych kolumn
- Z indeksem
    - Jeżeli indeks nie będzie na wszystkich wybranych kolumnach nadal baza danych będzie musiała przeskanować tabelę żeby odczytać wartości
    - Indeks obejmujący wszystkie kolumny sprawi że dane zostaną odczytane z indeksu

---

## 18. Strategia „tylko-indeks”

Baza danych nie skanuje tabeli jeżeli na kolumnach do których odnosi się zapytanie jest założony indeks.

---

## 19. Realizacja złączenia

### Algorytm Nested Loops Join

- Dla każdego wiersza tabeli E przeglądamy wszystkie wiersze Tabeli D
- Stosowany przez serwer, gdy rozmiar jednej z tabel jest niewielki, pozwalający zapisać ją całą w pamięci
- Każda strona z wierszami z drugiej tabeli jest sprowadzana do pamięci tylko raz

### Algorytm Index Nested Loops Join

- Dla każdego wiersza tabeli E przeglądamy indeks i wyszukujemy potrzebnej wartości

### Algorytm Sort-Merge Join

- Posortuj wiersze w tabelach D i E według wartości w kolumnach złączenia, następnie scal odpowiadające sobie wiersze w D i E
- Przy scalaniu każda z posortowanych tabel jest przeglądana liniowo tylko raz

### Algorytm Hash-Join

- Podziel tabele D i E na partycje względem wartości funkcji hashującej na kolumnach złączenia
- Wiersze z tabeli D w partycji “i” złączaj z wierszami tabeli E w partycji “i”

---

## 20. Użycie perspektyw zmaterializowanych

Perspektywa, której wynik jest zapisywany w postaci tabeli

- Można na niej założyć indeksy
- Można raz wyliczyć pracochłonne złączenia i statystyki
- Potrzebne jest odświeżanie zawartości po operacjach zmieniających wiersze w tabelach źródłowych

---

## 21. Optymalizacja zapytań, drzewo zapytania, plan wykonania zapytania

- Drzewo zapytania -> parser buduje, hierarchinczna reprezentacja zapytania która pokazuje kolejność stosowania operatorów relacyjnych
- Plan zapytania -> optymializator, instrukcja zawierająca szczegóły dotyczące sposobu realizacji zapytania przez bazę. Określa: drzewo wskazujące kolejność, sposób dostępu do rekordów na dysku, metody realizacji operatorów relacyjnych
- Plan wykonania zapytania to kompleksowa instrukcja, która poza hierarchicznym drzewem operatorów relacyjnych, określa szczegółowo:
    - Sposób dostępu do danych (indeksy, skanowanie tabel),
    - Metody łączenia i przetwarzania danych (nested loops, hash join, merge join),
    - Szacunkowe koszty operacji oraz przewidywaną liczbę przetwarzanych wierszy,
    - Ewentualne mechanizmy równoległości.

---

## 22. Równoległość w serwerze baz danych

- **Obiekty dzielone na partycje**
    - Operacje na partycjach wykonywane równolegle
    - Wyszukiwanie ograniczone do jednej partycji
- **Równoległość na poziomie systemu operacyjnego**
    - Wątki, procesy, macierze dyskowe
        - W tym procesy serwera
- **Klaster instancji serwera**
    - Jedna baza danych, wiele instancji serwera działających na niej równolegle
- **Rozproszona baza danych - na zbiorze połączonych ze sobą serwerów baz danych**
    - Duży poziom równoległości obliczeń
    - Dobra skalowalność
    - Potencjalne kłopoty z synchronizacją obliczeń

---

## 23. Architektura serwera bazy danych Oracle

Serwer bazy danych ORACLE składa się z:

- bazy danych - zbioru plików dyskowych, w których są składowane dane, indeksy i rekordy dziennika transakcji
- instancji - struktur danych w pamięci RAM i procesów dwóch typów
    - realizujących zlecenia użytkowników
    - wykonujących cyklicznie działania serwera jak zapisywanie zmienionych danych na dysk
    
<img src="Assets/server_oracle.png"/>

---

## 24. Aksjomaty ACID współbieżnego wykonywania transakcji

### Atomic (Atomowość)

Cała operacja jest traktowana jako pojedyncza jednostka.

### Consistent (Spójność)

Zakończona operacja pozostawi w bazę danych w spójnym stanie wewnętrznym

### Isolated (Izolacja)

Wynik działania transakcji powinien być taki sam, jakby od chwili rozpoczęcia transakcji nie działała na wspólnych danych żadna inna transakcja - użytkownik ma poczucie jakby sam korzystał z bazy danych.

### Durability (Trwałość)

Wyniki transakcji są stale przechowywane w systemie, nawet w przypadku awarii oprogramowania czy sprzętu.

---

## 25. Mechanizmy współdzielenia zasobów bazy danych

### Blokady

Ograniczają działanie innych transakcji na zablokowanym obiekcie

### Dzienniki transakcji

Opis działania transakcji - w oparciu o ten opis można zarówno wycofać transakcję jak i powtórzyć jej wykonanie w przypadku awarii.

### Kopia bazy danych

Wykonywana w regularnych odstępach czasu, np. raz na dzień

### Mechanizm wielowersyjności

Dostęp do różnych czasowych wersji tych samych danych - umożliwia odczytywane danych zmienianych równocześnie przez inne transakcje w takiej postaci, w jakiej istniały w chwili rozpoczynania się danej instrukcji lub transakcji 

---

## 26. Protokół ścisłego blokowania dwufazowego (2PL)

- Każda transakcja musi uzyskać blokadę S (współdzieloną) na obiekcie zanim odczyta ten obiekt oraz blokadę X (wyłączną) na obiekcie przed zapisaniem go.
- Jeśli transakcja trzyma blokadę X na obiekcie, żadna inna transakcja nie może założyć żadnej innej blokady (ani S ani X) na tym obiekcie
- Jeśli transakcja trzyma blokadę S na obiekcie, żadna inna transakcja nie może założyć blokady X na tym obiekcie, może założyć tylko blokadę S
- Wszystkie blokady trzymane przez transakcje są zwalniane gdy transakcja kończy się
- Dwie fazy działania transakcji
    - zakłada blokady i dokonuje wymaganych odczytów/zapisów na obiektach
    - wykonuje Commit/Rollback i zwalnia wszystkie blokady

---

## 27.Problem fantomów

Inaczej (duch) - wiersz, którego nie było w tabeli na początku wykonywania transakcji obliczającej zapytanie na tabeli, a który został wprowadzony przez inną zatwierdzoną transakcję w trakcie wykonywania rozważanej transakcji.

---

## 28. Poziomy izolacji

### Read uncommited

Umożliwienie innej transakcji czytania danych niezauktualizowanych/niezatwiedzonych. Dane blokowane tylko podczas update’u. Brak ustawionych blokad na danych przygotowywanych do zmiany.

### Read commited

Transakcje mogą odczytywać tylko dane, które zostały zatwierdzone przez inne transakcje. Zapewnia to, że transakcje nie będą odczytywać brudnych danych.
Może dochodzić do niespójnych odczytów, kiedy ta sama transakcja odczytuje różne wartości tego samego rekordu.

### Repeatable read

Gwarantuje, że wszystkie odczyty wykonane w ramach jednej transakcji będą takie same. oznacza to, że dane odczytane przez transakcję nie mogą być zmienione przez inną transakcję do zakończenia tej pierwszej.
Może dochodzić do phantom read, czyli nowo dodane wiersze mogą być widoczne podczas kolejnych operacji.

### Serializable

Najwyższy poziom izolacji, w którym transakcje są wykonywane tak, jakby były wykonane sekwencyjnie, jedna po drugiej. Żadne transakcje nie mogą wprowadzać zmian w danych, które są używane przez daną transakcję.

---

## 29. Wielowersyjność w bazie danych (MVCC)

(Z chata) Technika zarządzania współbieżnym dostępem do danych, która pozwala na przechowywanie wielu wersji tych samych danych w celu zapewnienia spójności i izolacji transakcji, minimalizując jednocześnie ryzyko blokad i zapewniając wysoką wydajność.

---

## 30. Dziennik transakcji

- rejestruje wszystkie zmiany zachodzące w bazie danych
- dokonuje się regularnie jego archiwizacji (Backup)
- zmiana danych na stronie dyskowej i informacja o zatwierdzeniu są najpierw zapisywane do dziennika transakcji, dopiero potem uwzględniane w pliku danych na dysku

---

## 31. Realizacja odtwarzania po awarii serwera

*z chata*

- Odtwarzanie po awarii serwera polega na wykorzystaniu mechanizmu **Write-Ahead Logging (WAL)** (zmiany w bazie danych są najpierw zapisywane w dzienniku transakcji, a dopiero potem w plikach danych na dysku). W przypadku awarii, system bazodanowy stosuje techniki **roll-forward recovery i redo/undo**, aby przywrócić spójny stan bazy. Dodatkowe zabezpieczenia, takie jak replikacja, snapshoty i backupy, zwiększają niezawodność systemu.

- Proces odtwarzania po awarii:
    - Analiza dziennika transakcji – sprawdzenie, które operacje zostały zatwierdzone, a które były w trakcie wykonania.
    - Powtórzenie zatwierdzonych operacji (redo) – odtworzenie zmian, które były zapisane w dzienniku transakcji, ale jeszcze nie znalazły się w bazie.
    - Cofnięcie niezakończonych operacji (undo) – anulowanie transakcji, które nie zostały zatwierdzone przed awarią.

---

## 32. Realizacja odtwarzania po awarii dysku

- Gdy nastąpi awaria dysku z danymi, rekordy dziennika transakcji zastosowane do backupu bazy danych pozwalają odtworzyć stan bazy danych w chwili awarii
- W dzienniku transakcji są również zapisane informacje potrzebne do wycofania transakcji, można wycofać niezatwierdzone transakcje, których działanie zostało przerwane z powodu awarii

---

## 33. Rezerwowa baza danych (standby)

- Dodatkowa instalacja bazy danych na osobnym komputerze. 
- Odtwarzanie danych na bieżąco z kopii dziennika transakcji generowanego przez główną bazę.
- Może służyć do zapewnienia wydajności poprzez obsługę zapytań SELECT, raportów i analiz.
- W przypadku awarii dysku lub katastrofy dotyczącej głównej bazy danych, rezerwowa baza danych przechodzi z trybu **stand-by** w tryb **read-write** i przejmuje obowiązki głównej bazy danych.
- Może być użyta podczas prac serwisowych nad główną bazą.

---

## 34. Protokół dwu-fazowego zatwierdzania (2PC)

- Cel: Zapewnienie atomowości transakcji rozproszonych.
- Koordynator: Węzeł inicjujący transakcję, zarządza procesem zatwierdzania.

### Fazy:

#### Faza przygotowania (Prepare):

- Koordynator wysyła do węzłów komunikat prepare.
- Węzły zapisują w dzienniku rekord prepare i odpowiadają yes (gotowość) lub no (brak gotowości).

#### Faza zatwierdzenia (Commit/Abort):

- Jeśli wszystkie węzły odpowiedzą yes, koordynator zapisuje commit w dzienniku i wysyła komunikat commit.
- Jeśli którykolwiek węzeł odpowie no, koordynator zapisuje abort i wysyła abort.
- Węzły zapisują commit/abort w dzienniku, wykonują odpowiednie działania (zatwierdzają lub wycofują zmiany) i wysyłają potwierdzenie ack.
- Koordynator kończy transakcję po otrzymaniu wszystkich ack, zapisując end w dzienniku.

#### Kluczowe cechy:

- Dwie rundy komunikacji: głosowanie (prepare) i kończenie (commit/abort).
- Liberum veto: Każdy węzeł może zablokować transakcję, odpowiadając no.
- Odporność na awarie: Decyzje są zapisywane w dzienniku przed wysłaniem komunikatów.
- Domyślne wycofanie: Brak rekordu w pamięci RAM oznacza wycofanie transakcji.

#### Wady:

- Blokady w przypadku awarii koordynatora lub węzłów.
- Złożoność komunikacyjna i czasowa.

---

## 35. Realizacja rozproszonych danych w Oracle

- Oracle dostarcza oprogramowanie sieciowe  umożliwiające komunikację między bazami danych oraz obsługę transakcji działających na więcej niż jednej bazie danych (w tym zatwierdzanie transakcji i ich wycofywanie).
- Opcja Oracle Sharding wspomaga tworzenie rozproszonej bazy danych na zbiorze węzłów sieciowych zawierających serwery baz danych w tym dodawanie nowych węzłów (tabele są partycjonowane i/lub replikowane między węzłami).

---

## 36. Problem integracji danych

- Problem integracji danych polega na połączeniu i spójnym wykorzystaniu danych pochodzących z różnych źródeł, które mogą różnić się pod wieloma względami.

### Główne przyczyny problemu:

- Różne lokalizacje danych: Powiązane dane są przechowywane w różnych miejscach, ale mogą być potrzebne jednocześnie przez jedną aplikację.
- Potrzeba integracji po połączeniu firm: Np. scalenie baz danych po fuzji lub przejęciu przedsiębiorstw.

- Różnorodność baz danych:
    - Modele danych: relacyjne, obiektowo-relacyjne, hierarchiczne, XML, pliki Excel itp.
    - Schematy: znormalizowane vs. nieznormalizowane.
    - Terminologia: np. różne definicje "pracownika" (konsultanci, emeryci).
    - Konwencje: różne jednostki miary (Celsjusz vs. Fahrenheit, mile vs. kilometry).

### Rozwiązanie:

- Rozproszona baza danych: Integruje dane z wielu źródeł, zapewniając jednolity dostęp i spójność.
- Narzędzia integracyjne: ETL (Extract, Transform, Load), middleware, systemy zarządzania metadanymi.

---

## 37. Zapytania temporalne (retrospektywne)

- Zapytania temporalne umożliwiają dostęp do danych według stanu bazy danych z przeszłości. Pozwalają na odczytanie informacji, które zostały zmienione lub usunięte.  

#### Zastosowanie: 
- **Analiza danych:** Badanie historycznych stanów danych.  
- **Naprawa błędów:** Odzyskiwanie danych błędnie wprowadzonych, zmienionych lub usuniętych.  

**Przykład (Oracle):**  
Aby wyświetlić dane sprzed godziny:  
```sql
SELECT * FROM Emp  
AS OF TIMESTAMP(SYSDATE - 1/24);  
```

**Tabele temporalne:**  
- Specjalne tabele przechowujące historię zmian danych.  
- Umożliwiają automatyczne śledzenie wersji danych w czasie.  

**Korzyści:**  
- Łatwy dostęp do historycznych danych bez ręcznego zarządzania wersjami.  
- Przydatne w audycie, analizie trendów i przywracaniu danych.

---


# Lenki Extras

### 1. **Strojenie baz danych (ZBD1)**
   - **Cel strojenia**: Poprawa wydajności bazy danych, zwiększenie przepustowości, skrócenie czasu reakcji aplikacji.
   - **Zasady strojenia**:
     - **Myśl globalnie, działaj lokalnie**: Nie lecz symptomów, lecz przyczyny. Unikaj drobnych optymalizacji, które mogą pogorszyć całość.
     - **Partycjonowanie**: Rozproszenie danych w przestrzeni (geograficzne, na wielu dyskach) i w czasie (przenoszenie zadań wsadowych na okresy mniejszego obciążenia).
     - **Inicjacja jest droga, używanie tanie**: Wykorzystuj prepared statements, pule połączeń, unikaj częstego otwierania i zamykania połączeń.
     - **Kompromisy**: Dodanie pamięci RAM przyspiesza system, ale kosztuje. Dodanie indeksów przyspiesza zapytania, ale spowalnia modyfikacje danych.
     - **Równoważenie obciążenia**: Rozkładaj obciążenie między klientem a serwerem. Interakcje z użytkownikiem powinny odbywać się poza transakcjami, aby uniknąć długotrwałych blokad.

### 2. **Indeksy (ZBD1)**
   - **Rodzaje indeksów**:
     - **B+drzewo**: Zrównoważone drzewo, które dobrze sprawdza się w zapytaniach zakresowych i sortowaniach. Liście zawierają ciągi par klucz-wskaźnik, co umożliwia szybki dostęp do danych.
     - **Indeks haszowany**: Szybki dostęp do danych w zapytaniach punktowych, ale nieefektywny w zapytaniach zakresowych. Wykorzystuje funkcję haszującą do mapowania kluczy na pozycje w tabeli.
     - **Indeks bitmapowy**: Skuteczny w zapytaniach z wieloma warunkami logicznymi (AND, OR), ale nie nadaje się do częstych modyfikacji danych. Każda unikalna wartość klucza ma swoją mapę bitową.
   - **Indeks pogrupowany vs niepogrupowany**:
     - **Pogrupowany**: Tabela może mieć tylko jeden taki indeks. Rekordy o podobnych wartościach klucza są fizycznie blisko siebie. Lepszy do sortowania i zapytań zakresowych.
     - **Niepogrupowany**: Tabela może mieć wiele takich indeksów. Nie narzuca fizycznej organizacji danych. Wskaźniki w indeksie wskazują na rekordy w tabeli.
   - **Indeks pokrywający**: Zawiera wszystkie kolumny potrzebne w zapytaniu, co pozwala na uniknięcie dostępu do tabeli (strategia „tylko indeks”). Jest szczególnie efektywny, gdy kolumny używane w WHERE i SELECT są zawarte w indeksie.
   - **Pielęgnacja indeksów**: Regularne odbudowywanie indeksów (np. w SQL Server) zapobiega fragmentacji i utrzymuje wydajność. W przypadku dużych tabel, indeksy mogą ulec fragmentacji, co prowadzi do spadku wydajności.

### 3. **Strojenie zapytań (ZBD2)**
   - **Optymalizacja zapytań**:
     - **Użycie indeksów**: Unikaj operatorów arytmetycznych, funkcji (np. SUBSTR) i porównań z NULL w warunkach WHERE, ponieważ mogą uniemożliwić użycie indeksów. Alternatywnie, można użyć indeksów funkcyjnych.
     - **Eliminacja DISTINCT**: DISTINCT jest kosztowny, ponieważ wymaga sortowania. Czasem można go uniknąć poprzez odpowiednie przeformułowanie zapytania.
     - **Poprawa podzapytań**: Przeformułowanie podzapytań nieskorelowanych na złączenia może znacznie poprawić wydajność. Nieskorelowane podzapytania są wykonywane raz, podczas gdy skorelowane są wykonywane dla każdego rekordu.
     - **Użycie tabel tymczasowych**: Tabele tymczasowe mogą pomóc w optymalizacji zapytań, ale ich nadużywanie może prowadzić do spadku wydajności. Tabele tymczasowe mogą „oślepiać” optymalizator, dlatego należy je używać z rozwagą.
     - **Warunki złączenia**: Złączenia powinny być wykonywane po atrybutach liczbowych, a nie napisowych, aby uniknąć kosztownych operacji sortowania. Złączenia po atrybutach liczbowych są szybsze, ponieważ porównania liczbowe są mniej kosztowne niż porównania tekstowe.
     - **Użycie HAVING**: HAVING powinno być używane tylko do selekcji po zagregowanych właściwościach grup. W innych przypadkach lepiej użyć WHERE. HAVING jest kosztowne, ponieważ działa na już zagregowanych danych.
     - **Perspektywy zmaterializowane**: Perspektywy zmaterializowane przechowują wyniki zapytań, co może znacznie przyspieszyć wykonywanie częstych zapytań. Są szczególnie przydatne w systemach analitycznych, gdzie dane nie zmieniają się często.

### 4. **Strojenie współbieżności (ZBD3)**
   - **Transakcje**: Transakcja to ciąg operacji odczytu i zapisu, które powinny być wykonywane w sposób izolowany od innych transakcji. Transakcje muszą być ACID (Atomicity, Consistency, Isolation, Durability).
   - **Poziomy izolacji**:
     - **READ UNCOMMITTED**: Najniższy poziom izolacji, pozwala na odczytywanie niezatwierdzonych danych. Może prowadzić do „brudnych odczytów”.
     - **READ COMMITTED**: Tylko zatwierdzone dane są widoczne. Zapobiega brudnym odczytom, ale może prowadzić do „niepowtarzalnych odczytów”.
     - **REPEATABLE READ**: Gwarantuje, że dane odczytane w trakcie transakcji nie zmienią się. Zapobiega niepowtarzalnym odczytom, ale może prowadzić do „fantomów”.
     - **SERIALIZABLE**: Najwyższy poziom izolacji, gwarantuje pełną szeregowalność transakcji. Zapobiega fantomom, ale może znacznie ograniczyć współbieżność.
   - **Blokady**:
     - **Współdzielona (S)**: Pozwala wielu transakcjom na jednoczesny odczyt danych.
     - **Wyłączna (X)**: Tylko jedna transakcja może mieć dostęp do danych. Blokady wyłączne są używane przy zapisie.
   - **Izolacja migawkowa (Snapshot Isolation)**: Transakcje pracują na migawce danych z momentu ich rozpoczęcia. Zmniejsza ryzyko zakleszczeń, ale może prowadzić do konfliktów aktualizacji. Jest szczególnie przydatna w systemach o wysokiej współbieżności.
   - **Zakleszczenia**: Transakcje wzajemnie czekają na zwolnienie blokad. Aby uniknąć zakleszczeń, należy odpowiednio zarządzać kolejnością blokad. Można również użyć mechanizmów wykrywania i usuwania zakleszczeń.

### 5. **Strojenie systemu operacyjnego i sprzętu (ZBD3, ZBD4)**
   - **Bufor bazy danych**: Wielkość bufora powinna być dostosowana do rozmiaru danych. Zbyt mały bufor prowadzi do częstych odczytów z dysku, zbyt duży może powodować stronicowanie. Monitoruj stosunek trafień (hit ratio) i zwiększaj rozmiar bufora, dopóki wykres się nie spłaszczy.
   - **RAID**: 
     - **RAID 1 (mirroring)**: Dobre dla dzienników transakcji, zapewnia wysoką dostępność. Zapisy są synchroniczne i sekwencyjne.
     - **RAID 5 (parity)**: Dobre dla aplikacji intensywnie czytających, ale ma negatywny wpływ na zapisy. Każdy zapis wymaga 4 operacji I/O (odczyt starej danej, odczyt starej parzystości, zapis nowej danej, zapis nowej parzystości).
     - **RAID 10 (striping + mirroring)**: Dobre dla aplikacji intensywnie piszących. Łączy zalety RAID 0 (striping) i RAID 1 (mirroring).
   - **Dodanie pamięci RAM**: Zwiększa rozmiar bufora bazy danych, co przyspiesza dostęp do danych. Pamięć RAM jest kluczowa dla wydajności systemu, szczególnie w systemach OLTP.
   - **Dodanie dysków**: Dziennik transakcji powinien być na oddzielnym dysku, aby uniknąć wąskich gardeł. Pliki danych i indeksów można rozłożyć na wielu dyskach, aby zwiększyć przepustowość.

### 6. **Strojenie interfejsu (ZBD4)**
   - **Minimalizacja komunikacji między aplikacją a bazą danych**: Unikaj częstego przekraczania granicy interfejsu aplikacji. Wykorzystuj pętle i kursory z umiarem. Każde przekroczenie interfejsu aplikacji wiąże się z kosztem, dlatego należy minimalizować liczbę komunikatów.
   - **Prepared statements**: Zmniejszają koszt kompilacji zapytań, szczególnie gdy zapytanie jest wykonywane wielokrotnie. Prepared statements pozwalają na ponowne wykorzystanie planów wykonania, co przyspiesza wykonywanie zapytań.
   - **Masowe ładowanie danych**: Wykorzystuj narzędzia do masowego ładowania danych, które omijają procesor zapytań i menedżera składowania, co znacznie przyspiesza operacje. Masowe ładowanie danych może być nawet o rzędy wielkości szybsze niż tradycyjne wstawianie rekordów.
   - **Pobieranie tylko potrzebnych danych**: Unikaj przesyłania niepotrzebnych kolumn i wierszy, co może uniemożliwić użycie indeksów pokrywających. Przesyłaj tylko te dane, które są rzeczywiście potrzebne użytkownikowi.

### 7. **Monitorowanie wydajności (ZBD4)**
   - **Cele monitorowania**: Identyfikacja wąskich gardeł, optymalizacja parametrów bazy danych, zapobieganie problemom wydajnościowym. Monitorowanie powinno być zarówno prewencyjne, jak i reakcyjne.
   - **Narzędzia monitorowania**: 
     - **Monitory zdarzeń** (np. SQL Server Profiler): Pozwalają na śledzenie i analizę zapytań SQL.
     - **Monitory wydajności** (np. Performance Monitor w Windows): Pozwalają na monitorowanie wskaźników systemowych, takich jak użycie procesora, dysków i pamięci.
     - **Tuning advisors**: Narzędzia do automatycznej optymalizacji zapytań i indeksów. Mogą sugerować zmiany w schemacie bazy danych lub w zapytaniach.
   - **Systematyczne podejście**: Identyfikacja krytycznych zapytań, analiza planów wykonania, monitorowanie wskaźników systemowych (użycie procesora, dysków, pamięci). Krytyczne zapytania to te, które mają długi czas wykonania, są często wykonywane lub konsumują dużo zasobów.

### 8. **Dane nierelacyjne i NoSQL (ZBD4)**
   - **Dane przestrzenne**: Przechowywanie i indeksowanie danych geograficznych (punkty, linie, wieloboki). Wykorzystanie indeksów siatkowych i R-drzew. Indeksy przestrzenne pozwalają na szybkie wyszukiwanie obiektów w przestrzeni.
   - **NoSQL**: 
     - **Klucz-wartość**: Prosty model danych, np. Redis. Szybki dostęp do danych, ale brak zaawansowanych funkcji zapytań.
     - **Bazy kolumnowe**: Wysoka wydajność w systemach analitycznych, np. Cassandra. Dane są przechowywane w kolumnach, co ułatwia agregacje.
     - **Bazy dokumentowe**: Przechowywanie dokumentów JSON, np. MongoDB. Elastyczny model danych, ale brak silnej spójności.
     - **Bazy grafowe**: Przechowywanie danych w postaci węzłów i relacji, np. Neo4J. Idealne do modelowania złożonych relacji.
   - **Bazy danych in-memory**: Przechowywanie całej bazy danych w pamięci RAM, co znacznie przyspiesza dostęp do danych (np. TimesTen, MS SQL in-memory). Bazy in-memory są szczególnie przydatne w systemach OLTP, gdzie wymagana jest bardzo niska latencja.

### 9. **Zaawansowane techniki (ZBD4)**
   - **Temporal Tables**: Przechowywanie danych historycznych wraz z aktualnymi, co umożliwia śledzenie zmian w czasie. Temporal Tables pozwalają na łatwe odtwarzanie stanu bazy danych z dowolnego momentu w przeszłości.
   - **Indeksy kolumnowe (Columnstore Index)**: Skuteczne w systemach analitycznych, gdzie wymagane są szybkie agregacje dużych zbiorów danych. Indeksy kolumnowe przechowują dane w kolumnach, co pozwala na efektywne kompresowanie danych i szybkie wykonywanie zapytań analitycznych.
   - **Grafy w relacyjnych bazach danych**: MS SQL od 2017 roku wspiera przechowywanie i przetwarzanie danych w postaci grafów. Grafowe bazy danych są szczególnie przydatne w aplikacjach społecznościowych, rekomendacjach i analizie sieci.

### 10. **Dodatkowe techniki i uwagi**
   - **Denormalizacja**: W niektórych przypadkach denormalizacja schematu bazy danych może poprawić wydajność, szczególnie w systemach analitycznych, gdzie dane są często odczytywane, ale rzadko modyfikowane. Denormalizacja polega na wprowadzeniu nadmiarowości danych, aby uniknąć kosztownych złączeń.
   - **Partycjonowanie tabel**: Partycjonowanie pozwala na rozbicie dużych tabel na mniejsze fragmenty, co może znacznie przyspieszyć wykonywanie zapytań. Partycjonowanie może być zakresowe, listowe lub haszowane.
   - **Perspektywy zmaterializowane**: Perspektywy zmaterializowane przechowują wyniki zapytań, co może znacznie przyspieszyć wykonywanie częstych zapytań. Są szczególnie przydatne w systemach analitycznych, gdzie dane nie zmieniają się często.
   - **Optymalizacja zapytań wsadowych**: W systemach wsadowych, gdzie wykonywane są duże ilości operacji, ważne jest optymalizowanie zapytań pod kątem minimalizacji czasu wykonania. Można to osiągnąć poprzez równoległe przetwarzanie, partycjonowanie danych i wykorzystanie narzędzi do masowego ładowania danych.
