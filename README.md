# CanisHub Opencart3 Integration #

This contains all the steps necessary to integarte CanisHub with Opencart-3

### Integration Steps  ###

* Add reference of CanisHub SDK JS file
* Update view template files
* Update controller files
* Update model files

### Add reference of CanisHub SDK JS file
Request CanisHub support team for integration javascript file and save it as *ch.js* at
```
<opencart3-installation-location>/catalog/view/javascript/ch.js
```
Search for: *$data['scripts'] = $this->document->getScripts('footer')* add follwoing snippet before that
```php
$this->document->addScript('catalog/view/javascript/ch.js');
```

### Update view template files
Include CanisHub code snippets into opencart view templte files

#### Update Home View Template
Add following code snippet to */catalog/view/theme/<theme-folder>/template/common/home.twig* file above *{{ footer }}*
```
<!-- CANISHUB Widgets START-->
<div class='canishub-index-widgets'>
  <div class='canishub-index-first-buyers'></div>
  <div class='canishub-index-user-interest'></div>
  <div class='canishub-index-recent-prod'></div>
</div>
<script>
ch_pub_sub_events.publish('index_page_view_before', {'item_id':'index'});
</script>
<!-- CANISHUB Widgets END-->
```

#### Update Product View Template
Add following code snippet to */catalog/view/theme/<theme-folder>/template/product/product.twig* file above *{{ footer }}*
```
<!-- CANISHUB START-->
<div class='canishub-pdp-widgets'>
  <div class='canishub-pdp-similar-prod-sorted'></div>
  <div class='canishub-pdp-cross-sell'></div>
  <div class='canishub-pdp-similar-prod'></div>
  <div class='canishub-pdp-bought-together'></div>
  <div class='canishub-pdp-also-bought'></div>
  <div class='canishub-pdp-collaborative-prod-view'></div>
  <div class='canishub-pdp-recent-prod'></div>
</div>

<script>
  var _special_char_format = /[!@#$%^*()_+\=\[\]{}':"\\|,<>\/?]+/;
  var _categories  = []
  var _variants    = []

  {% for option in options %}
    {% for option_value in option.product_option_value %}
      _variants.push({id:'{{ option_value.product_option_value_id }}',title:'{{ option_value.name }}'})
    {% endfor %}
  {% endfor %}

  {% for product_category in ch_product_categories %}
    _categories.push('{{product_category.name}}')
  {% endfor %}

  var _item = {
    item_id: '{{ product_id }}',
    title: '{{heading_title}}'.replace(/&quot;/g,'').replace(/[&\/\\#,+()$~%.'":*?<>{}]/g,' '),
    categories: _categories,
    description: '',
    product_url: window.location.origin + '{{product.url}}',
    image_url: '{{ popup }}',
    condition:  "new",
    availability: '{{ stock }}',
    price: '{{ price }}'.replace(/[^\d\.\,\s]+/g, ""),
    sale_price: ('{{ special }}' || '{{ price }}').replace(/[^\d\.\,\s]+/g, ""),
    brand: '{{ manufacturer }}',
    base_category: _categories[0],
    collection_name: '{{ model }}',
    base_categories: [_categories[0]],
    categories_1: '{{ model }}' ? ['{{ model }}'] : [],
    categories_2: '{{ manufacturer }}' ? ['{{ manufacturer }}'] : [],
    sku: '{{ product_id }}',
    variants: _variants
  };
  ch_pub_sub_events.publish('product_view_before', {'item_id':_item.item_id, 'item':_item});

  $('#button-cart').click(function(){
    ch_pub_sub_events.publish('add_to_cart_before', {'item_id':_item.item_id, 'item':_item});
  })
</script>
<!-- CANISHUB END-->
```

