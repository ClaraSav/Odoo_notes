
## Agregar una imagen en Excel con xlsxwrite

```python
product_image = io.BytesIO(base64.b64decode(product.image_1980))
sheet.insert_image(row, col, "image.png", {'image_data': product_image})
```

