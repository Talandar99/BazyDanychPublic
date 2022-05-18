### Zdefiniuj wyzwalacz jedna_pozycja, który gwarantuje, że dwóch kandydatów z tej samej partii ni ma tej samej pozycji na liście. Treść wyzwalacza z okna SQL wklej jako odpowiedź. Podaj również zapytania, za pomocą których sprawdzasz działanie wyzwalacza
```sql

```

### Zdefiniuj funkcje udział_w_wyborach(nrpunktu), która dla podanego numeru punktu wyborczego poda jaka część uprawnionych głosowała w tym punkcie. Wykorzystaj funkcję z zapytaniu: Podaj frekwencję w poczególnych punktach wyborczych. Jako odpowiedź wklej definicję funkcji z okna SQL oraz pytania sprawdzające.
```sql
1 próba

SELECT CONCAT(ROUND(CAST(SUM(glosowanie.lba_glosow) AS decimal) / punkty_wyb.lba_uprawnionych,2)*100,'%') * 100 INTO udzial, punkty_wyb.nrpunktu FROM glosowanie NATURAL JOIN punkty_wyb GROUP BY punkty_wyb.nrpunktu, punkty_wyb.lba_uprawnionych ORDER BY nrpunktu;
```


### Zdefiniuj wyzwalacz czerw_dorosly, który gwarantuje, że każdy kandydat z partii czerwonej ma więcej niż 30 lat. Treść wyzwalacza z okna SQL wklej jako odpowiedź. Podaj również zapytania, za pomocą których sprawdzasz działanie wyzwalacza.
```sql
    Wyzwalacz

CREATE TRIGGER wczerw_dorosly

    BEFORE INSERT OR UPDATE 

    ON public.kandydaci

    FOR EACH ROW

    EXECUTE FUNCTION public.czerw_dorosly();



Funkcja

CREATE OR REPLACE FUNCTION public.czerw_dorosly()

    RETURNS trigger

    LANGUAGE 'plpgsql'

    COST 100

    VOLATILE NOT LEAKPROOF

AS $BODY$

declare

wiek integer;



begin

SELECT INTO wiek EXTRACT(YEAR FROM CURRENT_DATE)-NEW.rok_ur;

if wiek<30 AND NEW.nr_listy IN (SELECT nr_listy FROM partie WHERE naz_partii='czerwona') then

    RAISE EXCEPTION 'Kandydaci z partii czerwonej muszą mieć co najmniej 30 lat. Ten ma % lat',wiek;

end if;

return NEW;

end;

$BODY$;



Pytania testowe

-- sprawdzanie zmian

--SELECT *,EXTRACT(YEAR FROM CURRENT_DATE)-rok_ur AS wiek FROM kandydaci;



--działa

/*

INSERT INTO kandydaci VALUES ('Czerwonka2',

(SELECT MAX(nr_listy) FROM partie WHERE naz_partii='czerwona'),

(SELECT MAX(poz_na_liscie) FROM kandydaci)+1,

1970,'wyższe');

*/

--nie działa

/*

INSERT INTO kandydaci VALUES ('Czerwonka3',

(SELECT MAX(nr_listy) FROM partie WHERE naz_partii='czerwona'),

(SELECT MAX(poz_na_liscie) FROM kandydaci),

2000,'wyższe');

*/



--działa

--UPDATE kandydaci SET rok_ur=1968 WHERE kandydat='Czerwonka';



--nie działa

--UPDATE kandydaci SET rok_ur=2002 WHERE kandydat='Czerwonka';
```



