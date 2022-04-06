### W punktach wyborczych na ulicy Dębowej i Akacjowej liczba uprawnionych zmalała o 10%.
```sql
Update punkty_wyb set lba_uprawnionych = lba_uprawnionych*0.9 
where adres_punktu like 'Dębowa%' or adres_punktu like 'Akacjowa%'
```
### Nazwiska i wiek kandydatów, dla których nie podano wyników fłosowania
```sql
select kandydaci.kandydat, DATE_PART('year',current_date)-rok_ur as Wiek_kandydata from kandydaci 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
Where lba_glosow is null
```
### Nazwisko i nazwa partii kandydatów z  wykształceniem średnim, którzy w punkcie zlokalizowanym na ulicy Dębowej  uzyskali więcej niż  300 głosów. Podaj uporządkowane alfabetycznie wg nazwisk.
```sql
Select Distinct kandydaci.kandydat, naz_partii from kandydaci 
left join partie on kandydaci.nr_listy=partie.nr_listy
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat
left join punkty_wyb on punkty_wyb.nr_punktu=glosowanie.nr_punktu
where wyksztalcenie = 'średnie' and adres_punktu like 'Dębowa%' and lba_glosow > 300
order by kandydat;
```
### Zdefiniuj więzy integralności referencyjnej między tabelami  kandydaci - glosowanie. W razie trudności, znajdź przyczyny i usuń problemy budując odpowiednie zapytania. Zbudowane pytania wklej jako odpowiedź. Wklej także definicje (zakładka SQL) zmienionych tabel.
```sql
--Januszko jest problemem jego dane zostały wprowadzone kilka razy
--Zakładamy że były to ostatnie głosy i ktoś wpisał to kilka razy przez przypadekwięc 
--bierzemy tylko najwyższą wartość
Select Max(lba_glosow) from glosowanie where kandydat = 'Januszko' and nr_punktu = '6';
Delete from glosowanie where kandydat = 'Januszko' and nr_punktu = '6' and (lba_glosow<218);

--więzy integralności referencyjnej między tabelami  kandydaci - glosowanie. 
--ALTER TABLE public.glosowanie
--    ADD CONSTRAINT pk_glosowanie PRIMARY KEY (kandydat, nr_punktu);

-- do poprawki
-- ALTER TABLE public.kandydaci
--     ADD CONSTRAINT pk_kandydaci PRIMARY KEY (kandydat);
-- ALTER TABLE public.kandydaci
--     ADD CONSTRAINT fk_kandydaci FOREIGN KEY (kandydat)
--     REFERENCES public.glosowanie (kandydat) MATCH SIMPLE
--     ON UPDATE NO ACTION
--     ON DELETE NO ACTION
--     NOT VALID;


--CREATE TABLE public.glosowanie
--(
--    kandydat character varying(20) COLLATE pg_catalog."default" NOT NULL,
--    nr_punktu integer NOT NULL,
--    lba_glosow integer,
--    CONSTRAINT pk_glosowanie PRIMARY KEY (kandydat, nr_punktu)
--)
```
### Średni wiek  kandydatów z poszczególnych  partii powstałych po 1980 roku
```sql
select naz_partii , avg(DATE_PART('year',current_date)-rok_ur) 
from kandydaci 
natural join partie 
where rok_zalozenia > '1980' 
group by naz_partii;
```

