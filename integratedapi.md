# Tích hợp trang đặt khám MeApp cho website, ứng dụng bên thứ 3

## 1. Web đặt khám

> Liên kết: https://datkham.meapp.vn?`QUERY_STRING`

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
partnerCode | `string` | `Tuỳ chọn` | | Mã đối tác
extraData | `string` | `Tuỳ chọn` | `""` | Encode **Base64** theo định dạng Json: `{"key": "value"}` <br>Ví dụ với dữ liệu: `{"username": "meapp"}` thì data extraData: `eyJ1c2VybmFtZSI6ICJtZWFwcCJ9`

* Mã đối tác dùng để xác định đối tác mở webview, chỉ hiển thị phương thức thanh toán online duy nhất của đối tác
* Dữ liệu `extraData` dùng để hiển thị thông tin người dùng khi mở trang đặt khám, theo định dạng

    ```json
    { 
        "phoneNumber": "123", 
        "fullName": "ABC", 
        "birthDay": "20/11/2021", 
        "gender": 1 
    }
    ``` 
    - `birthDay`: dd/MM/yyyy
    - `gender`: `1` - Nam, `2` - Nữ

### Ví dụ mẫu
```
https://datkham.meapp.vn?partnerCode=appota&extraData=eyAicGhvbmVOdW1iZXIiOiAiMTIzIiwgImZ1bGxOYW1lIjogIkFCQyIsICJiaXJ0aERheSI6ICIyNC8xMS8yMDIxIiwgImdlbmRlciI6IDEgfQ
```

<!-- ## 2. Sau khi đặt khám thành công
Call vào API thanh toán của đối tác nếu có phí dịch vụ -->

## 2. API kiểm tra đơn hàng Appota

### 2.1. Request

> Liên kết: https://gateway.meapp.vn/v1/appota/check_order

> Phương thức: POST

> Dữ liệu: Json

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
order_id | `string` | `Bắt buộc` | | Mã giao dịch
amount | `int` | `Bắt buộc` | `""` | Số tiền giao dịch
check_sum | `string` | `Bắt buộc` | | Chữ ký

Trong đó: 
* Định dạng chữ ký `check_sum`

    ```csharp
    SHA256($"{amount}.{order_id}.{SECRET_KEY}")
    ``` 
    - `SECRET_KEY`: được gửi riêng cho đối tác

### Ví dụ mẫu
```json
{
    "amount": 10000,
    "orderId": "123",
    "check_sum": "87013a41259c976cc1a1beb8218a5b177849ab1276560b2eabbbe32c60fd1aec"
    // Với secret ví dụ là MEAPP, format raw có dạng 10000.123.MEAPP
}
```

### 2.2. Response

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
status | `bool` | `Bắt buộc` | | Trạng thái
error_message | `string` | `Bắt buộc` | `""` | Thông báo
order_info | `object` | `Tùy chọn` | | Thông tin order nếu status là `true`
### Ví dụ
```json
{
  "status": true,
  "error_message": "Thành công",
  "order_info": {
    "id": "123",
    "name": "Tên dịch vụ hiển thị"
  }
}
```
## 3. API cập nhật thanh toán đơn hàng Appota

### 3.1. Request

> Liên kết: https://gateway.meapp.vn/v1/appota/update_order

> Phương thức: POST

> Dữ liệu: Json

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
order_id | `string` | `Bắt buộc` | | Mã giao dịch
amount | `int` | `Bắt buộc` | `""` | Số tiền giao dịch
check_sum | `string` | `Bắt buộc` | | Chữ ký
appotapay_transaction_id | `string` | `Bắt buộc` | | Mã giao dịch của Appota

Trong đó: 
* Định dạng chữ ký `check_sum`

    ```csharp
    SHA256($"{amount}.{order_id}.{appotapay_transaction_id}.{SECRET_KEY}")
    ``` 
    - `SECRET_KEY`: được gửi riêng cho đối tác

### Ví dụ mẫu
```json
{
    "amount": 10000,
    "orderId": "123",
    "appotapay_transaction_id": "TRANS_ABC",
    "check_sum": "a871d9e31e10e52840e5f21123ab764bba2a66ad97e32aa7c2499454cdb691e6"
    // Với secret ví dụ là MEAPP, format raw có dạng 10000.123.TRANS_ABC.MEAPP
}
```

### 3.2. Response

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
status | `bool` | `Bắt buộc` | | Trạng thái
error_message | `string` | `Bắt buộc` | `""` | Thông báo
### Ví dụ
```json
{
  "status": true,
  "error_message": "Thành công"
}
```
## 4. API hoàn tiền

### 4.1. Request

> Liên kết: https://gateway.meapp.vn/v1/appota/refund

> Phương thức: POST

> Dữ liệu: Json

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
order_id | `string` | `Bắt buộc` | | Mã giao dịch
check_sum | `string` | `Bắt buộc` | | Chữ ký
appotapay_transaction_id | `string` | `Bắt buộc` | | Mã giao dịch của Appota

Trong đó: 
* Định dạng chữ ký `check_sum`

    ```csharp
    SHA256($"{order_id}.{appotapay_transaction_id}.{SECRET_KEY}")
    ``` 
    - `SECRET_KEY`: được gửi riêng cho đối tác

### Ví dụ mẫu
```json
{
    "orderId": "123",
    "appotapay_transaction_id": "TRANS_ABC",
    "check_sum": "5e988777abcd36d484de3a2b1dc22b83a207f737afc61100ee78c4356a3734d2"
    // Với secret ví dụ là MEAPP, format raw có dạng 123.TRANS_ABC.MEAPP
}
```

### 3.2. Response

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| :--- | :--- | :--- | :--- | :--- |
status | `bool` | `Bắt buộc` | | Trạng thái
error_message | `string` | `Bắt buộc` | `""` | Thông báo
### Ví dụ
```json
{
  "status": true,
  "error_message": "Thành công"
}
```