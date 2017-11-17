## 分组TopN


1.
```
SELECT id, kw_type, keyword
FROM tb_keyword
WHERE (
   SELECT COUNT(*) FROM tb_keyword AS f
   WHERE f.kw_type = tb_keyword.kw_type AND f.id <= tb_keyword.id
) <= 10;



```

2.union all
```
(SELECT id,kw_type,keyword FROM tb_keyword WHERE kw_type='object' LIMIT 10)
UNION ALL
(SELECT id,kw_type,keyword FROM tb_keyword WHERE kw_type='operator' LIMIT 10)
UNION ALL
(SELECT id,kw_type,keyword FROM tb_keyword WHERE kw_type='praise' LIMIT 10)
UNION ALL
(SELECT id,kw_type,keyword FROM tb_keyword WHERE kw_type='problem' LIMIT 10)
```