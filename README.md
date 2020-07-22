# 触摸设备 DSDT 修补补充

## 目录

- [F&Q](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/%E8%A7%A6%E6%91%B8%E6%9D%BF%E8%A1%A5%E5%85%85.md#fq)
- [触摸板专有词汇集合](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/%E8%A7%A6%E6%91%B8%E6%9D%BF%E8%A1%A5%E5%85%85.md#%E8%A7%A6%E6%91%B8%E6%9D%BF%E4%B8%93%E6%9C%89%E8%AF%8D%E6%B1%87%E9%9B%86%E5%90%88)
- [触摸板必备软件集合](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/%E8%A7%A6%E6%91%B8%E6%9D%BF%E8%A1%A5%E5%85%85.md#%E8%A7%A6%E6%91%B8%E6%9D%BF%E5%BF%85%E5%A4%87%E8%BD%AF%E4%BB%B6%E9%9B%86%E5%90%88%E5%9D%87%E5%9C%A8%E7%BE%A4%E6%96%87%E4%BB%B6%E4%B8%AD)
- [DSDT 触摸板部分详解](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/%E8%A7%A6%E6%91%B8%E6%9D%BF%E8%A1%A5%E5%85%85.md#dsdt-%E8%A7%A6%E6%91%B8%E6%9D%BF%E9%83%A8%E5%88%86%E8%AF%A6%E8%A7%A3)
- [排错](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/%E8%A7%A6%E6%91%B8%E6%9D%BF%E8%A1%A5%E5%85%85.md#%E6%8E%92%E9%94%99)
- [鸣谢](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/触摸板补充.md#鸣谢)

## 致那些无脑放完驱动就能用了的小小小白

> VoodoI2C 不同于仍和一个其他的黑苹果驱动，直接将驱动放入 Clover 或者 L/E 是 **完全错误** 的做法，根据个人经验，高达 90% 以上的设备都默认使用 APIC 中断，只有极少数都设备默认  使用 GPIO 中断，然而，又因为绝大多数设备的 IOInterruptSpecifiers 都大于 2F，因此 APIC 中断控制器根本就不会工作，所以，即使你把驱动放进去能用了，也是极为低效的轮询模式，如果你阅读完 [群主博客](https://www.penghubingzhou.cn/) 和 本篇教程，你会发现这个问题显而易见。

![Circumstances](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Circumstances.png)

> 仅有极少数情况默认就能正确驱动

## 致刚接触触摸板的小白

> 在看这篇文章前，请仔细阅读[群主博客](https://www.penghubingzhou.cn/)，这篇文章旨在补充教程中没有的小细节，文章啰嗦但非常详细（废话连篇），请您耐心阅读（别光欣赏图片去了😝）

## F&Q

1. Dalao! 我虽然没有尝试，但是我想问问的触摸板到底能不能驱动啊！
   - “实践是检验真理的唯一标准”，我们并不能够通过您的三两句提问确定到底能不能成功，行动起来，发现问题，解决问题，才是真理！👨‍🔧
1. Dalao! 我该怎么用这些驱动，全部放进去吗?
   - 那就大错特错了，除非你同时拥有多个触摸设备，要不然就是使用其中的两个驱动 VoodooI2C + VoodooI2CXXX, 一个核心驱动，一个附属驱动，至于**怎么选择附属驱动**，请看[群主博客](https://www.penghubingzhou.cn/)。
1. Dalao! 群里的人都在说哪国语言啊，什么 HID, GPIO 是什么鬼？？？
   - 仔细阅读下文的👇`触摸板专有词汇集合`没什么你搞不懂的🤨。
1. Dalao! IORegisteryExplorer 里面触摸板设备名称里面没有 IOInterruptSpecifiers ，怎么办？
   - 确保你没有安装 VoodooI2C。
   - 如果在没有安装 VoodooI2C 时依然没有这就说明你无需对 DSDT 进行修改！你的运气帮了你一个大忙。不过也有极少数的例外，而且处理较为复杂，[请阅读此处（4. 例外 1）来通过 Windows 确认你的 Pin 值](https://github.com/ettingshausen/VoodooI2C-PreRelease/blob/master/%E8%A7%A6%E6%91%B8%E6%9D%BF%E8%A1%A5%E5%85%85.md#dsdt-%E8%A7%A6%E6%91%B8%E6%9D%BF%E9%83%A8%E5%88%86%E8%AF%A6%E8%A7%A3)，这需要你打包求助，由我们来为你解决。
1. Dalao! VoodooI2C 是不是万能的，能不能完美解决所有的触摸设备？
   - 不幸的是，答案是`不是`，VoodooI2C 目前仍然存在对一些设备的兼容问题。
1. Dalao! 什么是轮询？什么是中断？
   - 现实中的中断和轮询可以看看这个👉[🚪](https://baike.baidu.com/item/%E8%BD%AE%E8%AF%A2%E5%92%8C%E4%B8%AD%E6%96%AD/267433?fr=aladdin)。
   - 在触摸板上，我们这样定义：
   > 轮询就是每隔一个特定的时间扫描一次触摸板的状态，然后改变鼠标指针坐标。
   > 中断则是你触摸板的正常工作方式 (Windows / Linux 中即是如此)。

   - 在 DSDT 的修改中我们这样定义：（请仔细阅读下文👇）。
   > **GPIO 中断：** DSDT 在 `SBFG` 中存在有效 `GPIO Pin`，`CRS` 方法中返回 `(SBFB,SBFG)`，并应用启用 `GPIO` 控制器的补丁。  
   > **APIC 中断：** （注意，不是 ACPI）当你的 IOInterruptSpecifiers 值 **没有** 或者小于`2F(16进制)`时，此时就是 APIC 中断，无需对 DSDT 进行大量修改，只用应用 `Windows(20XX) 补丁`即可。此时使用的是 APIC 控制器而不是 GPIO 控制器。  
   > **轮询：** DSDT 中除了 `Windows(20XX)` 补丁以外，不应用其他补丁，CRS 方法中只返回 `SBFB` 或者 `(SBFB,SBFI)`。
1. Dalao! 到底是中断好还是轮询好？😖
   - 正如上文所述，中断才是触摸板的正确驱动方式，但受到 VoodooI2C 驱动，苹果触摸驱动及自身设备 GPIO 特殊性等等原因，有些设备只能通过**轮询**的方式驱动，轮询就像 Windows 里的**安全模式**，因为触摸板本身就不是设计来在一定时间不断轮询的，所以注定有 `Bug` 比如指针漂移，少手指等等，但是用这种方式驱动总比没有驱动强。
1. Dalao! DSDT 里我找不到我的触摸板啊，搜不到 `SBFG` 怎么办？
   - 你可以尝试搜索你的设备名，比如(`ETPD`, `TPDO`, `TPAD`等等)。
1. Dalao! DSDT 看得头晕眼花怎么办？
   - 请看下方的 DSDT 触摸板部分详解👇
1. Dalao! APIC 中断和 GPIO 中断有什么区别？
   - 两者使用不同的控制器
   - APIC 中断省时省力，但是要看看你有没有这个福气了
   - GPIO 则需要 VoodooGPIO (已包含在VoodooI2C中作为插件) 来启用 GPIO 控制器，同时也需要对 DSDT 进行相对复杂的修改（其实很简单）
1. Dalao! 能不能通过终端重新加载 VoodooI2C ？
   - 不幸的是，这个功能被某一次更新破坏了，官方至今未修复

## 触摸板专有词汇集合

1. HID
   > Human Interface Device 说白了就是人和电脑交互的设备
1. ELAN
   > ElanTech 生产的触摸板，简称 ELAN，常见于华硕电脑
1. SYNA
   > Synaptics 新思触摸板
1. CRS方法
   > DSDT 中用于返回值的“方法”
1. PS/2
   > PS/2 通道的触摸板，请使用 VoodooPS2 (Acidanthera自带手势), 如果出问题可以尝试 Rehabman 版本并配上手势文件，不建议使用 ApplePS2SmartTouchPad (多年未更新，且不开源)
1. I2C
   > I2C 通道的触摸板，请使用 VoodooI2C
1. 先暂时想到这么多。。。。

## 触摸板必备软件集合(均在群文件中)

- MaciASL -- 用于修改 DSDT
- IORegisteryExplorer v2.1 **(请勿使用其他版本)** -- 用于查看数据 / 排错
- FingerMgmt -- 用于查看识别手指个数
- maclog -- 用于查看日志排错

## DSDT 触摸板部分详解

1. 当你在 DSDT 里搜索 SBFG 或者你的触摸板名称时，会大致出来以下代码(是不是头晕眼花😵)

   ![Mess](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/ETPD.png)

1. 我们来提取提取主干，仔细看看不难发现其结构(我们需要了解的部分)如下，是不是一目了然？

   ![Main](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Main.png)

1. 接下来( GPIO 中断修改)
   1. 接下来请不断查看此处确保你一直位于你的触摸板里面

      ![Check](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Check.png)

   1. 应用所需的补丁，并按下编译，查看是否有错误，如有错误，请打包求助
   1. 移除 SBFI，或者有的 DSDT 的 SBFI 内容内嵌于 SBFB，请删除其中的代码
   1. 如果你的 DSDT 里本身就有 SBFG 且 `Pin List` 下方的内容不为零，那么直接跳到本小节的第 6 步，如果没有，请继续往下看
   1. 如果您没有 SBFG，请复制如下代码手动添加

      ![NO-SBFG](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/NO-SBFG.png)  

      ```asl
                Name (SBFG, ResourceTemplate ()
               {
                   GpioInt (Level, ActiveLow, ExclusiveAndWake, PullDefault, 0x0000,
                       "\\_SB.PCI0.GPI0", 0x00, ResourceConsumer, ,
                       )
                       {   // Pin list
                           0x修改
                       }
               })  
      ```

      添加完和上面一样就好了

   1. 确定 GPIO Pin，按照群主博客指示查找你的 GPIO Pin，把 `修改` 改成你的值(有的时候转化出来的值不止一个，请制作两份文件依次尝试)。
   1. 转到 `CRS` 方法(别翻过头了，数一数大括号，看一眼左下角，确保你还在你的触摸板里。

        ![Sure-CRS](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Sure-CRS.png)

      - 把所有的 Return (XXXX) 和 Return (**ConcatenateResTemplate** (`SBFB`, **XXXX**)) 都修改成 `(SBFB,SBFG)`。
      - **或者**
        1. 清空里面的一堆 If 条件语句（如果有），将里面的 `(SBFB,SBFI)` 改成 `(SBFB,SBFG)`。
        1. 按`编译`键查看是否有错误，删掉发生错误的行(此行必须位于触摸板中，且在报告中夹在在一堆警告⚠️中)。

      ![Return-SBFG](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Return-SBFG.png)

   1. 没有错误那就大功告成了。
1. 例外
   - 有些情况非常极端，需要通过 Windows 来确定你的 IOInterruptSpecifiers 具体方法如下：
     1. 打开设备管理器
     1. 找到 I2C HID 设备
     1. 双击点开，转到 `资源` 页面
     1. 里面的 `IRQ` 就是 IOInterruptSpecifiers

     ![Pin-1024](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Pin-1024.png)

   - 有的 DSDT 里，`SBFB`, `SBFG`, `SBFI` 这些值位于 `CRS` 方法中，与常规的结构有所不同，不过这种情况问题不大，你依然可以继续按教程修改。

     ![In-CRS](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/In-CRS.png)

   - 有的 DSDT 里，`SBFB`, `SBFG`, `SBFI` 这些值不仅设备的根目录里有，`CRS` 方法中也有，与常规的结构大有不同，这种情况就需要你灵活变通理解到底那个起作用，通常情况下修改 `CRS` 方法里的就可以了，如果你多次尝试还是搞不定，请打包求助。
   - 有的 DSDT 更加让人摸不着头脑🧠，除了有 `SBFB`, `SBFG`, `SBFI`，还有 `SBFS`, `SBFF`, `SBFE` 等等，仔细观察你会发现，这些例外里的内容和 `SBFB` 里的内容几乎一摸一样，这就说明他们是和 `SBFB` 是起到相同作用的，只是设备地址不同，因此，如果 `CRS` 方法里如果有大量的条件语句，你只要把 (SBF **(X)**,SBFI) 改成 (SBF **(X)**,SBFG) 即可，假如你的 `CRS` 里只有一个简单的返回 (SBF **(X)**,SBFI)，你则需要创建多个文件，分别尝试不同的 (SBF **(X)**,SBF**G**)。

## 排错

### I. 五国

- 由于五国状况复杂这里只提供一个能减少五国概率的方法，如果还是一样，请拍照发群里
  1. 打开 VoodooI2C.kext⁩ ▸ 右键 ▸ 显示包内容 ▸ ⁨Contents⁩ ▸ Info.plist (用 Xcode, PlistEdit Pro 或者 VS Code 打开 **千万别用 Clover Configurator**) 这里以 VS Code 为例
  1. ⌘ + F 搜索 `<string>VoodooI2CPCIController</string>`
  1. 往下翻一点找到这些

     ```plist
         <string>pci8086,9d60</string>
         <string>pci8086,9d61</string>
         <string>pci8086,9d62</string>
         <string>pci8086,9d63</string>
         <string>pci8086,9de8</string>
         <string>pci8086,9de9</string>
         <string>pci8086,a160</string>
         <string>pci8086,a161</string>
         <string>pci8086,a162</string>
         <string>pci8086,a368</string>
         <string>pci8086,a369</string>
     ```

  1. 在 Windows 里查看你设备使用的是哪一个，删掉其他的，保留你用的

     ![a369](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/a369.png)

### II. 学会使用 maclog 提取日志

![maclog](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/maclog.png)

- 下载完后双击解压
- 双击运行，程序会先调用终端，接着会弹出来 **“魔改过的”** 控制台日志窗口
- 在右上角的搜索栏中搜索 `Voodoo`

  ![Voodoo](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Voodoo.png)

- 将文本复制粘贴到记事本中
- 再在右上角的搜索栏中搜索 **你的设备名** ，比如 `ETPD`, `TPAD`, `TPD0`
- 将文本复制粘贴到记事本中
- 保存好文本文档

### III. 确保你使用了正确 `VoodooI2C` 及附属驱动，不能多也不能少

### IV. 查看 `IORegisteryExplorer` 里 `VoodooI2C` 是否正常加载

   1. 不正常1:

      ![LpssI2C](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/LpssI2C.png)

      - 解决方案：添加 `Clover` 里有屏蔽冲突驱动的补丁

        1. Clover Configurator 法

           ![Disable-CCG](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Disable.png)

        2. Plist 编辑器法

           ![Disable-XCode](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Disable2.png)

        3. 文本法
         <details>
         <summary>（点击此处以展开）</summary>

            ```
                 <key>KextsToPatch</key>
                     <array>
                         <dict>
                             <key>Comment</key>
                             <string>Disable AppleIntelLpssI2C (credit by Coolstar)    </string>
                             <key>Disabled</key>
                             <false/>
                             <key>Find</key>
                             <data>
                             SU9LaXQ=
                             </data>
                             <key>InfoPlistPatch</key>
                             <true/>
                             <key>Name</key>
                             <string>AppleIntelLpssI2C</string>
                             <key>Replace</key>
                             <data>
                             SU9LaXM=
                             </data>
                         </dict>
                         <dict>
                             <key>Comment</key>
                             <string>Disable AppleIntelLpssI2CCOntroller (credit by     Coolstar)</string>
                             <key>Disabled</key>
                             <false/>
                             <key>Find</key>
                             <data>
                             SU9LaXQ=
                             </data>
                             <key>InfoPlistPatch</key>
                             <true/>
                             <key>Name</key>
                             <string>AppleIntelLpssI2CController</string>
                             <key>Replace</key>
                             <data>
                             SU9LaXM=
                             </data>
                         </dict>
                     </array>
            ```

         </details>

      - 暴力删除法：进入 `/System/Library/Extensions/` 删除这两个驱动 **AppleIntelLpssI2C** & **AppleIntelLpssI2CController**
      > `sudo mount -uw /`
      > `sudo rm -rf AppleIntelLpssI2C.kext`
      > `sudo rm -rf AppleIntelLpssI2CController.kext`

   1. 不正常2:

      ![Wrong2](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Wrong2.png)

      - 提取日志并进行分析
      1. 出现如下字样

         ```log
         VoodooGPIOXXX:: pin XXX cannot be used as IRQ
         ```

         **解决方案：** 你转换的 GPIO Pin 超出范围了，换另一个

      1. 出现如下字样

         ```log
         VoodooI2C:: Failed to load
         ```

         **解决方案：** 确认你使用了正确的驱动及附属驱动并且加上了屏蔽冲突驱动的补丁

      1. 七代设备出现如下字样，或者 CPU 占用极高

         ```log
         VoodooI2CControllerDriver::pci8086,9XXX Warning: Error getting bus config, using defaults where necessary
         VoodooI2CControllerDriver::pci8086,9XXX I2C Transaction error details
         VoodooI2CControllerDriver::pci8086,9XXX lost arbitration
         VoodooI2CControllerDriver::pci8086,9XXX I2C Transaction error: 0x0e001000 - aborting
         Request for HID descriptor failed
         Request for HID descriptor failed
         ```

         **解决方案：**

         使用 SSDT-I2CxConf 修补 I2C 速度常量 SSCN 和 FMCN

         使用方法  
              1. 将 [SSDT-I2CxConf.dsl](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/SSDT-I2CxConf.dsl) 另存为编译成 SSDT-I2CxConf.aml  
              2. 在 Clover 中添加如下重命名：

         ![Rename SSCN FMCN](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Rename-SSCN-FMCN.png)

         **如果依旧没有效果**

         删除上方的改动，在你的DSDT的触摸板设备上方加上这个：

         ```asl
         Name (SSCN, Package () { 528, 640, 30 })
         Name (FMCN, Package () { 128, 160, 30 })
         ```

         **像这样：**

         ![7Gen](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/7-SSCN-FMCN.png)

         **如果不理想，再尝试这个：**

         ```asl
         Name (SSCN, Package () { 432, 507, 30 })
         Name (FMCN, Package () { 72, 160, 30 })
         ```

      1. 八代标压设备出现如下字样

         ```log
         VoodooI2CControllerNub::pci8086,aXXX SSCN not implemented in ACPI tables
         VoodooI2CControllerNub::pci8086,aXXX FMCN not implemented in ACPI tables
         VoodooI2CControllerDriver::pci8086,aXXX Warning: Error getting bus config, using defaults where necessary
         VoodooI2CControllerDriver::pci8086,a369 I2C Transaction error details
         VoodooI2CControllerDriver::pci8086,a369 slave address not acknowledged (7bit mode)
         VoodooI2CControllerDriver::pci8086,a369 I2C Transaction error: 0x08800001 - aborting
         Request for HID descriptor failed
         Could not get HID descriptor
         ```

         **解决方案：**

         使用 SSDT-I2CxConf 修补 I2C 速度常量 SSCN 和 FMCN

         使用方法  
              1. 将 [SSDT-I2CxConf.dsl](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/SSDT-I2CxConf.dsl) 另存为编译成 SSDT-I2CxConf.aml  
              2. 在 Clover 中添加如下重命名：

         ![Rename SSCN FMCN](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Rename-SSCN-FMCN.png)

         **如果依旧没有效果**

         删除上方的改动，在你的DSDT的触摸板设备上方加上这个

         ```asl
         Method (SSCN, 0, NotSerialized)
         {
             Return (PKG3 (SSH1, SSL1, SSD1))
         }

         Method (FMCN, 0, NotSerialized)
         {
             Name (PKG, Package (0x03)
             {
                 0x0101,
                 0x012C,
                 0x62
             })
             Return (PKG) /* \_SB_.PCI0.I2C1.FMCN.PKG_ */
         }
         ```

         **然后在 `Scope (_SB.PCI0.I2C1)` 上面加上**

         ```asl
         Method (PKG3, 3, Serialized)
         {
             Name (PKG, Package (0x03)
             {
                 Zero,
                 Zero,
                 Zero
             })
             PKG [Zero] = Arg0
             PKG [One] = Arg1
             PKG [0x02] = Arg2
             Return (PKG) /* \PKG3.PKG_ */
         }
         ```

         **像这样：**

         ![8Gen](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/8-SSCN-FMCN.png)

### V. 尝试**输入密码登陆界面**或者**不带鼠标开机**能不能使用触摸板

- 如果可以，请进入 偏好设置 ▸ 辅助功能 ▸ 鼠标与触控板 ▸ **取消勾选** `有鼠标或无线触控板时忽略内建触摸板`

  ![Ignore](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Ignore.png)

### VI. 驱动放进 `Library/Extensions`，然后重建缓存

   ![L/E](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/L-E.png)

   然后打开终端，输入

   ```Command
   sudo kextcache -i/
   ```

### VII. 尝试睡眠唤醒

### 如果还是不行，请提交一下内容

- 原始 DSDT (Clover F4 的完整 ACPI 表) 一份
- 针对触摸板修改过的 DSDT 一份
- 装了 VoodooI2C 时的 IOREG 一份（文件 -> 另存为）
- maclog 提取的日志一份
- 一份详细的描述
  - 包含：
    1. 笔记本机型
    1. 触摸板型号
    1. 触摸板 BIOS 设备地址
    1. 出厂 Windows 版本和当前 macOS 版本
    1. 使用的 VoodooI2C 版本以及使用的附属驱动名称
    1. 处理器型号
    1. 问题的详细描述

### 最后，祝你早日见到这个感人的界面，用上丝滑的触摸板！

   ![Success1](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Success.png)

   ![Success2](https://raw.githubusercontent.com/ettingshausen/VoodooI2C-PreRelease/master/IMG/Success2.png)

[//]: <> (## 手势问题及解决方案)

## 鸣谢

- [Startpenghubingzhou](https://github.com/penghubingzhou)
- [VoodooI2C 开发组](https://voodooi2c.github.io/#Credits%20and%20Acknowledgments/Credits%20and%20Acknowledgments)
