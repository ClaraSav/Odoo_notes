## Consultar un campo traducido por sql

Version 16

Se realiza el mismo tratamiendo de postgres para los json

```sql
SELECT name::json->'es_CL' AS name FROM crm_team;
```

En python quedaría

```python

self.env.cr.execute("SELECT name::json->%s AS name FROM crm_team", (self.env.user.lang,))


```

NOTA: si el idioma no se encuentra entonces retornará None, por eso se puede hacer uso de COALESCE para pasar un idioma por defecto

```sql
COALESCE(name::json->%s, name::json->'en_US')
```