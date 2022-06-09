**De meeste hier kunt ge alleen maar gebruiken bij de plpgsql taal! Dus niet gwn "language sql;" schrijven.**

## Function caching

Plaats deze jongens na de **Language** definition van uw functie.

```sql
--function cannot modify the db and always gives the same results for the same arg values.
IMMUTABLE

--function cannot modify the db and will consistently return the same result for the same arg values.
--BUT its result could change across SQL statements. Bv current_timestamp() omdat  de waarden niet veranderen binnen een transactie
STABLE

--self-explantory I guess, denk aan random(), timeofday(), currval(). Letterlijk elke functie die bijwerkingen heeft, zelfs al zijn ze voorspelbaar moeten gecached worden als volatile!
--ge kunt ze dus ook niet optimaliseren
VOLATILE
```

## Control flow

```sql
FOR i IN 1 ... numtimes LOOP
 statements
END LOOP;

FOR i IN REVERSE numtimes ... 1 LOOP
 statements
END LOOP;

FOR var_e IN EXECUTE('somedynamicsql') LOOP
 statements
 RETURN NEXT var_e;
END LOOP;

FOR var_e IN somesql LOOP
 statements
 RETURN NEXT var_e;
END LOOP;

IF condition THEN
  :
END IF;

IF condition THEN
  :
ELSE
  :
END IF;

IF condition. THEN
  :
ELSIF condition THEN
  :
ELSE
  :
END IF;

WHILE condition LOOP
  :
END LOOP;

LOOP
 -- some computations
 EXIT WHEN count > 100;
 CONTINUE WHEN count < 50;
 -- some computations for count IN [50 .. 100]
END LOOP;
```

## Return constructs

```sql
RETURN somevariable
RETURN NEXT rowvariable
RETURN QUERY
```

## Raise family

```sql
RAISE DEBUG[1-5]
RAISE EXCEPTION
RAISE INFO
RAISE LOG
RAISE NOTICE
```

## Exception Handling

```sql
RAISE EXCEPTION 'Exception notice: %', var
EXCEPTION
 WHEN condition THEN
	--do something or
	--leave blank to ignore
END;
```

## Common state and error constants

```sql
FOUND
ROW_COUNT
division_by_zero
no_data_found
too_many_rows
unique_violation
```

## Variable Setting

```sql
DECLARE
 somevar sometype :=  somevalue;
 somevar sometype
 curs1 refcursor;
 curs2 CURSOR FOR SELECT * FROM sometable;

somevar := somevalue

SELECT field1, field2
INTO somevar1, somevar2
FROM sometable WHERE .. LIMIT 1;
```

## Return types

```sql
RETURNS somedatatype
RETURNS SETOF somedatatype
RETURNS refcursor
RETURNS trigger
RETURNS void
```

## Qualifiers

```sql
(EXTERNAL) SECURITY DEFINER | INVOKER --je gaat bijna nooit Invoker moeten gebruiken hier btw
STRICT
COST cost_metric
ROWS est_num_rows
```

## Beastmaster Examples

"Give me a class of procrastinating students, a md file and an hour and I'll make them pass db2" > Dhr. Michiel Werbrouck 2022 (één dag voor het befaamde db2 examen)
_side note: imagine da ik nu kkr hard buis en de rest is door lmao_

```sql
CREATE OR REPLACE FUNCTION fn_test(param_arg1 integer, param_arg2 text)
  RETURNS text AS
$$
DECLARE
	var_a integer := 0;
	var_b text := 'test test test';
BEGIN
	RAISE NOTICE 'Pointless example to demonstrate a point';
	RETURN var_b || ' - ' ||
		CAST(param_arg1 As text) || ' - '
		|| param_arg2;
END
$$
  LANGUAGE 'plpgsql' STABLE;

SELECT fn_test(10, 'test');

--Example to RETURN QUERY --
CREATE OR REPLACE FUNCTION fnpgsql_get_peoplebylname_key(param_lname text)
  RETURNS SETOF int AS
$$
BEGIN
    RETURN QUERY SELECT name_key
        FROM people WHERE last_name LIKE param_lname;
END
$$
  LANGUAGE 'plpgsql' STABLE;

-- Example using dynamic query
CREATE OR REPLACE FUNCTION cp_addtextfield(param_schema_name text, param_table_name text,
	param_column_name text)
  RETURNS text AS
  $$
  BEGIN
	EXECUTE 'ALTER TABLE ' ||
		quote_ident(param_schema_name) || '.' || quote_ident(param_table_name)
		|| ' ADD COLUMN ' || quote_ident(param_column_name) || ' text ';
	RETURN 'done';
  END;
$$
  LANGUAGE 'plpgsql' VOLATILE;
  SELECT cp_addtextfield('public', 'employees', 'resume');

-- Perform action
 CREATE OR REPLACE FUNCTION cp_updatesometable(param_id bigint,
	param_lname varchar(50), param_fname varchar(50))
  RETURNS void AS
$$
BEGIN
   UPDATE people SET first_name = param_fname, last_name = param_lname
	 WHERE name_key = param_id;
END;
$$
  LANGUAGE 'plpgsql' VOLATILE SECURITY DEFINER;

 --Sample logging trigger taken from docs
 CREATE TABLE emp_audit(
    operation         char(1)   NOT NULL,
    stamp             timestamp NOT NULL,
    userid            text      NOT NULL,
    empname           text      NOT NULL,
    salary integer
);
 CREATE OR REPLACE FUNCTION process_emp_audit() RETURNS TRIGGER AS $$
    BEGIN
        -- Create a row in emp_audit to reflect the operation performed on emp,
        -- make use of the special variable TG_OP to work out the operation.
        IF (TG_OP = 'DELETE') THEN
            INSERT INTO emp_audit SELECT 'D', now(), current_user, OLD.*;
            RETURN OLD;
        ELSIF (TG_OP = 'UPDATE') THEN
            INSERT INTO emp_audit SELECT 'U', now(), current_user, NEW.*;
            RETURN NEW;
        ELSIF (TG_OP = 'INSERT') THEN
            INSERT INTO emp_audit SELECT 'I', now(), current_user, NEW.*;
            RETURN NEW;
        END IF;
        RETURN NULL; -- result is ignored since this is an AFTER trigger
    END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER emp_audit
AFTER INSERT OR UPDATE OR DELETE ON emp
    FOR EACH ROW EXECUTE PROCEDURE process_emp_audit();
```
