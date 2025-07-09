

# OTADeviceBLE

##### import RogoCore

### B1: Scan device BLE cần OTA
```
            RGCore.shared.device.scanBleMeshProxyDeviceWith(deviceId: String,
                                                        excludeUUIDs: [UUID]?,
                                                        timeout: Int?,
                                                        didUpdateProgessing: (_ completedPercent: Int) -> (),
                                                        completion: (_ response: CBPeripheral?, (any Error)?) -> Void)
```

- Scan các thiết bị BLE khả dụng

Trong đó:
- deviceId: truyền vào UUID của thiết bị đang được chọn (từ UUID trong core sẽ suy ra ProductID chỉ để quét đúng những loại thiết bị đó)
- excludeUUIDs: có thể truyền nil và không nhất thiết phải truyền vào (phần này được sử dụng trong trường hợp khi completion response trả ra CBPeripheral không đúng con thiết bị mà bạn cần OTA thì có thể ném phần res đó vào excludeUUIDs này thì lần sau scan hàm này sẽ bỏ qua thiết bị đó)
- timeout: Set thời gian timeOut (có thể truyền nil)
- didUpdateProgessing: đây là 1 closure mục đích dùng để cập nhật trạng thái tiến trình VD:
```
            RGCore.shared.device.scanBleMeshProxyDevice(deviceUUID: self.deviceID,
                                                        excludeUUIDs: nil,
                                                        timeout: nil) { [weak self] completedPercent in
                //                print("scanOTADeviceWith: \(completedPercent)")
                if completedPercent == -3 {
                    self?.lbScanWarning.text = "Tìm thấy 1 thiết bị"
                }
                
                if completedPercent == -2 {
                    self?.lbScanWarning.text = "Đang kết nối tới thiết bị"
                }
                
                if completedPercent == -1 {
                    self?.lbScanWarning.text = "Đang gửi thông tin nhận dạng thiết bị"
                }
            } completion: { response, err in
```
        
### B2: Yêu cầu cập nhật firmware cho thiết bị BLE
```
        RGCore.shared.device.requestUpdateFirmwareForBleMeshDeviceWith(deviceId: String,
                                                                       peripheral: CBPeripheral,
                                                                       firmwareVersion: String?,
                                                                       otaData: Data?,
                                                                       timeOut: Int?,
                                                                       downloadProgessing: (_ completedPercent: Int) -> (),
                                                                       didUpgadeFirmwareProgessing: (_ completedPercent: Int) -> (),
                                                                       completion: (_ response: Bool?, (any Error)?) -> Void)
```
Trong đó:
- deviceId: truyền vào UUID của thiết bị đang được chọn
- peripheral: truyền vào CBPeripheral của phần response của hàm scanBleMeshProxyDevice
- otaData: có thể truyền nil
- timeOutL Set thời gian timeOut của hàm có thể truyền nil
- downloadProgessing: trả ra kiểu Int, hiển thị tiến trình download
- didUpgadeFirmwareProgessing: hiển thị tiến trình update firmware VD:
``
            didUpgadeFirmwareProgessing: {[weak self] percent in
            guard let self = self else {
                return
            }

            if percent == -2 {
                self.lbProgressAction.text = "Đang kết nối tới thiết bị"
            } else if percent == -1 {
                self.lbProgressAction.text = "Đang gửi yêu cầu cập nhật thiết bị"
            } else {
                self.lbProgressAction.text = percent == 100 ? "Gửi thông tin hoàn tất. Thiết bị sẽ khởi động lại" : "Đang gửi thông tin cập nhật tới thiết bị"
                self.lbProgress.text = "\(percent)%"
            }
        }
``
- completion: check lỗi

### Lệnh huỷ tiến trình update firmware cho thiết bị BLE

Trong quá trình đang update nếu muốn huỷ tiến trình cập nhật firm gọi hàm:

```
    cancelUpdateFirmwareForBleMesh()
```
