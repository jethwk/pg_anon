# README #


### Installation ###
```python
apt install -y python3-pip
pip3 install -r requirements.txt
```

```sql
psql -c "CREATE USER anon_test_user WITH PASSWORD 'mYy5RexGsZ' SUPERUSER;" -U postgres

```

```bash
set TEST_DB_USER=anon_test_user
set TEST_DB_USER_PASSWORD=mYy5RexGsZ
set TEST_DB_HOST=127.0.0.1
set TEST_DB_PORT=5432
set TEST_SOURCE_DB=test_source_db
set TEST_TARGET_DB=test_target_db
```


```python
python3 test/full_test.py -v
>>
	Ran N tests in ...
	OK

# If all tests is OK then application ready to use

```


### How to escape/unescape complex names of objects ###

```python
python3

import json
j = {"k": "_TBL.$complex#имя;@&* a'2"}
json.dumps(j)
>>
	'{"k": "_TBL.$complex#\\u0438\\u043c\\u044f;@&* a\'2"}'

s = '{"k": "_TBL.$complex#\\u0438\\u043c\\u044f;@&* a\'2"}'
u = json.loads(s)
print(u['k'])
>>
	_TBL.$complex#имя;@&* a'2

```

### Generate dictionary by table rows ###


```sql
select
	jsonb_pretty(
		json_agg(json_build_object('schema', T.schm, 'table', T.tbl, 'fields', flds ))::jsonb
	)
from (
    select
        T.schm,
        T.tbl,
        JSON_OBJECT_AGG(fld, mrule) as flds
    from (
        select 'schm_1' as schm, 'tbl_a' as tbl, 'fld_1' as fld, 'md5(fld_1)' as mrule
        union all
        select 'schm_1', 'tbl_a', 'fld_2', 'md5(fld_2)'
        union all
        select 'schm_1','tbl_b', 'fld_1', 'md5(fld_1)'
        union all
        select 'schm_1','tbl_b', 'fld_2', 'md5(fld_2)'
    ) T
    group by schm, tbl
) T
>>
	[
	    {
	        "table": "tbl_b",
	        "fields": {
	            "fld_1": "md5(fld_1)",
	            "fld_2": "md5(fld_2)"
	        },
	        "schema": "schm_1"
	    },
	    {
	        "table": "tbl_a",
	        "fields": {
	            "fld_1": "md5(fld_1)",
	            "fld_2": "md5(fld_2)"
	        },
	        "schema": "schm_1"
	    }
	]

```