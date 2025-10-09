

# Learn Ir Raw For AC

##### import RogoCore

#### Các hàm sử dụng cho phần học lệnh mã raw cho điều hoà

### Phát hiện lệnh cho điều khiển AC

```
RGCore.shared.device.setIRLearningModeFor(deviceType: RGBProductCategoryType,
                                          hub: RGBDevice,
                                          observer: AnyObject?,
                                          isEnable: Bool,
                                          timeout: Int?,
                                          completion: (_ response: RGBIrRemoteRawInfo?, (any Error)?) -> Void)
```
Trong đó:

- deviceType: .AC
- hub: truyền vào device IR được chọn làm hub
- observer: self
- isEnable: true
- timeout: set timeout
- completion: check lỗi, response trả ra RGBIrRemoteRawInfo chính là mã raw vừa học được

### Tạo key cho nút điều khiển vừa học được

```
let cmdKey = RGBIrAcCmdValues.getAcControlKeyWith(isPowerOn: Bool?,
                                     mode: RGBIrAcModeType?,
                                     temperature: Int?,
                                     fanSpeed: RGBIrAcFanType?)
```
Lưu ý: Trong trường hợp người dùng học nút tắt điều hoà thì cmdKey = 0, vì nút tắt nguồn không cần quan tâm tới các giá trị như nhiệt độ, chế độ điều hoà và chế độ quạt

Trong đó:

key ở đây sẽ có kiểu Int?

- isPowerOn: nil (khi truyền là nil - sẽ là on nghĩa là trạng thái bật của điều khiển)
- RGBIrAcModeType: truyền vào 1 trong các mode của điều khiển điều hoà (.AC_MODE_AUTO, .AC_MODE_COOLING, .AC_MODE_DRY, .AC_MODE_HEATING, .AC_MODE_FAN)
- temperature: truyền vào nhiệt độ (15-30)
- fanSpeed: truyền vào tốc độ quạt (.FAN_SPEED_AUTO, .FAN_SPEED_LOW, .FAN_SPEED_NORMAL, .FAN_SPEED_HIGH, .FAN_SPEED_MAX, .FAN_SPEED_DISABLE)

### Sau khi set xong các giá trị cho key gán nó cho mã raw

VD: 
```
            guard let cmd = self.detectedRemoteCommand, cmd.irCmdKey == nil else { return }
            cmd.irCmdKey = cmdKey
            
            // Add cmd to lst
            if let index = self.learnedRawCmds.firstIndex(where: {$0.irCmdKey == cmdKey}) {
                self.learnedRawCmds[index] = cmd
            } else {
                self.learnedRawCmds.append(cmd)
            }

```
Trong đó: 
- cmdKey chính là cmdKey vừa được tạo ở trên và cmd: RGBIrRemoteRawInfo, chính là mã raw vừa học được, bước này set irCmdKey của mã raw là gía trị key vừa được tạo
-  if let index = self.learnedRawCmds.firstIndex(where: {$0.irCmdKey == cmdKey}) {
                self.learnedRawCmds[index] = cmd
            } else {
                self.learnedRawCmds.append(cmd)
            }
Bước này kiểm tra xem key bạn vừa tạo có bị trùng với key đã có sẵn trong list không, nếu key bị trùng thì sẽ thay thế ở vị trí đó, còn nếy key không bị trùng thì sẽ được thêm vào list để chuẩn bị cho việc add


### Gửi lệnh xác minh cho thiết bị IR AC bằng lệnh IR thô cụ thể.
Hàm cho phép kiểm tra một lệnh IR đã học (`testRawInfo`) duy nhất trên một thiết bị AC thông qua hub đã chỉ định.
Hàm này cũng lấy danh sách tất cả các lệnh IR đã học (`learnedRawCommands`) để kiểm tra chéo hoặc tham chiếu.
Sử dụng phương pháp này trong quá trình học mã raw hoặc để xác thực tính chính xác và phản hồi của từng lệnh IR.

```
            RGCore.shared.device.sendVerifyIrRawRemoteCommand(hub: RGBDevice,
                                                              testRawInfo: RGBIrRemoteRawInfo,
                                                              learnedRawCommands: [RGBIrRemoteRawInfo],
                                                              observer: AnyObject?,
                                                              timeout: Int?,
                                                              completion: (_ response: RGBIrRemoteRawInfo?, (any Error)?) -> Void)
```
Trong đó: 
- hub: thiết bị hub IR chịu trách nhiệm truyền tín hiệu IR
- testRawInfo: tệnh IR thô cần được xác minh
- learnedRawCommands: anh sách tất cả các lệnh IR thô đã học trước đó cho thiết bị
- observer: self
- timeOut: set thời gian timeOut (có thể truyền nil)
- completion: check lỗi, trả ra RGBIrRemoteRawInfo

### Add điều khiển vừa được học

```
RGCore.shared.device.addAcRemoteFromLearnedRaws(remoteInfos: [RGBIrRemoteRawInfo],
                                                label: String,
                                                group: RGBGroup?,
                                                toHub: RGBDevice,
                                                timeOut: Int?,
                                                observer: AnyObject?,
                                                completion: RGBCompletionObject<RGBDevice?>?)
```
Trong đó: 
- remoteInfos: truyền vào list raw vừa học được ở trên
- label: truyền vào tên thiết bị
- group: truyền vào nhóm phòng thiết bị - RGBGroup
- toHub: truyền vào hub điều khiển - RGBDevice
- timeOut: set thời gian timeOut (có thể truyền nil)
- observer: self
- completion: check lỗi, trả ra RGBDevice

### Hiển thị thiết bị sau khi add xong

Để phân biệt loại điều khiển điều hoà học raw và điều khiển điều hoà mã protocol trước đây sử dụng getLearnedAcRemoteRawCmds
VD:
```
        if let learnedCmd = device.getLearnedAcRemoteRawCmds() {
           // Nếu learnedCmd của thiết bị điều khiển điều hoà đó có tồn tại thì điều khiển điều hoà đó là điều khiển được học raw
                  learnedCmd: [RGBIrRemoteRawInfo]
                  sau khi lấy được 
        }
```
Tiếp theo lấy ra các lệnh điều khiển:

```
        if let cmd = device.getLearnedAcRemoteRawCmds()?.first(where: {$0.irCmdKey != 0}) {
        // irCmdKey == 0 (Nút tắt nguồn)
            self.currentTemperature = cmd.irCmdValues?.temperature
            self.fanSpeed = cmd.irCmdValues?.fanSpeed
            self.mode = cmd.irCmdValues?.mode
        }

```
Bằng cách sử dụng getLearnedAcRemoteRawCmds ta lấy được thông tin về nhiệt độ, mode điều hoà và mode quạt 

### Điều khiển điều hoà sau khi add xong

```
            let valueObj = RGBIRLearnedModeControlMessage(acCmdKey: irKey)
            
            RGCore.shared.device.sendControlDeviceMessageWith(deviceUUID: String,
                                                              value: RGBCmdValue,
                                                              elements: [Int],
                                                              completion: RGBCompletionObject<Bool>?)

```


Trong đó: 
- deviceUUID: truyền vào uuid của device
- value: truyền vào valueObj tương ứng với key
- elements: truyền vào element của thiết bị
- completion: check lỗi
