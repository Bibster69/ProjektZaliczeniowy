
SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0;
-- Komenda ta decyduje otym czy InnoDB sprawdza duplikaty kluczy obcych. 
-- Ominięcie tego etapu przyspiesza importowanie obszernych baz danych. 


SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0;
-- Komenda ta decyduje o tym czy serwer sprawdza istnienie tabeli do której odwołuje się dany klucz obcy.alter.
-- Opcja musie być wyłączona na czas importu aby możliwe było definiowanie kluczy obcych

SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION';
-- Komenda ta ustawia opcje trybu działania serwera.


DROP SCHEMA IF EXISTS `wypożyczalnia`; -- Komenda usuwa baazę danych "wypożyczalnia" w razie jej istnienia
CREATE SCHEMA IF NOT EXISTS `wypożyczalnia` DEFAULT CHARACTER SET utf8 ;-- Komenda tworzy bazę danych
USE `wypożyczalnia` ;

-- Tworzenie tabeli "adres"
CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`adres` (
  `kod_pocztowy` INT NOT NULL,
  `Ulica` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`kod_pocztowy`)) -- Określenie kolumny "kod_pocztowy" jako klucz główny.
ENGINE = InnoDB -- Określenie używaniego silnika pamięci masowej.
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`garaż` (
  `Garaz_ID` INT NOT NULL AUTO_INCREMENT, -- Kolumna "Garaz_ID" jest kolumną inkrementującą się automatycznie.
  `Miejsca` INT NOT NULL,
  `Lokalizacja` VARCHAR(45) NOT NULL,
  PRIMARY KEY (`Garaz_ID`))
