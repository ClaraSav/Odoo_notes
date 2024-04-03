## Clase Intervals en Odoo para rango de Fechas ###

### Donde se utiliza: ###

- recalculo de entradas de trabajo (hr.work.entry).

### Cómo funciona: ###

1. Para poder utilizarlo, se importa desde el módulo resource
```python
import pytz # libreria que nos permite utilizar zonas horarias
from odoo.addons.resource.models.resource import Intervals
```

2. Se generan los rangos de fecha y hora con la zona horaria correspondiente.

Ejemplo:

```python
# Lista donde se guardarán los rangos de fecha
result = []

# Usando zona horaria UTC para el ejemplo 
tz = pytz.timezone('UTC')

# Aplicamos solo para el empleado con el id = 1 para este ejemplo
for work in self.env['hr.work.entry'].search([('employee_id', '=', 1)]):
    start = work.date_start.astimezone(tz)
    end = work.date_stop.astimezone(tz)
    # por cada intervalo de fecha se coloca una tupla:
    # (fecha_inicio, fecha_fin, registro_de_donde_sale)
    result.append((start, end, work))

# ahora los convertimos en Intervals
intervals_dates = Intervals(result)
```

3. Uso de los Intervals:

Descripcion grafica:
```python
# interval_A = [('4-2-2022 00:00:00', '4-2-2022 18:00:00', hr.work.entry(1,)), ('5-2-2022 00:00:00', '5-2-2022 18:00:00', hr.work.entry(2,))]
# interval_B = [('5-2-2022 00:00:00', '5-2-2022 18:00:00', hr.work.entry(3,)), ('6-2-2022 00:00:00', '6-2-2022 18:00:00', hr.work.entry(4,)),]
```

a. resta de intervalos:
```python
result = interval_A - interval_B

# result = [('4-2-2022 00:00:00', '4-2-2022 18:00:00', hr.work.entry(1,)),]
```

b. interseccion de intervalos:
```python
result = interval_A & interval_B
```

c. union de intervalos:

Este permite que choquen las fechas
```python
result = interval_A | interval_B
```

