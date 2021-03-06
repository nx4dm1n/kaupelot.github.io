---
layout: post
title:  "iOS逆向之使用class-dump来查看app的头文件"
date:   2016-05-07 08:04:20
categories: iosre kaupelot
---
上一篇文章我们学习了怎么在reveal里面查看别人的app的结构,也就可以看到很多自定义的类名称.这篇文章我们将了解怎么将ipa的类文件导出,直接查看相应的类方法名.
class-dump 官网地址：[这里](http://stevenygard.com/)

我这里下载的是 class-dump-3.5.dmg 版本的。双击.dmg 文件，将 ![](http://img.blog.csdn.net/20140914172328750) 拉倒 /usr / local / bin 目录下，这样就可以在终端使用 class-dump 命令了。

这里我演示dump系统自带的计算器，导出它的头文件。

命令如下：

    **class-dump -H /Applications/Calculator.app -o /Users/Rio/Desktop/calculate\ heads**

解释：

/Applications/Calculator.app 是计算器app的路径

/Users/Rio/Desktop/calculate\ heads 是存放dump出来头文件的文件夹路径

    class-dump 3.5 (64 bit)
    Usage: class-dump [options] <mach-o-file>
    
      where options are:
            -a             show instance variable offsets
            -A             show implementation addresses
            --arch <arch>  choose a specific architecture from a universal binary (ppc, ppc64, i386, x86_64)
            -C <regex>     only display classes matching regular expression
            -f <str>       find string in method name
            -H             generate header files in current directory, or directory specified with -o
            -I             sort classes, categories, and protocols by inheritance (overrides -s)
            -o <dir>       output directory used for -H
            -r             recursively expand frameworks and fixed VM shared libraries
            -s             sort classes and categories by name
            -S             sort methods by name
            -t             suppress header in output, for testing
            --list-arches  list the arches in the file, then exit
            --sdk-ios      specify iOS SDK version (will look in /Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS<version>.sdk
            --sdk-mac      specify Mac OS X version (will look in /Developer/SDKs/MacOSX<version>.sdk
            --sdk-root     specify the full SDK root path (or use --sdk-ios/--sdk-mac for a shortcut)

运行命令，可以看到已经dump出头文件了，如下所示：

![](http://img.blog.csdn.net/20140914173142843)

   class-dump 虽然非常有用，但有时我们会发现 class-dump 执行失败，无法得到我们想要的 .h 文件，或者 .h 文件的内容是加密的密文。出现这种现象的原因是：class-dump 额作用对象必须是未经加密的可执行文件，而**从App Store 下载的 App 都是经过签名加密的**，可执行文件被加上了一层“壳”。可以使用 AppCrackr 来自动砸壳。

---------- 2016.01.07 更新 ----------------

从App Store 上下载的 App 我这里是通过 Clutch 这个来砸壳的。

先 ssh 到 iPhone 上，可能会遇到这样的问题：

MacBook:~ aaron$ ssh root@172.17.24.70

ssh: connect to host 172.17.24.70 port 22: Connection refused

不急，去 Cydia 安装 OpenSSH 即可，安装完后再次 ssh 到 iPhone 发现已经可以了。

去网上下载 Clutch 这个软件，改名为 clutch (不改也可以，这里改名只是为了以后敲命令的时候老是要第一个字母大写，麻烦！)，然后 copy 到 iPhone 的 /usr/bin/ 目录下面。

    MacBook:~ aaron$ scp /Users/aaron/Documents/kugou/doc/越狱开发/hack/clutch root@172.17.24.70:/usr/bin/

    root@172.17.24.70's password: 

    clutch                                        100%  915KB 914.8KB/s   00:00 

呐，已经 ok 了，接下来修改 clutch 为最高权限 777

    iPhone:/usr/bin root# chmod 777 ./clutch

查看是否修改成功，

    iPhone:/usr/bin root# ls -al | grep clutch

    -rwxrwxrwx 1 root   wheel  936752 Jan  7 16:02 clutch

可以看到已经是最高权限了。

接下来是使用 clutch 了，网上说还要安装 Mobile Terminal 啥啥啥的一律不用管它，直接开干。

    iPhone:/usr/bin root# clutch -i 

输入上面的命令就可以看到一大堆可以砸壳的应用：

    Installed apps:

    1) <AlipayWallet bundleID: com.alipay.iphoneclient>

    2) <WeChat bundleID: com.tencent.xin>

    3) <yidian bundleID: com.yidian.zixun>

    4) <YoukuiPhone bundleID: com.youku.YouKu>

    5) <嘀嘀打车 bundleID: com.xiaojukeji.didi>

    6) <Taobao4iPhone bundleID: com.taobao.taobao4iphone>

    7) <NewsBoard bundleID: com.netease.news>

    8) <iLady bundleID: cn.com.modernmedia.imodernlady>

    9) <netdisk_iPhone bundleID: com.baidu.netdisk>

    ..........

