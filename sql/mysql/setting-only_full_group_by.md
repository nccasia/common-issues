### Issue in MySQL
```
Expression #1 of SELECT list is not in GROUP BY clause and contains nonaggregated column 'db.table.col' which is not functionally dependent on columns in GROUP BY clause; this is incompatible with sql_mode=only_full_group_by
```

### When enabled
Reject queries for which the select list, HAVING condition, or ORDER BY list refer to nonaggregated columns that are neither named in the GROUP BY clause nor are functionally dependent on (uniquely determined by) GROUP BY columns.

For example, when this setting is enabled, the following query is not valid because the nonaggregated `address` column in the select list is not named in the `GROUP BY` clause:
```
SELECT name, address, MAX(age) FROM t GROUP BY name;
```

### Check value
  ```
  SELECT @@sql_mode;
  ```

### To enable
- Use command:
  ```
  SET GLOBAL sql_mode=(SELECT CONCAT(@@sql_mode, ',ONLY_FULL_GROUP_BY'));
  ```
- OR modify the config file `my.cnf`. Usually it’s in `/etc/my.cnf` or `/etc/mysql/my.cnf` to add ONLY_FULL_GROUP_BY to setting `sql_mode` value
  ```
  sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION,ONLY_FULL_GROUP_BY
  ```

### To disable
- Use command: 
  ```
  SET GLOBAL sql_mode=(SELECT REPLACE(@@sql_mode,'ONLY_FULL_GROUP_BY',''));
  ```
- OR modify the config file `my.cnf`. Usually it’s in `/etc/my.cnf` or `/etc/mysql/my.cnf` to remove ONLY_FULL_GROUP_BY from setting `sql_mode` value 
  ```
  sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_ENGINE_SUBSTITUTION
  ```

### Recommended
Follow the SQL/92 standard for GROUP BY:
- A column used in an expression of the SELECT clause must be in the GROUP BY clause. Otherwise, the expression using that column is an aggregate function.
- A GROUP BY expression can only contain column names from the select list, but not those used only as arguments for vector aggregates.

In MySQL when the setting ONLY_FULL_GROUP_BY is disabled, the server is free to choose any value from each group, so unless they are the same, the values chosen are nondeterministic, so it might not be what you want.

### Reference
https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by

https://dev.mysql.com/doc/refman/5.7/en/group-by-handling.html