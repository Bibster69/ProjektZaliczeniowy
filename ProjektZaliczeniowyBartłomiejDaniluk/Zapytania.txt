-- Projekcje: 

SELECT * FROM adres;

SELECT Lokalizacja, Miejsca FROM garaż;

SELECT Nazwisko, email, Region_Nazwa FROM klient;

SELECT Model, Marka, Przebieg FROM pojazd;

SELECT Imie, Nazwisko, Pozycja FROM pracownicy;

SELECT Nazwa FROM region;

SELECT * FROM region_pracownicy;

SELECT Pojazd_ID, data_wyp, data_zw FROM wypożyczenia;

SELECT * FROM wypożyczenia_pracownicy;

SELECT opis, czy_naprawiony FROM pojazd_uszkodzenia;

SELECT * FROM uszkodzenia;

-- -----------------------------------------------------------------------------------------------
-- Selekcje:

SELECT Ulica FROM adres WHERE kod_pocztowy > 20100 ORDER BY Ulica;

SELECT Lokalizacja FROM garaż WHERE Miejsca > 2 ORDER BY Miejsca;

SELECT Imie, Nazwisko FROM klient WHERE email LIKE '%gmail%' ORDER BY Imie;

SELECT CONCAT(Imie,' ', Nazwisko) AS 'Imie i Nazwisko' from klient WHERE CONVERT(telefon, CHAR(16)) LIKE '%0%';

SELECT Pojazd_ID, Przebieg , Stan FROM pojazd WHERE Model = 'Fabia';

SELECT Pojazd_ID, Przebieg , CONCAT(Marka, ' ', Model) AS 'Samochod' FROM pojazd WHERE przebieg BETWEEN 20000 AND 100000;

SELECT CONCAT(Imie , ' ', Nazwisko) AS 'Godność' FROM pracownicy WHERE Pozycja LIKE '%Gł%';

SELECT Imie, Wynagrodzenie, Pozycja FROM pracownicy WHERE Wynagrodzenie > 15000;

SELECT data_zw, Klient_ID FROM wypożyczenia WHERE czy_uszkodzony = true ORDER BY data_wyp;

SELECT COUNT(Nazwa) AS 'Ilość średnio zamożnych regionów' FROM region WHERE Zamożność = 'Srednio'; 

SELECT Pracownik_ID, COUNT(Wypo_ID) FROM wypożyczenia_pracownicy WHERE Pracownik_ID = 4;

SELECT * FROM uszkodzenia WHERE typ LIKE '%Układ%';

-- ------------------------------------------------------------------------------------------------------
-- Złączenia 2 tabel

SELECT Nazwa
FROM region JOIN region_pracownicy ON region.Nazwa=region_pracownicy.Region_Nazwa;

SELECT * FROM pracownicy
INNER JOIN adres ON pracownicy.kod_pocztowy=adres.kod_pocztowy;

SELECT Marka, Model
FROM pojazd RIGHT JOIN garaż ON pojazd.Garaz_ID=garaż.Garaz_ID;

SELECT Imie, Nazwisko
FROM klient LEFT JOIN wypożyczenia ON klient.Klient_ID=wypożyczenia.Klient_ID;

SELECT CONCAT(Marka, ' ', Model) AS 'Samochod'
FROM pojazd INNER JOIN pojazd_uszkodzenia ON pojazd.Pojazd_ID=pojazd_uszkodzenia.Pojazd_ID;

SELECT DISTINCT CONCAT(Marka, ' ', Model) AS Samochod
FROM pojazd RIGHT JOIN pojazd_uszkodzenia ON pojazd.Pojazd_ID=pojazd_uszkodzenia.Pojazd_ID
AND pojazd.Stan = 'Uszkodzony'
WHERE pojazd.Stan IS NOT NULL;

SELECT  uszk.typ , pojuszk.opis
FROM uszkodzenia AS uszk
INNER JOIN pojazd_uszkodzenia AS pojuszk ON uszk.Uszkodz_ID = pojuszk.Uszkodzenia_ID
WHERE uszk.typ LIKE '%Układ%';

SELECT * FROM
pojazd LEFT JOIN wypożyczenia ON pojazd.Pojazd_ID=wypożyczenia.Pojazd_ID
UNION
SELECT * FROM
pojazd RIGHT JOIN wypożyczenia ON pojazd.Pojazd_ID=wypożyczenia.Pojazd_ID

