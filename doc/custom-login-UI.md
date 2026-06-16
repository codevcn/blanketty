Đây là flow đầy đủ khi dùng **Classic Customer Accounts**:

---

## 1. Bật Classic Customer Accounts

Vào **Settings > Customer accounts** → chọn **Classic accounts**.

---

## 2. Các trang có sẵn trên domain store

Shopify tự tạo sẵn các route:

| Route               | Chức năng                   |
| ------------------- | --------------------------- |
| `/account/login`    | Trang đăng nhập             |
| `/account/register` | Trang đăng ký               |
| `/account`          | Trang tài khoản (sau login) |
| `/account/logout`   | Đăng xuất                   |
| `/account/reset`    | Reset mật khẩu              |

---

## 3. Tùy chỉnh giao diện trong theme

Bạn chỉnh UI qua các file Liquid:

```
theme/
└── templates/
    └── customers/
        ├── login.liquid        ← UI trang đăng nhập
        ├── register.liquid     ← UI trang đăng ký
        ├── account.liquid      ← UI trang tài khoản
        └── reset_password.liquid
```

Bạn có thể viết HTML/CSS tùy ý trong các file này.

---

## 4. Form đăng nhập hoạt động thế nào

Shopify xử lý authentication qua **form action chuẩn**:

```html
<!-- login.liquid -->
<form method="POST" action="/account/login">
  <input type="email" name="customer[email]" />
  <input type="password" name="customer[password]" />
  <button type="submit">Đăng nhập</button>
</form>
```

Shopify tự xử lý logic xác thực, bạn chỉ cần đúng `action` và `name` của input.

---

## 5. Form đăng ký tương tự

```html
<!-- register.liquid -->
<form method="POST" action="/account">
  <input type="text" name="customer[first_name]" />
  <input type="text" name="customer[last_name]" />
  <input type="email" name="customer[email]" />
  <input type="password" name="customer[password]" />
  <button type="submit">Đăng ký</button>
</form>
```

---

## 6. Kiểm tra trạng thái đăng nhập trong Liquid

```liquid
{% if customer %}
  Xin chào, {{ customer.first_name }}!
  <a href="/account/logout">Đăng xuất</a>
{% else %}
  <a href="/account/login">Đăng nhập</a>
{% endif %}
```
