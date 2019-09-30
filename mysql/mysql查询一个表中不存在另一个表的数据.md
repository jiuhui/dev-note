



```sql
SELECT * FROM A 
WHERE  id  NOT  IN  ( SELECT id FROM B);

或者
SELECT * FROM A 
WHERE 
    NOT  EXISTS  ( 
        SELECT 1 
        FROM B 
        WHERE B.id = A.id );

或者
SELECT 
  A.* 
FROM 
  A  LEFT JOIN B
    ON (A.id = B.id)
WHERE
  b.id  IS  NULL
```

最好用最后一种，可以分页