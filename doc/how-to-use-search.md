Dưới đây là các đoạn code Liquid mẫu để tích hợp Search đúng cách với Shopify, bao gồm cả **Search Results page** và **Predictive Search** (autocomplete khi gõ).

---

## 1. Search Form — `sections/search-bar.liquid`

Form tìm kiếm cơ bản, submit đến trang `/search`:

```liquid
<form action="{{ routes.search_url }}" method="get" role="search">
  <input
    type="search"
    name="q"
    value="{{ search.terms | escape }}"
    placeholder="Tìm kiếm sản phẩm..."
    aria-label="Tìm kiếm"
    autocomplete="off"
  >
  <!-- Giới hạn chỉ tìm products (có thể bỏ để tìm tất cả) -->
  <input type="hidden" name="type" value="product">
  <button type="submit">Tìm</button>
</form>
```

---

## 2. Search Results Page — `templates/search.liquid`

```liquid
{% if search.performed %}
  {% if search.results_count == 0 %}
    <p>Không tìm thấy kết quả cho "<strong>{{ search.terms | escape }}</strong>".</p>
  {% else %}
    <p>{{ search.results_count }} kết quả cho "<strong>{{ search.terms | escape }}</strong>"</p>

    <ul class="search-results">
      {% for item in search.results %}
        <li class="search-result-item">
          {% case item.object_type %}

            {% when 'product' %}
              <a href="{{ item.url }}">
                {% if item.featured_image %}
                  <img
                    src="{{ item.featured_image | image_url: width: 200 }}"
                    alt="{{ item.featured_image.alt | escape }}"
                    width="200"
                    loading="lazy"
                  >
                {% endif %}
                <h3>{{ item.title }}</h3>
                <p>{{ item.price | money }}</p>
              </a>

            {% when 'article' %}
              <a href="{{ item.url }}">
                <h3>{{ item.title }}</h3>
                <p>{{ item.excerpt_or_content | strip_html | truncate: 100 }}</p>
              </a>

            {% when 'page' %}
              <a href="{{ item.url }}">
                <h3>{{ item.title }}</h3>
              </a>

          {% endcase %}
        </li>
      {% endfor %}
    </ul>

    {% comment %} Pagination {% endcomment %}
    {% if paginate.pages > 1 %}
      {% paginate search.results by 12 %}
        {{ paginate | default_pagination }}
      {% endpaginate %}
    {% endif %}
  {% endif %}

{% else %}
  <p>Nhập từ khóa để tìm kiếm sản phẩm.</p>
{% endif %}
```

> ⚠️ Để pagination hoạt động, bạn cần bọc toàn bộ trong `{% paginate search.results by 12 %}...{% endpaginate %}`.

---

## 3. Predictive Search (Autocomplete) — dùng Predictive Search API

Shopify cung cấp endpoint AJAX: `/search/suggest.json`

### HTML + Liquid (`sections/predictive-search.liquid`):

```liquid
<div class="predictive-search-wrapper" style="position: relative;">
  <input
    type="text"
    id="predictive-search-input"
    placeholder="Tìm kiếm..."
    autocomplete="off"
    aria-autocomplete="list"
    aria-controls="predictive-search-results"
  >
  <ul id="predictive-search-results" role="listbox" style="display:none; position:absolute; background:#fff; border:1px solid #ccc; width:100%; z-index:100; list-style:none; margin:0; padding:0;"></ul>
</div>
```

### JavaScript (thêm vào `assets/predictive-search.js`):

```javascript
const input = document.getElementById("predictive-search-input");
const results = document.getElementById("predictive-search-results");

let debounceTimer;

input.addEventListener("input", () => {
  clearTimeout(debounceTimer);
  const query = input.value.trim();

  if (query.length < 2) {
    results.style.display = "none";
    results.innerHTML = "";
    return;
  }

  debounceTimer = setTimeout(() => {
    fetch(
      `/search/suggest.json?q=${encodeURIComponent(query)}&resources[type]=product&resources[limit]=5`,
    )
      .then((res) => res.json())
      .then((data) => {
        const products = data.resources.results.products;
        results.innerHTML = "";

        if (products.length === 0) {
          results.style.display = "none";
          return;
        }

        products.forEach((product) => {
          const li = document.createElement("li");
          li.style.cssText =
            "padding: 8px 12px; cursor: pointer; display: flex; align-items: center; gap: 10px;";
          li.innerHTML = `
            ${product.image ? `<img src="${product.image}" width="40" height="40" style="object-fit:cover;">` : ""}
            <span>${product.title}</span>
          `;
          li.addEventListener("click", () => {
            window.location.href = product.url;
          });
          results.appendChild(li);
        });

        results.style.display = "block";
      });
  }, 300); // debounce 300ms
});

// Đóng dropdown khi click ra ngoài
document.addEventListener("click", (e) => {
  if (!e.target.closest(".predictive-search-wrapper")) {
    results.style.display = "none";
  }
});
```

### Load JS trong theme layout (`layout/theme.liquid`):

```liquid
{{ 'predictive-search.js' | asset_url | script_tag }}
```

---

## Tóm tắt các điểm quan trọng

| Tính năng         | Cách dùng                                                |
| ----------------- | -------------------------------------------------------- |
| Search form       | `routes.search_url`, param `q` và `type`                 |
| Search results    | Object `search` trong template `search.liquid`           |
| Predictive search | Endpoint `/search/suggest.json` qua AJAX                 |
| Tùy chỉnh kết quả | Dùng app **Search & Discovery** (boost, synonym, filter) |

Các cấu hình như **boost sản phẩm**, **synonym**, **filter** bạn thiết lập trong app Search & Discovery sẽ tự động được áp dụng vào cả search results lẫn predictive search mà không cần thêm code.