输入你想要砸壳的应用 bundle identifier 进行砸壳

    iPhone:/usr/bin root# clutch -d com.yidian.zixun

成功后你就可以看到已经破解完后的路径：

    DONE: /private/var/mobile/Documents/Dumped/com.yidian.zixun-iOS7.0-(Clutch-2.0 RC2).ipa

之后，copy .ipa 文件到你的电脑上，

    MacBook:~ aaron$ scp root:172.17.24.70:/private/var/mobile/Documents/Dumped/com.yidian.zixun-iOS7.0-(Clutch-2.0 RC2).ipa ~/Desktop/

    -bash: syntax error near unexpected token `('

诶，有错误，识别不了`('，那么改一下.ipa 名字后在 scp 吧

iPhone:/private/var/mobile/Documents/Dumped root# mv com.yidian.zixun-iOS7.0-\(Clutch-2.0\ RC2\).ipa yidian.ipa

再 copy 到电脑桌面：

    MacBook:~ aaron$ scp root@172.17.24.70:/private/var/mobile/Documents/Dumped/yidian.ipa ~/Desktop/yidian.ipa

    root@172.17.24.70's password: 

    yidian.ipa                                    100%   25MB   1.5MB/s   00:16

至此，砸壳就已经完成了，现在可以用 class-dump 把头文件 dump 出来了。cheer！

-------------------------------- 分割线 -----------------------------------

**安装 apt-get ：**

打开cydia –管理—设置—选择“开发者”—完成, 搜索apt，安装APT 0.6 Transitional,它会安装四五个其它依赖包，都不大.

安装后就能用apt-get了，例如apt-get install netstat, apt-get install ps等。

-------------------------------- 分割线 -----------------------------------

在新建工程之前还有一个很重要的事情要做，安装打印日志用的 log。

# 看log

    apt-get install socat

安装完后，输入以下命令看log

    socat - UNIX-CONNECT:/var/run/lockdown/syslog.sock

    >watch

安装 socat 的时候出现这个错误提示的话：

    E: Could not get lock /var/lib/dpkg/lock - open (35: Resource temporarily unavailable) 

    E: Unable to lock the administration directory (/var/lib/dpkg/), is another process using it?

可以尝试用[这里](http://askubuntu.com/questions/15433/unable-to-lock-the-administration-directory-var-lib-dpkg-is-another-process)的办法解决。（先试一下这个帖子提到的其他办法，最后不得已再使用如下办法）

![](http://img.blog.csdn.net/20160108170233736)

为了在使用 iosOpenDev 开发时调试更加便利，机子还需要安装这样一些环境：

# 这个是使用iosOpenDev开发时，xcode把deb包安装到手机时，需要手机先具备的环境条件

    apt-get install coreutils diskdev-cmds file-cmds system-cmds com.saurik.substrate.safemode mobilesubstrate preferenceloader

若出现“Package diskdev-cmds is not available, but is referred to by another package. This may mean that the package is missing, has been obsoleted, or is only available from another source”这样的安装错误的时候，执行如下命令后再次安装过，

    sudo apt-get update && && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt-get install packagename

(or)

    apt-get update && apt-get upgrade -y && apt-get dist-upgrade -y && apt-get install packagename

参考来源：http://www.blackmoreops.com/2014/12/13/fixing-error-package-packagename-not-available-referred-another-package-may-mean-package-missing-obsoleted-available-another-source-e-pa/

有些机子环境，安装不了netstat、socat等环境，于是就不能通过iosOpenDev来拷贝dylib，也不能用socat来查看log。这时，解决办法是：

1. 在iosOpenDev编译完后，手动把dylib拷贝到手机：

        scp hookKTV.dylib root@192.168.2.42:/Library/MobileSubstrate/DynamicLibraries/

2. 把原本的NSLog换成写文件，把hook信息写到文件，之后用手机助手之类的工具导出文件。

3. 重启springboard：

        killall SpringBoard

-------------------------------- 分割线 -----------------------------------

以上所有环境搞定之后就可以新建一个工程了：

![](http://img.blog.csdn.net/20160107204901359)

选择Logos Tweak ，然后一路next 到创建好新工程。

新建好的 .xm 文件中会有一个一个提示，要你去 /opt/iOSOpenDev/lib/ 目录下拉一个libsubstrate.dylib 动态链接库到工程，因为我们这里是要hook SpringBoard，要弹出一个弹出框，所以需要再导入 UIKit.framework.

    #error iOSOpenDev post-project creation from template requirements (remove these lines after completed) -- \

Link to libsubstrate.dylib: \

(1) go to TARGETS > Build Phases > Link Binary With Libraries and add /opt/iOSOpenDev/lib/libsubstrate.dylib \

(2) remove these lines from *.xm files (not *.mm files as they're automatically generated from *.xm files)

然后在 .xm 里面编写代码如下：

    // Logos by Dustin Howett
    // See http://iphonedevwiki.net/index.php/Logos
    #import <UIKit/UIKit.h>
    
    %hook SpringBoard
    
    - (void)applicationDidFinishLaunching:(id)application {
        %orig;
        
        NSLog(@"hook SpringBoard:这是测试log输出。。。。");
        
        UIAlertView *alert =
        [[UIAlertView alloc]initWithTitle:@"welcome" message:@"hellowrold" delegate:nil cancelButtonTitle:@"thanks" otherButtonTitles:nil];
        [alert show];
        [alert release];
    }
    
    %end
    
    %hook AppDelegate
    
    - (void)applicationDidEnterBackground:(UIApplication *)application{
        %orig;
        
        NSLog(@"hook AppDelegate:这是测试log输出。。。。");
        UIAlertView *alert =
        [[UIAlertView alloc]initWithTitle:@"hook" message:@"hellowrold" delegate:nil cancelButtonTitle:@"thanks" otherButtonTitles:nil];
        [alert show];
        [alert release];
    }
    
    %end

编译的时候有可能会遇到如下的编译错误：

![](http://img.blog.csdn.net/20160506214025136)

解决办法是：修改 TARGETS - Build Settings - Code Signing - Provisioning Profile 为 **
iOS Team Provisioning Profile: *** 就好了。

ssh 到手机，运行

    socat - UNIX-CONNECT:/var/run/lockdown/syslog.sock

    >watch

这个命令，准备查看输出 log。

电脑和越狱设备连接到同一个网络下，然后查看设备的 wifi 地址，记下来，然后填写到 TARGETS - Build Settings - User-Defined - iOSOpenDevDevice 里面去，比如我这里是 192.168.2.42 ,为的是等下 Profiling 的时候把动态链接库 HookTest.dylb copy 到           root@192.168.2.42:/Library/MobileSubstrate/DynamicLibraries/ 目录下。

假如你以上环境都能成功搭建的话，那接下来就简单了，点击 XCode -> Product -> Build For -> Profiling ，XCode 就会帮你把动态链接库 HookTest.dylib （我新建的工程叫HookTest，所以这里是HookTest.dylib）和 .plist 文件copy 到 root@192.168.2.42:/Library/MobileSubstrate/DynamicLibraries/ 这个路径下了，然后重启springboard：

运行命令 killall SpringBoard，就可以看到手机在启动的时候就会去运行刚刚的 HookTest.dylib 然后弹出一个弹出框了。copy 进去的 .plist 的作用是在hook的时候系统会根据 .plist 去过滤 hook 那个应用，如果 .plist 里面啥过滤条件都不写的话是默认所有应用都会去加载刚刚的copy进去的动态库的，

，比如说你现在hook 应用启动时的 

    <del>- (void)applicationDidEnterBackground:(UIApplication *)application</del>

这个方法的话，如果.plist里面不指定是哪个应用去加载HookTest.dylib 的话，默认所有应用启动的时候都会去加载 HookTest.dylib 动态库的。

Profiling 完后查看了一下，Library/MobileSubstrate/DynamicLibraries 这个目录并没有生成，而是把工程打包成 .deb 文件然后 copy 到越狱设备的 iOSOpenDevPackages 目录下，比如我生成的是 

com.aaron.hooksb_1.0-1_iphoneos-arm.deb 是这个，Profiling 完之后，iphone 会自动重启（kill SpringBoard）然后执行hook代码。

用 scp 命令可以将设备里面的文件拷贝到电脑里查看：

    scp root@192.168.2.42:/var/root/iOSOpenDevPackages/com.aaron.hooksb_1.0-1_iphoneos-arm.deb ~/Desktop/

NOTE：在 Profiling 可能遇到的问题： 

1、Permission denied (publickey,password,keyboard-interactive) 

相关的解决方法是：

第一步：拷贝公钥到 iPhone 的 ~/home/userName 目录（没有该目录新建即可，其实我觉得应该是iPhone的任何目录都行的，其目的都是为了第二步）

    scp ~/.ssh/id_rsa.pub serverUsername@host.com:/home/serverUsername

第二步：把公钥的key写到 ~/.ssh/authorized_keys （同样没有该目录新建）

    cat id_rsa.pub >> ~/.ssh/authorized_keys

然后再次 Profiling 就编译通过了。

参考链接：[这里](http://www.linuxquestions.org/questions/linux-networking-3/ssh-permission-denied-publickey-password-keyboard-interactive-405049/)

2、Permission denied，Failed to create directory /var/root/iOSOpenDevPackages on device 192.168.2.42 Command /bin/sh failed with exit code 255。

问题看起来应该是跟 1 一样的。网上找到了一个更优雅的解决方式：直接在 MacBook 命令行上运行命令：iosod sshkey -h 192.168.2.42 ，然后再次 Profiling 就好了。

参考链接：[这里](http://www.cnblogs.com/wangshengl9263/p/3464953.html)

假如你以上环境安装出错的话，我最好建议你Google一下，当然你也可以选择每次修改之后都手动copy到 root@192.168.2.42:/Library/MobileSubstrate/DynamicLibraries/ 目录下，明明可以只是点击一下按钮就可以搞定的事为什么每次都要手动去copy一下呢，那样不觉得很蛋疼吗？

ok，就到这里了。现在大家可以自由自在的根据类名去查看别人写了些什么样的类方法了,是不是觉得只看到类方法的名称,还是差点意思呢?没事,下一次咱一起看看怎么用hopper来查看这些类方法是怎么写的.
本文非原创,作为基础教学,仅供参考.附带:使用class-dump-z

为了了解支付宝 app 的源码结构，我们可以使用 class-dump-z 工具来分析支付宝二进制。

## 下载配置 `class_dump_z`

前往 [[https://code.google.com/p/networkpx/wiki/class_dump_z](https://code.google.com/p/networkpx/wiki/class_dump_z)](https://code.google.com/p/networkpx/wiki/class_dump_z) ，下载 tar 包，然后解压配置到本地环境   

    $ tar -zxvf class-dump-z_0.2a.tar.gz  
    $ sudo cp mac_x86/class-dump-z /usr/bin/  

## class_dump 支付宝 App

    $ class-dump-z Portal > Portal-dump.txt  
    
    @protocol XXEncryptedProtocol_10764b0  
    -(?)XXEncryptedMethod_d109df;  
    -(?)XXEncryptedMethod_d109d3;  
    -(?)XXEncryptedMethod_d109c7;  
    -(?)XXEncryptedMethod_d109bf;  
    -(?)XXEncryptedMethod_d109b8;  
    -(?)XXEncryptedMethod_d109a4;  
    -(?)XXEncryptedMethod_d10990;  
    -(?)XXEncryptedMethod_d1097f;  
    -(?)XXEncryptedMethod_d10970;  
    -(?)XXEncryptedMethod_d10968;  
    -(?)XXEncryptedMethod_d10941;  
    -(?)XXEncryptedMethod_d10925;  
    -(?)XXEncryptedMethod_d10914;  
    -(?)XXEncryptedMethod_d1090f;  
    -(?)XXEncryptedMethod_d1090a;  
    -(?)XXEncryptedMethod_d10904;  
    -(?)XXEncryptedMethod_d108f9;  
    -(?)XXEncryptedMethod_d108f4;  
    -(?)XXEncryptedMethod_d108eb;  
    @optional  
    -(?)XXEncryptedMethod_d109eb;  
    @end  

查看得到的信息是加过密的，这个加密操作是苹果在部署到 app store时做的，所以我们还需要做一步解密操作。

## 使用 Clutch 解密支付宝 App

### 下载 Clutch

iOS7 越狱后的 Cydia 源里已经下载不到 Clutch 了，但是我们可以从网上下载好推进 iPhone   

地址：[Clutch 传送门](https://github.com/KJCracks/Clutch-dl/releases)

### 查看可解密的应用列表

    root# ./Clutch   
    
    Clutch-1.3.2  
    usage: ./Clutch [flags] [application name] [...]  
    Applications available: 9P_RetinaWallpapers breadtrip Chiizu CodecademyiPhone FisheyeFree food GirlsCamera IMDb InstaDaily InstaTextFree iOne ItsMe3 linecamera Moldiv MPCamera MYXJ NewsBoard Photo Blur Photo Editor PhotoWonder POCO 相机 Portal QQPicShow smashbandits Spark tripcamera Tuding_vITC_01 wantu WaterMarkCamera WeiBo Weibo 

### 解密支付宝 App

    root# ./Clutch Portal  
    
    Clutch-1.3.2  
    Cracking Portal...  
    Creating working directory...  
    Performing initial analysis...  
    Performing cracking preflight...  
    dumping binary: analyzing load commands  
    dumping binary: obtaining ptrace handle  
    dumping binary: forking to begin tracing  
    dumping binary: successfully forked  
    dumping binary: obtaining mach port  
    dumping binary: preparing code resign  
    dumping binary: preparing to dump  
    dumping binary: ASLR enabled, identifying dump location dynamically  
    dumping binary: performing dump  
    dumping binary: patched cryptid  
    dumping binary: writing new checksum  
    Censoring iTunesMetadata.plist...  
    Packaging IPA file...  
    
    compression level: 0  
        /var/root/Documents/Cracked/支付宝钱包-v8.0.0-(Clutch-1.3.2).ipa  
    
    elapsed time: 7473ms  
    
    Applications Cracked:   
    Portal  
    
    Applications that Failed:  
    
    Total Success: 1 Total Failed: 0  

### 导出已解密的支付宝 App

从上一步骤得知，已解密的 ipa 位置为：`/var/root/Documents/Cracked/支付宝钱包-v8.0.0-(Clutch-1.3.2).ipa`
将其拷贝到本地去分析

## `class_dump` 已解密的支付宝 App

解压 .ipa 后，到支付宝钱包-`v8.0.0-(Clutch-1.3.2)/Payload/Portal.app` 目录下，class_dump 已解密的二进制文件
`$ class-dump-z Portal > ~/Portal-classdump.txt`

这回就可以得到对应的信息了：   

    @protocol ALPNumPwdInputViewDelegate <NSObject>  
    -(void)onPasswordDidChange:(id)onPassword;  
    @end  
    
    @protocol ALPContactBaseTableViewCellDelegate <NSObject>  
    -(void)shareClicked:(id)clicked sender:(id)sender;  
    @end  
    
    @interface MMPPayWayViewController : XXUnknownSuperclass <SubChannelSelectDelegate, UITableViewDataSource, UITableViewDelegate, CellDelegate, UIAlertViewDelegate> {  
    @private  
        Item* channelSelected;  
        BOOL _bCheck;  
        BOOL _bOpenMiniPay;  
        BOOL _bNeedPwd;  
        BOOL _bSimplePwd;  
        BOOL _bAutopayon;  
        BOOL _bHasSub;  
        BOOL _bFirstChannel;  
        BOOL _bChangeSub;  
        BOOL _bClickBack;  
        UITableView* _channelListTableView;  
        NSMutableArray* _channelListArray;  
        NSMutableArray* _subChanneSelectedlList;  
        NSMutableArray* _unCheckArray;  
        UIButton* _saveButton;  
        UILabel* _tipLabel;  
        MMPPasswordSwichView* _payWaySwitch;  
        MMPPopupAlertView* _alertView;  
        UIView* _setView;  
        int _originalSelectedRow;  
        int _currentSelectedRow;  
        NSString* _statusCode;  
        ChannelListModel* _defaultChannelList;  
    }  
    @property(assign, nonatomic) BOOL bClickBack;  
    @property(retain, nonatomic) ChannelListModel* defaultChannelList;  
    @property(retain, nonatomic) NSString* statusCode;  
    @property(assign, nonatomic) int currentSelectedRow;  
    @property(assign, nonatomic) int originalSelectedRow;  
    @property(retain, nonatomic) UIView* setView;  
    @property(retain, nonatomic) MMPPopupAlertView* alertView;  
    @property(retain, nonatomic) MMPPasswordSwichView* payWaySwitch;  
    @property(assign, nonatomic, getter=isSubChannelChanged) BOOL bChangeSub;  
    @property(assign, nonatomic) BOOL bFirstChannel;  
    @property(assign, nonatomic) BOOL bHasSub;  
    @property(assign, nonatomic) BOOL bAutopayon;  
    @property(assign, nonatomic) BOOL bSimplePwd;  
    @property(assign, nonatomic) BOOL bNeedPwd;  
    @property(assign, nonatomic) BOOL bOpenMiniPay;  
    @property(assign, nonatomic) BOOL bCheck;  
    @property(retain, nonatomic) UILabel* tipLabel;  
    @property(retain, nonatomic) UIButton* saveButton;  
    @property(retain, nonatomic) NSMutableArray* unCheckArray;  
    @property(retain, nonatomic) NSMutableArray* subChanneSelectedlList;  
    @property(retain, nonatomic) NSMutableArray* channelListArray;  
    @property(retain, nonatomic) UITableView* channelListTableView;  
    -(void).cxx_destruct;  
    -(void)subChannelDidSelected:(id)subChannel;  
    -(void)switchCheckButtonClicked:(id)clicked;  
    -(void)checkboxButtonClicked:(id)clicked;  
    -(void)onCellClick:(id)click;  
    -(void)showSubChannels;  
    -(void)tableView:(id)view didSelectRowAtIndexPath:(id)indexPath;  
    -(id)tableView:(id)view cellForRowAtIndexPath:(id)indexPath;  
    -(int)tableView:(id)view numberOfRowsInSection:(int)section;  
    -(float)tableView:(id)view heightForRowAtIndexPath:(id)indexPath;  
    -(int)numberOfSectionsInTableView:(id)tableView;  
    -(void)setTableViewFootView:(id)view;  
    -(void)setTableViewHeaderView:(id)view;  
    -(id)tableView:(id)view viewForHeaderInSection:(int)section;  
    -(id)tableView:(id)view viewForFooterInSection:(int)section;  
    -(float)tableView:(id)view heightForHeaderInSection:(int)section;  
    -(float)tableView:(id)view heightForFooterInSection:(int)section;  
    -(void)alertView:(id)view clickedButtonAtIndex:(int)index;  
    -(void)clickSave;  
    -(void)netWorkRequestWithPwd:(id)pwd;  
    -(void)setPayWaySwitchStates:(id)states;  
    -(void)changePayWaySwitch:(id)aSwitch;  
    -(void)scrollToSelectedRow;  
    -(void)didReceiveMemoryWarning;  
    -(void)viewDidLoad;  
    -(void)applicationEnterBackground:(id)background;  
    -(void)dealloc;  
    -(void)goBack;  
    -(BOOL)isChannelsSetChanged;  
    -(id)subChannelCode:(int)code;  
    -(id)subChannelDesc:(int)desc;  
    -(id)initWithDefaultData:(id)defaultData;  
    -(id)initWithNibName:(id)nibName bundle:(id)bundle;  
    -(void)commonInit:(id)init;  
    @end  

## 分析支付宝源码片段

### 使用了 @private 关键字限制成员访问权限

但是实际上，在 Objective-C 编程中，使用 @private 连 Keypath 访问都拦不住的

### 抛出了冗长的成员对象

这非常有利分析程序结构

## 进一步思考

1）如何利用 class-dump 结果，结合 cycript 进行攻击呢？   

2）class-dump-z 如此强大，有什么方法可以减少暴露的信息吗？

接下来的博文将针对上面的思考，继续总结～