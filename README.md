# Testament theme mobile product image issue
Open Templates > product.liquid
Replace this code:

```html
<!-- For Mobile -->
  <div id="mobile-product" class="desktop-12 tablet-6 mobile-3">
    <ul class="bxslider">
      {% assign featured_image = product.selected_or_first_available_variant.featured_image | default: product.featured_image %}
      <li><img data-image-id="{{ image.id }}" src="{{ featured_image | img_url: '1024x1024' }}" alt="{{ image.alt | escape }}"></li>
      {% for image in product.images %}
      <li><img data-image-id="{{ image.id }}" src="{{ image.src | product_img_url: '1024x1024' }}" alt="{{ image.alt | escape }}"></li>
      {% endfor %}      
    </ul>

    <div id="bx-pager" style="display: none">
      {% for image in product.images %}
      <a class="thumbnail" data-slide-index="{{ forloop.index }}" data-image-id="{{ image.id }}" href=""><img src="{{ image.src | product_img_url: 'compact' }}" /></a>
      {% endfor %}
    </div>

  </div>  
```

With this:

```html
  <div id="mobile-product" class="mobile-3">
    <div class="mobile-gallery">
      <ul class="slides">    
        {% for image in product.images %}
        <li data-thumb="{{ image | product_img_url: 'small' }}" data-image-id="{{ image.id }}"><img data-image-id="{{ image.id }}" src="{{ image.src | product_img_url: 'grande' }}" alt="{{ image.alt | escape }}"></li>
        {% endfor %}
      </ul>
    </div>  
  </div> 
```


Then replace this:
```html
      $('.bxslider').bxSlider({
        pagerCustom: '#bx-pager'
      });
```

with this:

```html
    $('.mobile-gallery').flexslider({
      animation: "fade",
      controlNav: false,
      slideshow: false,
      directionNav: true,
      controlNav: "thumbnails"
    }); 
 ```   
 
 Now open: Snippets > short-form.liquid and scroll down to the Select Callback control for variants.
 Carefully replace this: ( normally lines: 89 - 101 ).
 
 ```html
     if (variant && variant.featured_image) {
      var original_image = $("#{{ product.id }}"), new_image = variant.featured_image;
      Shopify.Image.switchImage(new_image, original_image[0], function (new_image_src, original_image, element) {
        
        $(element).parents('a').attr('href', new_image_src);
        $(element).attr('src', new_image_src);   
        $(element).attr('data-image', new_image_src);   
        $(element).attr('data-zoom-image',new_image_src);
		
        $('.thumbnail[data-image-id="' + variant.featured_image.id + '"]').trigger('click');
             
      });
    }
 ```  
 
 with this: 
 
 ```html
 if (variant && variant.featured_image) {
      var original_image = $("#{{ product.id }}"), new_image = variant.featured_image;

	{% if template == 'product' %}
      var $slider = $('.mobile-gallery');
      var original_image = $(".flex-active-slide img"), new_image = variant.featured_image;    
        {% endif %}                         
                             
      Shopify.Image.switchImage(new_image, original_image[0], function (new_image_src, original_image, element) {
        
        $(element).parents('a').attr('href', new_image_src);
        $(element).attr('src', new_image_src);   
        $(element).attr('data-image', new_image_src);   
        $(element).attr('data-zoom-image',new_image_src);
		
        $('.thumbnail[data-image-id="' + variant.featured_image.id + '"]').trigger('click');

	{% if template == 'product' %}        
		$slider.each(function() { $(this).flexslider($('[data-image-id="' + variant.featured_image.id + '"]').data('index')); });
	{% endif %}
             

      });
      
    }
 ``` 
 
 click save and open: Assets > stylesheet.css.liquid and add this to the bottom:

 ```html
 #mobile-product { position: relative; }
ol.flex-control-nav {
    display: none;
}
.flex-direction-nav a { text-align: center; }
 ``` 
 
 Click save when done.
 
 
 Test that images still switch when changed on all devices and double check the console for any JS errors.
