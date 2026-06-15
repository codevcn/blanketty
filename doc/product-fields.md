# Shopify Product Fields Documentation

Tài liệu này mô tả chi tiết các trường dữ liệu (fields) của một sản phẩm trên hệ thống Shopify, được trích xuất từ file data import mẫu (`shopify-blanket-products-12-fixed.csv`). 

Những trường này đại diện cho cấu trúc dữ liệu mà bạn có thể cấu hình cho mỗi sản phẩm trong trang quản trị (Admin Panel) và đồng thời cũng là những trường dữ liệu bạn có thể truy xuất thông qua mã Liquid để hiển thị lên giao diện (front-end).

## 1. Thông tin cơ bản (Basic Information)
- **Title (Tên sản phẩm):** Tên hiển thị chính thức của sản phẩm. *Trong Liquid: `{{ product.title }}`*
- **URL handle:** Đường dẫn rút gọn không dấu (slug) dùng cho URL của sản phẩm (VD: `chan-long-cuu`). *Trong Liquid: `{{ product.handle }}`*
- **Description (Mô tả):** Đoạn văn bản (hỗ trợ HTML) mô tả chi tiết về sản phẩm. *Trong Liquid: `{{ product.description }}` hoặc `{{ product.content }}`*
- **Vendor (Nhà cung cấp / Thương hiệu):** Tên hãng hoặc người bán (VD: `CozyNest Home`). *Trong Liquid: `{{ product.vendor }}`*
- **Product category (Danh mục chuẩn):** Mã danh mục tiêu chuẩn của Google/Shopify (VD: `hg-15-1-4` tương ứng với Chăn mền).
- **Type (Loại sản phẩm):** Loại tùy chỉnh do chủ shop tự phân loại (VD: `Chăn mền`). *Trong Liquid: `{{ product.type }}`*
- **Tags (Thẻ):** Các từ khóa phân cách bằng dấu phẩy dùng để lọc và tìm kiếm (VD: `Chăn dệt kim, Sofa, Trang trí`). *Trong Liquid lặp qua mảng `{{ product.tags }}`*

## 2. Trạng thái và Phân phối (Status & Publishing)
- **Published on online store:** Trạng thái cho phép (TRUE/FALSE) sản phẩm có hiển thị trên website hay không.
- **Status (Trạng thái):** Trạng thái hoạt động của sản phẩm (`active`, `draft`, hoặc `archived`).

## 3. Quản lý Kho và Phân loại (Inventory & Variants)
- **SKU:** Mã quản lý nội bộ của sản phẩm (Mã vạch kho, VD: `BLK-COZY-001`). *Trong Liquid, SKU thuộc về Variant: `{{ product.selected_or_first_available_variant.sku }}`*
- **Barcode:** Mã vạch toàn cầu (UPC, GTIN, v.v.).
- **Option1/2/3 name & value (Tùy chọn biến thể):** Các thuộc tính để phân loại biến thể như Kích thước, Màu sắc, Chất liệu. Mỗi sản phẩm hỗ trợ tối đa 3 Option name (VD: Option1 Name = `Title`, Value = `Default Title` - Sản phẩm không có biến thể phức tạp). *Trong Liquid: `{{ product.options }}` và `{{ product.variants }}`*
- **Inventory tracker:** Hệ thống theo dõi tồn kho (thường là `shopify`).
- **Inventory quantity:** Số lượng hàng còn trong kho. *Truy xuất qua variant trong Liquid: `{{ variant.inventory_quantity }}`*
- **Continue selling when out of stock:** Tùy chọn (deny/continue) cho phép khách hàng tiếp tục mua khi tồn kho bằng 0.

## 4. Định giá (Pricing)
- **Price (Giá bán):** Giá thực tế khách hàng phải trả. *Trong Liquid: `{{ product.price | money }}`*
- **Compare-at price (Giá gốc / Giá so sánh):** Giá ban đầu dùng để tạo hiệu ứng gạch bỏ (giảm giá). *Trong Liquid: `{{ product.compare_at_price | money }}`*
- **Cost per item (Giá vốn):** Giá gốc nhập hàng (chỉ dùng nội bộ, không hiển thị cho khách).
- **Charge tax:** Tùy chọn có tính thuế hay không (TRUE/FALSE).
- **Tax code:** Mã thuế đặc thù.

## 5. Vận chuyển (Shipping)
- **Weight value & Weight unit:** Trọng lượng sản phẩm và đơn vị (VD: `1600 g`). Dùng để tính phí ship.
- **Requires shipping:** Sản phẩm có cần giao hàng vật lý không (TRUE/FALSE).
- **Fulfillment service:** Đơn vị xử lý hoàn tất đơn hàng (mặc định là `manual`).

## 6. Hình ảnh (Media/Images)
- **Product image URL:** Đường dẫn file ảnh chính của sản phẩm. *Trong Liquid: `{{ product.featured_image }}` hoặc lặp qua `{{ product.images }}`*
- **Image position:** Thứ tự sắp xếp của ảnh (1, 2, 3...).
- **Image alt text:** Văn bản thay thế cho ảnh, cực kỳ quan trọng cho SEO và người khiếm thị. *Trong Liquid: `{{ image.alt }}`*

## 7. Metafields và Cấu hình Nâng cao (Advanced & Metafields)
- **Gift card:** Xác định xem đây có phải là thẻ quà tặng ảo hay không (TRUE/FALSE).
- **Color (product.metafields.shopify.color-pattern):** Một ví dụ về **Metafield** (trường dữ liệu tùy biến mở rộng) dùng để lưu mã màu chuẩn hoặc quy tắc màu đặc thù. *Trong Liquid: `{{ product.metafields.shopify.color-pattern }}`*

## 8. SEO (Tối ưu hóa công cụ tìm kiếm)
- **SEO title:** Tiêu đề thẻ `<title>` tối ưu hóa cho Google.
- **SEO description:** Thẻ `<meta description>` chứa đoạn văn giới thiệu tối ưu hóa cho Google.

## 9. Google Shopping / Kênh bán hàng (Sales Channels)
Các trường bắt đầu bằng `Google Shopping / ...` dùng riêng cho mục đích đồng bộ dữ liệu chuẩn lên Google Merchant Center để chạy quảng cáo Google Shopping. Bao gồm:
- **Google product category:** Danh mục phân loại chuẩn của Google.
- **Gender & Age group:** Giới tính và độ tuổi mục tiêu.
- **Custom label 0-4:** Nhãn tùy chỉnh để chia chiến dịch quảng cáo.
- **Condition:** Tình trạng sản phẩm (Mới/Cũ).
