# MiniShop — Buổi 5

Tiếp nối trọn Buổi 4, hoàn thiện theo [giáo trình Drive 158 trang](https://drive.google.com/file/d/1gSqNqCIeJO6ZPmCOMRtSOEPbuv7Wlb-S/view), **trang in 127–155 / trang PDF 130–158**.

Đã làm các chức năng quản trị, bảo mật và **cả 6 bài tập tổng kết**. Đã chuẩn bị gói triển khai cho Lab 5.5. Việc gắn tên miền, cấp SSL và nghiệm thu trên hosting thật **chưa thực hiện**, vì chưa có thông tin hosting/tên miền. Xem [hướng dẫn triển khai](docs/DEPLOYMENT.md) để làm tiếp phần này.

## Chạy nhanh bằng XAMPP

1. Giải nén **nội dung ZIP** vào `C:\xampp\htdocs\minishop`, để `index.php` nằm ngay trong thư mục này.
2. Bật **Apache** và **MySQL**.
3. Mở `http://localhost/phpmyadmin/`:
   - Cài mới: nhập `database/minishop.sql` để có 6 sản phẩm mẫu.
   - Khôi phục bản đã xuất: chọn CSDL đích, nhập `database/minishop-export.sql` trong ZIP. Không cần nhập thêm seed.
   - Đang có Buổi 4: chỉ nhập `database/upgrade_buoi5.sql`.
   - Đang có Buổi 3: nhập `upgrade_buoi4.sql` rồi `upgrade_buoi5.sql`. Các migration giữ dữ liệu và có thể chạy lại.
4. Mặc định CSDL `minishop`, host `127.0.0.1`, port `3306`, user `root`, mật khẩu rỗng. Nếu khác, sao chép `config/local.example.php` thành **`config/local.php`** và sửa thông tin. Tệp này không đưa vào Git/ZIP để tránh ghi đè cấu hình riêng.
5. Mở **`http://localhost/minishop/`**. Dùng PHP **8.1+**, PDO MySQL; khuyến nghị bật extension Fileinfo cho kiểm tra ảnh.

### Vào khu vực quản trị

Không có tài khoản hay mật khẩu quản trị mặc định trong mã nguồn. Đăng ký một tài khoản trên website, rồi chạy trong thẻ SQL của phpMyAdmin (thay email của bạn):

```sql
UPDATE users SET role = 'admin' WHERE email = 'email-cua-ban@example.com';
```

Đăng xuất, đăng nhập lại. Chọn **Quản trị**, hoặc mở `/minishop/admin/index.php`.

Nếu dùng CLI, có thể chạy `php tools/promote_admin.php email-cua-ban@example.com`. Công cụ chỉ cấp quyền cho tài khoản đã đăng ký; không tạo mật khẩu cố định.

### Dùng máy chủ PHP tích hợp

Trong thư mục dự án:

```powershell
php -S 127.0.0.1:8000 -t . router.php
```

Mở `http://127.0.0.1:8000/`. MySQL phải đang chạy. **Cần router.php**, vì máy chủ PHP tích hợp không đọc `.htaccess`. Router chặn các thư mục riêng, ngăn chạy PHP trong uploads và trả trang 404 của MiniShop.

## Các phần đã hoàn thành

| Phần giáo trình | Thực hiện |
| --- | --- |
| Lab 5.1 — phân quyền, danh sách, xóa | `require_admin()`, menu theo role; danh sách 10 sản phẩm/trang; xóa bằng POST; chặn xóa sản phẩm có đơn |
| Lab 5.2 — thêm/sửa và tải ảnh | Form dùng chung; kiểm tra danh mục, tên, giá ≥ 1.000 đ, kho nguyên không âm; ảnh JPG/PNG/WebP ≤ 2 MB |
| Lab 5.3 — quản lý đơn | Xem mọi đơn, mở chi tiết, lọc trạng thái, cập nhật bằng POST; khách thấy trạng thái mới |
| Lab 5.4 — bảo mật | CSRF tập trung trong init; escape HTML; PDO; cookie HttpOnly/SameSite; chặn tệp riêng; bảo vệ uploads; trang 404 |
| Lab 5.5 — hosting | ZIP triển khai không kèm bài thử/SQL; cấu hình riêng; mẫu Apache/Nginx, HTTPS và hướng dẫn panel. Chưa triển khai công khai |

### Sáu bài tập tổng kết

1. **Danh mục:** `admin/categories.php` thêm/sửa/xóa; đường dẫn ngắn duy nhất; không xóa danh mục còn sản phẩm.
2. **Xóa ảnh cũ:** sau khi đổi ảnh và lưu CSDL thành công mới dọn ảnh cũ, chỉ trong `uploads/` và chỉ khi không còn sản phẩm tham chiếu. Nếu lưu CSDL thất bại, ảnh vừa tải lên được dọn, ảnh cũ còn nguyên.
3. **Phân trang trang chủ:** 8 sản phẩm/trang, giữ danh mục và từ khóa trên liên kết chuyển trang.
4. **Tổng quan:** đếm đơn theo trạng thái, doanh thu đơn hoàn tất, tổng sản phẩm và 5 sản phẩm có tồn kho thấp nhất.
5. **Đăng nhập sai:** 5 lần sai trong session thì khóa 5 phút; trả HTTP 429 và `Retry-After`; hết hạn có thể thử lại. Đây là giới hạn theo phiên của bài thực hành.
6. **Đổi mật khẩu:** xác nhận mật khẩu hiện tại, kiểm tra mật khẩu mới/nhập lại, băm lại, đổi session ID. `auth_version` khiến các phiên đăng nhập cũ phải đăng nhập lại.

## Cách xử lý quan trọng

- `includes/init.php` là điểm nạp chung. Các trang quản trị đều gọi `require_admin()` trước khi đọc/ghi dữ liệu quản trị. Ẩn menu không thay thế kiểm tra quyền.
- `csrf_token` nằm trong mọi form POST và được kiểm tra tập trung. Dữ liệu thay đổi qua POST/Redirect/Get; thiếu token trả 403.
- Tên ảnh do máy chủ sinh ngẫu nhiên; loại ảnh được đọc từ nội dung, không tin tên/MIME do trình duyệt gửi. Không nhận SVG/PHP. Ảnh mặc định trong `images/` không bị xóa.
- Giao dịch đặt/hủy đơn của Buổi 4 được giữ nguyên. Khi admin hủy đơn, kho được hoàn **một lần** trong transaction. Đơn đã hoàn tất/hủy không được mở lại làm sai kho. Các bước thông thường: chờ xử lý → đang giao → hoàn tất; có thể hủy đơn chưa kết thúc. Trạng thái `confirmed` cũ được giữ để đọc dữ liệu trước đây.
- Tài khoản thường chỉ xem đơn của mình; admin có thể xem mọi đơn. Đổi role/đổi mật khẩu khiến phiên cũ mất quyền ở request tiếp theo.
- Liên kết/ảnh/form dùng base URL chung nên chạy ở gốc tên miền và thư mục con. `BASE_URL` có thể là `/` hoặc `/minishop/`; mặc định tự nhận theo vị trí cài.
- `APP_DEBUG` mặc định false; lỗi được ghi log. Không đưa mật khẩu hoặc thông tin kết nối lên giao diện.

## Cấu trúc chính

```text
admin/        index.php, products.php, product-form.php, product-delete.php,
              categories.php, orders.php
config/       config.php, db.php, local.example.php, .htaccess
includes/     init, bootstrap, functions, auth, session, products, cart,
              orders, admin, uploads, header, footer
uploads/      Ảnh tải lên và .htaccess cấm thực thi PHP
database/     minishop.sql, migration Buổi 4/5, .htaccess
deployment/   Mẫu Nginx và HTTPS cho panel
tools/        promote_admin.php, package_buoi5.py
tests/        Buổi 4 và Buổi 5, dữ liệu kiểm thử được dọn sau khi chạy
docs/         Bảng đối chiếu giáo trình, hướng dẫn hosting, tài liệu buổi trước
router.php    Bảo vệ và định tuyến cho php -S tại localhost
404.php       Trang không tìm thấy của MiniShop
```

Apache/LiteSpeed dùng `.htaccess`. Nginx không đọc tệp này: phải ghép quy tắc ở `deployment/nginx.conf.example` vào site trước khi đưa lên mạng. Các đường dẫn `config/`, `includes/`, `database/`, `tests/`, `tools/`, `docs/`, `deployment/`, `labs/` bị chặn trên web nhưng PHP vẫn nạp tệp nội bộ được.

## Kiểm tra

Chạy với CSDL thực hành riêng, cấu hình `DB_*` của CLI phải trùng máy chủ web. Các script có ghi dữ liệu tạo tài khoản/sản phẩm có tiền tố ngẫu nhiên rồi dọn trong `finally`. Kiểm thử rollback cần quyền tạo trigger.

```powershell
Get-ChildItem -Recurse -Filter *.php | ForEach-Object { php -l $_.FullName }
php tests/smoke.php
python tests/http_smoke.py http://127.0.0.1:8000/
python tests/buoi4_flow.py http://127.0.0.1:8000/
python tests/buoi5_flow.py http://127.0.0.1:8000/
```

Đặt `PHP_BIN` nếu PHP không có trong PATH. Đã kiểm tra backend với PDO/MariaDB, gốc `/` và thư mục `/minishop/`, giao diện quản trị trên desktop/điện thoại. Quy tắc `.htaccess`/Nginx vẫn cần nghiệm thu trên máy chủ hosting thật.

## Nén và xuất bài

```powershell
python tools/package_buoi5.py --dump C:\xampp\mysql\bin\mysqldump.exe
```

Với cấu hình CSDL riêng, đặt `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` cùng giá trị ứng dụng, hoặc truyền `--sql DUONG_DAN_BAN_XUAT.sql` nếu đã xuất bằng phpMyAdmin.

Tệp trong thư mục `../output/`:

- **`minishop-buoi5.zip`**: trọn mã nguồn, hướng dẫn, bài kiểm thử và `database/minishop-export.sql`; `index.php` nằm ngay gốc ZIP.
- **`minishop-hosting.zip`**: chỉ tệp chạy website, không có SQL, bài thử, công cụ hay mật khẩu riêng; dùng để tải lên `public_html`.
- **`minishop-export.sql`**: bản xuất không gắn cứng tên CSDL, có thể nhập vào database có tiền tố do panel cấp.

Bản xuất chứa dữ liệu CSDL của bạn, giữ ở máy hoặc nhập bằng phpMyAdmin; không tải SQL/ZIP nộp bài vào thư mục web công khai. Xem chi tiết [Lab 5.5](docs/DEPLOYMENT.md) và [đối chiếu giáo trình](docs/BUOI5_DRIVE.md).

Tham khảo: [PHP upload](https://www.php.net/manual/en/features.file-upload.php), [Apache access configuration](https://httpd.apache.org/docs/2.4/sections.html).
