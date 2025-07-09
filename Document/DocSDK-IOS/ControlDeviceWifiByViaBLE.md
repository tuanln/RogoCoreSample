

# Control Device Wifi By Via BLE

##### import RogoCore

### B1: Kiểm tra trạng thái kết nối của thiết bị Wifi
```
                RGCore.shared.device.isConnectedBleWith(deviceId: String)
```

Trong đó:
- deviceId: truyền vào uuid của thiết bị Wifi

Nếu RGCore.shared.device.isConnectedBleWith(deviceId: String) là true thì đang tồn tại kết nối BLE còn nếu là false thì đang không có kết nối BLE với thiết bị này

Vd: 
```
        self.isConnectedViaBle = RGCore.shared.device.isConnectedBleWith(deviceId: device.uuid ?? "")
 ```
        
### B2: Mở kết nối BLE cho thiết bị Wifi được chọn

```
                RGCore.shared.device.openBleConnectionWith(deviceId: String,
                                                           timeout: Int?,
                                                           didDisconnectedHandler: (String?) -> (),
                                                           completion: (_ response: Bool?, (any Error)?) -> Void)
```

Trong đó:
- deviceId: truyền vào uuid của thiết bị Wifi
- timeOut: thời gian timeOut đơn vị giây (Vd: 30)
- didDisconnectedHandler: trả ra deviceMac của thiết bị ở dạng String
- completion: check lỗi, nếu là error nil và response == true => đã kết nối thành công

Vd:
```
            if device != nil {
                RGCore.shared.device.openBleConnectionWith(deviceId: device?.uuid ?? "", timeout: 30) { deviceMac in
                    
                } completion: { response, error in
                    if error == nil {
                        let viewSucess = RGUIAnimationSuccessV2.loadNib()
                        viewSucess.device = self.device
                        viewSucess.grouppedDevice = self.grouppedDevice
                        viewSucess.lbContentAnimation.text = "Chuyển đổi thành công"
                        RGUIPopup.showPopupWith(contentView: viewSucess, popsition: .bottom)
                    } else {
                        let viewFail = RGUIAnimationFailV2.loadNib()
                        viewFail.device = self.device
                        viewFail.grouppedDevice = self.grouppedDevice
                        viewFail.lbError.text = "Chuyển đổi thất bại"
                        RGUIPopup.showPopupWith(contentView: viewFail, popsition: .bottom)
                    }
                }
            }
```
### B3: Sau khi đã kết nối thành công tiến hành điều khiển thiết bị Wifi qua kênh BLE

```

            RGCore.shared.device.sendControlMessageViaBleWith(deviceUUID: String,
                                                              value: RGBCmdValue,
                                                              elements: [Int],
                                                              completion: (_ response: Bool?, (any Error)?) -> Void)
```

Trong đó:
- deviceUUID: truyền vào uuid của thiết bị Wifi
- value: truyền vào RGBCmdValue (VD: RGBValueOpenClose => .open, .stop, .close)
- elements: truyền vào element muốn điều khiển
- completion: check lỗi, nếu là error nil và response == true => điều khiển thiết bị thành công

Vd:
```
    private func sendControlMessageViaBle(value: RGBValueOpenClose,
                                          elements: [Int],
                                          completion: RGBCompletionObject<Bool?>? = nil) {
        self.openBleConnetion { [weak self] isConnected in
            guard let self = self,
                  self.device.productType?.protocolType == .WILE,
                  isConnected == true else {
                return
            }
            RGCore.shared.device.sendControlMessageViaBleWith(device.uuid ?? "",
                                                              value: value,
                                                              elements: elements,
                                                              completion: completion)
        }
    }

```

### Huỷ kết nối
```
        RGCore.shared.device.cancelBleConnection()
```