ENGINE = InnoDB
AUTO_INCREMENT = 11 -- Określenie liczby od której ma się rozpocząć autoinkrementacja.
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`region` (
  `Nazwa` CHAR(30) NOT NULL,
  `Zamożność` CHAR(30) NOT NULL,
  CONSTRAINT `check_zamożność` CHECK (Zamożność IN('Zamożny', 'Średnio', 'Mało')), -- Dodanie więzu integralności do kolumny "Zamożność" ograniczającą ją do podanych wartości
  PRIMARY KEY (`Nazwa`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`klient` (
  `Klient_ID` INT NOT NULL AUTO_INCREMENT,
  `Imie` CHAR(30) NOT NULL,
  `Nazwisko` CHAR(30) NOT NULL,
  `email` VARCHAR(45) NOT NULL,
  `telefon` INT NOT NULL,
  `Region_Nazwa` CHAR(30) NULL DEFAULT 'Brak', -- Ustawnienie artości domyślnej kolumny na "Brak"
  PRIMARY KEY (`Klient_ID`),
  UNIQUE INDEX `email_UNIQUE` (`email` ASC) VISIBLE,
  UNIQUE INDEX `telefon_UNIQUE` (`telefon` ASC) VISIBLE, -- Każda podana wartość kolumn email i telefon musi być unikatowa.
  INDEX `fk_Klient_Region_Klient1_idx` (`Region_Nazwa` ASC) VISIBLE,
  CONSTRAINT `fk_Klient_Region_Klient1`
    FOREIGN KEY (`Region_Nazwa`)
    REFERENCES `wypożyczalnia`.`region` (`Nazwa`) -- Ustawienie kolumny "Region_Nazwa" jako klucz obcy odwołujący się do tabeli "region"
    ON UPDATE CASCADE) -- Komenda pozwala na automatyczne uaktualnienie rekordu w razie uaktualnienia rekordu macierzystego
ENGINE = InnoDB
AUTO_INCREMENT = 11
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`pojazd` (
  `Pojazd_ID` INT NOT NULL AUTO_INCREMENT,
  `Przebieg` INT NOT NULL DEFAULT '0',
  `Garaz_ID` INT NULL DEFAULT NULL,
  `Stan` CHAR(30) NOT NULL DEFAULT 'Sprawny',
  `Marka` VARCHAR(30) NOT NULL,
  `Model` CHAR(30) NOT NULL,
  PRIMARY KEY (`Pojazd_ID`),
  INDEX `fk_Pojazd_Garaż1_idx` (`Garaz_ID` ASC) VISIBLE,
  CONSTRAINT `check_Stan` CHECK (Stan IN('Sprawny', 'Niesprawny', 'Uszkodzony')),
  CONSTRAINT `fk_Pojazd_Garaż1`
    FOREIGN KEY (`Garaz_ID`)
    REFERENCES `wypożyczalnia`.`garaż` (`Garaz_ID`)
    ON DELETE SET NULL -- Komenda pozwala na automatyczne ustawienie wartości w razie usuniecia wartości do któej się odwołuje na podaną w tym przypadku NULL 
    ON UPDATE CASCADE)
ENGINE = InnoDB
AUTO_INCREMENT = 13
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`uszkodzenia` (
  `Uszkodz_ID` INT NOT NULL AUTO_INCREMENT,
  `typ` VARCHAR(25) NOT NULL,
  PRIMARY KEY (`Uszkodz_ID`))
ENGINE = InnoDB
AUTO_INCREMENT = 12
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`pojazd_uszkodzenia` (
  `Pojazd_ID` INT NOT NULL,
  `Uszkodzenia_ID` INT NOT NULL,
  `opis` VARCHAR(100) NOT NULL,
  `czy_naprawiony` TINYINT NOT NULL,
  PRIMARY KEY (`Pojazd_ID`, `Uszkodzenia_ID`),
  INDEX `fk_Pojazd_has_Uszkodzenia_Uszkodzenia1_idx` (`Uszkodzenia_ID` ASC) VISIBLE,
  INDEX `fk_Pojazd_has_Uszkodzenia_Pojazd1_idx` (`Pojazd_ID` ASC) VISIBLE,
  CONSTRAINT `fk_Pojazd_has_Uszkodzenia_Pojazd1`
    FOREIGN KEY (`Pojazd_ID`)
    REFERENCES `wypożyczalnia`.`pojazd` (`Pojazd_ID`)
    ON UPDATE CASCADE,
  CONSTRAINT `fk_Pojazd_has_Uszkodzenia_Uszkodzenia1`
    FOREIGN KEY (`Uszkodzenia_ID`)
    REFERENCES `wypożyczalnia`.`uszkodzenia` (`Uszkodz_ID`)
    ON UPDATE CASCADE)
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8;


CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`pracownicy` (
  `Pracownik_ID` INT NOT NULL AUTO_INCREMENT,
  `Imie` CHAR(30) NOT NULL,
  `Nazwisko` CHAR(30) NOT NULL,
  `Szef_ID` INT NULL DEFAULT NULL,
  `Wynagrodzenie` FLOAT NOT NULL,
  `Pozycja` CHAR(30) NOT NULL,
  `kod_pocztowy` INT NULL DEFAULT NULL,
  PRIMARY KEY (`Pracownik_ID`),
  INDEX `fk_Pracownicy_Pracownicy1_idx` (`Szef_ID` ASC) VISIBLE,
  INDEX `fk_Pracownicy_table11_idx` (`kod_pocztowy` ASC) VISIBLE,
  CONSTRAINT `fk_Pracownicy_Pracownicy1`
    FOREIGN KEY (`Szef_ID`)
    REFERENCES `wypożyczalnia`.`pracownicy` (`Pracownik_ID`)
    ON DELETE SET NULL,
  CONSTRAINT `fk_Pracownicy_table11`
    FOREIGN KEY (`kod_pocztowy`)
    REFERENCES `wypożyczalnia`.`adres` (`kod_pocztowy`)
    ON DELETE SET NULL
    ON UPDATE CASCADE)
ENGINE = InnoDB
AUTO_INCREMENT = 12
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`region_pracownicy` (
  `Region_Nazwa` CHAR(30) NOT NULL,
  `Pracownik_ID` INT NOT NULL,
  PRIMARY KEY (`Region_Nazwa`, `Pracownik_ID`),
  INDEX `fk_Region_Klient_has_Pracownicy_Pracownicy1_idx` (`Pracownik_ID` ASC) VISIBLE,
  INDEX `fk_Region_Klient_has_Pracownicy_Region_Klient1_idx` (`Region_Nazwa` ASC) VISIBLE,
  CONSTRAINT `fk_Region_Klient_has_Pracownicy_Pracownicy1`
    FOREIGN KEY (`Pracownik_ID`)
    REFERENCES `wypożyczalnia`.`pracownicy` (`Pracownik_ID`)
    ON UPDATE CASCADE,
  CONSTRAINT `fk_Region_Klient_has_Pracownicy_Region_Klient1`
    FOREIGN KEY (`Region_Nazwa`)
    REFERENCES `wypożyczalnia`.`region` (`Nazwa`)
    ON UPDATE CASCADE)
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8;


CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`wypożyczenia` (
  `Wypo_ID` INT NOT NULL AUTO_INCREMENT,
  `data_wyp` DATE NOT NULL,
  `data_zw` DATE NULL DEFAULT NULL,
  `czy_uszkodzony` TINYINT NOT NULL,
  `Pojazd_ID` INT NOT NULL,
  `Klient_ID` INT NULL DEFAULT '0',
  CONSTRAINT `check_data_zw_` CHECK(DATEDIFF(data_zw, data_wyp) >= 0), -- Komenda dopilnowuje aby data zwortu była puzniejsza niż data wypożyczenia
  PRIMARY KEY (`Wypo_ID`),
  INDEX `fk_Wypozyczenia_Pojazd1_idx` (`Pojazd_ID` ASC) VISIBLE,
  INDEX `fk_Wypozyczenia_Klient1_idx` (`Klient_ID` ASC) VISIBLE,
  CONSTRAINT `fk_Wypozyczenia_Klient1`
    FOREIGN KEY (`Klient_ID`)
    REFERENCES `wypożyczalnia`.`klient` (`Klient_ID`)
    ON DELETE SET NULL
    ON UPDATE CASCADE,
  CONSTRAINT `fk_Wypozyczenia_Pojazd1`
    FOREIGN KEY (`Pojazd_ID`)
    REFERENCES `wypożyczalnia`.`pojazd` (`Pojazd_ID`))
ENGINE = InnoDB
AUTO_INCREMENT = 11
DEFAULT CHARACTER SET = utf8;



CREATE TABLE IF NOT EXISTS `wypożyczalnia`.`wypożyczenia_pracownicy` (
  `Wypo_ID` INT NOT NULL,
  `Pracownik_ID` INT NOT NULL,
  PRIMARY KEY (`Wypo_ID`, `Pracownik_ID`),
  INDEX `fk_Wypozyczenia_has_Pracownicy_Pracownicy1_idx` (`Pracownik_ID` ASC) VISIBLE,
  INDEX `fk_Wypozyczenia_has_Pracownicy_Wypozyczenia1_idx` (`Wypo_ID` ASC) VISIBLE,
  CONSTRAINT `fk_Wypozyczenia_has_Pracownicy_Pracownicy1`
    FOREIGN KEY (`Pracownik_ID`)
    REFERENCES `wypożyczalnia`.`pracownicy` (`Pracownik_ID`),
  CONSTRAINT `fk_Wypozyczenia_has_Pracownicy_Wypozyczenia1`
    FOREIGN KEY (`Wypo_ID`)
    REFERENCES `wypożyczalnia`.`wypożyczenia` (`Wypo_ID`))
ENGINE = InnoDB
DEFAULT CHARACTER SET = utf8;


SET SQL_MODE=@OLD_SQL_MODE;
SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS;
SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS;
-- Komendy ustawiają opcje bazy danych na te zdefiniowane na początku.


-- Wypełnienie tabel

-- wypełnienie tabeli Region

insert into region(Nazwa, Zamożność)
values
	('Czechów N', 'Zamożny'),
	('Czuby S', 'Zamożny'),
    ('Czechów S', 'Średnio'),
    ('Sławin', 'Średnio'),
    ('Kalinowszczyzna', 'Mało'),
    ('Rury', 'Mało'),
    ('Wrotków', 'Mało'),
    ('Konstantynów', 'Zamożny'),
    ('Środmiście', 'Średnio'),
    ('Czuby N', 'Średnio');

-- wypełnienie tabeli Garaż

insert into garaż(Garaz_ID, Miejsca, Lokalizacja)
values
	(1, 2, 'Różana 6'),
    (2, 2, 'Turkusowa 4'),
    (3, 2, 'Leonarda 3'),
    (4, 8, 'Baza'),
    (5, 2, 'Magdaleny Brzeskiej 32'),
    (6, 3, 'Stefana Okrzei 16'),
    (7, 4, 'Skalista 9'),
    (8, 3, 'Diamentowa 2'),
    (9, 2, 'Agatowa 4'),
    (10, 3, 'Chmielna 11');


-- wypełnienie tabeli Pojazd

insert into pojazd(Pojazd_ID, Przebieg, Garaz_ID, Stan, Marka, Model)
values
	(1, 20001 , 2, 'Sprawny', 'Ford', 'Mndeo'),
    (2, 120540 , 1, 'Uszkodzony', 'Skoda', 'Fabia'),
    (3, 160743 , 8, 'Uszkodzony', 'Skoda', 'Fabia'),
    (4, 29873 , 4, 'Sprawny', 'Mini', 'Countryman'),
    (5, 40291 , 4, 'Sprawny', 'BMW', '330i'),
    (6, 80765 , 3, 'Uszkodzony', 'Ford', 'Mndeo'),
    (7, 30674 , 3, 'Sprawny', 'Ford', 'Fiesta ST'),
    (8, 10786 , 6, 'Niesprawny', 'Mini', 'Cooper S'),
    (9, 56321 , 7, 'Sprawny', 'Mercedes', 'CLA'),
    (10, 12345, 7, 'Sprawny', 'BMW', 'X5'),
    (11, 0 , 4, 'Sprawny', 'BMW', '850i'),
    (12, 0 , 4, 'Sprawny', 'Mercedes', 'C63');

-- wypełnienie tabeli Klient

insert into Klient(Klient_ID, Imie, Nazwisko, email, telefon, Region_Nazwa)
values
	(1, 'Stefan' , 'Byk', 'byk69@gmail.com', 798405377 , 'Sławin'),
    (2, 'Marcin' , 'Straszak', 'marc12@wp.pl', 793058862 , 'Czuby N'),
    (3, 'Arkadiusz' , 'Kulawy', 'ark45@gmail.com', 502935360 , 'Czechow S'),
    (4, 'Bartłomeij' , 'Wspaniały', 'bartekx66@gmail.com', 793508626 , 'Czuby S'),
    (5, 'Bartosz' , 'Cholewka', 'chole88@wp.pl', 695744337 , 'Kalinowszczyzna'),
    (6, 'Ignacy' , 'Krasicki', 'pisarz@wp.pl', 69497146 , 'Konstantynów'),
    (7, 'Czesław' , 'Dziadek', 'czesio30@gmail.com', 501351734 , 'Środmiście'),
    (8, 'Kamil' , 'Nowak', 'kamyk89@gmail.com', 519038791 , 'Wrotków'),
    (9, 'Jan' , 'Kowal', 'janKow44@gmail.com', 512631975 , 'Rury'),
    (10, 'Janusz' , 'Dzik', 'dzik69@gmail.com', 513745673, 'Czechów N');

-- wypełnienie tabeli Adres

insert into adres(kod_pocztowy, Ulica)
values
	(20054, 'Ambonowa 6/5'),
    (20078, 'Fiołkowa 12/5'),
    (20123, 'Ceglana 15/4'),
    (20456, 'Maglowa 9/1'),
    (20789, 'Topazowa 2/67'),
    (20321, 'Poranna 8/11'),
    (20011, 'Wyżynna 7/10'),
    (20097, 'Nizinna 23/3'),
    (20154, 'Bursztynowa 4/25'),
    (20211, 'Lwia 3/15'),
    (20666, 'Pikantna 6/66');

-- wypełnienie tabeli pracownicy

insert into pracownicy(Pracownik_ID, Imie, Nazwisko, Szef_ID, Wynagrodzenie, Pozycja, kod_Pocztowy)
values
	(1,'Bartosz','Danilczuk',NULL,1000000,'CEO',20211),
    (2,'Janusz','Tokot',1,12000,'Manager Placówki',20078),
    (3,'Miłosz','Przestępny',2,5250,'Kierowca Gł.',20054),
    (4,'Elżbieta','Dadejczyk-Maniluk',2,5000,'Recepcjonista Gł.',20123),
    (5,'Kacper','Piłkarz',4,3500,'Recepcjonista',20011),
    (6,'Aleksandra','Nowak',3,2800,'Kierowca',20154),
    (7,'Sebastioan','Zaradny',3,2800,'Kierowca',20097),
    (8,'Janusz','Topies',3,3000,'Kierowca',20789),
    (9,'Dominik','Marcinowski',4,3000,'Kierowca',20456),
    (10,'Steve','Smith',2,4200,'Ochrona',20321),
    (11,'Jesus','Sanchez',4,2200,'Sprzątanie',20666);

-- wypełnienie tabeli uszkodzenia
	
insert into uszkodzenia(Uszkodz_ID, typ)
values
	(1, 'Światła'),
    (2, 'Karoseria'),
    (3, 'Klimatyzacja'),
    (4, 'Układ wydechowy'),
    (5, 'Układ napędowy'),
    (6, 'Skrzynia biegów'),
    (7, 'Układ chłodzenia'),
    (8, 'Koła'),
    (9, 'Układ chamulcowy'),
    (10, 'Opony'),
    (11, 'Elektornika');

-- wypełnienie tabeli pojazd_uszkodzenia

insert into pojazd_uszkodzenia(Pojazd_ID, Uszkodzenia_ID, opis, czy_naprawiony)
values
	(2, 8, 'Wygięta prawa przednia alufelga', true),
    (3, 10, 'Przebita lewa tylnia opona', true),
    (3, 2, 'Lekkie wgniecenie prawego tylniego nadkola', false),
    (1, 1, 'Przepalona żarówka lewego przedniego kierunkowskazu', true),
    (8, 9, 'Przerwany przewód chamulcowy tylniego prawego zacisku', true),
    (10, 4, 'Przebity tłumik', true),
    (2, 11, 'Przepalinie silniczka prawego tylniego okna', false),
    (4, 8, 'Otarte alufelgi po prawej stronie', true),
    (8, 5, 'Przepalone sprzęgło', false),
    (6, 2, 'Zadapanie lewego lusterka oraz tylniego nadkola', true),
    (7, 3, 'Zużyty filtr kabiny', true);

-- wypełnienie tabeli region_pracownicy

insert into region_pracownicy(Region_Nazwa, Pracownik_ID)
values
    ('Czechów N',3),
    ('Czechów S',3),
    ('Środmiście',3),
    ('Czuby N',6),
    ('Czuby S',6),
    ('Rury',6),
    ('Sławin',7),
    ('Konstantynów',7),
    ('Środmiście',8),
    ('Kalinowszczyzna',8),
    ('Czechów S',8),
    ('Wrotków',9),
    ('Czuby S',9),
    ('Czuby N',9);

-- wypełnienie tabeli wypożyczenia

insert into wypożyczenia(Wypo_ID, data_wyp, data_zw, czy_uszkodzony, Pojazd_ID, Klient_ID)
values
	(1, '2020-04-11', '2020-04-15', false, 4, 4),
    (2, '2020-04-12', '2020-04-17', false, 3, 6),
    (3, '2020-04-28', '2020-05-05', false, 9, 1),
    (4, '2020-05-01', '2020-05-05', false, 1, 2),
    (5, '2020-05-02', '2020-05-02', true, 2, 8),
    (6, '2020-05-10', '2020-05-13', true, 6, 5),
    (7, '2020-05-11', '2020-05-18', false, 8, 7),
    (8, '2020-05-11', '2020-05-17', false, 1, 6),
    (9, '2020-05-23', '2020-05-26', true, 8, 1),
    (10, '2020-05-24', '2020-05-28', true, 3, 4);

-- wypełnienie tabeli wypożyczenia_pracownicy

insert into wypożyczenia_pracownicy(Wypo_ID, Pracownik_ID)
values
	(1,4),
    (1,3),
    (2,4),
    (2,7),
    (3,4),
    (4,5),
    (4,6),
    (5,4),
    (5,8),
    (6,5),
    (6,8),
    (7,5),
    (7,3),
    (8,5),
    (8,8),
    (9,4),
    (9,7),
    (10,5),
    (10,9),
    (10,10);