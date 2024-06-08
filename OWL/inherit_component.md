## Modificando un Componente existente de OWL  

### Parte 1: backend

### **Version 16:** ###  

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


### **Version 17:** ###  


```js
/** @odoo-module **/

import { QtyAtDateWidget } from "@sale_stock/widgets/qty_at_date_widget";
import { patch } from "@web/core/utils/patch";


patch(QtyAtDateWidget.prototype, {
    initCalcData() {
        /** Acá lo que se le va a agregar **/
        return this._super();
    },

    /** Acá metodos adicionales **/

});

```


### Parte 2: Punto de Venta

### **Version 16:** ###  

Para heredar del punto de venta es un poco distinto y mas sencillo, debido a que ya existe ```Registries``` que nos facilita bastante.


El siguiente ejemplo nos inicializa el numpadMode (la seleccion de botones cantidad, descuento y precio) como vacio o sin ninguna opcion seleccionada.

```js
odoo.define('custom_pos.models', function (require) {
"use strict";

var { PosGlobalState } = require('point_of_sale.models');
const Registries = require('point_of_sale.Registries');

const CustomPosGlobalState = (PosGlobalState) => class CustomPosGlobalState extends PosGlobalState {
    constructor(obj) {
        super(obj);
        this.numpadMode = '';
    }
}
Registries.Model.extend(PosGlobalState, CustomPosGlobalState);

});
```
En este caso estamos extendiendo un model, para hacerlo con un Componente de OWL es basicamente lo mismo pero cambiando ```Registries.Model.extend``` por ```Registries.Component.extend``` y la cabecera de la herencia seria:

```js
const PosAdyenPaymentScreen = PaymentScreen => class extends PaymentScreen {
        ...
}

```

Los js del punto de venta se cargan en el manifest de la siguiente manera:
```python
'assets': {
    'point_of_sale.assets': [
        'custom_pos/static/src/js/**/*',
    ],
},
```



