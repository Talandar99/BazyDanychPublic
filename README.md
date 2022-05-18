### Zdefiniuj wyzwalacz jedna_pozycja, który gwarantuje, że dwóch kandydatów z tej samej partii ni ma tej samej pozycji na liście. 
Treść wyzwalacza z okna SQL wklej jako odpowiedź. Podaj również zapytania, za pomocą których sprawdzasz działanie wyzwalacza
```sql
--code:

begin
	if new.poz_na_liscie IN (Select poz_na_liscie from kandydaci where nr_listy = new.nr_listy) then 
		RAISE EXCEPTION 'dwóch kandydatów z tej samej partii nie może mieć tej samej pozycji na liście. ';
	end if;
	return new;
end;

--do sprawdzania:
    select kandydat, nr_listy, poz_na_liscie from kandydaci;

--nie działa:
    update kandydaci set nr_listy=2 WHERE kandydat='Antecki';

--działa:
    update kandydaci set nr_listy=2137 WHERE kandydat='Antecki';
   

```

### Zdefiniuj funkcje udzial_w_wyborach(nrpunktu), która dla podanego numeru punktu wyborczego poda jaka część uprawnionych głosowała w tym punkcie.
Wykorzystaj funkcję z zapytaniu: Podaj frekwencję w poczególnych punktach wyborczych. Jako odpowiedź wklej definicję funkcji z okna SQL oraz pytania sprawdzające.
```sql
--code

declare
wszyscy integer;
suma integer;
begin
	select into wszyscy lba_uprawnionych from punkty_wyb ;
	select into suma sum(lba_glosow) from glosowanie where nr_punktu = nrpunktu;
	udzial = concat(suma,'/',wszyscy) ;
end;

--sprawdzenie:
select udzial_w_wyborach(3);

```


### Zdefiniuj wyzwalacz czerw_dorosly, który gwarantuje, że każdy kandydat z partii czerwonej ma więcej niż 30 lat. Treść wyzwalacza z okna SQL wklej jako odpowiedź. Podaj również zapytania, za pomocą których sprawdzasz działanie wyzwalacza.
```sql
--code:

declare
wiek integer;
begin
SELECT INTO wiek EXTRACT(YEAR FROM CURRENT_DATE)-NEW.rok_ur;
if wiek<30 AND NEW.nr_listy IN (SELECT nr_listy FROM partie WHERE naz_partii='czerwona') then
    RAISE EXCEPTION 'Kandydaci z partii czerwonej muszą mieć co najmniej 30 lat. Ten ma % lat',wiek;
end if;
return NEW;
end;

--Pytania testowe

--Cydzik zmienia team:
UPDATE kandydaci SET nr_listy=3 WHERE kandydat='Cydzik';

--sprawdzanie zmian:
SELECT *,EXTRACT(YEAR FROM CURRENT_DATE)-rok_ur AS wiek FROM kandydaci;

--Wprowadzanie do bazy
--działa:
INSERT INTO kandydaci VALUES ('Czerwonka2',
(SELECT MAX(nr_listy) FROM partie WHERE naz_partii='czerwona'),
(SELECT MAX(poz_na_liscie) FROM kandydaci)+1,
1970,'wyższe');

--nie działa:
INSERT INTO kandydaci VALUES ('Czerwonka3',
(SELECT MAX(nr_listy) FROM partie WHERE naz_partii='czerwona'),
(SELECT MAX(poz_na_liscie) FROM kandydaci),
2000,'wyższe');


--Zmiana W bazie
--działa:
UPDATE kandydaci SET rok_ur=1960 WHERE kandydat='Cydzik';

--nie działa:
UPDATE kandydaci SET rok_ur=2002 WHERE kandydat='Cydzik';
```
