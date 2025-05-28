# HTML & CSS

## HTML

* 自動Refresh (url為可選參數)

  ```HTML
  <meta http-equiv="refresh" content="5; url=https://www.fooish.com">
  ```

* Custom Attribute

- 以`data-`為前綴，可儲存string資料
- 注意attribute name限定為"**全小寫!**"
- jQuery: 
  - `.data("myattribute")` or `.attr("data-myattribute")`皆可用
  - Set: `.data("myattribute", "new value")` or `.attr("data-myattribute", "new value")`

```HTML
<!DOCTYPE html>
<html lang="en">
<head></head>
<body>
    <div id="product" data-product-id="12345" data-product-name="Laptop" data-product-price="999.99">
        Product Information
    </div>

    <script>
        $(document).ready(function() {
            // Accessing data attributes using jQuery
            var productDiv = $('#product');
            var productId = productDiv.data('product-id');
            var productName = productDiv.data('product-name');
            var productPrice = productDiv.data('product-price');

            console.log('Product ID: ' + productId);
            console.log('Product Name: ' + productName);
            console.log('Product Price: ' + productPrice);

            // Modifying data attributes using jQuery
            productDiv.data('product-price', '899.99');
            console.log('Updated Product Price: ' + productDiv.data('product-price'));
        });
    </script>
</body>
</html>
```

# CSS
* SCSS
  * gulp & node-sass 自動偵測變更compile成.css
* [如何使用CSS的「text-overflow: ellipsis;」屬性限制內容字數？](https://www.astralweb.com.tw/css-ellipsis/)
