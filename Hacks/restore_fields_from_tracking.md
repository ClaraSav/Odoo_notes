## Restaurar valores de campos eliminados con tracking

El otro dia sucedió un evento en un proyecto que hizo que casi 2000 registros se le borraran un campo personalizado. Por suerte el campo tenia el tracking=True y pudimos recuperar la información. Acá dejo un ejemplo del query que ejecuté para recuperar todos los registros.

En primer lugar es importate saber que el historico de los campos con tracking se guarda en la tabla mail_tracking_value del modelo mail.tracking.value 

En este caso mi modelo se llamaba account.collect.list asi que la consulta es la siguiente:

```sql
UPDATE account_collect_list
SET classification_id = origen.classification_id
FROM (
	SELECT 
		alist.id,
		typi.id AS classification_id,
		value_t.new_value_char
	FROM account_collect_list alist
	JOIN mail_message ON (mail_message.model = 'account.collect.list' AND mail_message.res_id = alist.id)
	JOIN mail_tracking_value value_t ON (value_t.field_desc = 'Tipificación' AND value_t.mail_message_id = mail_message.id)
	JOIN collection_classification typi ON typi.name = value_t.new_value_char
	WHERE alist.classification_id IS NULL AND alist.typification_date IS NOT NULL AND value_t.new_value_char IS NOT NULL AND value_t.new_value_char <> ''
	ORDER BY value_t.create_date
) origen
WHERE account_collect_list.id = origen.id
```