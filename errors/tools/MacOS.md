# MacOS文件已损坏时运行sudo xattr -r -d com.apple.quarantine 报错xattr: \[Errno 1] Operation not permitted: '/Applications/PicGo.app/Contents/MacOS/xxx'
## 原因

这是由于你的命令行工具没有足够的App管理权限所致，需要在隐私与安全性赋予足够命令行工具的权限（App管理的权限）

## 解决方案

左上角苹果图标->系统设置->隐私与安全性->隐私->App管理->添加对应命令行工具权限

此时再次运行
sudo xattr -r -d com.apple.quarantine Applications/xxx.app 
输入密码后即可