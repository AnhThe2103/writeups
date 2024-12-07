# LAB 7: NULLY CYBERSECURITY

## **Máy ảo**

- [Kali Linux](https://www.kali.org/get-kali/#kali-virtual-machines)

- [Nully](https://www.vulnhub.com/entry/nully-cybersecurity-1,549/)
   - Network Adapter: NAT cho cả 2 máy

---

## **Scan**
- **IP máy Kali Linux**: `192.168.208.180/24`.
![image](https://github.com/user-attachments/assets/a3fe549b-eb0b-4fc7-926b-c6d48840fc61)

- **IP mục tiêu**: `192.168.208.186/24`.
![image](https://github.com/user-attachments/assets/62869b39-199c-46ad-819b-4a9cf70b561c)

---

## **Mục tiêu**

- Truy cập web trên công 80 thu được thông tin về máy ảo này:
  - Mục tiêu của challenge này là chiếm quyền root trên 3 server ảo.
  - Không được tấn công port 80 và port 8000, 9000 (chạy dịch vụ quản lý máy ảo Docker).
  - Story...
- Gợi ý:
  - Bắt đầu với việc kiểm tra email với tài khoản được cho.
![image](https://github.com/user-attachments/assets/fc850ccb-31ec-4eca-897d-4720c0c8d315)

---

## **Mail server**
Truy cập đến cổng 110 bằng netcat với tài khoản được cấp.
![image](https://github.com/user-attachments/assets/c2af4455-3b7d-4e1d-8b01-ed8c79003ed8) 
Có 1 email từ Bob Smith với yêu cầu giúp đỡ truy cập lại mail server do quên password.
![image](https://github.com/user-attachments/assets/e10d324b-4c20-4dab-9804-5e283da6813d)
Dựa vào một gợi ý khác của tác giả máy ảo trên VulnHub thì có thể tạo wordlist với từ khóa bobby.
![image](https://github.com/user-attachments/assets/9767d16a-7154-4410-987a-bb668d37b81b)
![image](https://github.com/user-attachments/assets/fd24b18f-8a1a-4548-9da9-b8f248940c09)
Do Bob không đề cập đến tên tài khoản nên ta cũng phải tạo một wordlist username.
![image](https://github.com/user-attachments/assets/943d6505-b939-479e-af17-dea6942b8525)
Tiến hành tấn công 
![image](https://github.com/user-attachments/assets/464ed854-0227-4870-94ab-f47f571ab687)
Thu được thông tin đăng nhập bob:bobby1985
![image](https://github.com/user-attachments/assets/cc3a270f-754d-4980-8187-fabc9dd850bf)
Thông tin đăng nhập này có thể dùng để đăng nhập vào dịch vụ mail và ssh vào mail server.
![image](https://github.com/user-attachments/assets/ba87dbe9-5ec2-4aec-bb38-4841f3b72feb)
![image](https://github.com/user-attachments/assets/f0f5ce68-9fb6-460e-bd36-43074853537c)
Trong home của Bob có 1 list các task cần thực hiện:
  - Cài đặt postfix và dovecot. (2 phần mềm open source liên quan tới mail)
  - Gửi mail tới pentester về cái server.
  - Tạo một script để kiểm tra thông tin server.
  - Tạo một người dùng tên my2user để backup thông tin quan trọng. 
![image](https://github.com/user-attachments/assets/4c0ef27f-cc8e-4390-83b7-028ab312e9b7)
Dựa vào thông tin tài khoản đọc từ `/etc/passwd` thì có vẻ các task trong list trên đã được thực hiện phần nào.
![image](https://github.com/user-attachments/assets/bac559bf-6edc-44f2-9c17-7ebfc2c8fab6)
Tìm thử thì không thấy thông tin backup nào trên mail server, nhưng tìm được một file script tên `check.sh`.
![image](https://github.com/user-attachments/assets/2d74233c-872d-4f79-ba87-473425366308)
![image](https://github.com/user-attachments/assets/7d0a8c35-cd24-40e8-a120-b0a5f8fe9a92)
Bob chỉ có thể đọc ghi và không thể trực tiếp chạy được `check.sh` nhưng có thể chạy dưới quyền my2user.
![image](https://github.com/user-attachments/assets/d0068b7b-85af-4e51-916e-9e0b69ad31be)
`check.sh` thực thi Bash script, in ra thông tin
![image](https://github.com/user-attachments/assets/a3624916-7be8-40a6-bf42-7a7eff1716f5)
Sửa lại file này để mở shell với tư cách my2user
![image](https://github.com/user-attachments/assets/3d9233d7-a269-4232-b777-3d5abfc7ffe8)
![image](https://github.com/user-attachments/assets/66fdd498-703a-46d3-8125-1605f9ccea2d)
my2user có thể chạy zip với quyền sudo mà không cần mật khẩu, thông qua đó có thể leo thang đặc quyền lên root.
![image](https://github.com/user-attachments/assets/3df8324c-3d96-4fc0-ad6b-93f9aeffc429)

---

## **DatabaseServer**
Từ MailServer, dò quét các máy trong mạng với nmap có sẵn.
![image](https://github.com/user-attachments/assets/2156ca24-d148-4102-993b-a6a8ee6f8df3)
![image](https://github.com/user-attachments/assets/492c253b-48bb-4eba-a0de-3e1b0889b5be)
Có 1 máy mở 2 cổng 8000 và 9000 là server quản lý máy ảo, không phải mục tiêu tấn công. Một máy mở cổng 80 có vẻ là WebServer, máy còn lại mở cổng 21 có vẻ là DatabaseServer.
![image](https://github.com/user-attachments/assets/cb889cc1-cf28-42d4-8239-02d8215aba12)
Thu thập thông tin trên cổng 21 ftp, có cho phép đăng nhập anonymous và có 1 folder pub. (ip thay đổi do reset máy ảo Nully)
![image](https://github.com/user-attachments/assets/dacd6c11-60d7-4501-bfd3-5f6e3e1c623e)
Kết nối tới dịch vụ ftp này và tải xuống toàn bộ folder pub về MailServer
![image](https://github.com/user-attachments/assets/e8d77f0d-a7a7-49a8-971d-1eb05a94eff9)
![image](https://github.com/user-attachments/assets/3f2472ed-2d3c-4679-ba1e-a256827804fe)
Trong pub có 1 file `.backup.zip` và `file.txt`, `file.txt` không có thông tin gì còn `.backup.zip` thì yêu cầu mật khẩu để giải nén file `creds.txt` (có vẻ chứa credentials) 
![image](https://github.com/user-attachments/assets/4a03ddff-bac9-4744-8774-fde078835ea7)
![image](https://github.com/user-attachments/assets/943ca431-cad9-404a-9dc0-fe0ec7e8cd93)
![image](https://github.com/user-attachments/assets/b9064320-548c-4c03-8e53-45e476867cc9)
Đưa file `.backup.zip` này về máy Kali để tiện phá mật khẩu với fcrackzip.
![image](https://github.com/user-attachments/assets/293cd247-7885-4802-8899-6d6171369171)
Thu được mật khẩu 1234567890
![image](https://github.com/user-attachments/assets/81b9fd37-4ec6-4bef-88e8-47b5b9c9ad49)
Đọc `creds.txt` thì thu được 1 thông tin đăng nhập.
![image](https://github.com/user-attachments/assets/b1108c12-6e0f-4a91-aa09-1e9646f32afd)
![image](https://github.com/user-attachments/assets/27304a17-1bd3-4560-a563-aa0b2a40156c)
Dùng thông tin đăng nhập vừa thu được đăng nhập ssh vào DatabaseServer
![image](https://github.com/user-attachments/assets/f9722a87-f638-4bc4-8adf-f627cb43a42a)
![image](https://github.com/user-attachments/assets/a437b6e8-b276-4409-b4e6-67580cc202d5)
Trong số các file SUID trên DatabaseServer thì có Screen 4.5.0 có lỗ hổng có thể khai thác để leo thang đặc quyền. [EDI-ID: 41154](https://www.exploit-db.com/exploits/41154)
![image](https://github.com/user-attachments/assets/698b9fd4-c086-4f64-8eeb-1549312e056c)
Sử dụng script khai thác chương trình này, thành công thu được quyền root trên DatabaseServer.
![image](https://github.com/user-attachments/assets/135eb5b7-2263-4370-bb51-a367871bc9dd)

---

## **WebServer**
Từ MailServer truy cập website trên Webserver thấy có thông báo đang trong quá trình phát triển, và xuất hiện thêm một nhân vật tên Oliver.
![image](https://github.com/user-attachments/assets/19b16f65-53dc-401f-8fb5-2caea5901d3f)
Dùng dirb (không có sẵn trên MailServer) scan thu được thông tin về 1 file `robots.txt` và một folder `/ping`
![image](https://github.com/user-attachments/assets/9251ebcb-8caf-422e-aac7-0ee3837c1331)
`robots.txr` chứa thông tin về `ping`
![image](https://github.com/user-attachments/assets/e53f7d51-16f8-47d9-a4a3-49a57afc6b09)
Trong `/ping` có `ping.php` và `For-Oscar.txt`
![image](https://github.com/user-attachments/assets/423aad7e-1ff1-4c9d-aeea-f6af816ba26c)
Bức thư cho Oscar từ Oliver thông báo rằng anh ta để file ping ở đây để tiện ping sang server khác.
![image](https://github.com/user-attachments/assets/87d45daa-de13-4262-b2d5-1c61871aa47c)
`ping.php` yêu cầu 1 tham số host 
![image](https://github.com/user-attachments/assets/906ade9a-f946-4482-9f16-02773eaf802a)
Thử truyền vào ip của MailServer thu được kết quả ping.
![image](https://github.com/user-attachments/assets/e6c348c5-2b9c-4817-b736-b67c4397d951)
Có vẻ `ping.php` thực hiện lệnh ping và trả về kết quả, thử chèn lệnh khác vào xác nhận.
![image](https://github.com/user-attachments/assets/08debf6a-b16b-4a9d-b0ba-aa21b922b631)
Tiến hành upload php reverse shell lên WebServer, ip của file reverse shell được chỉnh để kết nối tới cổng 6666 của MailServer.
![image](https://github.com/user-attachments/assets/1feec882-6efa-4416-84b3-48bb81bba33f)
Trên MailServer có sẵn python3, host 1 http server để đưa file này sang WebServer
![image](https://github.com/user-attachments/assets/a23bb25b-4153-467f-a4c7-6c238f236e70)
Thực hiện tải reverse shell xuống WebServer thông qua `ping.php`
![image](https://github.com/user-attachments/assets/81837526-48e4-489f-84a0-d9e3984c6bc1)
Trên MailServer, thực hiện lắng nghe cổng 6666 với netcat (không có sẵn, nhưng có thể dễ dàng cài)
![image](https://github.com/user-attachments/assets/2f75c12b-d418-449f-a221-32ae5582abcd)
Kích hoạt reverse shell thoogn qua `ping.php`, thu được shell trên WebServer
![image](https://github.com/user-attachments/assets/ae908d64-3147-425b-93ee-9d75eeb4c543)
![image](https://github.com/user-attachments/assets/d5866231-3a6d-4dd8-88fd-aec36047830d)
Có 2 user là Oliver và Oscar
![image](https://github.com/user-attachments/assets/deed8996-010b-4433-8a55-90a78599c054)
Trong số các file SUID thì có python3, kích hoạt shell với python3 thu được EUID của Oscar
![image](https://github.com/user-attachments/assets/11784264-a945-4540-a8a3-8316950a9be0)
![image](https://github.com/user-attachments/assets/afb7f75a-5f56-427c-80d8-d20d1856f02a)
User Oscar để password tài khoản của mình ở trong home, đọc thu được password của user này.
![image](https://github.com/user-attachments/assets/f6abcaee-04b0-45cf-a857-efabf4e11c9c)
![image](https://github.com/user-attachments/assets/2cfedfa9-44e7-4c10-bfa7-8ba40a37de12)
Ngoài ra trong home của Oscar còn có folder `script`, bên trong có `current-date` với SUID root, `current-date` có chức năng hiển thị thời gian hiện tại. 
![image](https://github.com/user-attachments/assets/9a41b5a9-aaaf-4123-b2cb-1a4a714417ec)
![image](https://github.com/user-attachments/assets/df70003f-7558-4b07-8865-76a32344d08a)
Phân tích nhanh thì nhận thấy có vẻ `current-date` thực hiện lệnh date để lấy thời gian.
![image](https://github.com/user-attachments/assets/09749cc8-5a87-4158-8d36-e5cc5c186bf3)
![image](https://github.com/user-attachments/assets/c263a787-95f1-4f73-96bc-1692d6a10414)
Dùng kỹ thuật chèn PATH độc hại để hoán đổi hàm date được gọi sang /bin/sh, thu được quyền root
![image](https://github.com/user-attachments/assets/b02d8cd7-b624-454b-9a7c-f4bb2f8d0bfe)

---

## **Tài liệu tham khảo**
- [Relative path calls](https://www.thehacker.recipes/infra/privilege-escalation/unix/suid-sgid-binaries#relative-path-calls)
- [GNU Screen 4.5.0 - Local Privilege Escalation](https://www.exploit-db.com/exploits/41154)
- [ZIP Privilege Escalation](https://www.hackingarticles.in/linux-for-pentester-zip-privilege-escalation/)
