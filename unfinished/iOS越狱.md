# iOS 越狱

## 通过 USB 建立 ssh 链接

安装 usbmuxd

```shell
brew install usbmuxd
```

端口映射

```shell
# 把远程22端口映射到本地的2222端口上
iproxy 2222 22
```

建立 ssh 链接

```
ssh -p 2222 root:localhost
```

## 加载动态库

```c
#define SBSERVPATH "/System/Library/PrivateFrameworks/SpringBoardServices.framework/SpringBoardServices"

    int ret;

    if (argc < 2) {
        fprintf(stderr, "Usage: %s com.application.identifier \n", argv[0]);
        return -1;
    }
    
    void *sbserv = dlopen(SBSERVPATH, RTLD_LAZY);
    int (*SBSLaunchApplicationWithIdentifier)(CFStringRef identifier, Boolean suspended) = dlsym(sbserv, "SBSLaunchApplicationWithIdentifier");
    CFStringRef (*SBSApplicationLaunchingErrorString)(int ret) = dlsym(sbserv, "SBSApplicationLaunchingErrorString");
    
    CFStringRef identifier = CFStringCreateWithCString(kCFAllocatorDefault, argv[1], kCFStringEncodingUTF8);
    assert(identifier != NULL);
    
    ret = SBSLaunchApplicationWithIdentifier(identifier, FALSE);
    
    if (ret != 0) {
        fprintf(stderr, "Couldn't open application: %s. Reason: %i, ", argv[1], ret);
        CFShow(SBSApplicationLaunchingErrorString(ret));
    }
    dlclose(sbserv);
    CFRelease(identifier);
```

# vim 设置

cydia 里面找到 vim 源 `Vi IMproved`，下载后就可以使用，但是使用中发现键盘映射不对，上下左右键有问题，到 `~/.vimrc` 下加上下面两行

```
set nocompatible
set backspace=2
```

# 没有 ps 等命令

cydia 里面安装 `adv-cmds`

# NSUserDefault

##### NSUserDefault 是 Fundation 库的一部分，所以依然可用，只不过数据 plist 文件保存在 `~/Library/Preferences/` 目录下，而不是沙盒下。

# 查看 plist 内容

```
plutil aa.plist
```

# tail 命令

The "tail" command is in the package "Core Utilities" (coreutils) in the Cydia/Telesphoreo repository.


