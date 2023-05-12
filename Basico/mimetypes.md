## Tipos de archivos (Mimetype)

Version 11

para descargar un archivo desde Odoo suele encontrarse con el siguiente ejemplo
```python
return request.make_response(filecontent, [('Content-Type', 'application/vnd.ms-excel'), ('Content-Length', len(filecontent)),
											 ('Content-Disposition', 'attachment; filename= Reporte.xls;')])
```

Donde se descarga un archivo excel.

Dependiendo del tipo de documento a descargar se puede ser mas especifico de acuerdo a su mimetype. 

Acá una recopilació de algunos:
- javascript: 'application/javascript'
- xls: 'application/vnd.ms-excel'
- xlsx: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
- xlsm: 'application/vnd.ms-excel.sheet.macroEnabled.12'
- ies: 'application/octet-stream'
- pdf: 'application/pdf'
- xml: 'application/xml or application/xml;charset=utf-8'
- zip: 'application/zip'
- gif: 'image/gif'
- jpeg: 'image/jpeg'
- png: 'image/png'
- tiff: 'image/tiff'
- eml: 'message/rfc822'
- ics: 'text/calendar'
- css: 'text/css'
- text plain: 'text/plain'
- video mp4: 'video/mp4'

