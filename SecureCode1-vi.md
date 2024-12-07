# LAB 8: SECURECODE1

## **Máy ảo**

- [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines)

- [SecureCode](https://www.vulnhub.com/entry/securecode-1,651/)
   - Network Adapter: NAT cho cả 2 máy

---

## **Scan**
- **IP máy Kali Linux**: `192.168.208.180/24`.
![image](https://github.com/user-attachments/assets/96295593-fc34-4229-89e3-bb320fdd3370)

- **IP mục tiêu**: `192.168.208.188/24`. Có cổng **80** mở.
![image](https://github.com/user-attachments/assets/3d118320-b152-400c-9523-9c8bc41fc142)

---

## **Phân tích trang web**
Trên trang này không có nhiều thông tin lắm, chỉ hiển thị website đang được xây dựng.
![image](https://github.com/user-attachments/assets/8b3e39b9-8c14-453a-8b2e-8c4755df4521)

1. **Mã nguồn trang web**:
   - Tất cả các file CSS và JS được tải về từ máy mục tiêu, không phải thư viện ngoài.
   - Có một form bị vô hiệu hóa trong mã nguồn trang chủ trang web.
  ![image](https://github.com/user-attachments/assets/fd34b522-9a03-40e5-a6cd-7b0335fc5168)


2. **Lưu lượng mạng**:
   - Một số tài nguyên không tải được.
  ![image](https://github.com/user-attachments/assets/f6eb8ab9-9515-4c8c-9e17-c0b63df0ed1b)

3. **Các trang quan trọng**: 
Dùng Dirb scan trang web này thu được một số kết quả:
![image](https://github.com/user-attachments/assets/2538872b-b359-47c0-845a-58c6de5213ee)

   - Truy cập `/item`, `/profile`, `/user`: Redirect sang `/login` với thông báo chưa có quyền.
  ![image](https://github.com/user-attachments/assets/85742b63-dcde-4a9f-87f6-8f1ed11f0223)

   - `/resetPassword.php`: Có thể dùng để kiểm tra tên tài khoản hợp lệ hay không, thử ngẫu nhiên thì tìm được 1 username admin.
  ![image](https://github.com/user-attachments/assets/b5925c3f-228c-4409-a20a-e0b0a031068e)
  ![image](https://github.com/user-attachments/assets/c5b117e3-5ab4-4bfa-8533-63bdf2759352)
  ![image](https://github.com/user-attachments/assets/b2cd8176-5163-425c-ad7a-82c4860023a2)

---

## **Thu thập thông tin mã nguồn**
- Dùng công cụ gobuster, tìm được file `source_code.zip`, tải xuống và giải nén file này.
![image](https://github.com/user-attachments/assets/618cafbe-6266-4e88-bef2-924a44704a8c)

- **Phân tích mã nguồn**:
  - `connection.php`: Chứa thông tin cơ sở dữ liệu (DB).
  ![image](https://github.com/user-attachments/assets/098b054a-6cf7-4d68-8aaf-02ce643e53f5)

  - `db.sql`: Tiết lộ cấu trúc các bảng và các dữ liệu trong đó.
  ![image](https://github.com/user-attachments/assets/242f6ba0-efe5-437e-bcf0-18107330124f)
     - Mô hình các bảng
     ![image](https://github.com/user-attachments/assets/3a6c51b5-e7ac-48ac-85d8-84b3890ff4bc)
      
  - `/users/store.php`: Sử dụng MD5 để băm mật khẩu. 
  ![image](https://github.com/user-attachments/assets/2bd42fe8-670d-40b9-9a27-d95b097a34d6)
      - Thử decrypt mật khẩu thu được từ db.sql nhưng thất bại.

  - `/login/resetPassword.php`:
    - Tạo token 15 ký tự ngẫu nhiên bằng `rand()` (dễ bị crack).
    ![image](https://github.com/user-attachments/assets/7d2a49f4-7d19-492b-baa6-8ec673c4c973)

    - Cơ chế kiểm tra token bằng `in_array()` và `ctype_alnum()`.
    ![image](https://github.com/user-attachments/assets/386bed10-4d28-4791-b469-ed3ea84e5810)

---

## **Lỗ hổng**
### **1. Lỗ hổng SQL Injection tại `/item/viewItem.php`**
- **Phân tích mã nguồn**:
  - Các trang trong website này đều sử dụng `mysql_real_escape_string` để lọc đầu vào, hàm này chỉ lọc một số ký tự đặc biệt và không đủ an toàn.
  ![image](https://github.com/user-attachments/assets/56a1ebf4-a2f4-430a-9ea7-adeee8523f11)

  - Điểm yếu: nếu id được truyền vào trang viewItem không chứa ký tự đặc biệt thì sẽ không bị `mysql_real_escape_string` escape.
  ![image](https://github.com/user-attachments/assets/a172b1ca-d861-404a-91df-3e2548319ba8)

  - Nếu truy vấn trả về kết quả -> Status 404, ngược lại -> Không trả về gì. =_=
  ![image](https://github.com/user-attachments/assets/59ca7128-1db0-4482-aafc-be4b1679bf50)


- **Thử nghiệm SQLi**:
  - Nhập ID đúng (dữ liệu từ `db.sql`): Trả về **404**.
  ![image](https://github.com/user-attachments/assets/cbbfd5ee-6dab-46da-9592-572c3187b8db)

  - Nhập ID sai: Trả về **302**.
  ![image](https://github.com/user-attachments/assets/df1fc59a-8103-43b8-b1bc-613138cc2761)

  - Thử `sleep(15)`: Hoạt động -> Lỗ hổng SQL Injection tồn tại.
  ![image](https://github.com/user-attachments/assets/4bd965ad-b808-4e78-a24a-23a9cb087f44)


- **Phương hướng khai thác**:
  - Thực hiện SQL Injection blind-based để lấy token reset password.
  - Script sử dụng tìm kiếm nhị phân tìm từng ký tự trong token.
  ![image](https://github.com/user-attachments/assets/d1e51251-961e-4da7-8bf0-17d411efa93a)

### **2. Lỗ hổng Upload File tại `/item/updateItem.php`**
- **Mã nguồn**:
  - Chỉ kiểm tra extension file, không kiểm tra nội dung.
  ![image](https://github.com/user-attachments/assets/77c82034-7fec-475a-86e9-65ac7cc695c5)


- **Khai thác**:
  - Tạo reverse shell với đuôi `.phar`.
  - Tải lên qua trang `editItem`.
  - Reload trang để kích hoạt.

---

## **Kết quả**
1. Lấy được token để đổi mật khẩu admin.
![image](https://github.com/user-attachments/assets/0d930e4d-d313-4c59-a9ba-87c629e23141)

2. Tải lên reverse shell và truy cập máy chủ mục tiêu.
![image](https://github.com/user-attachments/assets/cef1a6e2-9199-4917-a8da-1a8c31fef1ba)
![image](https://github.com/user-attachments/assets/b3a70231-ccfa-406f-a0d6-107e472810b8)

---

## **Tài liệu tham khảo**
- [OSWE Methodology](https://github.com/R-s0n/OSWE-Methodology/blob/main/methodology.txt)
- [PHP mysql_real_escape_string](https://www.php.net/manual/en/function.mysql-real-escape-string.php)
- [PHP rand](https://www.php.net/manual/en/function.rand.php)
