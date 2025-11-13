# Hướng Dẫn Flow Thêm Thiết Bị - Rogo Core SDK

## Tổng Quan Nhanh

Flow thêm thiết bị mới vào tài khoản gồm **8 bước chính**:

```
1️⃣ Add Devices (Khởi tạo)
    ↓
2️⃣ Scan Available Devices (Quét thiết bị)
    ↓
3️⃣ Connect to Target Device (Kết nối)
    ↓
4️⃣ Send Config Message → Get MAC Address (Cấu hình & nhận MAC)
    ↓
5️⃣ Device Returns WiFi SSID List (Thiết bị trả về danh sách WiFi)
    ↓
6️⃣ Send WiFi Credentials (SSID/Password) (Gửi thông tin WiFi)
    ↓
7️⃣ Device Connects to WiFi & Response (Thiết bị kết nối WiFi)
    ↓
8️⃣ Config Success & Sync to Cloud via API (Đồng bộ lên Cloud)
    ↓
✅ HOÀN THÀNH
```

---

## API Mapping - Từng Bước

### 📍 Bước 1: Add Devices
```swift
RGCore.shared.device.wileDelegate = self
```

### 📍 Bước 2: Scan Available Devices
```swift
RGCore.shared.device.scanAvailableWileDevice(
    timeout: 60,
    limitRssi: nil,
    completion: { response, error in
        // response: RGBMeshScannedDevice
    }
)
```

### 📍 Bước 3: Connect to Target Device
```swift
RGCore.shared.device.startConfigWileDevice(device: detectedDevice)

// Delegate callback
func didConnectDeviceSuccess(setDeviceInfoHandler: ((String?, String?) -> ())?) {
    setDeviceInfoHandler?(deviceName, groupId)
}
```

### 📍 Bước 4: Send Config & Get MAC
```swift
// SDK tự động xử lý
func didUpdateProgessing(percent: Int) {
    // Hiển thị progress: 0% -> 100%
}
```

### 📍 Bước 5: Device Returns WiFi List
```swift
func didScannedWifiInfo(
    _ listWifiInfos: [RGBWifiInfo],
    wifiSelectionHandler: ((String, String) -> ())?
) {
    // listWifiInfos chứa: ssid, rssi, authType
    // Lưu wifiSelectionHandler để dùng ở bước 6
}
```

### 📍 Bước 6: Send WiFi Credentials
```swift
// Khi user chọn WiFi và nhập password
wifiSelectionHandler?(selectedSSID, password)
```

### 📍 Bước 7: Device Connects to WiFi
```swift
// Nếu thành công → chuyển bước 8
// Nếu thất bại → callback này được gọi
func didFailedToConnectWifi(
    _ ssid: String?,
    _ password: String?,
    _ wifiConnectionState: RGBWifiConnectionErrorType
) {
    switch wifiConnectionState {
    case .PASSWORD_WRONG:       // Sai password
    case .SSID_NOTFOUND:        // Không tìm thấy WiFi
    case .SOMETHING_WENT_WRONG: // Lỗi khác
    }
}
```

### 📍 Bước 8: Config Success & Sync to Cloud
```swift
func didFinishAddWileDevice(response: RGBDevice?, error: Error?) {
    // response chứa thông tin thiết bị đã được thêm
    // Thiết bị đã tự động đồng bộ lên Cloud
}
```

---

## Quick Start Code

```swift
import RogoCore

class AddDeviceVC: UIViewController, RGBWileDelegate {

    var detectedDevice: RGBMeshScannedDevice?
    var wifiSelectionHandler: ((String, String) -> ())?

    // Bắt đầu flow
    func startAddDevice() {
        RGCore.shared.device.wileDelegate = self

        RGCore.shared.device.scanAvailableWileDevice(
            timeout: 60,
            limitRssi: nil
        ) { [weak self] response, error in
            if let device = response {
                self?.detectedDevice = device
                RGCore.shared.device.startConfigWileDevice(device: device)
            }
        }
    }

    // MARK: - RGBWileDelegate

    func didConnectDeviceSuccess(setDeviceInfoHandler: ((String?, String?) -> ())?) {
        setDeviceInfoHandler?("My Device", "room-uuid")
    }

    func didUpdateProgessing(percent: Int) {
        print("Progress: \(percent)%")
    }

    func didScannedWifiInfo(
        _ listWifiInfos: [RGBWifiInfo],
        wifiSelectionHandler: ((String, String) -> ())?
    ) {
        self.wifiSelectionHandler = wifiSelectionHandler
        // Hiển thị UI chọn WiFi
        showWifiList(listWifiInfos)
    }

    func didFailedToConnectWifi(
        _ ssid: String?,
        _ password: String?,
        _ wifiConnectionState: RGBWifiConnectionErrorType
    ) {
        // Hiển thị lỗi và cho user nhập lại
        showError(wifiConnectionState)
    }

    func didFinishAddWileDevice(response: RGBDevice?, error: Error?) {
        if let device = response {
            print("✅ Success! Device ID: \(device.uuid ?? "")")
        }
    }

    // Khi user chọn WiFi
    func userSelectedWifi(ssid: String, password: String) {
        wifiSelectionHandler?(ssid, password)
    }
}
```

---

