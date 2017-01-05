# DanmuManager 一个简单的弹幕工具

## 使用方法 Usage

`DanmuManager` 和 `VideoDanmuManager` 有不同的应用场景，后者用于视频弹幕。

你可以运行项目中的测试，来了解二者的使用方法。

<img src="https://raw.githubusercontent.com/cheng-kang/DanmuManager/master/DanmuManager-1.gif" width="200">
<img src="https://raw.githubusercontent.com/cheng-kang/DanmuManager/master/DanmuManager-2.gif" width="200">
<img src="https://raw.githubusercontent.com/cheng-kang/DanmuManager/master/DanmuManager-3.gif" width="200">

### DanmuManager

1. 创建 DanmuManager

	```
	let dm = DanmuManager(with: self.view)
	```
	
	`init` 方法允许自定义： 
	
	- `top`、`bottom`弹幕显示在 `view` 中的上下位置范围;
	- `speed` 弹幕的速度;
	- `customFont` 弹幕字体（`UIFont`, 包括字体家族和大小，默认为系统字体，字号 20）。

2. 添加一条弹幕 

	```
	dm.add(with: “Wow so cool!!!”, at: 4)
	```
	
	`add` 方法有两个必传参数，`text`（弹幕文字） 和 `line`（弹幕所在行）;可选参数 `hasBorder` 用于设置该条弹幕是否有边框（默认 borderColor = UIColor.black.cgColor，borderWidth = 1）,默认值为 false;可选参数 `isAdvance` 用于设置该条弹幕是否开启高级功能（目前支持修改 文字颜色 和 背景颜色），默认值为 false。
	
	如果需要弹幕出现在随机行，可以使用
	`func addRandom(with text: String = "This is a test Danmu.", at line: Int = 0, hasBorder: Bool = false, isAdvanced: Bool = false)`。
	
3. 暂停/继续 弹幕

	```
	dm.pause()
	dm.resume()
	```
	
	你也可以使用 `dm.toggle()` 来快捷切换 暂停/继续。
	
	
### VideoDanmuManager

1. 创建 VideoDanmuManager
	
	```
	let vdm = VideoDanmuManager(view: self.view,
                                videoLength: 10,
                                danmuData: [
                                    (3.4, "3.4 Wowowowowowow!"),
                                    (3.4, "3.4 SOOOO COOOOOOOOOOL!"),
                                    (3.4, "3.4 Amazing!!!!"),
                                    (3.4, "3.4 I love you~"),
                                    (3.4, "3.4 MY BABY!!!!"),
                                    (1.1, "1.1 This is a test Danmu!!!"),
                                    (2.0, "2.0 Another test Danmu."),
                                    (4.1, "4.1 Amazing!!!!"),
                                    (6.1, "6.1 Test!!!!"),
                                    (8.1, "8.1 Test!!!!"),
                                    (9.1, "9.1 Test!!!!"),
                                    (10, "10 Test!!!!"),
            ],
                                isSorted: false
        )
	```
	
	`VideoDanmuManager` 的创建有三个必选参数，`view`（显示弹幕的视图），`videoLength`（视频时长，精确到 0.1 秒）和 `danmuData`（已经存在的弹幕列表）。
	
	可选参数 `videoCurrent` 默认为 0，即开始时刻；你可以通过传入不同的时间值来设置显示弹幕的初始时间（比如在视频从 1 分 15 秒开始播放，则应设置 `videoCurrent` 为 75）。
	可选参数 `isSorted` 默认为 true，即默认数据集已经按照弹幕显示时间从先到后排序；如传入 false，则将自动调用 `func sort()` 对数据进行排序。
	
	可选参数 `top` 和 `bottom` 同 `DanmuManager`。
	
	
2. 开启弹幕
	
	```
	vdm.start()
	```
	
	你需要在视频开始播放的同时，手动开启弹幕。
	
3. 暂停/继续 弹幕
	
	```
	vdm.pause()
	vdm.resume()
	```
	
	你也可以使用 `vdm.toggle()` 来快捷切换 暂停/继续。
	
4. 中止弹幕

	```
	vdm.stop()
	```
	
	
5. 重新开始弹幕

	```
	vdm.restart()
	```
	
	你也可以在指定时间点重新开始弹幕：
	
	```
	vdm.restart(at: 75)
	```
	
	
## 设计思路

`DanmuManager` 通过设置的字体大小计算文本显示高度 `lineHeight`，通过 `numberOfLines = Int(floor((bottom - top) / lineHeight))` 得到可以显示的弹幕行数；使用 `inUsingLines`，`enteringTimers` 和 `waitingQueues` 来分别记录 正在使用的行（有尾部还没有完全进入视图的弹幕的行），正在进入视图的弹幕的计时器 和 正在等待进入的弹幕队列。

为了防止弹幕重叠，只有当 当前行中前一条弹幕尾部进入视图 时（之后），下一条弹幕才可以发射。

`enteringTimers` 中每一个 Timer 都对应一条正在进入视图的弹幕，当计时器结束时，通过 NotificationCenter 发通知将该弹幕所在的行的状态更改为 false。

因为弹幕存在暂停状态，如果弹幕对应的 Timer 不同时暂停，将导致 弹幕所在行状态 提前更改为 false。为了解决这个问题，我参考他人的想法并改进后创建了 PauseableTimer，并在本项目中使用上了。我的另外一篇博客更详细地介绍了 [PauseableTimer]()。

为了避免手动发射弹幕和自动发射队列中的弹幕出现冲突（弹幕重叠），所有弹幕通过 `taskTimer` 定时任务统一调度。

## 下一步 Next Step

现在流行的弹幕功能越来越复杂、高级，DanmuManager 只是完成了最基础的滚动文字弹幕功能。未来可能会学习常见的弹幕，添加一些更高级的功能，比如 对滚动弹幕的操作，居中弹幕 等。	