SELECT * FROM
pojazd LEFT JOIN wypożyczenia ON pojazd.Pojazd_ID=wypożyczenia.Pojazd_ID
UNION ALL
SELECT * FROM
pojazd RIGHT JOIN wypożyczenia ON pojazd.Pojazd_ID=wypożyczenia.Pojazd_ID
WHERE pojazd.Pojazd_ID IS NULL;

SELECT p.Pracownik_ID, p.Nazwisko, p.Szef_ID AS 'Melduje się do', s.Nazwisko, s.Pracownik_ID
FROM pracownicy AS p
LEFT JOIN pracownicy AS s ON p.Szef_ID=s.Pracownik_ID
ORDER BY s.Pracownik_ID;


SELECT CONCAT(pra.Imie, ' ', pra.Nazwisko) AS 'Pracownik', COUNT(wyppra.Wypo_ID) AS 'Ilosc Wypożyczeń'
FROM pracownicy AS pra
INNER JOIN wypożyczenia_pracownicy AS wyppra ON pra.Pracownik_ID = wyppra.Pracownik_ID
WHERE wyppra.Pracownik_ID = 4;

-- ------------------------------------------------------------------------------------------------
-- Złączenia 3 lub więcej tabel

SELECT Imie, Nazwisko, Pozycja, wyp.data_wyp
FROM pracownicy
INNER JOIN wypożyczenia_pracownicy ON pracownicy.Pracownik_ID=wypożyczenia_pracownicy.Pracownik_ID
INNER JOIN wypożyczenia AS wyp ON wypożyczenia_pracownicy.Wypo_ID=wyp.Wypo_ID;

SELECT wyp.Wypo_ID, wyp.data_wyp, CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochód', gar.Lokalizacja 
FROM wypożyczenia AS wyp
INNER JOIN pojazd AS poj ON wyp.Pojazd_ID=poj.Pojazd_ID
INNER JOIN garaż AS gar ON poj.Garaz_ID=gar.Garaz_ID;

SELECT CONCAT(kl.Imie, ' ', kl.Nazwisko) AS 'Godność', wyp.data_wyp, wyp.data_zw, CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochód'
FROM klient AS kl
INNER JOIN wypożyczenia AS wyp ON kl.Klient_ID=wyp.Klient_ID
INNER JOIN pojazd AS poj ON wyp.Pojazd_ID=poj.Pojazd_ID;


SELECT CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochod', Stan, przebieg, uszk.typ, pojuszk.opis
FROM pojazd AS poj
INNER JOIN pojazd_uszkodzenia AS pojuszk ON poj.Pojazd_ID=pojuszk.Pojazd_ID
INNER JOIN uszkodzenia AS uszk ON pojuszk.Uszkodzenia_ID=uszk.Uszkodz_ID;

SELECT DISTINCT CONCAT(kli.Imie, ' ', kli.Nazwisko) AS 'Klient', email, telefon, wyp.Wypo_ID, wyp.data_wyp, wyp.Pojazd_ID, CONCAT(pra.Imie, ' ', pra.Nazwisko) AS 'Pracownik' 
FROM klient AS kli
INNER JOIN wypożyczenia AS wyp ON kli.Klient_ID=wyp.Klient_ID
INNER JOIN wypożyczenia_pracownicy AS wyppra ON wyp.Wypo_ID=wyppra.Wypo_ID
INNER JOIN pracownicy AS pra ON wyppra.Pracownik_ID=pra.Pracownik_ID
ORDER BY Wypo_ID;

SELECT  reg.Nazwa, reg.Zamożność , CONCAT(kli.Imie, ' ', kli.Nazwisko) AS 'Klient', CONCAT(pra.Imie, ' ', pra.Nazwisko) AS 'Pracownik' 
FROM pracownicy AS pra
RIGHT JOIN region_pracownicy AS regpra ON pra.Pracownik_ID=regpra.Pracownik_ID
INNER JOIN region AS reg ON regpra.Region_Nazwa=reg.Nazwa
INNER JOIN klient AS kli ON reg.Nazwa=kli.Region_Nazwa
WHERE pra.Imie LIKE '%Janusz%',
ORDER BY reg.Zamożność;

SELECT CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochod', poj.Przebieg, uszk.typ AS 'Rodzaj Uszkodzenia', gar.Lokalizacja,
CASE
	WHEN pojuszk.czy_naprawiony = TRUE THEN 'Naprawiony'
    ELSE 'Nie naprawiony'
