
### Problemas en contabilidad
**Calcular los impuestos de las facturas sin que den error de diferencia credito-debito**

```python
move.with_context(check_move_validity=False, move_reverse_cancel=False)
move._recompute_dynamic_lines(recompute_all_taxes=True)
```

Nota: importante el atributo check_move_validity=False pasado por contexto. Ese es el que evita que realice la validación.

