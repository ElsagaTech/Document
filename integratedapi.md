## Tích hợp trang đặt khám MeApp cho website, ứng dụng bên thứ 3

> **POST:** /v1/appointment

| Attribute | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
partnerCode | `string` | `Required` | | Mã đối tác
partnerName | `string` | `Optional` | `""` | Tên hiển thị của đối tác
requestId | `string` | `Required` | | Mã duy nhất sinh ra từ đối tác, nên sử dụng UUID v4
amount | `long` | `Optional` | `0` | Giá tiền gói thanh toán
orderId | `string` | `Required` | | Id thanh toán của đối tác
orderInfo | `string` | `Required` | | Thông tin thanh toán
redirectUrl | `string` | `Required` | | Url nhận callback của đối tác, dùng để nhận kết quả đặt khám thành công
extraData | `string` | `Optional` | `""` | Encode Base64 theo định dạng Json: `{"key": "value"}` Ví dụ với dữ liệu: `{"username": "meapp"}` thì data extraData: `eyJ1c2VybmFtZSI6ICJtZWFwcCJ9`
signature | `string` | `Required` | | Chữ ký để xác nhận bảo toàn dữ liệu. Sử dụng thuật toán Hmac_SHA256 với data phía trên theo định dạng được sort từ a-z: *accessKey=`$accessKey`&amount=`$amount`&extraData=`$extraData`&orderId=`$orderId`&orderInfo=`$orderInfo`&partnerCode=`$partnerCode`&partnerName=`$partnerName`&redirectUrl=`$redirectUrl`&requestId=`$requestId`*

> accessKey tạm thời fixed gủi cho bên đối tác

> Dữ liệu extraData dùng để hiển thị thông tin người dùng khi mở trang đặt khám, theo format 
```json
{ 
    "phoneNumber": "123", 
    "fullName": "ABC", 
    "birthDay": "11/11/2021", 
    "gender": 1 
}
``` 
> Chú ý trường gender thì `1` là nam, `2` là nữ

**Sample**
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

## Response

| Attribute | Type | Description |
| --- | --- | --- |
partnerCode | `string` | Mã đối tác
requestId | `string` | Mã duy nhất sinh ra từ đối tác, nên sử dụng UUID v4
amount | `long` | Giá tiền gói thanh toán
orderId | `string` | Id thanh toán của đối tác
appointmentUrl | `string` | Url trang đặt khám, đối tác có thể mở webview hoặc mở trình duyệt ngoài

**Sample**
```json
{
    "partnerCode": "MEAPP",
    "requestId": "",
    "amount": 10000,
    "orderId": "",
    "appointmentUrl": "http://datkham.meapp.vn?sessionId=123456"
}
```

## Sau khi đặt khám thành công

Call `GET` vào redirectUrl bên đối tác gửi lúc đăng ký đặt khám theo format

*`$redirectUrl`?resultCode=`0`&requestId=`$requestId`*
