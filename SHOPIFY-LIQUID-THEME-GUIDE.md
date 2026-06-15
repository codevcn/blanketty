# Hướng dẫn dùng Liquid để phát triển Shopify Theme đúng chuẩn

> Tài liệu thực hành dành cho lập trình viên xây dựng hoặc tùy chỉnh Shopify Online Store theme bằng Liquid, HTML, CSS và JavaScript.
>
> Mục tiêu: code dễ đọc, dễ mở rộng, thân thiện với Theme Editor, có hiệu năng tốt, hỗ trợ accessibility, SEO, đa ngôn ngữ và quy trình làm việc an toàn với Shopify CLI/Git.

---

## Mục lục

1. [Liquid trong Shopify là gì?](#1-liquid-trong-shopify-là-gì)
2. [Nguyên tắc kiến trúc cốt lõi](#2-nguyên-tắc-kiến-trúc-cốt-lõi)
3. [Cấu trúc thư mục Shopify Theme](#3-cấu-trúc-thư-mục-shopify-theme)
4. [Cú pháp Liquid nền tảng](#4-cú-pháp-liquid-nền-tảng)
5. [Objects, properties, tags và filters](#5-objects-properties-tags-và-filters)
6. [Quy tắc viết Liquid sạch và an toàn](#6-quy-tắc-viết-liquid-sạch-và-an-toàn)
7. [Layouts và `theme.liquid`](#7-layouts-và-themeliquid)
8. [JSON templates](#8-json-templates)
9. [Sections và section schema](#9-sections-và-section-schema)
10. [Blocks](#10-blocks)
11. [Snippets và `render`](#11-snippets-và-render)
12. [Theme settings và config](#12-theme-settings-và-config)
13. [Locales và đa ngôn ngữ](#13-locales-và-đa-ngôn-ngữ)
14. [Làm việc với product, collection và cart](#14-làm-việc-với-product-collection-và-cart)
15. [Metafields và metaobjects](#15-metafields-và-metaobjects)
16. [Ảnh và media responsive](#16-ảnh-và-media-responsive)
17. [CSS và JavaScript trong theme](#17-css-và-javascript-trong-theme)
18. [Theme Editor và JavaScript events](#18-theme-editor-và-javascript-events)
19. [Performance](#19-performance)
20. [Accessibility](#20-accessibility)
21. [SEO và structured data](#21-seo-và-structured-data)
22. [Bảo mật và escaping](#22-bảo-mật-và-escaping)
23. [Shopify CLI, Theme Check và Git workflow](#23-shopify-cli-theme-check-và-git-workflow)
24. [Cấu trúc mẫu cho một feature](#24-cấu-trúc-mẫu-cho-một-feature)
25. [Những lỗi thường gặp](#25-những-lỗi-thường-gặp)
26. [Checklist trước khi bàn giao hoặc publish](#26-checklist-trước-khi-bàn-giao-hoặc-publish)
27. [Tài liệu chính thức](#27-tài-liệu-chính-thức)

---

## 1. Liquid trong Shopify là gì?

Liquid là template language được Shopify dùng để kết hợp:

- Dữ liệu của cửa hàng: sản phẩm, collection, cart, customer, blog, article, menu, metafield...
- Cấu hình của merchant trong Theme Editor.
- HTML, CSS và JavaScript của giao diện.

Liquid được xử lý trên server của Shopify để tạo HTML gửi về trình duyệt. Liquid **không phải** JavaScript và không chạy trực tiếp trong browser.

Ví dụ:

```liquid
<h1>{{ product.title | escape }}</h1>
<p>{{ product.price | money }}</p>
```

Trong ví dụ này:

- `product` là một Liquid object.
- `title` và `price` là properties.
- `money` và `escape` là filters.
- `{{ ... }}` xuất dữ liệu ra HTML.

### Liquid nên chịu trách nhiệm gì?

Liquid phù hợp để:

- Render dữ liệu Shopify thành HTML.
- Điều khiển điều kiện hiển thị.
- Lặp qua danh sách dữ liệu.
- Đọc settings từ Theme Editor.
- Chia nhỏ giao diện thành sections, blocks và snippets.

Liquid không nên bị dùng để thay thế hoàn toàn JavaScript cho các tương tác phía client hoặc chứa logic nghiệp vụ quá phức tạp.

---

## 2. Nguyên tắc kiến trúc cốt lõi

Một Shopify theme hiện đại nên tuân theo hướng sau:

```text
Layout
└── Section groups
└── JSON template
    └── Sections
        └── Blocks
            └── Snippets
```

### Trách nhiệm của từng tầng

| Thành phần | Trách nhiệm chính |
|---|---|
| Layout | Khung HTML toàn trang, `<head>`, assets chung, header/footer groups, `content_for_layout` |
| JSON template | Khai báo section nào xuất hiện trên từng loại trang và thứ tự của chúng |
| Section | Một vùng giao diện có thể cấu hình trong Theme Editor |
| Block | Một đơn vị nội dung nhỏ hơn bên trong section |
| Snippet | Partial tái sử dụng, nhận dữ liệu qua tham số |
| Asset | CSS, JavaScript, SVG, hình tĩnh hoặc font được theme sử dụng |
| Config | Cấu hình global theme settings và dữ liệu settings |
| Locales | Chuỗi dịch cho storefront và Theme Editor |

### Nguyên tắc phân tách

- JSON template chỉ nên quản lý cấu trúc section, không chứa Liquid logic.
- Section chịu trách nhiệm orchestration và cấu hình merchant.
- Snippet chịu trách nhiệm render component tái sử dụng.
- CSS và JavaScript nên được tổ chức theo component hoặc feature.
- Không nhồi toàn bộ theme vào `theme.liquid`.
- Không hard-code nội dung mà merchant cần chỉnh sửa.
- Không tạo quá nhiều settings cho các chi tiết không có giá trị kinh doanh.

---

## 3. Cấu trúc thư mục Shopify Theme

Cấu trúc phổ biến:

```text
my-theme/
├── assets/
│   ├── base.css
│   ├── theme.js
│   ├── component-card.css
│   └── product-form.js
├── blocks/
│   └── text.liquid
├── config/
│   ├── settings_schema.json
│   └── settings_data.json
├── layout/
│   ├── theme.liquid
│   └── password.liquid
├── locales/
│   ├── en.default.json
│   ├── en.default.schema.json
│   ├── vi.json
│   └── vi.schema.json
├── sections/
│   ├── header-group.json
│   ├── footer-group.json
│   ├── main-product.liquid
│   ├── main-collection-product-grid.liquid
│   └── image-with-text.liquid
├── snippets/
│   ├── product-card.liquid
│   ├── price.liquid
│   ├── icon.liquid
│   └── meta-tags.liquid
└── templates/
    ├── index.json
    ├── product.json
    ├── collection.json
    ├── cart.json
    ├── page.json
    └── search.json
```

> Thư mục `blocks/` được dùng cho theme blocks. Không phải theme nào cũng cần dùng ngay từ đầu; section blocks vẫn phù hợp cho nhiều component đơn giản.

### Quy tắc đặt tên file

Dùng `kebab-case`:

```text
main-product.liquid
featured-collection.liquid
product-card.liquid
component-price.css
product-form.js
```

Tên file cần mô tả rõ vai trò, tránh tên chung chung như:

```text
custom.liquid
new-section.liquid
style2.css
script-final.js
```

---

## 4. Cú pháp Liquid nền tảng

### 4.1 Output

```liquid
{{ product.title }}
```

Khi hiển thị text do merchant hoặc customer nhập, ưu tiên escape:

```liquid
{{ section.settings.heading | escape }}
```

### 4.2 Logic tags

```liquid
{% if product.available %}
  <span>{{ 'products.product.in_stock' | t }}</span>
{% else %}
  <span>{{ 'products.product.sold_out' | t }}</span>
{% endif %}
```

### 4.3 Comment

```liquid
{% comment %}
  Nội dung này không được render ra HTML.
{% endcomment %}
```

### 4.4 Gán biến

```liquid
{% assign selected_variant = product.selected_or_first_available_variant %}
```

### 4.5 Capture HTML hoặc chuỗi

```liquid
{% capture accessible_label %}
  {{ product.title }} — {{ product.price | money }}
{% endcapture %}
```

Không lạm dụng `capture` cho các đoạn HTML lớn. Khi component đủ lớn, chuyển thành snippet.

### 4.6 Điều kiện

```liquid
{% if product.available and product.price > 0 %}
  ...
{% elsif product.price == 0 %}
  ...
{% else %}
  ...
{% endif %}
```

### 4.7 `case`

```liquid
{% case block.type %}
  {% when 'heading' %}
    <h2 {{ block.shopify_attributes }}>
      {{ block.settings.text | escape }}
    </h2>

  {% when 'button' %}
    <a href="{{ block.settings.url }}" {{ block.shopify_attributes }}>
      {{ block.settings.label | escape }}
    </a>
{% endcase %}
```

### 4.8 Loop

```liquid
{% for product in collection.products %}
  {% render 'product-card', card_product: product %}
{% else %}
  <p>{{ 'collections.general.no_matches' | t }}</p>
{% endfor %}
```

`for ... else` giúp xử lý danh sách rỗng rõ ràng.

### 4.9 Giới hạn loop

```liquid
{% for product in collection.products limit: 8 %}
  ...
{% endfor %}
```

### 4.10 Whitespace control

Dấu `-` loại bỏ whitespace dư:

```liquid
{%- assign heading = section.settings.heading -%}
```

Chỉ dùng khi thực sự cần. Việc thêm `-` vào mọi tag có thể khiến template khó đọc hoặc nối text ngoài ý muốn.

---

## 5. Objects, properties, tags và filters

### 5.1 Object

Một số global hoặc contextual objects thường gặp:

```liquid
{{ shop.name }}
{{ request.page_type }}
{{ routes.cart_url }}
{{ localization.country.name }}
{{ cart.item_count }}
{{ customer.first_name }}
```

Contextual object phụ thuộc trang:

- Product page: `product`
- Collection page: `collection`
- Article page: `article`
- Blog page: `blog`
- Cart page: `cart`
- Search page: `search`

Không giả định object luôn tồn tại ở mọi template.

```liquid
{% if product != blank %}
  {{ product.title | escape }}
{% endif %}
```

### 5.2 Filters

Filters biến đổi output:

```liquid
{{ product.title | escape }}
{{ product.price | money }}
{{ article.published_at | date: '%d/%m/%Y' }}
{{ product.url | within: collection }}
{{ text | strip_html | truncate: 120 }}
```

Có thể nối filters thành pipeline:

```liquid
{{ article.content | strip_html | strip | truncatewords: 30 }}
```

### 5.3 Tags

Tags điều khiển logic hoặc tạo cấu trúc Shopify:

```liquid
{% if %}
{% for %}
{% case %}
{% assign %}
{% capture %}
{% render %}
{% form %}
{% paginate %}
{% schema %}
```

### 5.4 Kiểm tra `blank`, `empty`, `nil`

Ưu tiên cách dễ hiểu:

```liquid
{% if section.settings.heading != blank %}
  ...
{% endif %}
```

```liquid
{% if collection.products == empty %}
  ...
{% endif %}
```

```liquid
{% if customer == nil %}
  ...
{% endif %}
```

### 5.5 Tránh truy cập dữ liệu động thiếu kiểm soát

Không nên tạo handle rồi truy cập resource mà không kiểm tra:

```liquid
{% assign featured_collection = collections[section.settings.collection] %}

{% if featured_collection != blank %}
  ...
{% endif %}
```

Với setting loại `collection`, Shopify thường trả resource object trực tiếp. Khi đó dùng:

```liquid
{% assign featured_collection = section.settings.collection %}
```

---

## 6. Quy tắc viết Liquid sạch và an toàn

### 6.1 Dùng tên biến mô tả rõ ý nghĩa

Tốt:

```liquid
{% assign selected_variant = product.selected_or_first_available_variant %}
{% assign has_compare_at_price = false %}
```

Không tốt:

```liquid
{% assign v = product.selected_or_first_available_variant %}
{% assign x = false %}
```

### 6.2 Một biến chỉ nên có một ý nghĩa

Không tái sử dụng cùng một biến cho nhiều loại dữ liệu khác nhau.

### 6.3 Giới hạn logic lồng nhau

Không tốt:

```liquid
{% if product %}
  {% if product.available %}
    {% if product.featured_image %}
      {% if section.settings.show_image %}
        ...
      {% endif %}
    {% endif %}
  {% endif %}
{% endif %}
```

Tốt hơn:

```liquid
{% assign should_show_image = false %}

{% if product != blank and product.available and product.featured_image != blank and section.settings.show_image %}
  {% assign should_show_image = true %}
{% endif %}

{% if should_show_image %}
  ...
{% endif %}
```

Hoặc tách component thành snippet có API rõ ràng.

### 6.4 Dùng `render`, không phụ thuộc biến ngầm

```liquid
{% render 'product-card',
  card_product: product,
  show_vendor: section.settings.show_vendor,
  image_ratio: section.settings.image_ratio
%}
```

### 6.5 Dùng routes object

Ưu tiên:

```liquid
<a href="{{ routes.cart_url }}">Cart</a>
<a href="{{ routes.search_url }}">Search</a>
<a href="{{ routes.root_url }}">Home</a>
```

Không hard-code:

```liquid
<a href="/cart">Cart</a>
```

Routes object an toàn hơn với localization/subfolder routing.

### 6.6 Escape đúng ngữ cảnh

Text:

```liquid
{{ section.settings.heading | escape }}
```

URL từ Shopify resource hoặc setting URL có thể dùng trực tiếp trong thuộc tính `href`, nhưng vẫn phải quote attribute:

```liquid
<a href="{{ section.settings.button_link }}">
```

Nội dung rich text được Shopify quản lý có thể render như HTML:

```liquid
<div class="rte">
  {{ section.settings.text }}
</div>
```

Không thêm `escape` vào rich text nếu bạn muốn giữ HTML hợp lệ do Shopify tạo.

### 6.7 Không dùng Liquid để tạo JSON thủ công thiếu escaping

Không tốt:

```liquid
<script>
  const title = "{{ product.title }}";
</script>
```

Tốt hơn:

```liquid
<script type="application/json" data-product-json>
  {{ product | json }}
</script>
```

Hoặc:

```liquid
<script>
  const title = {{ product.title | json }};
</script>
```

Filter `json` giúp encode chuỗi đúng cách.

---

## 7. Layouts và `theme.liquid`

`layout/theme.liquid` là khung chính của storefront.

Một cấu trúc tối giản:

```liquid
<!doctype html>
<html class="no-js" lang="{{ request.locale.iso_code }}">
  <head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">

    <link rel="canonical" href="{{ canonical_url }}">

    <title>
      {{ page_title }}
      {% if current_tags %} &ndash; {{ current_tags | join: ', ' }}{% endif %}
      {% if current_page != 1 %} &ndash; Page {{ current_page }}{% endif %}
      {% unless page_title contains shop.name %} &ndash; {{ shop.name }}{% endunless %}
    </title>

    {% if page_description %}
      <meta name="description" content="{{ page_description | escape }}">
    {% endif %}

    {% render 'meta-tags' %}

    {{ 'base.css' | asset_url | stylesheet_tag }}

    <script src="{{ 'theme.js' | asset_url }}" defer="defer"></script>

    {{ content_for_header }}
  </head>

  <body>
    <a class="skip-to-content-link" href="#MainContent">
      {{ 'accessibility.skip_to_text' | t }}
    </a>

    {% sections 'header-group' %}

    <main id="MainContent" role="main" tabindex="-1">
      {{ content_for_layout }}
    </main>

    {% sections 'footer-group' %}
  </body>
</html>
```

### Quy tắc quan trọng

- Phải giữ `{{ content_for_header }}` trong `<head>`.
- Phải giữ `{{ content_for_layout }}` ở nơi nội dung template được render.
- Không chỉnh sửa hoặc lọc output của `content_for_header`.
- Dùng `request.locale.iso_code` cho thuộc tính `lang`.
- Script không quan trọng nên dùng `defer`.
- CSS/JS chỉ dùng toàn site mới nên nằm trong layout.
- Feature chỉ xuất hiện ở một số trang nên tải asset theo điều kiện hoặc theo section/component.

---

## 8. JSON templates

JSON template mô tả danh sách sections và thứ tự hiển thị.

Ví dụ `templates/product.json`:

```json
{
  "sections": {
    "main": {
      "type": "main-product",
      "blocks": {
        "title": {
          "type": "title",
          "settings": {}
        },
        "price": {
          "type": "price",
          "settings": {}
        },
        "buy_buttons": {
          "type": "buy_buttons",
          "settings": {}
        }
      },
      "block_order": [
        "title",
        "price",
        "buy_buttons"
      ],
      "settings": {}
    }
  },
  "order": [
    "main"
  ]
}
```

### Quy tắc

- `type` phải trùng tên file section, bỏ `.liquid`.
- Mỗi section instance có ID riêng trong object `sections`.
- Thứ tự section được quyết định bởi mảng `order`.
- Thứ tự block được quyết định bởi `block_order`.
- Không viết Liquid trong JSON template.
- Không chỉnh trực tiếp JSON được merchant tạo nếu không hiểu ảnh hưởng đến dữ liệu Theme Editor.

### Alternate template

Có thể tạo:

```text
templates/product.preorder.json
templates/page.about.json
templates/collection.featured.json
```

Sau đó gán template cho resource trong Shopify Admin.

---

## 9. Sections và section schema

Section là đơn vị giao diện chính merchant có thể thêm, xóa, sắp xếp và cấu hình.

### 9.1 Section hoàn chỉnh mẫu

`sections/image-with-text.liquid`:

```liquid
{% liquid
  assign heading = section.settings.heading
  assign text = section.settings.text
  assign image = section.settings.image
  assign image_position = section.settings.image_position
%}

<section
  class="image-with-text image-with-text--{{ image_position }}"
  id="ImageWithText-{{ section.id }}"
>
  <div class="image-with-text__inner page-width">
    {% if image != blank %}
      <div class="image-with-text__media">
        {{
          image
          | image_url: width: 1600
          | image_tag:
            widths: '320, 480, 750, 990, 1200, 1600',
            sizes: '(min-width: 990px) 50vw, 100vw',
            loading: 'lazy',
            class: 'image-with-text__image'
        }}
      </div>
    {% endif %}

    <div class="image-with-text__content">
      {% if heading != blank %}
        <h2>{{ heading | escape }}</h2>
      {% endif %}

      {% if text != blank %}
        <div class="rte">{{ text }}</div>
      {% endif %}

      {% if section.settings.button_label != blank and section.settings.button_link != blank %}
        <a class="button" href="{{ section.settings.button_link }}">
          {{ section.settings.button_label | escape }}
        </a>
      {% endif %}
    </div>
  </div>
</section>

{% schema %}
{
  "name": "Image with text",
  "tag": "section",
  "class": "section",
  "disabled_on": {
    "groups": ["header", "footer"]
  },
  "settings": [
    {
      "type": "image_picker",
      "id": "image",
      "label": "Image"
    },
    {
      "type": "inline_richtext",
      "id": "heading",
      "label": "Heading",
      "default": "Tell your story"
    },
    {
      "type": "richtext",
      "id": "text",
      "label": "Text",
      "default": "<p>Pair text with an image to communicate a key message.</p>"
    },
    {
      "type": "text",
      "id": "button_label",
      "label": "Button label"
    },
    {
      "type": "url",
      "id": "button_link",
      "label": "Button link"
    },
    {
      "type": "select",
      "id": "image_position",
      "label": "Image position",
      "default": "left",
      "options": [
        {
          "value": "left",
          "label": "Left"
        },
        {
          "value": "right",
          "label": "Right"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Image with text"
    }
  ]
}
{% endschema %}
```

### 9.2 Quy tắc schema

- Một section chỉ có một `{% schema %}` block.
- Schema phải là JSON hợp lệ.
- Schema không hỗ trợ Liquid interpolation.
- `id` nên dùng `snake_case` và không thay đổi tùy tiện sau khi theme đã được dùng.
- `label` phải rõ nghĩa với merchant.
- Có `default` hợp lý để section hiển thị tốt ngay khi được thêm.
- Section cần `presets` nếu muốn merchant thêm từ Theme Editor.
- Dùng `enabled_on` hoặc `disabled_on` để giới hạn section ở đúng template/group.
- Không tạo settings trùng lặp hoặc quá chi tiết khiến Theme Editor khó dùng.

### 9.3 Khi nào dùng section setting?

Dùng setting khi merchant thực sự cần thay đổi:

- Heading, text, image, video.
- Collection/product/menu/page được chọn.
- Bật/tắt một tính năng.
- Layout có số lượng lựa chọn giới hạn.
- Color scheme, spacing hoặc alignment đã được thiết kế có kiểm soát.

Không nên cho merchant nhập tùy ý:

- CSS selector.
- JavaScript.
- Class nội bộ khó hiểu.
- Pixel value không giới hạn cho mọi thuộc tính.
- HTML không được kiểm soát, trừ tính năng Custom Liquid có chủ đích.

### 9.4 Section ID

Mỗi section instance có `section.id`. Dùng nó để tránh trùng ID DOM:

```liquid
<section id="FeaturedCollection-{{ section.id }}">
```

---

## 10. Blocks

Blocks giúp merchant thêm, xóa và sắp xếp các đơn vị nội dung nhỏ bên trong section.

### 10.1 Section blocks

Ví dụ:

```liquid
<div class="content-blocks">
  {% for block in section.blocks %}
    {% case block.type %}
      {% when 'heading' %}
        <h2 {{ block.shopify_attributes }}>
          {{ block.settings.heading | escape }}
        </h2>

      {% when 'text' %}
        <div class="rte" {{ block.shopify_attributes }}>
          {{ block.settings.text }}
        </div>

      {% when 'button' %}
        {% if block.settings.label != blank and block.settings.link != blank %}
          <a
            class="button"
            href="{{ block.settings.link }}"
            {{ block.shopify_attributes }}
          >
            {{ block.settings.label | escape }}
          </a>
        {% endif %}
    {% endcase %}
  {% endfor %}
</div>

{% schema %}
{
  "name": "Flexible content",
  "settings": [],
  "blocks": [
    {
      "type": "heading",
      "name": "Heading",
      "limit": 1,
      "settings": [
        {
          "type": "inline_richtext",
          "id": "heading",
          "label": "Heading",
          "default": "Heading"
        }
      ]
    },
    {
      "type": "text",
      "name": "Text",
      "settings": [
        {
          "type": "richtext",
          "id": "text",
          "label": "Text",
          "default": "<p>Add supporting text.</p>"
        }
      ]
    },
    {
      "type": "button",
      "name": "Button",
      "settings": [
        {
          "type": "text",
          "id": "label",
          "label": "Label",
          "default": "Learn more"
        },
        {
          "type": "url",
          "id": "link",
          "label": "Link"
        }
      ]
    }
  ],
  "presets": [
    {
      "name": "Flexible content",
      "blocks": [
        {
          "type": "heading"
        },
        {
          "type": "text"
        },
        {
          "type": "button"
        }
      ]
    }
  ]
}
{% endschema %}
```

### 10.2 Luôn dùng `block.shopify_attributes`

Thuộc tính này giúp Theme Editor nhận biết block được merchant chọn:

```liquid
<div {{ block.shopify_attributes }}>
```

Đặt nó trên element gốc đại diện cho block.

### 10.3 App blocks

Section trong JSON template nên cân nhắc hỗ trợ app blocks:

```json
{
  "type": "@app"
}
```

Trong Liquid:

```liquid
{% for block in section.blocks %}
  {% case block.type %}
    {% when '@app' %}
      {% render block %}
    {% else %}
      ...
  {% endcase %}
{% endfor %}
```

App blocks cho phép app chèn nội dung mà không phải sửa trực tiếp theme code.

### 10.4 Theme blocks

Theme blocks nằm trong thư mục `blocks/` và có thể tái sử dụng/nesting linh hoạt hơn. Chỉ dùng khi kiến trúc theme thực sự cần composition sâu hoặc component dùng chung giữa nhiều sections.

Tránh chuyển mọi section block đơn giản thành theme block nếu điều đó làm dự án phức tạp hơn mà không có lợi ích rõ ràng.

---

## 11. Snippets và `render`

Snippet là partial tái sử dụng.

### 11.1 Gọi snippet

```liquid
{% render 'product-card',
  card_product: product,
  show_vendor: true,
  image_ratio: 'square'
%}
```

### 11.2 Viết API cho snippet

Đầu file `snippets/product-card.liquid` nên có comment mô tả:

```liquid
{% comment %}
  Renders a product card.

  Accepts:
  - card_product: {Product} Required product object.
  - show_vendor: {Boolean} Show product vendor.
  - image_ratio: {String} "adapt", "square", or "portrait".

  Usage:
  {% render 'product-card',
    card_product: product,
    show_vendor: true,
    image_ratio: 'square'
  %}
{% endcomment %}
```

Sau đó chuẩn hóa default:

```liquid
{% liquid
  assign show_vendor = show_vendor | default: false
  assign image_ratio = image_ratio | default: 'adapt'
%}
```

> Lưu ý: filter `default` có thể thay cả giá trị `false` trong một số trường hợp. Với boolean cần phân biệt rõ, hãy kiểm tra hoặc gán từ caller thay vì phụ thuộc mù quáng vào `default`.

### 11.3 Scope của `render`

Snippet được render trong scope cô lập. Biến local ở file cha không tự động xuất hiện trong snippet nếu không truyền vào.

Tốt:

```liquid
{% assign featured_product = section.settings.product %}
{% render 'product-card', card_product: featured_product %}
```

Không nên viết snippet phụ thuộc vào biến không được document.

### 11.4 `render ... for`

Có thể render một snippet cho mỗi phần tử:

```liquid
{% render 'product-card' for collection.products as card_product %}
```

Cách này ngắn, nhưng khi cần nhiều tham số hoặc xử lý empty state, loop thường dễ đọc hơn.

### 11.5 Khi nào tách snippet?

Tách khi:

- Component được dùng ở từ hai nơi trở lên.
- File section trở nên dài và khó hiểu.
- Component có input/output rõ ràng.
- Cần test hoặc thay đổi component độc lập.

Không tách từng thẻ HTML nhỏ thành snippet vì sẽ làm luồng đọc bị phân mảnh.

---

## 12. Theme settings và config

### 12.1 `settings_schema.json`

`config/settings_schema.json` định nghĩa global theme settings:

```json
[
  {
    "name": "theme_info",
    "theme_name": "Example Theme",
    "theme_version": "1.0.0",
    "theme_author": "Your Company",
    "theme_documentation_url": "https://example.com/docs",
    "theme_support_url": "https://example.com/support"
  },
  {
    "name": "Colors",
    "settings": [
      {
        "type": "color",
        "id": "color_background",
        "label": "Background",
        "default": "#ffffff"
      },
      {
        "type": "color",
        "id": "color_text",
        "label": "Text",
        "default": "#121212"
      }
    ]
  }
]
```

Truy cập trong Liquid:

```liquid
{{ settings.color_background }}
```

### 12.2 Xuất settings thành CSS variables

```liquid
{% style %}
  :root {
    --color-background: {{ settings.color_background }};
    --color-text: {{ settings.color_text }};
  }
{% endstyle %}
```

Hoặc chuyển giá trị vào một snippet style tokens để quản lý tập trung.

### 12.3 `settings_data.json`

File này chứa giá trị merchant đã lưu trong Theme Editor.

Quy tắc làm việc nhóm:

- Không ghi đè `settings_data.json` production một cách vô ý.
- Trước khi pull/push cần xác định có lấy/đẩy settings hay không.
- Review diff kỹ vì thay đổi nhỏ trong Theme Editor có thể tạo diff lớn.
- Không lưu secret trong theme settings.

### 12.4 Global setting hay section setting?

Dùng global theme setting cho:

- Typography.
- Color schemes.
- Container width.
- Button style toàn theme.
- Social links.
- Logo/favicon.

Dùng section setting cho:

- Nội dung và layout chỉ áp dụng cho một section instance.
- Image, heading, text, selected collection của section.

---

## 13. Locales và đa ngôn ngữ

Không hard-code text giao diện lặp lại.

### 13.1 Storefront locale

`locales/en.default.json`:

```json
{
  "accessibility": {
    "skip_to_text": "Skip to content"
  },
  "products": {
    "product": {
      "add_to_cart": "Add to cart",
      "sold_out": "Sold out",
      "quantity": "Quantity"
    }
  }
}
```

`locales/vi.json`:

```json
{
  "accessibility": {
    "skip_to_text": "Chuyển đến nội dung"
  },
  "products": {
    "product": {
      "add_to_cart": "Thêm vào giỏ hàng",
      "sold_out": "Hết hàng",
      "quantity": "Số lượng"
    }
  }
}
```

Dùng trong Liquid:

```liquid
{{ 'products.product.add_to_cart' | t }}
```

### 13.2 Translation có biến

Locale:

```json
{
  "cart": {
    "items_count": {
      "one": "{{ count }} item",
      "other": "{{ count }} items"
    }
  }
}
```

Liquid:

```liquid
{{ 'cart.items_count' | t: count: cart.item_count }}
```

### 13.3 Schema locales

Dùng file `.schema.json` để dịch label trong Theme Editor.

Trong schema:

```json
{
  "name": "t:sections.featured_collection.name",
  "settings": [
    {
      "type": "text",
      "id": "heading",
      "label": "t:sections.featured_collection.settings.heading.label"
    }
  ]
}
```

### 13.4 Quy tắc đặt translation key

Tốt:

```text
products.product.add_to_cart
sections.featured_collection.settings.heading.label
general.search.search
```

Không tốt:

```text
text1
button_new
label_final
```

Key cần phản ánh ngữ nghĩa, không phản ánh vị trí tạm thời.

---

## 14. Làm việc với product, collection và cart

## 14.1 Product

### Variant được chọn

```liquid
{% assign selected_variant = product.selected_or_first_available_variant %}
```

### Product form chuẩn

```liquid
{% assign product_form_id = 'ProductForm-' | append: section.id %}
{% assign selected_variant = product.selected_or_first_available_variant %}

{% form 'product', product, id: product_form_id, class: 'product-form' %}
  <input
    type="hidden"
    name="id"
    value="{{ selected_variant.id }}"
  >

  <label for="Quantity-{{ section.id }}">
    {{ 'products.product.quantity' | t }}
  </label>

  <input
    id="Quantity-{{ section.id }}"
    type="number"
    name="quantity"
    value="1"
    min="1"
    inputmode="numeric"
  >

  <button
    type="submit"
    name="add"
    {% unless selected_variant.available %}disabled{% endunless %}
  >
    {% if selected_variant.available %}
      {{ 'products.product.add_to_cart' | t }}
    {% else %}
      {{ 'products.product.sold_out' | t }}
    {% endif %}
  </button>
{% endform %}
```

Khi JavaScript thay variant, cần cập nhật tối thiểu:

- Input `name="id"`.
- Giá.
- Compare-at price.
- Availability/button state.
- SKU nếu hiển thị.
- Media tương ứng nếu có.
- URL variant nếu thiết kế yêu cầu deep-link.
- Pickup availability hoặc selling plan nếu theme hỗ trợ.

Không giả định tất cả variant của sản phẩm luôn nên được serialize vào HTML. Với sản phẩm có số lượng variant lớn, dùng API/pattern hỗ trợ high-variant products và chỉ tải dữ liệu cần thiết.

### Giá

```liquid
{% assign target = product.selected_or_first_available_variant %}

<div class="price">
  {% if target.compare_at_price > target.price %}
    <s>{{ target.compare_at_price | money }}</s>
  {% endif %}

  <span>{{ target.price | money }}</span>
</div>
```

Không tự format tiền bằng cách chia `100` hoặc nối ký hiệu tiền tệ thủ công. Dùng money filters để tôn trọng currency format của shop.

## 14.2 Collection

### Pagination

```liquid
{% paginate collection.products by section.settings.products_per_page %}
  <ul class="product-grid" role="list">
    {% for product in collection.products %}
      <li>
        {% render 'product-card', card_product: product %}
      </li>
    {% else %}
      <li>{{ 'collections.general.no_matches' | t }}</li>
    {% endfor %}
  </ul>

  {% if paginate.pages > 1 %}
    {% render 'pagination', paginate: paginate %}
  {% endif %}
{% endpaginate %}
```

Không dùng loop toàn bộ catalog qua `collections.all.products` để giả lập search/filter.

### Sorting

```liquid
<form method="get">
  <label for="SortBy">Sort by</label>
  <select id="SortBy" name="sort_by">
    {% assign sort_by = collection.sort_by | default: collection.default_sort_by %}

    {% for option in collection.sort_options %}
      <option
        value="{{ option.value }}"
        {% if option.value == sort_by %}selected{% endif %}
      >
        {{ option.name }}
      </option>
    {% endfor %}
  </select>

  <button type="submit">Apply</button>
</form>
```

Khi có storefront filtering, render từ `collection.filters` hoặc `search.filters` thay vì tự suy luận filters từ tags.

## 14.3 Cart

### Cart form

```liquid
{% form 'cart', cart %}
  {% for item in cart.items %}
    <article class="cart-item">
      <a href="{{ item.url }}">
        {{ item.product.title | escape }}
      </a>

      <label for="Updates-{{ item.key }}">
        {{ 'products.product.quantity' | t }}
      </label>

      <input
        id="Updates-{{ item.key }}"
        type="number"
        name="updates[]"
        value="{{ item.quantity }}"
        min="0"
      >

      <span>{{ item.final_line_price | money }}</span>
    </article>
  {% endfor %}

  <label for="CartNote">Order note</label>
  <textarea id="CartNote" name="note">{{ cart.note }}</textarea>

  <button type="submit" name="update">Update cart</button>
  <button type="submit" name="checkout">Checkout</button>
{% endform %}
```

Nếu xây AJAX cart, vẫn nên giữ fallback HTML form hoặc hành vi cơ bản hoạt động khi JavaScript thất bại.

---

## 15. Metafields và metaobjects

### 15.1 Đọc metafield

```liquid
{% assign material = product.metafields.custom.material %}

{% if material != blank %}
  <p>{{ material.value | escape }}</p>
{% endif %}
```

### 15.2 Dùng `metafield_tag`

Với rich text, dimension, date hoặc các kiểu dữ liệu hỗ trợ:

```liquid
{% if product.metafields.custom.care_instructions != blank %}
  {{ product.metafields.custom.care_instructions | metafield_tag }}
{% endif %}
```

### 15.3 List metafield

```liquid
{% assign features = product.metafields.custom.features.value %}

{% if features != blank %}
  <ul>
    {% for feature in features %}
      <li>{{ feature | escape }}</li>
    {% endfor %}
  </ul>
{% endif %}
```

### 15.4 Metaobject reference

```liquid
{% assign author = product.metafields.custom.author.value %}

{% if author != blank %}
  <div class="author-card">
    <h3>{{ author.name | escape }}</h3>

    {% if author.bio != blank %}
      {{ author.bio | metafield_tag }}
    {% endif %}
  </div>
{% endif %}
```

### Quy tắc

- Luôn kiểm tra metafield có dữ liệu trước khi render wrapper.
- Dùng `.value` để lấy typed value khi phù hợp.
- Dùng `metafield_tag` nếu muốn Shopify render đúng kiểu dữ liệu.
- Không thể tạo metafield bằng Liquid; Liquid chỉ đọc dữ liệu đã được định nghĩa/lưu.
- Namespace và key nên ổn định, có tài liệu rõ ràng.

---

## 16. Ảnh và media responsive

### 16.1 Dùng `image_url` và `image_tag`

```liquid
{% if product.featured_image != blank %}
  {{
    product.featured_image
    | image_url: width: 1200
    | image_tag:
      widths: '320, 480, 750, 990, 1200',
      sizes: '(min-width: 990px) 33vw, 50vw',
      loading: 'lazy',
      alt: product.featured_image.alt
  }}
{% endif %}
```

### 16.2 Alt text

- Ảnh truyền tải nội dung: alt mô tả ngắn gọn.
- Ảnh chỉ trang trí: dùng `alt=""`.
- Không nhồi keyword SEO.
- Không lặp “image of” hoặc “picture of” khi không cần.

```liquid
{% assign image_alt = image.alt | default: product.title %}
```

Chỉ fallback sang product title nếu hợp ngữ cảnh.

### 16.3 Loading

- Ảnh hero/LCP phía trên fold: cân nhắc `loading: 'eager'` và preload có chọn lọc.
- Ảnh phía dưới fold: `loading: 'lazy'`.
- Không preload hàng loạt ảnh.
- Cung cấp width/height để giảm layout shift.

### 16.4 CSS responsive

```css
.responsive-image {
  display: block;
  width: 100%;
  height: auto;
}
```

### 16.5 Product media

Sản phẩm có thể có:

- Image.
- Video.
- External video.
- 3D model.

Không giả định `product.media` chỉ chứa ảnh. Kiểm tra `media.media_type` hoặc dùng media filters/component phù hợp.

---

## 17. CSS và JavaScript trong theme

## 17.1 CSS

### Asset toàn theme

```liquid
{{ 'base.css' | asset_url | stylesheet_tag }}
```

### Asset theo component

```liquid
{{ 'component-product-card.css' | asset_url | stylesheet_tag }}
```

Chỉ tải khi component được render, nhưng tránh lặp quá nhiều stylesheet nhỏ gây request overhead nếu chiến lược build không tối ưu.

### CSS custom properties từ settings

```liquid
<div
  class="promo-banner"
  style="--promo-background: {{ section.settings.background }};"
>
```

Giới hạn inline style ở việc truyền design token hoặc value cấu hình. Không đưa toàn bộ stylesheet lớn vào thuộc tính `style`.

### Naming

Có thể dùng BEM hoặc convention nhất quán:

```text
product-card
product-card__media
product-card__title
product-card--featured
```

Ưu tiên class theo component, tránh selector sâu:

Không tốt:

```css
main .page-width .grid > div article h3 a span {
  ...
}
```

Tốt:

```css
.product-card__title-link {
  ...
}
```

## 17.2 JavaScript

### Script asset

```liquid
<script src="{{ 'product-form.js' | asset_url }}" defer="defer"></script>
```

### Progressive enhancement

HTML cơ bản cần hoạt động trước khi JavaScript chạy:

- Link vẫn điều hướng được.
- Product form vẫn submit được.
- Navigation vẫn có fallback.
- Nội dung cốt lõi không bị phụ thuộc hoàn toàn vào client rendering.

### Custom element mẫu

```js
class QuantityInput extends HTMLElement {
  connectedCallback() {
    this.input = this.querySelector('input[type="number"]');
    this.addEventListener('click', this.onClick.bind(this));
  }

  onClick(event) {
    const button = event.target.closest('button[data-action]');

    if (!button || !this.input) return;

    const currentValue = Number(this.input.value || 0);
    const step = Number(this.input.step || 1);

    if (button.dataset.action === 'increase') {
      this.input.value = currentValue + step;
    }

    if (button.dataset.action === 'decrease') {
      const min = Number(this.input.min || 0);
      this.input.value = Math.max(min, currentValue - step);
    }

    this.input.dispatchEvent(new Event('change', { bubbles: true }));
  }
}

if (!customElements.get('quantity-input')) {
  customElements.define('quantity-input', QuantityInput);
}
```

HTML:

```liquid
<quantity-input>
  <button type="button" data-action="decrease" aria-label="Decrease quantity">−</button>
  <input type="number" name="quantity" value="1" min="1">
  <button type="button" data-action="increase" aria-label="Increase quantity">+</button>
</quantity-input>
```

### Quy tắc JavaScript

- Không gắn cùng event listener nhiều lần khi section re-render.
- Không phụ thuộc ID cố định nếu section có thể xuất hiện nhiều instance.
- Dùng `section.id` hoặc query trong component root.
- Cleanup listeners, observers và timers khi section bị unload nếu cần.
- Không dùng thư viện lớn cho chức năng browser native có thể xử lý.
- Tránh global variables; dùng module hoặc custom elements.
- Không nhúng dữ liệu Liquid vào JS bằng string không được `json` encode.

---

## 18. Theme Editor và JavaScript events

Trong Theme Editor, section có thể được thêm, xóa, sắp xếp hoặc re-render mà không reload toàn trang.

JavaScript cần xử lý các event liên quan, ví dụ:

```js
document.addEventListener('shopify:section:load', (event) => {
  initializeSection(event.target);
});

document.addEventListener('shopify:section:unload', (event) => {
  destroySection(event.target);
});

document.addEventListener('shopify:block:select', (event) => {
  event.target.scrollIntoView({ behavior: 'smooth', block: 'center' });
});
```

### Pattern khởi tạo an toàn

```js
function initializeSection(root = document) {
  root.querySelectorAll('[data-slider]:not([data-initialized])').forEach((element) => {
    element.dataset.initialized = 'true';
    // Initialize feature here.
  });
}

document.addEventListener('DOMContentLoaded', () => {
  initializeSection(document);
});

document.addEventListener('shopify:section:load', (event) => {
  initializeSection(event.target);
});
```

### Kiểm tra design mode trong Liquid

```liquid
{% if request.design_mode %}
  <p class="theme-editor-note">Visible only in the Theme Editor.</p>
{% endif %}
```

Không ẩn lỗi thực tế chỉ vì đang ở Theme Editor. `request.design_mode` nên dùng cho hướng dẫn hoặc behavior dành riêng cho editor.

---

## 19. Performance

### 19.1 HTML/CSS trước, JavaScript sau

- Dùng HTML semantic.
- Dùng CSS cho layout, animation đơn giản, disclosure khi browser hỗ trợ.
- JavaScript chỉ là progressive enhancement.

### 19.2 Chỉ tải asset cần thiết

Không tải script product gallery trên blog page.

Có thể conditionally load:

```liquid
{% if request.page_type == 'product' %}
  <script src="{{ 'product.js' | asset_url }}" defer="defer"></script>
{% endif %}
```

Tuy nhiên, nếu feature được gắn với section động, tốt hơn là tải asset cùng section/component để tránh phụ thuộc page type quá cứng.

### 19.3 Tránh loop nặng

Không tốt:

```liquid
{% for collection in collections %}
  {% for product in collection.products %}
    ...
  {% endfor %}
{% endfor %}
```

Giới hạn dữ liệu hoặc dùng setting để merchant chọn resource cần render.

### 19.4 Pagination

Dùng `{% paginate %}` cho danh sách lớn thay vì cố render tất cả.

### 19.5 Ảnh

- CDN transformations qua `image_url`.
- Responsive `srcset` qua `image_tag`.
- Lazy-load ảnh dưới fold.
- Không tải ảnh lớn hơn kích thước hiển thị cần thiết.
- Width/height để hạn chế cumulative layout shift.

### 19.6 Fonts

- Giảm số font families và font weights.
- Chỉ preload font thực sự cần cho nội dung above-the-fold.
- Dùng `font-display` phù hợp.
- Ưu tiên system fonts nếu branding cho phép.

### 19.7 Third-party scripts

- Kiểm tra tác động của analytics, chat, reviews, popup và pixels.
- Không chèn app code trực tiếp vào theme nếu app hỗ trợ app embed hoặc theme app extension.
- Tải script theo consent khi quy định pháp lý yêu cầu.

### 19.8 Đo lường

Kiểm tra ít nhất:

- Home page.
- Product page.
- Collection page.
- Cart.
- Mobile viewport.
- Store có dữ liệu thực tế, không chỉ demo ít sản phẩm.

Dùng:

```bash
shopify theme check
shopify theme profile
```

Kết hợp Lighthouse/Web Vitals trong quy trình QA.

---

## 20. Accessibility

### 20.1 Semantic HTML

Dùng đúng element:

```html
<button type="button">Open menu</button>
<a href="/collections/all">Shop all</a>
```

Không dùng `<div>` giả làm button.

### 20.2 Keyboard

- Mọi chức năng tương tác phải dùng được bằng bàn phím.
- Focus indicator phải nhìn thấy.
- Modal/drawer cần quản lý focus.
- Escape nên đóng dialog phù hợp.
- Không tạo keyboard trap.

### 20.3 Form labels

```liquid
<label for="Email-{{ section.id }}">Email</label>
<input id="Email-{{ section.id }}" type="email" name="contact[email]">
```

Placeholder không thay thế label.

### 20.4 Button icon

```liquid
<button type="button" aria-label="{{ 'general.search.search' | t }}">
  {% render 'icon-search' %}
</button>
```

SVG trang trí:

```html
<svg aria-hidden="true" focusable="false" ...>
```

### 20.5 Heading hierarchy

- Mỗi trang nên có heading chính rõ ràng.
- Không chọn heading level chỉ để đạt kích thước chữ mong muốn.
- Style bằng CSS, giữ cấu trúc semantic.

### 20.6 Status messages

AJAX cart hoặc validation cần thông báo cho assistive technology:

```html
<div role="status" aria-live="polite" aria-atomic="true"></div>
```

### 20.7 Reduced motion

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    scroll-behavior: auto !important;
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
  }
}
```

### 20.8 Color contrast

Merchant có thể chọn màu làm giảm contrast. Hãy:

- Cung cấp color schemes đã được kiểm tra.
- Hạn chế combination nguy hiểm.
- Không truyền thông tin chỉ bằng màu sắc.

---

## 21. SEO và structured data

### 21.1 Title, description, canonical

Trong layout:

```liquid
<link rel="canonical" href="{{ canonical_url }}">

<title>{{ page_title | escape }}</title>

{% if page_description %}
  <meta name="description" content="{{ page_description | escape }}">
{% endif %}
```

Theme hoàn chỉnh thường bổ sung shop name, pagination và tags vào title một cách có kiểm soát.

### 21.2 Open Graph/social metadata

Tách thành snippet:

```liquid
{% render 'meta-tags' %}
```

Snippet cần xử lý đúng context product, article, collection và fallback image.

### 21.3 Structured data

Dùng JSON-LD và luôn encode dữ liệu:

```liquid
<script type="application/ld+json">
  {
    "@context": "https://schema.org",
    "@type": "Product",
    "name": {{ product.title | json }},
    "url": {{ request.origin | append: product.url | json }},
    "description": {{ product.description | strip_html | json }}
  }
</script>
```

Đây chỉ là ví dụ tối giản. Product structured data production cần xử lý offers, variants, availability, currency, images, identifiers và tránh trùng schema do app/theme khác tạo.

### 21.4 Nội dung SEO

- Product title dùng heading phù hợp.
- Collection có description khi merchant nhập.
- Link dùng anchor text có nghĩa.
- Không render text ẩn chỉ để nhồi keyword.
- Không tạo nhiều URL duplicate không cần thiết.
- Giữ canonical và localized URLs đúng theo Shopify.

---

## 22. Bảo mật và escaping

### 22.1 Merchant/customer-generated text

```liquid
{{ customer.first_name | escape }}
{{ section.settings.heading | escape }}
```

### 22.2 JSON

```liquid
<script>
  window.themeData = {
    productTitle: {{ product.title | json }},
    cartUrl: {{ routes.cart_url | json }}
  };
</script>
```

### 22.3 Rich text

Chỉ render HTML không escape từ nguồn Shopify đã được thiết kế cho rich text:

```liquid
{{ section.settings.rich_text }}
```

Không render chuỗi customer nhập dưới dạng raw HTML.

### 22.4 Secrets

Không lưu trong theme:

- Admin API access token.
- App client secret.
- Private API key.
- Database credentials.
- Webhook signing secret.

Mọi file theme đều có thể bị tải về bởi người có quyền theme và phần frontend được browser nhận. Theme không phải nơi lưu secret.

### 22.5 External links

Khi mở tab mới:

```liquid
<a
  href="{{ section.settings.external_link }}"
  target="_blank"
  rel="noopener noreferrer"
>
  {{ section.settings.external_label | escape }}
</a>
```

---

## 23. Shopify CLI, Theme Check và Git workflow

## 23.1 Khởi tạo hoặc tải theme

Khởi tạo theme:

```bash
shopify theme init
```

Kéo theme từ store:

```bash
shopify theme pull --store your-store.myshopify.com
```

Nên xác định đúng theme cần pull khi CLI yêu cầu chọn, tránh lấy nhầm live theme.

## 23.2 Chạy local development

```bash
shopify theme dev --store your-store.myshopify.com
```

Lệnh này tạo hoặc dùng development theme để preview, hot reload và test thay đổi mà không ghi trực tiếp lên live theme.

## 23.3 Kiểm tra code

```bash
shopify theme check
```

Chạy trước mỗi commit hoặc pull request.

Có thể cấu hình `.theme-check.yml`:

```yaml
extends: theme-check:recommended

ignore:
  - node_modules/**
  - dist/**
```

Không tắt check chỉ để làm CI xanh. Nếu phải ignore, ghi rõ lý do.

## 23.4 Push theme

Đẩy lên một unpublished theme trước:

```bash
shopify theme push --unpublished --store your-store.myshopify.com
```

Hoặc chọn theme cụ thể theo prompt/ID khi cần.

Không push thẳng live theme trong workflow thông thường nếu chưa preview và QA.

## 23.5 Publish

Sau khi QA:

```bash
shopify theme publish --store your-store.myshopify.com
```

Publish là hành động production, cần xác nhận đúng theme.

## 23.6 Package bàn giao

```bash
shopify theme package
```

Lệnh tạo ZIP theme để upload hoặc bàn giao.

## 23.7 Theme environments

Dự án nhiều store/môi trường có thể dùng `shopify.theme.toml`:

```toml
[environments.development]
store = "dev-store.myshopify.com"

[environments.production]
store = "production-store.myshopify.com"
```

Ví dụ sử dụng:

```bash
shopify theme dev --environment development
shopify theme push --environment production
```

Không commit password/token nhạy cảm vào repository.

## 23.8 Git workflow đề xuất

```text
main
└── develop
    ├── feature/product-gallery
    ├── feature/mega-menu
    └── fix/cart-quantity
```

Quy trình:

1. Pull code/theme state mới nhất.
2. Tạo branch theo feature.
3. Chạy `shopify theme dev`.
4. Code và test nhiều page/context.
5. Chạy `shopify theme check`.
6. Review Git diff, đặc biệt JSON templates và `settings_data.json`.
7. Commit nhỏ, message rõ nghĩa.
8. Push lên unpublished/staging theme.
9. QA.
10. Merge và publish theo quy trình dự án.

Ví dụ commit:

```text
feat(product): add accessible variant picker
fix(cart): preserve line item properties on quantity update
perf(images): add responsive widths to product cards
```

## 23.9 Không sửa đồng thời hai nguồn mà không đồng bộ

Nếu một người sửa code local trong khi người khác sửa trực tiếp Online Code Editor, thay đổi có thể bị ghi đè.

Chọn một source of truth:

- Git repository là nguồn chính.
- Theme trên Shopify là deployment target.
- Hạn chế sửa production trực tiếp.
- Nếu có hotfix trong Admin, pull và commit ngược về Git ngay sau đó.

---

## 24. Cấu trúc mẫu cho một feature

Ví dụ feature “Featured Collection”:

```text
sections/featured-collection.liquid
snippets/product-card.liquid
snippets/price.liquid
assets/section-featured-collection.css
assets/component-product-card.css
assets/product-card.js
locales/en.default.json
locales/vi.json
```

### Section

- Nhận collection, heading, số sản phẩm, columns, color scheme.
- Paginate hoặc giới hạn hợp lý.
- Loop products.
- Gọi `product-card` snippet.
- Có empty state trong Theme Editor.

### Product card snippet

- Nhận `card_product` bắt buộc.
- Render ảnh responsive.
- Render title, price, vendor theo options.
- Không phụ thuộc biến local của section.

### CSS

- Grid/layout của section.
- Component style của card.
- Responsive breakpoints.
- Focus/hover/reduced-motion states.

### JavaScript

Chỉ cần nếu có slider, quick add hoặc tương tác không thể giải quyết bằng HTML/CSS.

### Locale

Mọi text hệ thống như “View all”, “Sold out”, “Quick add” dùng translation keys.

---

## 25. Những lỗi thường gặp

### 25.1 Hard-code URL

Sai:

```liquid
<a href="/cart">Cart</a>
```

Đúng:

```liquid
<a href="{{ routes.cart_url }}">Cart</a>
```

### 25.2 Hard-code text giao diện

Sai:

```liquid
<button>Add to cart</button>
```

Đúng:

```liquid
<button>{{ 'products.product.add_to_cart' | t }}</button>
```

### 25.3 Dùng ảnh gốc không resize

Sai:

```liquid
<img src="{{ product.featured_image | image_url }}">
```

Tốt hơn:

```liquid
{{
  product.featured_image
  | image_url: width: 900
  | image_tag: widths: '320, 480, 750, 900', loading: 'lazy'
}}
```

### 25.4 Không kiểm tra `blank`

Sai:

```liquid
<div class="banner">
  <img src="{{ section.settings.image | image_url }}">
</div>
```

Đúng:

```liquid
{% if section.settings.image != blank %}
  <div class="banner">
    {{ section.settings.image | image_url: width: 1600 | image_tag }}
  </div>
{% endif %}
```

### 25.5 Trùng DOM ID

Sai:

```liquid
<div id="ProductGallery">
```

Đúng:

```liquid
<div id="ProductGallery-{{ section.id }}">
```

### 25.6 Snippet dùng biến ngầm

Sai:

```liquid
{% render 'card' %}
```

Trong snippet lại tự giả định có biến `product` local.

Đúng:

```liquid
{% render 'product-card', card_product: product %}
```

### 25.7 Bỏ `block.shopify_attributes`

Điều này làm trải nghiệm chọn/edit block trong Theme Editor kém hoặc sai.

### 25.8 JavaScript chỉ khởi tạo ở `DOMContentLoaded`

Section được Theme Editor re-render sau đó sẽ không hoạt động. Cần xử lý `shopify:section:load`.

### 25.9 Push nhầm live theme

Luôn xem theme list/status và ưu tiên unpublished/development theme cho QA.

### 25.10 Commit secret

Theme không được chứa Admin API token, client secret hoặc password.

### 25.11 Dùng quá nhiều settings

Một section có hàng chục setting về từng pixel/margin/color riêng thường khó dùng, khó giữ consistency và khó bảo trì. Xây design system với các option có kiểm soát.

### 25.12 Render toàn bộ variant data không cần thiết

Có thể làm HTML/JSON nặng và không phù hợp với high-variant products. Chỉ tải dữ liệu cần cho UI hiện tại.

---

## 26. Checklist trước khi bàn giao hoặc publish

### Liquid và kiến trúc

- [ ] JSON templates chỉ quản lý sections/order.
- [ ] Logic tái sử dụng đã tách thành snippets hợp lý.
- [ ] Snippets có tham số rõ ràng và được document.
- [ ] Không phụ thuộc biến ngầm không cần thiết.
- [ ] Section có schema JSON hợp lệ.
- [ ] Section merchant cần thêm có `presets`.
- [ ] Blocks có `block.shopify_attributes`.
- [ ] DOM IDs có `section.id` hoặc giá trị unique.
- [ ] Object/metafield tùy chọn được kiểm tra `blank`.
- [ ] URLs nội bộ dùng `routes` hoặc resource URL.

### Theme Editor

- [ ] Setting names dễ hiểu với merchant.
- [ ] Defaults tạo giao diện hợp lý.
- [ ] Không có setting kỹ thuật không cần thiết.
- [ ] JavaScript hoạt động khi section load/unload/reorder.
- [ ] Empty state có hướng dẫn trong design mode khi cần.
- [ ] App block/app embed được hỗ trợ đúng nơi nếu feature cần.

### Product/commerce

- [ ] Variant picker cập nhật đúng variant ID.
- [ ] Price, compare-at price và availability đúng.
- [ ] Add-to-cart hoạt động không cần JavaScript ở mức cơ bản.
- [ ] Quantity có min/value hợp lệ.
- [ ] Selling plans/subscriptions không bị phá nếu store dùng.
- [ ] Line item properties không bị mất.
- [ ] Cart totals và discounts render đúng.
- [ ] Theme được thử với sold-out, zero-price, one-variant và nhiều-variant products.

### Performance

- [ ] Ảnh dùng Shopify CDN resizing.
- [ ] Ảnh có responsive widths/sizes.
- [ ] Ảnh dưới fold lazy-load.
- [ ] Ảnh LCP không bị lazy-load sai.
- [ ] Width/height được cung cấp để giảm layout shift.
- [ ] Không tải JavaScript không cần thiết.
- [ ] Script dùng `defer` khi phù hợp.
- [ ] Không có loop dữ liệu quá lớn.
- [ ] Third-party scripts đã được đánh giá.
- [ ] Đã test mobile và dữ liệu thực tế.

### Accessibility

- [ ] Có skip link.
- [ ] Form controls có label.
- [ ] Icon buttons có accessible name.
- [ ] Keyboard navigation hoạt động.
- [ ] Focus state nhìn thấy.
- [ ] Modal/drawer quản lý focus đúng.
- [ ] Heading hierarchy hợp lý.
- [ ] Alt text đúng ngữ cảnh.
- [ ] Live regions dùng cho cập nhật AJAX quan trọng.
- [ ] Tôn trọng `prefers-reduced-motion`.
- [ ] Color contrast đủ tốt.

### SEO

- [ ] Title và meta description đúng context.
- [ ] Canonical URL được giữ.
- [ ] Open Graph/social metadata hợp lệ.
- [ ] Structured data không lỗi/trùng không cần thiết.
- [ ] Nội dung chính có semantic headings.
- [ ] Link text có nghĩa.

### Security

- [ ] Text không tin cậy được `escape`.
- [ ] Dữ liệu truyền sang JavaScript dùng `json`.
- [ ] Không có secret/token/password trong repo.
- [ ] External links mở tab mới có `noopener noreferrer`.

### Workflow

- [ ] `shopify theme check` không có lỗi nghiêm trọng.
- [ ] Git diff đã được review.
- [ ] Không ghi đè `settings_data.json` ngoài ý muốn.
- [ ] Đã preview trên development/unpublished theme.
- [ ] Đã test home, product, collection, search, cart và customer flow liên quan.
- [ ] Có backup hoặc rollback theme trước khi publish.
- [ ] Theme version/changelog được cập nhật nếu dự án yêu cầu.

---

## 27. Tài liệu chính thức

Các nguồn nên dùng làm source of truth:

- Liquid reference: <https://shopify.dev/docs/api/liquid>
- Liquid basics: <https://shopify.dev/docs/api/liquid/basics>
- Theme architecture: <https://shopify.dev/docs/storefronts/themes/architecture>
- Layouts: <https://shopify.dev/docs/storefronts/themes/architecture/layouts>
- Templates: <https://shopify.dev/docs/storefronts/themes/architecture/templates>
- JSON templates: <https://shopify.dev/docs/storefronts/themes/architecture/templates/json-templates>
- Sections: <https://shopify.dev/docs/storefronts/themes/architecture/sections>
- Section schema: <https://shopify.dev/docs/storefronts/themes/architecture/sections/section-schema>
- Blocks: <https://shopify.dev/docs/storefronts/themes/architecture/blocks>
- App blocks: <https://shopify.dev/docs/storefronts/themes/architecture/blocks/app-blocks>
- Settings: <https://shopify.dev/docs/storefronts/themes/architecture/settings>
- `settings_schema.json`: <https://shopify.dev/docs/storefronts/themes/architecture/config/settings-schema-json>
- Locales: <https://shopify.dev/docs/storefronts/themes/architecture/locales>
- Best practices: <https://shopify.dev/docs/storefronts/themes/best-practices>
- Performance: <https://shopify.dev/docs/storefronts/themes/best-practices/performance>
- Accessibility: <https://shopify.dev/docs/storefronts/themes/best-practices/accessibility>
- Sections and blocks best practices: <https://shopify.dev/docs/storefronts/themes/best-practices/templates-sections-blocks>
- Shopify CLI for themes: <https://shopify.dev/docs/storefronts/themes/tools/cli>
- Shopify CLI theme commands: <https://shopify.dev/docs/api/shopify-cli/theme>
- Theme Check: <https://shopify.dev/docs/storefronts/themes/tools/theme-check>
- Product media: <https://shopify.dev/docs/storefronts/themes/product-merchandising/media/support-media>
- High-variant products: <https://shopify.dev/docs/storefronts/themes/product-merchandising/variants/support-high-variant-products>

---

## Kết luận

Một Shopify theme “đúng chuẩn” không chỉ là Liquid render đúng dữ liệu. Theme cần đồng thời:

- Có kiến trúc rõ ràng.
- Dễ cấu hình trong Theme Editor.
- Không phá chức năng commerce cốt lõi.
- Tái sử dụng component hợp lý.
- Chạy nhanh và ổn định.
- Truy cập được bằng bàn phím và assistive technology.
- Hỗ trợ localization.
- An toàn với dữ liệu đầu vào.
- Có workflow Git/CLI giúp tránh ghi đè production.

Khi không chắc về object, tag, filter, schema attribute hoặc giới hạn dữ liệu, hãy kiểm tra Shopify Liquid reference và theme architecture documentation thay vì suy đoán.
