
#Android电源管理的结构
Android平台基于 Linux内核，所以设计时也以 Linux内核电源管理为基础。传统的 Linux内核没有考虑到移动设备的特点，因此在 Android平台中，创建了考虑到移动设备的特点，即有限电池容量的控制电源管理的服务，这就是用户空间的 Power Manager Service。另外，内核空间也引入了由用户空间 Power Manager Service 控制的 EarlySuspend、Late Resume以及 Wake Lock新概念。
本文将介绍 Android平台电源管理的结构及相应对象的作用。
Android电源管理的层级结构下面将介绍Android平台的电源管理层级结构及各层级作用，以及之后在何处使用何种内容。
下图只显示了 Android平台层级结构下的电源管理相关对象。

Android 应 用 层 包 括 利 用 Power Manager Service 的 应 用 程 序 及Power Manager。
Android框架层中有 Power Manager Service这一远程服务。JNI（Java Native Interface，Java本地接口）层是由 Power ManagerService 的 本 地 方 法 调 用 的 本 地 函 数，或 是 本 地 函 数 调 用 Power
Manager Service方法的本地空间。com_android_server_PowerManagerService.cpp（framewoks/
base/services/jni）文件中定义了本地函数。HAL（Hardware Adaptation Layer，硬 件 适 配 层）层 抽 象 化 了 从
Android用户空间到内核空间硬件设备的访问，存在用于Android 电源
管理的Legacy HAL 库—libpower。libpower 将Wake Lock 的信息
传递到内核空间。
系统核心库libsuspend 触发内核空间的Early Suspend 及Late Resume
操作。sysfs 虚拟文件系统是用户空间及内核空间通信的方法之一。
图2不能准确说明Power Manager Service 这一远程服务。从客户端
与服务器端角度体现的Android 电源管理，如图2所示。
远程服务—Power Manager Service 相当于服务的服务器端，有用
于连接远程服务的RPC接口IPowerManager。使用远程服务的服务
客户端通过Power Manager 连接并使用远程服务—Power Manager
Service。
![这里写图片描述](http://img.blog.csdn.net/20160121165432288)

##Power Manager
为了使应用程序及服务等的客户端访问远程服务，Android 通过提供包装（Wrapper）类的管理器类提供利用远程服务的间接性标准化方法。
×××Manager 类由系统Fetcher 类（静态类）生成对象，此时，RPC
Binder 的 asInterface() 方 法 返 回 ×××Manager Service 的 代 理
对象引用。×××Manager 可以通过获取返回的代理对象引用访问
×××Manager Service，如图所示。
![这里写图片描述](http://img.blog.csdn.net/20160121165607634)
另外， 客户端获取 ××× Manager 类的对象引用后， 可以由 ×××Manager 访问××× Manager Service， 为获取××× Manager 类
的对象引用，Android Java 空间提供了 getSystemService() 方法。
因此，客户端使用getSystemService() 方法即可随时访问所需的远程
服务，如图 所示。
![这里写图片描述](http://img.blog.csdn.net/20160121165722901)
客户端利用 Power Manager Service的方法有两种，一种通过 Power
Manager间接调用，另一种通过 Power Manager Service的代理对象
引用直接调用。
##Power Manager Service
Power Manager Service 类 在 Android Java 空 间 继Manager.Stub服务存根类，实3 现 Power Manager性功能。Power Manager Service的主要操作是控制Sleep（睡眠），并将相应信息传递至本地及内核空间。
下图 为决定 Power Manager Service 状态的核心方State()方法将屏幕亮度信息传递给本地空间的过程。
![这里写图片描述](http://img.blog.csdn.net/20160121170142456)
从 Gingerbread（2.3）版 本 到 Jelly Bean（4.1）版 本，PowerManager Service的结构没有变化。但从 Jelly Bean Plus（4.2）版本开始，结构上完全更改为全新的代码。
setPowerState()方法决定 Power Manager Service的电源状态，并
根据电源状态控制灯光对象的亮度。
Java本地方法 nativeSetPowerState()方法通过 JNI向本地空间发送屏幕 On（开）/Off（关）状态及屏幕调光状态。输入管理器在发生按键事件及触控事件时，浏览相应信息以控制屏幕。Java本地方法 nativeSetScreenState()方法通过 JNI调用系统内核库 libsuspend 的 autosuspend_ disable() 函 数 或 autosuspend_enable()函数控制屏幕 On（开）/Off（关）状态。另外， autosuspend_
disable()函数根据内核的电源管理方式选择并调用 autosuspend_earlysuspend_disable()、autosuspend_ autosleep_disable()、autosuspend_wakeup_count_disable() 函 数 之 一， autosuspend_
enable() 函 数 选 择 并 调 用 autosuspend_earlysuspend_enable() 、 autosuspend_autosleep_enable() 、 autosuspend_wakeup_count_enable ()函数之一。
制造商及平台开发商为了与已有 Android 版本兼容，选择 state 文件管理电源的方式，所以此处也调用 autosuspend_earlysuspend_
disable()函数或 autosuspend_earlysuspend_enable ()函数进行介绍。 autosuspend_earlysuspend_disable()函数或 autosuspend_
earlysuspend_enable ()函数决定内核的电源状态，由 sysfs接口触
发内核空间的 Late Resume及 Early Suspend操作。