#### Update Category View Template
Add following code snippet to */catalog/view/theme/<theme-folder>/template/product/category.twig* file above *{{ footer }}*
```
<!-- CANISHUB Widgets START-->
<div class='canishub-collection-widgets'>
  <div class='canishub-collection-trending'></div>
</div>
<script>
ch_pub_sub_events.publish('collection_page_view_before', {'item_id':'{{heading_title}}'});
</script>
<!-- CANISHUB Widgets END-->
```
#### Update Cart View Template
Add following code snippet to */catalog/view/theme/<theme-folder>/template/checkout/cart.twig* file above *{{ footer }}*
```
<!-- CANISHUB Widgets START-->
<div class='canishub-cart-widgets'>
  <div class='canishub-cart-also-bought'></div>
  <div class='canishub-cart-bought-together'></div>
  <div class='canishub-cart-recent-prod'></div>
</div>

<script>
  let _item_ids = [];
  let _items = [];
  let _total_value = '{{totals[0].text}}'.replace(/[^\d\.\,\s]+/g, "")

  {% for product in products %}
    _items.push({
      id:'{{product.cart_id}}',
      name:'{{product.name}}',
      quantity:{{ product.quantity }},
      price:'{{product.price}}'.replace(/[^\d\.\,\s]+/g, ""),
      line_price:'{{product.total}}'.replace(/[^\d\.\,\s]+/g, ""),
      sku:'{{product.cart_id}}',
    });
    _item_ids.push('{{product.cart_id}}');

  {% endfor %}

  ch_pub_sub_events.publish('checkout_initiate_before', { item_id: "cart",item_ids: _item_ids,items: _items,total_value: _total_value});

</script>
<!-- CANISHUB Widgets END-->
```

### Update Checkout-Success View Template
Add following code snippet to */catalog/view/theme/<them-folder>/template/common/success.twig* file above *{{ footer }}*
```
<!-- CANISHUB START-->
<div class='canishub-thankyou-widgets'>
  <div class='canishub-thankyou-also-bought'></div>
  <div class='canishub-thankyou-bought-together'></div>
  <div class='canishub-thankyou-recent-prod'></div>
</div>
<script>
  let _ch_order_items = []
  let _ch_order_item_ids = []
  {% for product in ch_order_products %}
    _ch_order_items.push({
      item_id: '{{ product.product_id }}',
      name: '{{product.name}}'.replace(/&quot;/g,'').replace(/[&\/\\#,+()$~%.'":*?<>{}]/g,' '),
      price: '{{ product.price }}'.replace(/[^\d\.\,\s]+/g, ""),
      line_price: '{{ product.price }}'.replace(/[^\d\.\,\s]+/g, ""),
      sku: '{{ product_id }}',
      variant_id: '{{ product_id }}'
    })
    _ch_order_item_ids.push('{{ product.product_id }}')
  {% endfor %}

  ch_pub_sub_events.publish('purchase_complete_before', {
  	item_id: "thankyou",
  	order_id: '{{ch_order_details.order_id}}',
  	order_price: '{{ch_order_details.total}}',
  	order_shipping_zip: '{{ch_order_details.shipping_postcode}}',
  	order_shipping_city: '{{ch_order_details.shipping_city}}',
  	payment_transactions:  [{
  	    amount: '{{ch_order_details.total}}',
  	    gateway: '{{ch_order_details.payment_code}}',
  	    status:"pending"
  	}],
  	coupons:  [],
  	item_ids: _ch_order_item_ids,
  	items: _ch_order_items,
  	customer: {
  		id: '{{ch_order_details.customer_id}}',
  		f_name: '{{ch_order_details.firstname}}',
  		l_name: '{{ch_order_details.lastname}}',
  		email: '{{ch_order_details.email}}',
  		contact_no: '{{ch_order_details.telephone}}'
  	}
  })
</script>
<!-- CANISHUB END-->
```

### Update controller files
Include CanisHub code snippets into opencart controller files

#### Update Product Controller File
Open *catalog/controller/product/product.php* add following shippet after *$this->load->model('catalog/product');*
```php
$data['ch_product_categories'] = $this->model_catalog_category->ch_get_product_categories($product_id);
```
#### Update Checkout-Success Controller File
Open *catalog/controller/checkout/success.php* addd following shipper before *$this->cart->clear();*
```php
$data['ch_order_details'] = $this->model_checkout_order->getOrder($this->session->data['order_id']);
$data['ch_order_products'] = $this->model_checkout_order->getOrderProducts($this->session->data['order_id']);
```

### Update model files
Include CanisHub code snippets into opencart model files

#### Update Category Model File
Open *catalog/model/catalog/category.php* add this new method
```php
public function ch_get_product_categories($product_id) {
     $query = $this->db->query("SELECT DISTINCT(cd.name),p2c.product_id,cd.category_id FROM " . DB_PREFIX . "product_to_category p2c, " . DB_PREFIX . "category_description cd WHERE p2c.product_id = '".$product_id."' AND cd.category_id = p2c.category_id AND cd.category_id != 1");
     return $query->rows;
}

```
