Đây là code Liquid mẫu để hiển thị sản phẩm của một collection:

```liquid
{% comment %}
  Template: collection.liquid
  Dùng để hiển thị danh sách sản phẩm trong một collection.
  File này thường nằm ở: templates/collection.liquid
{% endcomment %}

{% comment %} Tiêu đề collection {% endcomment %}
<h1>{{ collection.title }}</h1>

{% comment %} Mô tả collection (nếu có) {% endcomment %}
{% if collection.description != blank %}
  <p>{{ collection.description }}</p>
{% endif %}

{% comment %}
  Vòng lặp qua tất cả sản phẩm trong collection.
  collection.products trả về danh sách sản phẩm.
{% endcomment %}
{% for product in collection.products %}
  <div class="product-card">

    {% comment %} Ảnh sản phẩm - lấy ảnh đầu tiên {% endcomment %}
    {% if product.featured_image %}
      <a href="{{ product.url }}">
        <img
          src="{{ product.featured_image | image_url: width: 400 }}"
          alt="{{ product.featured_image.alt | escape }}"
          width="400"
        >
      </a>
    {% endif %}

    {% comment %} Tên sản phẩm - có link dẫn đến trang chi tiết {% endcomment %}
    <h2>
      <a href="{{ product.url }}">{{ product.title }}</a>
    </h2>

    {% comment %} Giá sản phẩm {% endcomment %}
    <p class="price">
      {% if product.compare_at_price > product.price %}
        {% comment %} Sản phẩm đang giảm giá {% endcomment %}
        <span class="original-price">
          {{ product.compare_at_price | money }}
        </span>
        <span class="sale-price">
          {{ product.price | money }}
        </span>
      {% else %}
        {{ product.price | money }}
      {% endif %}
    </p>

    {% comment %} Nút thêm vào giỏ hàng (chỉ hiện nếu còn hàng) {% endcomment %}
    {% if product.available %}
      <form action="/cart/add" method="post">
        {% comment %}
          Nếu sản phẩm có nhiều variant, cần cho khách chọn variant.
          Ở đây dùng variant đầu tiên làm mặc định.
        {% endcomment %}
        <input type="hidden" name="id" value="{{ product.first_available_variant.id }}">
        <input type="number" name="quantity" value="1" min="1">
        <button type="submit">Thêm vào giỏ</button>
      </form>
    {% else %}
      <p class="sold-out">Hết hàng</p>
    {% endif %}

  </div>
{% else %}
  {% comment %} Hiển thị khi collection không có sản phẩm nào {% endcomment %}
  <p>Collection này chưa có sản phẩm nào.</p>
{% endfor %}

{% comment %}
  PHÂN TRANG - quan trọng khi collection có nhiều sản phẩm.
  Mặc định Shopify chỉ load 50 sản phẩm mỗi trang.
{% endcomment %}
{% if paginate.pages > 1 %}
  {% paginate collection.products by 12 %}
    {{ paginate | default_pagination }}
  {% endpaginate %}
{% endif %}
```

> ⚠️ **Lưu ý về phân trang:** Để phân trang hoạt động đúng, bạn cần bọc vòng lặp `for` bên trong block `{% paginate %}`:

```liquid
{% paginate collection.products by 12 %}
  {% for product in collection.products %}
    {% comment %} ... code hiển thị sản phẩm ... {% endcomment %}
  {% endfor %}

  {{ paginate | default_pagination }}
{% endpaginate %}
```

**Một số biến hữu ích của `product` trong Liquid:**

| Biến                     | Ý nghĩa            |
| ------------------------ | ------------------ |
| `product.title`          | Tên sản phẩm       |
| `product.url`            | URL trang chi tiết |
| `product.price`          | Giá thấp nhất      |
| `product.featured_image` | Ảnh đại diện       |
| `product.available`      | Còn hàng hay không |
| `product.vendor`         | Thương hiệu        |
| `product.tags`           | Danh sách tags     |
