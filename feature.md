# Tổng quan về hệ thống Thẻ lệnh


<!-- @import "[TOC]" {cmd="toc" depthFrom=1 depthTo=6 orderedList=false} -->

<!-- code_chunk_output -->

- [Tổng quan về hệ thống Thẻ lệnh](#tổng-quan-về-hệ-thống-thẻ-lệnh)
    - [Hiển thị khi quét QRCode trên thẻ](#hiển-thị-khi-quét-qrcode-trên-thẻ)
      - [Thông tin thẻ](#thông-tin-thẻ)
      - [Danh sách cửa hàng trong Phường/ Xã, được bán:](#danh-sách-cửa-hàng-trong-phường-xã-được-bán)
      - [Lịch sử tới chợ siêu thị:](#lịch-sử-tới-chợ-siêu-thị)
    - [MobileApp](#mobileapp)
      - [Đăng nhập](#đăng-nhập)
      - [Kiểm tra thẻ tại siêu thị, chợ](#kiểm-tra-thẻ-tại-siêu-thị-chợ)
      - [Kiểm tra thẻ khi di chuyển trong Phường](#kiểm-tra-thẻ-khi-di-chuyển-trong-phường)
      - [Danh sách Thẻ đã kiểm tra](#danh-sách-thẻ-đã-kiểm-tra)

<!-- /code_chunk_output -->


### Hiển thị khi quét QRCode trên thẻ
> Thẻ gia đình https://tl.qrcare.vn/id/{10_so}/p/{8_so}
> Thẻ đi làm: https://tl.qrcare.vn/lv/{11_so}/p/{8_so}
> Thẻ tài xế: https://tl.qrcare.vn/tx/{11_so}/p/{8_so}


#### Thông tin thẻ 
- ID (10 số)
- Phường / xã đang sống
- địa chỉ nhà:  nếu có hoặc "Chưa cập nhật"
- CMND/ CCCD: nếu có hoặc "Xem trên thẻ"

Trường hợp status : INACTIVE: Hiện thông tin: thẻ này bị khoá

Backend

- `/api/qrpackage/getInfo` 
- input:
```
id: 10_so
p: 8_so
```

#### Danh sách cửa hàng trong Phường/ Xã, được bán: 
được cán bộ Phường cập nhật danh sách từ file excel lên hệ thống: 

- tên cửa hàng (orgTitle)
- địa điểm (orgLocation)
- số điện thoại đặt hàng tại nhà (orgPhone)

Backend
- `/api/qrpackage/{id_5so}/dscuahang`

- output
```json
[ {
  }
]
```
#### Lịch sử tới chợ siêu thị: 

được cập nhật mỗi ngày do siêu thị, chợ gửi file excel lên hệ thống

Check-in History 

- vào ngày, lúc `dd/mm/yyyy hh:mm`
- địa điểm : `orgLocation`  || `scannerLocation` 
- tên cửa hàng `orgTitle` hoặc tên điểm checkpoint yêu cầu quét thẻ) 
- loại hành động: tới chợ, mua hàng, check-in...

Data 
```json 
[
{
  "id": number : 10 số,
  "scanAt": date ,
 
  "actionType" : string, //ENTER: vào chợ | BUY: hàng, OUT: ra chợ | CHECKIN: quét tại điểm ...
  //thông tin điểm đến 
  "orgTitle" : string, //"tên siêu thị, chợ, hoăc mô tả điểm checkpoint: ngã 3 đường",
  "orgType" : "COMPANY" | "WARD" , 
  "orgId" : "number", 
  "checkpointId" : "string", //null or uuid 
  "scannerLocation" : "string",
  "scannerGPS" : {log, lat},
     
  "scannerId: "number"
},

]
```

### MobileApp

#### Đăng nhập
- bằng tài khoản của đơn vị 

- đơn vị là Phường / xã, có 2 loại 
    - orgType: WARD 
    - ADMIN
    - SCANNER: cán bộ quét thẻ

- doanh nghiệp: có thể là chợ, siêu thị
    - orgType: COMPANY
    - ADMIN
    - SCANNER: nhân viên quét thẻ

- Thông báo sau khi đăng nhập thành công
    - chọn điểm đang đứng `scannerLocation`

```json 
{
  "scannerId" : number, //id 
  "orgId" : number,
  "orgType" : "COMPANY | WARD", 
  "orgLocation" : string, //null 
  "scannerLocation" : string,
      //nếu COMPANY thì = orgLocation
      //nếu WARD thì hiện popup yêu cầu nhập vào" 
}
```

#### Kiểm tra thẻ tại siêu thị, chợ 

Loại hành động khi quét QRCode, hoặc nhập ID trên thẻ. 

- xác nhận vào cổng: quét qrcode
- Xác nhận mua hàng: 
- Xác nhận ra cổng: quét qrcode
- xác nhận checkpoint tại siêu thị: 

#### Kiểm tra thẻ khi di chuyển trong Phường

- Kiểm tra tại checkpoint

- Xác nhận Vi phạm: 

#### Danh sách Thẻ đã kiểm tra
> quét qrcode hoặc nhập ID 

Liệt kê theo thời gian: không sửa xoá

Display data
- ID
- actionType: ra chợ, vào chợ, ...
- lúc : `dd/mm/yyyy hh:mm` 


Data 
```json 
[
{
  "id": number : 10 số,
  "scanAt": date ,
 
  "actionType" : string, //ENTER: vào chợ | BUY: hàng, OUT: ra chợ | CHECKIN: quét tại điểm ...
  //thông tin điểm đến 
  "orgTitle" : string, //"tên siêu thị, chợ, hoăc mô tả điểm checkpoint: ngã 3 đường",
  "orgType" : "COMPANY" | "WARD" , 
  "orgId" : "number", 
  "checkpointId" : "string", //null or uuid 

  "scannerId": "number", //id tài khoản đăng nhập 
  "scannerLocation" : "string", //scanner nhập vào 
  "scannerGPS" : {log, lat},//scanner lấy toạ độ gps 
},

]