## Các Mô Hình Sử Dụng

### 🎯 Mô hình 1: Sử dụng Delegate (Khuyến nghị)

**Ưu điểm:** Tách biệt logic, dễ test, code sạch hơn

```swift
RGCore.shared.device.wileDelegate = self
RGCore.shared.device.startConfigWileDevice(device: device)
```

### 🎯 Mô hình 2: Sử dụng Closures

**Ưu điểm:** Tất cả logic ở một chỗ, nhanh cho prototype

```swift
RGCore.shared.device.startConfigWileDevice(
    device: device,
    didUpdateProgessing: { percent in
        // Progress
    },
    wifiScanCompletedHandler: { wifiList in
        // Bước 5
    },
    wifiSelectionHandler: { ssid, password in
        // Bước 6
    },
    wifiConnectErrorHandler: { ssid, password, errorType in
        // Bước 7 - error
    },
    didCompletedHandler: { device in
        // Bước 8
    }
)
```

---

## State Diagram

```
[IDLE]
  ↓ (User taps "Add Device")
[SCANNING] → Timeout → [ERROR: Scan Failed]
  ↓ (Device found)
[CONNECTING]
  ↓ (Connected)
[CONFIGURING] (0% → 100%)
  ↓ (Get MAC success)
[WIFI_SCANNING]
  ↓ (WiFi list received)
[WAITING_USER_INPUT]
  ↓ (User selects WiFi + Password)
[CONNECTING_WIFI]
  ├─ Success → [SYNCING_CLOUD] → [COMPLETED] ✅
  └─ Failed → [WAITING_USER_INPUT] (Retry)
```

---

## Xử Lý Lỗi

| Bước | Lỗi Có Thể Xảy Ra | Cách Xử Lý |
|------|-------------------|------------|
| **2** | Timeout scanning | Retry hoặc di chuyển gần thiết bị |
| **3** | Connection failed | Kiểm tra thiết bị, reset nếu cần |
| **7** | PASSWORD_WRONG | Yêu cầu nhập lại password |
| **7** | SSID_NOTFOUND | Kiểm tra WiFi router |
| **8** | Cloud sync failed | SDK tự động retry |

---

## Checklist Implementation

- [ ] Import RogoCore framework
- [ ] Implement RGBWileDelegate trong ViewController
- [ ] Thiết lập wileDelegate trước khi scan
- [ ] Xử lý timeout khi scan (hiển thị retry button)
- [ ] UI hiển thị progress bar (0-100%)
- [ ] UI chọn WiFi từ danh sách
- [ ] UI nhập password WiFi
- [ ] Xử lý 3 loại lỗi WiFi (PASSWORD_WRONG, SSID_NOTFOUND, SOMETHING_WENT_WRONG)
- [ ] Hiển thị thông báo thành công
- [ ] Refresh danh sách thiết bị sau khi add
- [ ] Implement nút Cancel để hủy quá trình (cancelWileConfig)
- [ ] Test với WiFi có password
- [ ] Test với Open WiFi (không password)
- [ ] Test xử lý lỗi sai password

---

## Flow Diagram Chi Tiết

Xem file: [DeviceOnboardingFlow.md](./DeviceOnboardingFlow.md)

---

## Thời Gian Ước Tính

| Bước | Thời gian trung bình |
|------|---------------------|
| Scan (Bước 2) | 10-60 giây |
| Connect (Bước 3) | 5-10 giây |
| Config & MAC (Bước 4) | 5-15 giây |
| WiFi Scan (Bước 5) | 10-20 giây |
| WiFi Connect (Bước 7) | 10-30 giây |
| Sync Cloud (Bước 8) | 2-5 giây |
| **Tổng** | **~1-3 phút** |

---

## Tips & Best Practices

### ✅ DO:
- Hiển thị progress bar cho user
- Cho phép user cancel quá trình
- Validate password trước khi gửi (độ dài tối thiểu 8 ký tự cho WPA2)
- Hiển thị cường độ tín hiệu WiFi (RSSI)
- Sắp xếp WiFi theo cường độ tín hiệu (mạnh nhất ở trên)
- Cache danh sách WiFi để không cần scan lại nếu user nhập sai password
- Hiển thị icon khóa cho WiFi có mật khẩu

### ❌ DON'T:
- Không để timeout quá ngắn (< 30s)
- Không block UI khi đang config
- Không gửi password dưới dạng plain text qua log
- Không tự động retry vô hạn khi fail
- Không bỏ qua error handling

---

## Tài Liệu Liên Quan

- **Chi tiết Flow**: [DeviceOnboardingFlow.md](./DeviceOnboardingFlow.md) ⭐
- **API WiFi Device**: [AddWile.md](../DocSDK-IOS/AddWile.md)
- **API BLE Device**: [AddDeviceBLE.md](../DocSDK-IOS/AddDeviceBLE.md)
- **Đổi WiFi**: [ChangeWifi.md](../DocSDK-IOS/ChangeWifi.md)
- **Điều khiển qua BLE**: [ControlDeviceWifiByViaBLE.md](../DocSDK-IOS/ControlDeviceWifiByViaBLE.md)

---

**📱 Rogo IoT Mobile SDK**
**Version:** 1.0
**Last Updated:** 2025-11-13
