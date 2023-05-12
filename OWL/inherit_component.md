## Vista Formulario Odoo

Version 16

Ejemplo realizado con el componente QtyAtDateWidget del modulo de sale_stock

```js
/** @odoo-module **/

import { QtyAtDateWidget } from '@sale_stock/widgets/qty_at_date_widget';

import { patch } from 'web.utils';

const components = { QtyAtDateWidget };

patch(components.QtyAtDateWidget.prototype, 'qty_at_date_widget', {
    initCalcData() {
        /** Acá lo que se le va a agregar **/
        return this._super();
    }

});


```

La primera linea del archivo es `/** @odoo-module **/`

para importar el Componente que vamos a heredar especificamos la ruta de este en el modulo.
Como es el caso de `'@sale_stock/widgets/qty_at_date_widget'`. 
Nota: Por alguna razón se obvia la carpeta src en la ruta.

Para poder heredarlo, necesitamos importar patch desde web.utils.

En el manifest de nuestro modulo tendremos que agregar los assets de la siguiente forma

```python
'assets': {
    'web.assets_backend': [
        'modulo_ejemplo/static/src/js/*',
    ],
},
```

Se puede colocar la ruta especifica. En este caso se colocó el * para agregar todos los archivos encontrados.