END
AS 'Czy naprawiony'
FROM garaż AS gar
INNER JOIN pojazd AS poj ON gar.Garaz_ID = poj.Garaz_ID
INNER JOIN pojazd_uszkodzenia AS pojuszk ON poj.Pojazd_ID=pojuszk.Pojazd_ID
INNER JOIN uszkodzenia AS uszk ON pojuszk.Uszkodzenia_ID=uszk.Uszkodz_ID
ORDER BY poj.Przebieg;

SELECT CONCAT(pra.Imie, ' ', pra.Nazwisko) AS 'Pracownik', CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochod', wyp.data_wyp, wyp.data_zw
FROM pracownicy AS pra
INNER JOIN wypożyczenia_pracownicy AS wyppra ON pra.Pracownik_ID = wyppra.Pracownik_ID
INNER JOIN wypożyczenia AS wyp on wyppra.Wypo_ID=wyp.Wypo_ID
INNER JOIN pojazd AS poj ON wyp.Pojazd_ID=poj.Pojazd_ID
WHERE ABS(DATEDIFF(wyp.data_wyp, wyp.data_zw)) > 5;

SELECT CONCAT(kli.Imie, ' ', kli.Nazwisko) AS 'Klient', kli.email, kli.telefon, CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochod', wyp.data_wyp, wyp.data_zw, pojuszk.opis
FROM klient AS kli
INNER JOIN wypożyczenia AS wyp ON kli.Klient_ID=wyp.Klient_ID
INNER JOIN pojazd AS poj ON wyp.Pojazd_ID=poj.Pojazd_ID
INNER JOIN pojazd_uszkodzenia AS pojuszk ON poj.Pojazd_ID=pojuszk.Pojazd_ID
WHERE wyp.czy_uszkodzony = TRUE
ORDER BY wyp.Wypo_ID;

SELECT CONCAT(pra.Imie, ' ', pra.Nazwisko) AS 'Pracownik', CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochod', wyp.data_wyp, gar.Lokalizacja
FROM pracownicy AS pra
INNER JOIN wypożyczenia_pracownicy AS wyppra ON pra.Pracownik_ID = wyppra.Pracownik_ID
INNER JOIN wypożyczenia AS wyp on wyppra.Wypo_ID=wyp.Wypo_ID
INNER JOIN pojazd AS poj ON wyp.Pojazd_ID=poj.Pojazd_ID
INNER JOIN garaż AS gar ON poj.Garaz_ID=gar.Garaz_ID
WHERE pra.Pozycja = 'Kierowca'
ORDER BY wyp.data_wyp;

-- ----------------------------------------------------------------------------------------------
-- Skrypty modyfikujące obecny schemat

CREATE TABLE IF NOT EXISTS klient_uszkodzenie AS -- Tworzenie nowej tabeli o nazwie: klient_uszkodzenie
SELECT kli.Klient_ID, wyp.data_wyp, wyp.data_zw  -- Wypełnionej rekordami z tabel klient oraz wypożyczenia
FROM klient AS kli 
INNER JOIN wypożyczenia AS wyp ON kli.Klient_ID = wyp.Klient_ID
WHERE wyp.czy_uszkodzony = TRUE; -- Gdzie pojazd był uszkodzony podczas wypożyczenia

ALTER TABLE klient_uszkodzenie
ADD CONSTRAINT FK_Klient_ID
FOREIGN KEY (Klient_ID) REFERENCES klient(Klient_ID); -- Dodanie klucza obcego odwołującego się do tabeli klient

ALTER TABLE pojazd_uszkodzenia
ADD COLUMN koszt INT NOT NULL AFTER opis; -- Dodanie kolumny "koszt" do tabeli pojazd_uszkodzenia

-- Wypełnienie kolumny

UPDATE pojazd_uszkodzenia SET koszt=200 WHERE Uszkodzenia_ID = 8 AND Pojazd_ID = 2;
UPDATE pojazd_uszkodzenia SET koszt=150 WHERE Uszkodzenia_ID = 10 AND Pojazd_ID = 3;
UPDATE pojazd_uszkodzenia SET koszt=100 WHERE Uszkodzenia_ID = 2 AND Pojazd_ID = 3;
UPDATE pojazd_uszkodzenia SET koszt=50 WHERE Uszkodzenia_ID = 1 AND Pojazd_ID = 1;
UPDATE pojazd_uszkodzenia SET koszt=800 WHERE Uszkodzenia_ID = 9 AND Pojazd_ID = 8;
UPDATE pojazd_uszkodzenia SET koszt=2500 WHERE Uszkodzenia_ID = 4 AND Pojazd_ID = 10;
UPDATE pojazd_uszkodzenia SET koszt=400 WHERE Uszkodzenia_ID = 11 AND Pojazd_ID = 2;
UPDATE pojazd_uszkodzenia SET koszt=200 WHERE Uszkodzenia_ID = 8 AND Pojazd_ID = 4;
UPDATE pojazd_uszkodzenia SET koszt=4500 WHERE Uszkodzenia_ID = 5 AND Pojazd_ID = 8;
UPDATE pojazd_uszkodzenia SET koszt=500 WHERE Uszkodzenia_ID = 2 AND Pojazd_ID = 6;
UPDATE pojazd_uszkodzenia SET koszt=100 WHERE Uszkodzenia_ID = 3 AND Pojazd_ID = 7;


