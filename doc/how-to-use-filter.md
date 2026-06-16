Chào bạn, để triển khai tính năng lọc sản phẩm một cách chuẩn chỉnh nhất trên môi trường code theme cục bộ (local theme) của Shopify, chúng ta sẽ kết hợp sử dụng ứng dụng **Search & Discovery** của Shopify để quản lý dữ liệu, và đối tượng `collection.filters` (hoặc `search.filters`) trong mã nguồn Liquid để hiển thị.

Dưới đây là hướng dẫn chi tiết từng bước để bạn có thể xây dựng tính năng này, bao gồm cả cấu trúc Liquid tiêu chuẩn và tư duy xử lý JavaScript để không phải tải lại trang.

### Bước 1: Cấu Hình Dữ Liệu Qua Ứng Dụng Search & Discovery

Trước khi can thiệp vào mã nguồn, bạn bắt buộc phải cài đặt ứng dụng **Shopify Search & Discovery** (miễn phí từ Shopify) trong trang quản trị. Ứng dụng này dùng để thiết lập các bộ lọc (ví dụ: tình trạng còn hàng, giá cả, màu sắc, kích thước, nhà cung cấp, hoặc metafields). Chỉ những bộ lọc được bật trong ứng dụng này mới có thể được gọi ra thông qua đối tượng Liquid trên giao diện cửa hàng.

### Bước 2: Khởi Tạo Biểu Mẫu (Form) Lọc Trong Liquid

Trong tệp mã nguồn hiển thị danh sách sản phẩm (ví dụ: `main-collection-product-grid.liquid` hoặc tệp tương tự trong thư mục `sections`), bạn cần bọc toàn bộ các khối bộ lọc trong một thẻ `<form>`. Thẻ biểu mẫu này giúp hệ thống thu thập tất cả các tham số một cách có hệ thống khi người dùng lựa chọn.

```liquid
<form id="CollectionFiltersForm" class="facets__form">
  {%- comment -%} Giữ lại tùy chọn sắp xếp (sort) hiện tại nếu có {%- endcomment -%}
  {%- if collection.sort_by -%}
    <input type="hidden" name="sort_by" value="{{ collection.sort_by }}">
  {%- endif -%}

  {%- for filter in collection.filters -%}
    <details class="filter-group">
      <summary class="filter-group-summary">
        <span>{{ filter.label }}</span>
      </summary>

      <div class="filter-group-display">
        {%- case filter.type -%}
          {%- when 'boolean' or 'list' -%}
            <ul class="filter-group-display__list">
              {%- for filter_value in filter.values -%}
                <li class="filter-group-display__list-item">
                  <label for="Filter-{{ filter.param_name }}-{{ forloop.index }}">
                    <input type="checkbox"
                      name="{{ filter_value.param_name }}"
                      value="{{ filter_value.value }}"
                      id="Filter-{{ filter.param_name }}-{{ forloop.index }}"
                      {% if filter_value.active -%}checked{%- endif %}
                      {% if filter_value.count == 0 and filter_value.active == false -%}disabled{%- endif %}
                    >
                    <span>{{ filter_value.label }} ({{ filter_value.count }})</span>
                  </label>
                </li>
              {%- endfor -%}
            </ul>

          {%- when 'price_range' -%}
            <div class="filter-group-display__price-range">
              <div class="filter-group-display__price-range-from">
                <label>Từ</label>
                <input name="{{ filter.min_value.param_name }}"
                  id="Filter-{{ filter.min_value.param_name }}"
                  {% if filter.min_value.value -%}
                    value="{{ filter.min_value.value | divided_by: 100 }}"
                  {%- endif %}
                  type="number"
                  placeholder="0"
                  min="0"
                  max="{{ filter.range_max | divided_by: 100 | ceil }}">
              </div>
              <div class="filter-group-display__price-range-to">
                <label>Đến</label>
                <input name="{{ filter.max_value.param_name }}"
                  id="Filter-{{ filter.max_value.param_name }}"
                  {% if filter.max_value.value -%}
                    value="{{ filter.max_value.value | divided_by: 100 }}"
                  {%- endif %}
                  type="number"
                  placeholder="{{ filter.range_max | divided_by: 100 | ceil }}"
                  min="0"
                  max="{{ filter.range_max | divided_by: 100 | ceil }}">
              </div>
            </div>
        {%- endcase -%}
      </div>
    </details>
  {%- endfor -%}
</form>

```

### Bước 3: Hiển Thị Các Bộ Lọc Đang Được Áp Dụng (Active Filters)

Người dùng cần nhìn thấy rõ những bộ lọc nào đang được kích hoạt và có khả năng xóa chúng một cách dễ dàng. Bạn có thể duyệt qua các giá trị đang hoạt động để hiển thị nút xóa:

```liquid
<div class="active-filters">
  <a href="{{ collection.url }}?sort_by={{ collection.sort_by }}" class="active-filters__clear">Xóa tất cả</a>
  {%- for filter in collection.filters -%}
    {%- for filter_value in filter.active_values -%}
      <a class="active-filters__remove-filter" href="{{ filter_value.url_to_remove }}">
        {{ filter.label }}: {{ filter_value.label }} ✕
      </a>
    {%- endfor -%}
  {%- endfor -%}
</div>

```

### Bước 4: Xử Lý JavaScript Chuyên Sâu (Sử Dụng Section Rendering API)

Cách thức hoạt động chuẩn của các theme hiện đại (như theme Dawn mặc định của Shopify) là không tải lại toàn bộ trang web khi người dùng nhấn chọn một bộ lọc. Thay vào đó, hệ thống sử dụng JavaScript để cập nhật giao diện thông qua API.

Quy trình hoạt động logic của JavaScript sẽ diễn ra như sau:

1. Lắng nghe sự kiện thay đổi (`change`) trên biểu mẫu `<form id="CollectionFiltersForm">`.
2. Khởi tạo một đối tượng `FormData` từ biểu mẫu và chuyển đổi các giá trị thu thập được thành chuỗi tham số URL (URL search parameters).
3. Sử dụng hàm `fetch()` của JavaScript gọi đến đường dẫn hiện tại kết hợp cùng chuỗi tham số vừa tạo, đặc biệt phải đính kèm tham số `?section_id=ID_CUA_SECTION_SAN_PHAM`.
4. Khi nhận được phản hồi (response) từ máy chủ, phân tích cú pháp HTML trả về để trích xuất riêng phần danh sách sản phẩm đã được lọc, sau đó tiến hành thay thế phần tử cũ trên giao diện bằng đoạn HTML mới này.
5. Cập nhật đường dẫn trên thanh địa chỉ của trình duyệt web bằng hàm `history.pushState()`, để người dùng có thể sao chép đường dẫn bộ lọc chính xác và chia sẻ cho người khác.

Cách tiếp cận này đảm bảo mang lại trải nghiệm người dùng mượt mà nhất, tối ưu hóa tốc độ tải trang, và hoàn toàn tuân thủ theo tiêu chuẩn phát triển theme cao cấp của Shopify.
