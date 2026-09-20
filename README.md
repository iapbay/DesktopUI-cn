# ZeroTier 1.16.1 中文汉化版

ZeroTier 1.16.1 Windows **桌面 GUI 汉化版**，仅汉化界面文本，不修改 ZeroTier 核心功能。

我本人只在 **Windows 11** 上实际测试过，目前这套流程可以正常运行。其他 Windows 版本以及 macOS、Ubuntu 等平台未实际测试，不保证兼容。

## 使用方法

Release 中提供对应平台的 ZIP 压缩包，下载后解压即可。

Windows 用户解压后找到：

```text
zerotier_desktop_ui.exe
```

将其替换到：

```text
C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe
```

建议替换前先备份原版。

### 如果提示“目标文件夹访问被拒绝”

我自己替换时遇到了这个问题。即使使用管理员权限、停止 ZeroTier 相关服务，甚至进入 Windows 安全模式，仍然无法替换文件。

因此开始排查是不是文件本身的权限问题。

#### 1. 先排除 ZeroTier 占用

先退出 ZeroTier GUI，并在任务管理器中检查 ZeroTier 相关进程。

如果仍然无法替换，可以尝试进入 Windows 安全模式后操作。

我的情况在这些操作之后仍然提示：

```text
目标文件夹访问被拒绝
```

所以问题应该不只是 ZeroTier 进程占用。

#### 2. 检查 EXE 本身的权限

尝试对原来的：

```text
C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe
```

取得所有权：

```powershell
takeown /F "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe" /A
```

然后：

```powershell
icacls "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe" /grant "Administrators:F"
```

这一步可以修改 EXE 本身的权限。

但是之后尝试把原来的 EXE 复制到同一个目录进行备份时，仍然无法写入：

```text
C:\Program Files (x86)\ZeroTier\One
```

这时候才发现，问题并不只是 EXE 本身。

#### 3. 检查父文件夹权限

检查：

```powershell
icacls "C:\Program Files (x86)\ZeroTier\One"
```

当时看到的权限类似：

```text
Everyone:(OI)(CI)(RX)
NT AUTHORITY\SYSTEM:(OI)(CI)(F)
```

也就是说，`One` 文件夹本身没有给 Administrators 提供正常的写入权限。

所以即使已经取得 `zerotier_desktop_ui.exe` 本身的所有权，管理员仍然无法在父目录中创建、删除或替换文件。

也就是说，真正的问题是：

```text
zerotier_desktop_ui.exe
        ↓
EXE 本身的权限不是唯一问题
        ↓
C:\Program Files (x86)\ZeroTier\One
        ↓
父文件夹的所有权 / ACL 不允许管理员写入
```

#### 4. 取得 `One` 文件夹的所有权

使用**管理员 PowerShell**执行：

```powershell
takeown /F "C:\Program Files (x86)\ZeroTier\One" /A
```

这里的 `/A` 会把所有权交给 `Administrators` 管理员组。

成功后应该会看到类似：

```text
成功: 此文件(或文件夹): "C:\Program Files (x86)\ZeroTier\One"
现在由管理员组所有。
```

#### 5. 给 Administrators 添加完全控制权限

继续执行：

```powershell
icacls "C:\Program Files (x86)\ZeroTier\One" /grant "Administrators:(OI)(CI)F"
```

然后检查：

```powershell
icacls "C:\Program Files (x86)\ZeroTier\One"
```

应该至少能看到类似：

```text
BUILTIN\Administrators:(OI)(CI)(F)
Everyone:(OI)(CI)(RX)
NT AUTHORITY\SYSTEM:(OI)(CI)(F)
```

如果原本存在：

```text
NT SERVICE\TrustedInstaller:(I)(F)
```

之类的系统权限，不需要为了替换 EXE 而删除它们。

#### 6. 备份原版

父目录权限修改完成后，就可以正常写入目录了。

先备份原版：

```powershell
Copy-Item "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe" "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui_backup.exe"
```

如果这一步成功，说明之前的问题确实出在父目录权限。

#### 7. 替换汉化版

解压 Release 中对应的 ZIP，在解压后的文件夹中找到：

```text
zerotier_desktop_ui.exe
```

然后使用它的**实际文件路径**替换：

```powershell
Copy-Item "你解压后的实际文件路径\zerotier_desktop_ui.exe" "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe" -Force
```

`你解压后的实际文件路径` 只是占位示例，请替换成你自己的实际路径。

替换完成后可以检查：

```powershell
Get-Item "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe" | Select-Object Name,Length,LastWriteTime
```

然后运行：

```powershell
Start-Process "C:\Program Files (x86)\ZeroTier\One\zerotier_desktop_ui.exe"
```

## 其他说明

这个项目只是分享 **ZeroTier 1.16.1 Desktop GUI 的中文汉化版**，以及记录我自己替换文件时遇到的权限问题和解决过程。

权限部分只是我本人这次实际遇到问题后的处理方法，**不代表所有人的情况都一样**。

如果按照上面的流程仍然无法解决，请把自己的具体报错和相关信息整理后**找 AI 提问**。我也只是分享这次遇到的问题和汉化后的 GUI，其他 Windows 权限问题我没有办法保证能够解决。

**本汉化版对应 ZeroTier 1.16.1，其他版本不保证兼容。**