### Dopisz do bazy informację o kandydacie Czerwionke z partii czerwonej urodzonym w 1977 roku,  z wykształceniem średnim, który ma na swojej liście partyjnej pozycję  6
```sql
INSERT INTO kandydaci (kandydat, nr_listy, poz_na_liscie, rok_ur, wyksztalcenie) 
VALUES ('Czerwionka', 3, 6, 1977, 'średnie' );
```
### Nazwy partii, z których kandydaci urodzeni po roku 1960 uzyskali łącznie mniej niż 1000 głosów
```sql
select  SUM(lba_glosow), naz_partii from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
where rok_ur>1960 
group by naz_partii
having SUM(lba_glosow)<1000;
```
### Adresy punktów wyborczych, w których kandydat   Czesławski  uzyskał więcej  głosów niż w punkcie numer 3
```sql
select adres_punktu from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
left join punkty_wyb on glosowanie.nr_punktu=punkty_wyb.nr_punktu 
where kandydaci.kandydat = 'Czesławski' and lba_glosow > (select lba_glosow from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
where kandydaci.kandydat = 'Czesławski' and nr_punktu = '3');
```
### Przeciętny wiek (dziś, currrent_date) kandydata na  poszczególnych listach  (nr_listy, średni wiek kandydata)
```sql
select nr_listy , avg(DATE_PART('year',current_date)-rok_ur) as sredni_wiek
from kandydaci 
group by nr_listy;
```
### Zdefiniuj warunek ograniczający: Punkty zlokalizowane w biurach obsługują co najwyżej 1000 uprawnionych
```sql
CREATE TABLE public.punkty_wyb
(
    nr_punktu integer,
    adres_punktu character varying(20) COLLATE pg_catalog."default",
    przewkomisji character varying(20) COLLATE pg_catalog."default",
    miejsce character varying(20) COLLATE pg_catalog."default",
    lba_uprawnionych integer,
    CONSTRAINT limit_under_1000 CHECK (lba_uprawnionych <= 1000 OR miejsce::text <> 'biuro'::text)
)
```
### Nazwy, przewodniczący partii założonych przed rokiem 1980 z liczbą członków mniejszą niż 100000
```sql
select naz_partii, przewodniczacy from partie 
where rok_zalozenia < '1980' and liczba_czlonkow < '100000';
```
### Jak układały się udziały (podaj w postaci ułamków) poszczególnych partii w sumie głosów oddanych w punkcie wyborczym na  Bajkowej
```sql
select Concat (SUM(lba_glosow),'/', 
	(select SUM(lba_glosow) as lba_glosow_sum from partie 
	left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
	natural join glosowanie
	left join punkty_wyb on glosowanie.nr_punktu=punkty_wyb.nr_punktu
	where adres_punktu like 'Bajkowa%'))
as lba_glosow_sum, naz_partii from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
natural join glosowanie
left join punkty_wyb on glosowanie.nr_punktu=punkty_wyb.nr_punktu 
where adres_punktu like 'Bajkowa%'
group by naz_partii; 
```
### Dopisz do bazy, że kandydat Niebieski uzyskał w punkcie 4 576 głosów
```sql
INSERT INTO glosowanie (kandydat, nr_punktu, lba_glosow) VALUES ('Niebieski', 4, 576);
```
### Zdefiniuj więzy integralności referencyjnej między tabelami kandydaci-partie. W razie trudności, znajdź przyczyny i usuń problemy budując odpowiednie zapytania. Zbudowane pytania wklej jako odpowiedź. Wklej także definicje (Zakładka SQL) zmienionych tabel
```sql
--niektórzy kandydaci nie mają partii więc trzeba ich usunąć
delete from kandydaci where nr_listy not in (select nr_listy from partie);

CREATE TABLE public.kandydaci
(
    kandydat character varying(20) COLLATE pg_catalog."default" NOT NULL,
    nr_listy integer,
    poz_na_liscie integer,
    rok_ur integer,
    wyksztalcenie character varying(20) COLLATE pg_catalog."default",
    CONSTRAINT pk_kandydaci PRIMARY KEY (kandydat),
    CONSTRAINT fk_kandydaci FOREIGN KEY (nr_listy)
        REFERENCES public.partie (nr_listy) MATCH SIMPLE
        ON UPDATE RESTRICT
        ON DELETE RESTRICT
)

CREATE TABLE public.partie
(
    nr_listy integer NOT NULL,
    naz_partii character varying(20) COLLATE pg_catalog."default",
    rok_zalozenia integer,
    przewodniczacy character varying(20) COLLATE pg_catalog."default",
    liczba_czlonkow integer,
    CONSTRAINT partie_pkey PRIMARY KEY (nr_listy)
)
```
### Po ile głosów uzyskali poszczególni kandydaci z partii białej
```sql
select  SUM(lba_glosow) from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
where naz_partii = 'biała'
```
### Wiek najmłodszego kandydata w każdej partii (naz_partii, wiek najmłodszego kandydata)
```sql
select naz_partii , min(DATE_PART('year',current_date)-rok_ur) 
from kandydaci 
natural join partie 
group by naz_partii;
```
### W których punktach wyborczych (numery) głosowano na kandydatów nie pochodzących z oficjalnej listy kandydatów. Podaj też nazwiska takich kandydatów
```sql
select adres_punktu, glosowanie.kandydat from punkty_wyb
natural join glosowanie
left join kandydaci on kandydaci.kandydat=glosowanie.kandydat where kandydaci.poz_na_liscie is null
```
### jak nazywał się kandydat, który w punkcie nr 2 uzyskał największą liczbę głosów
```sql
select glosowanie.kandydat from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
where nr_punktu = '2' and lba_glosow=
(select max (lba_glosow) from partie 
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
left join glosowanie on kandydaci.kandydat=glosowanie.kandydat 
where nr_punktu = '2');
```
### Nazwy partii, któe wystawiły przynajmniej 3 kandydatów z wykształceniem wyższym lub średnim
```sql
select naz_partii from partie
left join kandydaci on kandydaci.nr_listy=partie.nr_listy 
where wyksztalcenie = 'średnie' or wyksztalcenie = 'wyższe'
group by naz_partii
having count(kandydat) >='3';
```
### Zdefiniuj warunek ograniczający: Minimalny i maksymalny wiek kandydata to: 35 do 75 lat
```sql
CREATE TABLE public.kandydaci
(
    kandydat character varying(20) COLLATE pg_catalog."default" NOT NULL,
    nr_listy integer,
    poz_na_liscie integer,
    rok_ur integer,
    wyksztalcenie character varying(20) COLLATE pg_catalog."default",
    CONSTRAINT pk_kandydaci PRIMARY KEY (kandydat),
    CONSTRAINT fk_kandydaci FOREIGN KEY (nr_listy)
        REFERENCES public.partie (nr_listy) MATCH SIMPLE
        ON UPDATE RESTRICT
        ON DELETE RESTRICT,
    CONSTRAINT min_max CHECK ((date_part('year'::text, CURRENT_DATE) - rok_ur::double precision) >= 35::double precision AND (date_part('year'::text, CURRENT_DATE) - rok_ur::double precision) <= 75::double precision)
)
```
### Nazwiska kandydatów oraz adresy i miejsca punktów wyborczych z liczbą uprawnionych powyżej 2000 w których głosowano na kandydatów z partii której przewodzi Czerwiński. Podaj uporządkowane według adresów punktów i nazwisk kandydatów.
```sql
select glosowanie.kandydat, adres_punktu, miejsce, lba_uprawnionych, przewodniczacy from punkty_wyb
left join glosowanie on glosowanie.nr_punktu=punkty_wyb.nr_punktu
left join kandydaci on kandydaci.kandydat=glosowanie.kandydat
left join partie on kandydaci.nr_listy=partie.nr_listy
where lba_uprawnionych >2000 and przewodniczacy='Czerwiński' order by adres_punktu,glosowanie.kandydat
-- XD kurwa
```
### Nazwisko, wiek w bierzącym roku kandydatów z listy numer 1 z wykształceniem zawodowym lub wyższym
```sql
select kandydat, DATE_PART('year',current_date)-rok_ur as Wiek from kandydaci 
where nr_listy = '1' and wyksztalcenie in ('zawodowe','wyższe')
```
