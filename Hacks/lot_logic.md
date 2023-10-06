## Lógica de Lotes por javascript para que no se caiga el proceso

Version 13 y 14

Con esta lógica se podrá tratar los procesos que necesiten ser ejecutados por lotes de manera automática.

> Nota: Falta mejorar el tratamiento de errores

### Archivo .js

```javascript
odoo.define('modulo.import_lot', function (require){

var Widget= require('web.Widget');
var widgetRegistry = require('web.widget_registry');
var framework = require('web.framework');
var AbstractAction = require('web.AbstractAction');
var FieldManagerMixin = require('web.FieldManagerMixin');
var core = require('web.core');

var _t = core._t;

// Author: Clara Savelli (AlixKill)
// version: 3.0
//                                       ___-------___
//                                   _-~~             ~~-_
//                                _-~                    /~-_
//             /^\__/^\         /~  \                   /    \
//           /|  O|| O|        /      \_______________/        \
//          | |___||__|      /       /                \          \
//          |          \    /      /                    \          \
//          |   (_______) /______/                        \_________ \
//          |         / /         \                      /            \
//           \         \^\\         \                  /               \     /
//             \         ||           \______________/      _-_       //\__//
//               \       ||------_-~~-_ ------------- \ --/~   ~\    || __/
//                 ~-----||====/~     |==================|       |/~~~~~
//                  (_(__/  ./     /                    \_\      \.
//                         (_(___/                         \_____)_)
// ************************************************************************************

var LotProcess = AbstractAction.extend({
    init: function(parent, action) {
        this._super.apply(this, arguments);
        this.model = action.context.active_model || false;
        this.method = action.context.method || false;
        this.redirect = action.context.redirect || false;
        // si se va a ejecutar en la vista tree entonces se seleccionan los active_ids por contexto
        this.active_ids = action.context.active_ids || [];
    },
    start: async function() {
        await this._super(...arguments);
        this.callGenerate();
    },
    generate_options: function() {
        // the options parameter contains useful data to be used on the python side
        // everything you need can go through here
        var self = this;
        var options = {
            // start at row 1 = skip 0 lines
            skip: 0,
            limit: 50,
            percentage: 0,
        };

        return options;
    },
    callGenerate: function (kwargs) {
        var opts = this.generate_options();

        this.trigger_up('with_client', {callback: function () {
            this.loading.ignore_events = true;
        }});

        return this._batchedGenerate(opts, kwargs, {done: 0, prev: 0})
            .then(null, function (reason) {
                var error = reason.message;
                var event = reason.event;
                // In case of unexpected exception, convert
                // "JSON-RPC error" to an failure, and
                // prevent default handling (warning dialog)
                if (event) { event.preventDefault(); }

                var msg;
                var errordata = error.data || {};
                if (errordata.type === 'xhrerror') {
                    var xhr = errordata.objects[0];
                    switch (xhr.status) {
                    case 504: // gateway timeout
                        msg = _t("Generate timed out. Please retry.");
                        break;
                    default:
                        msg = _t("An unknown issue occurred during generared (possibly lost connection, data limit exceeded or memory limits exceeded). Please retry in case the issue is transient.");
                    }
                } else {
                    msg = errordata.arguments && (errordata.arguments[1] || errordata.arguments[0])
                        || error.message;
                }

                return Promise.resolve({'messages': [{
                    type: 'error',
                    record: false,
                    message: msg,
                }]});
            });
    },

    _batchedGenerate: function (opts, kwargs, rec){
        // this method is the one that is executed in each batch
        opts.callback && opts.callback(rec.done || 0);
        var self = this;
        framework.blockUI({
                        'message': '<div class="oe_blockui_spin" style="height: 50px"> ' +
                                        _t('<img src="/web/static/src/img/spin.png" style="animation: fa-spin 1s infinite steps(12);" alt="Loading..."/>') +
                                    '</div>' +
                                    '<br />' +
                                    _t('<div class="oe_throbber_message" style="color:white">Progreso: ') + opts.percentage.toFixed(2).toString() + '% </div>'
                    });

        // the id of the record where the method will be executed is passed. and the opts parameter
        return this._rpc({
            model: self.model,
            method: self.method,
            args: [self.active_ids, opts], // si no se ejecuta en la vista tree entonces no se pasa el self.active_ids
            context: self.res_context,
        }).then(function (results) {
            framework.unblockUI(); // screen locks every time a block ends
            if (!results.continues) {
                // we're done
                return self.do_action(self.redirect, {});
            }
            // do the next batch
            // the percentage is received from the python method
            return self._batchedGenerate(
                _.extend(opts, {percentage: results.percentage, skip: results.skip}), kwargs, {
                    done: rec.done + results.qty,
                }
            ).then(function (r2) {
                framework.unblockUI();
                return {
                    qty: results.qty + r2.qty,
                    messages: results.messages.concat(r2.messages),
                    continues: r2.continues
                }
            });

        });
    },

});

core.action_registry.add('import_process', LotProcess);
return LotProcess;


});

```



### **Python code**

Se define una accion de cliente que bien puede ser por xml o por python como lo hice en este caso

Es importante pasar por contexto:
- el modelo donde se encontrará la función a ejecutar en cada lote
- el metodo a ejecutar
- una accion de de ventana que se ejecutará cuando finalice todos los lotes

```python
    def execute_import(self):
        return {
            'type': 'ir.actions.client',
            'name': 'GUARDAR STOCK',
            'tag': 'import_process',
            'context': {'model': 'modelo.con.funcion', 'method': 'import_stock', 'redirect': 'modulo.action_a_redirigir'},
        }
```

Posteriormente Tendremos que redefinir la funcion que ejecutaremos por cada lote, acá un ejemplo:

```python
    @api.model
    def import_stock(self, options):
        count_records = self.env['product.template'].search_count([])

        limit = options.get('limit')
        skip = options.get('skip')
        records = self.env['product.template'].search([], offset=skip, limit=limit)
        stop = 0  # si está en 0 entonces termina el proceso
        if records:
            # aca va la logica para todos los records
            stop = 1  # 1 para continuar con el siguiente lote
        percentage = (skip + limit) * 100 / count_records if count_records else 100
        return {
            'qty': limit,
            'messages': [],
            'continues': stop,
            'skip': skip + limit,
            'percentage': percentage,
        }
```


> Important: Debe colocarse el api.model para que no pida un id de registro al llamar a la función.

En dado caso que se agregue un boton en la vista tree entonces se puede pasar los registros seleccionados al metodo (verificar notas de active_ids en el codigo de javascript)

```python
 def import_stock(self, records, options):
```

