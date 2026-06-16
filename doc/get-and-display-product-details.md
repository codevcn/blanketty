Đây là code Liquid mẫu đầy đủ để hiển thị thông tin chi tiết sản phẩm:

---

## `templates/product.liquid` (hoặc `sections/product-info.liquid`)

```liquid
{% assign product = product %}

<div class="product-detail">

  <!-- ẢNH SẢN PHẨM -->
  <div class="product-images">
    {% for image in product.images %}
      <img
        src="{{ image | image_url: width: 800 }}"
        alt="{{ image.alt | escape }}"
        width="800"
        loading="{% if forloop.first %}eager{% else %}lazy{% endif %}"
      >
    {% endfor %}
  </div>

  <!-- THÔNG TIN CHÍNH -->
  <div class="product-info">

    <!-- Vendor -->
    {% if product.vendor %}
      <p class="product-vendor">{{ product.vendor }}</p>
    {% endif %}

    <!-- Tên sản phẩm -->
    <h1 class="product-title">{{ product.title }}</h1>

    <!-- Giá -->
    <div class="product-price">
      {% if product.compare_at_price > product.price %}
        <span class="price-sale">{{ product.price | money }}</span>
        <span class="price-compare">{{ product.compare_at_price | money }}</span>
      {% else %}
        <span class="price">{{ product.price | money }}</span>
      {% endif %}
    </div>

    <!-- Variants / Options -->
    {% unless product.has_only_default_variant %}
      <form id="product-form" action="/cart/add" method="POST">
        <input type="hidden" name="id" id="variant-id" value="{{ product.selected_or_first_available_variant.id }}">

        {% for option in product.options_with_values %}
          <div class="product-option">
            <label>{{ option.name }}</label>
            <select name="option{{ forloop.index }}" id="option-{{ forloop.index }}">
              {% for value in option.values %}
                <option value="{{ value }}"
                  {% if option.selected_value == value %}selected{% endif %}>
                  {{ value }}
                </option>
              {% endfor %}
            </select>
          </div>
        {% endfor %}

        <!-- Số lượng -->
        <div class="product-quantity">
          <label for="quantity">Số lượng</label>
          <input type="number" name="quantity" id="quantity" value="1" min="1">
        </div>

        <!-- Trạng thái tồn kho -->
        {% if product.selected_or_first_available_variant.available %}
          <button type="submit">Thêm vào giỏ hàng</button>
        {% else %}
          <button type="button" disabled>Hết hàng</button>
        {% endif %}

      </form>
    {% else %}
      <!-- Sản phẩm không có variant -->
      <form action="/cart/add" method="POST">
        <input type="hidden" name="id" value="{{ product.variants.first.id }}">
        <input type="number" name="quantity" value="1" min="1">
        {% if product.variants.first.available %}
          <button type="submit">Thêm vào giỏ hàng</button>
        {% else %}
          <button type="button" disabled>Hết hàng</button>
        {% endif %}
      </form>
    {% endunless %}

    <!-- Mô tả sản phẩm -->
    {% if product.description != blank %}
      <div class="product-description">
        {{ product.description }}
      </div>
    {% endif %}

    <!-- Metafields (nếu có) -->
    {% if product.metafields.custom.material != blank %}
      <p>Chất liệu: {{ product.metafields.custom.material }}</p>
    {% endif %}

    <!-- Tags -->
    {% if product.tags.size > 0 %}
      <div class="product-tags">
        {% for tag in product.tags %}
          <span class="tag">{{ tag }}</span>
        {% endfor %}
      </div>
    {% endif %}

  </div>
</div>
```

---

## Các điểm quan trọng

| Điểm                                  | Giải thích                                           |
| ------------------------------------- | ---------------------------------------------------- | ------------------------------- |
| `image_url: width: 800`               | Dùng CDN Shopify, không dùng `                       | img_url` (deprecated)           |
| `money` filter                        | Tự động format theo currency của store               |
| `selected_or_first_available_variant` | Lấy variant đang chọn hoặc variant đầu tiên còn hàng |
| `has_only_default_variant`            | Kiểm tra sản phẩm có variant hay không               |
| `loading="lazy"`                      | Tối ưu performance cho ảnh phụ                       |
| `                                     | escape`                                              | Bảo mật, tránh XSS cho alt text |
