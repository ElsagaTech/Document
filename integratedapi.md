# Tích hợp trang đặt khám MeApp cho website, ứng dụng bên thứ 3

## 1. API

### 1.1. Request
> **POST:** https://gateway.meapp.vn/v1/appointment

| Giá trị | Kiểu | Yêu cầu | Mặc định | Mô tả |
| --- | --- | --- | --- | --- |
partnerCode | `string` | `Bắt buộc` | | Mã đối tác
partnerName | `string` | `Tuỳ chọn` | `""` | Tên hiển thị của đối tác
requestId | `string` | `Bắt buộc` | | Mã duy nhất sinh ra từ đối tác, nên sử dụng UUID v4
amount | `long` | `Tuỳ chọn` | `0` | Giá tiền gói thanh toán
orderId | `string` | `Bắt buộc` | | Id thanh toán của đối tác
orderInfo | `string` | `Bắt buộc` | | Thông tin thanh toán
redirectUrl | `string` | `Bắt buộc` | | Url nhận callback của đối tác, dùng để nhận kết quả đặt khám thành công
extraData | `string` | `Tuỳ chọn` | `""` | Encode **Base64** theo định dạng Json: `{"key": "value"}` <br>Ví dụ với dữ liệu: `{"username": "meapp"}` thì data extraData: `eyJ1c2VybmFtZSI6ICJtZWFwcCJ9`
signature | `string` | `Bắt buộc` | | Chữ ký để xác nhận bảo toàn dữ liệu. <br>Sử dụng thuật toán **Hmac_SHA256** với data phía trên theo định dạng được sort từ a-z:<br> *accessKey=`$accessKey`&amount=`$amount`&extraData=`$extraData`&<br>orderId=`$orderId`&orderInfo=`$orderInfo`&partnerCode=`$partnerCode`&<br>partnerName=`$partnerName`&redirectUrl=`$redirectUrl`&requestId=`$requestId`*

Trong đó: 
* `partnerCode` và `accessKey` được gửi riêng cho đối tác
* Dữ liệu `extraData` dùng để hiển thị thông tin người dùng khi mở trang đặt khám, theo định dạng

    ```json
    { 
        "phoneNumber": "123", 
        "fullName": "ABC", 
        "birthDay": "11/11/2021", 
        "gender": 1 
    }
    ``` 
* `birthDay`: dd/MM/yyyyy
* `gender`: `1` - Nam, `2` - Nữ

### Ví dụ
```json
{
    "partnerCode": "MEAPP",
    "partnerName": "Test",
    "requestId": "",
    "amount": 10000,
    "orderId": "",
    "orderInfo": "Test",
    "redirectUrl": "http://localhost:4006",
    "extraData": "eyAicGhvbmVOdW1iZXIiOiAiMTIzIiwgImZ1bGxOYW1lIjogIkFCQyIsICJiaXJ0aERheSI6ICIyNC8xMS8yMDIxIiwgImdlbmRlciI6IDEgfQ",
    "signature": "2b3cdad4295dcc5cc92edf36149d6dc1192e48aa4762ed1f97af40906763571e"
}
```

### 1.2. Response

| Giá trị | Kiểu | Mô tả |
| --- | --- | --- |
partnerCode | `string` | Mã đối tác
requestId | `string` | Mã duy nhất sinh ra từ đối tác, nên sử dụng UUID v4
amount | `long` | Giá tiền gói thanh toán
orderId | `string` | Id thanh toán của đối tác
appointmentUrl | `string` | Url trang đặt khám, đối tác có thể mở webview hoặc mở trình duyệt ngoài

### Ví dụ
```json
{
    "partnerCode": "MEAPP",
    "requestId": "",
    "amount": 10000,
    "orderId": "",
    "appointmentUrl": "https://datkham.meapp.vn?sessionId=xxxxxx"
}
```

## 2. Sau khi đặt khám thành công

Call `GET` vào redirectUrl bên đối tác gửi lúc đăng ký đặt khám theo định dạng

*`redirectUrl`?resultCode=`0`&requestId=`$requestId`*