CREATE TABLE IF NOT EXISTS pojazd_koszta AS
SELECT poj.Pojazd_ID 
FROM pojazd AS poj; -- Tworzenie nowej tabeli : "pojazd_koszta" zawierającej kolumnę Pojazd_ID z tabeli pojazd

ALTER TABLE pojazd_koszta
ADD PRIMARY KEY (Pojazd_ID),
ADD COLUMN koszta_eksploatacji FLOAT NULL DEFAULT 0,
ADD COLUMN ilość_uszkodzeń FLOAT NULL DEFAULT 0,
ADD COLUMN koszta_uszkodzeń FLOAT NULL DEFAULT 0, -- Dodawanie kolumn do tabeli.
ADD CONSTRAINT FK_Pojazd_ID
FOREIGN KEY (Pojazd_ID) REFERENCES pojazd(Pojazd_ID); -- Dodanie klucza obcego odwołującego się do tabeli pojazd

-- Wypełnienie kolumny ilość_uszkodzeń korzystając z informacji z tabel pojazd oraz pojazd uszkodzenia

UPDATE pojazd_koszta AS pojkosz
INNER JOIN pojazd AS poj ON pojkosz.Pojazd_ID = poj.Pojazd_ID
SET pojkosz.ilość_uszkodzeń = (SELECT COUNT(Uszkodzenia_ID) FROM pojazd_uszkodzenia WHERE Pojazd_ID = poj.Pojazd_ID)
WHERE pojkosz.Pojazd_ID = poj.Pojazd_ID;

-- Wypełnienie kolumny koszta_uszkodzeń korzystając z informacji z tabel pojazd oraz pojazd uszkodzenia

UPDATE pojazd_koszta AS pojkosz
INNER JOIN pojazd AS poj ON pojkosz.Pojazd_ID = poj.Pojazd_ID
SET pojkosz.koszta_uszkodzeń = (SELECT SUM(KOSZT) FROM pojazd_uszkodzenia WHERE Pojazd_ID = poj.Pojazd_ID)
WHERE pojkosz.Pojazd_ID = poj.Pojazd_ID;

-- Ustawienie wartości 0 zamiast wartości NULL

UPDATE pojazd_koszta 
SET koszta_uszkodzeń = 0
WHERE ISNULL(koszta_uszkodzeń) = 1;

-- Wypełnienie kolumny koszta_eksploatacji

UPDATE pojazd_koszta SET koszta_eksploatacji = 2000 WHERE Pojazd_ID = 1;
UPDATE pojazd_koszta SET koszta_eksploatacji = 1500 WHERE Pojazd_ID = 2;
UPDATE pojazd_koszta SET koszta_eksploatacji = 1200 WHERE Pojazd_ID = 3;
UPDATE pojazd_koszta SET koszta_eksploatacji = 2250 WHERE Pojazd_ID = 4;
UPDATE pojazd_koszta SET koszta_eksploatacji = 3500 WHERE Pojazd_ID = 5;
UPDATE pojazd_koszta SET koszta_eksploatacji = 1240 WHERE Pojazd_ID = 6;
UPDATE pojazd_koszta SET koszta_eksploatacji = 1200 WHERE Pojazd_ID = 7;
UPDATE pojazd_koszta SET koszta_eksploatacji = 2000 WHERE Pojazd_ID = 8;
UPDATE pojazd_koszta SET koszta_eksploatacji = 4000 WHERE Pojazd_ID = 9;
UPDATE pojazd_koszta SET koszta_eksploatacji = 4500 WHERE Pojazd_ID = 10;
UPDATE pojazd_koszta SET koszta_eksploatacji = 400 WHERE Pojazd_ID = 11;
UPDATE pojazd_koszta SET koszta_eksploatacji = 400 WHERE Pojazd_ID = 12;

-- Dodanie kolumny typ do tabeli pojazd_uszkodzenia

