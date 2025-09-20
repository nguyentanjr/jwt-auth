🔐 Authentication & Authorization Flow
1️⃣ Login Flow

Client gửi login request (username/password) đến Controller.

Controller gọi AuthenticationManager để xác thực thông tin.

Xử lý kết quả:

❌ Nếu không hợp lệ → throw exception, thông báo login thất bại.

✅ Nếu hợp lệ:

Tạo Access Token (AT) và Refresh Token (RT).

Lưu RT vào cookie và AT vào local storage của client.

Lưu Authentication vào SecurityContextHolder để backend biết user đang đăng nhập.

Trả về response → client chuyển hướng đến Home.

2️⃣ Register Flow

Client gửi register request đến Controller.

Controller kiểm tra user đã tồn tại:

❌ Nếu tồn tại → throw exception, thông báo không thể đăng ký.

✅ Nếu hợp lệ:

Lưu user data vào DB.

Lưu Authentication vào SecurityContextHolder.

Tạo AT và RT, lưu vào local storage + cookie.

Client được chuyển hướng đến Home.

3️⃣ Refresh Token Flow

Client gửi refresh token request.

Server lấy RT từ cookie và kiểm tra:

❌ Nếu không hợp lệ → throw exception.

✅ Nếu hợp lệ → thực hiện rotation:

Revoke RT cũ (đánh dấu là used để chống replay attack).

Tạo AT mới và RT mới, lưu vào cookie/local storage.

Trả response → client tiếp tục sử dụng AT mới.

4️⃣ Logout Flow

Client gửi logout request.

Server lấy RT từ cookie và thực hiện:

Xóa RT trong cookie.

Revoke RT trong DB → token không còn hợp lệ.

Xóa AT trong local storage.

Kết quả: user hoàn toàn bị logout, không thể sử dụng token cũ.

5️⃣ Refresh Token Rotation

Khi refresh token được dùng:

Revoke RT cũ (invalidate trong DB).

Grant RT mới → gửi về client.

Mục tiêu: đảm bảo mỗi RT chỉ dùng một lần → tăng cường bảo mật.