ALTER TABLE pojazd_uszkodzenia
ADD COLUMN typ VARCHAR(20) NOT NULL
AFTER Uszkodzenia_ID;

-- Wypełnienie kolumny typ korzystając z informacji z tabeli uszkodzenia

UPDATE pojazd_uszkodzenia AS pojuszk
INNER JOIN uszkodzenia AS uszk ON pojuszk.Uszkodzenia_ID = uszk.Uszkodz_ID
SET pojuszk.typ = uszk.typ
WHERE pojuszk.Uszkodzenia_ID = uszk.Uszkodz_ID;

-- Usuwanie tabeli uszkodenia

ALTER TABLE pojazd_uszkodzenia DROP FOREIGN KEY `fk_Pojazd_has_Uszkodzenia_Uszkodzenia1`;
ALTER TABLE pojazd_uszkodzenia DROP INDEX `fk_Pojazd_has_Uszkodzenia_Uszkodzenia1_idx`;
DROP TABLE uszkodzenia;

-- Dostosowanie zawartości kolumny uszkodzenia_ID do nowego schematu

UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 1
WHERE uszkodzenia_id = 1 AND pojazd_id = 1;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 2
WHERE uszkodzenia_id = 2 AND pojazd_id = 3;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 3
WHERE uszkodzenia_id = 2 AND pojazd_id = 6;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 4
WHERE uszkodzenia_id = 3 AND pojazd_id = 7;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 5
WHERE uszkodzenia_id = 4 AND pojazd_id = 10;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 6
WHERE uszkodzenia_id = 5 AND pojazd_id = 8;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 7
WHERE uszkodzenia_id = 8 AND pojazd_id = 2;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 8
WHERE uszkodzenia_id = 8 AND pojazd_id = 4;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 9
WHERE uszkodzenia_id = 9 AND pojazd_id = 8;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 10
WHERE uszkodzenia_id = 10 AND pojazd_id = 3;
UPDATE pojazd_uszkodzenia SET Uszkodzenia_ID = 11
WHERE uszkodzenia_id = 11 AND pojazd_id = 2;

-- Ustawienie kkolumny Uszkodzenia_ID jako nowy klucz główny tabeli pojazd_uszkodzenia z funkcją auto-inkrementacji

ALTER TABLE pojazd_uszkodzenia DROP PRIMARY KEY;
ALTER TABLE pojazd_uszkodzenia ADD PRIMARY KEY(Uszkodzenia_ID);
ALTER TABLE pojazd_uszkodzenia MODIFY Uszkodzenia_ID INTEGER NOT NULL AUTO_INCREMENT;
ALTER TABLE pojazd_uszkodzenia AUTO_INCREMENT = 12;
-- -------------------------------------------------------------
-- Przykładowe zapytania do zmodyfikowanego schematu


SELECT CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochód', poj.Przebieg, pojuszk.typ, pojuszk.opis , pojuszk.koszt
FROM pojazd AS poj 
INNER JOIN pojazd_uszkodzenia AS pojuszk ON poj.Pojazd_ID = pojuszk.Pojazd_ID
WHERE pojuszk.koszt > 1000;

SELECT CONCAT(poj.Marka, ' ', poj.Model) AS 'Samochód', pojuszk.typ, pojuszk.opis, pojuszk.koszt
FROM pojazd_koszta AS pojkosz
INNER JOIN pojazd AS poj ON pojkosz.Pojazd_ID = poj.Pojazd_ID
INNER JOIN pojazd_uszkodzenia AS pojuszk ON poj.Pojazd_ID = pojuszk.Pojazd_ID
WHERE pojkosz.ilość_uszkodzeń > 1;

SELECT CONCAT(kli.Imie , ' ', kli.Nazwisko) AS 'Klient', CONCAT(kli.telefon, '     ', kli.email) AS 'Dane kontaktowe', kliuszk.data_wyp, kliuszk.data_zw
FROM klient AS kli
INNER JOIN klient_uszkodzenie AS kliuszk ON kli.Klient_ID = kliuszk.Klient_ID;

SELECT CONCAT(poj.Marka, ' ', poj.Model) AS Samochod, pojuszk.opis, pojuszk.koszt,
CASE
	WHEN pojuszk.czy_naprawiony = TRUE THEN 'Naprawione'
    ELSE 'Nie naprawione'
END
AS 'Czy naprawione'
FROM pojazd AS poj
RIGHT JOIN pojazd_uszkodzenia AS pojuszk ON poj.Pojazd_ID=pojuszk.Pojazd_ID;


