制作 macOS 平台可启动 USB 安装盘

1. 先在 app store 下载 macOS
2. 执行以下命令
```shell
sudo /Applications/Install\ macOS\ Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/mac_bootable  --applicationpath /Applications/Install\ macOS\ Sierra.app
```
`mac_bootable`是USB盘的名称.
USB盘容量>=8G

3. 重启开始用优盘引导
4. 安装时如果说文件已经损坏，需删除优盘上Sierra.app内的一个plist文件。
5. 在线校验如果出错可能是机器时间不对，可以与 time.apple.com 同步一下时间。

参考：
优盘的format格式，及 mac 启动选项
http://www.techrepublic.com/blog/apple-in-the-enterprise/how-to-create-a-bootable-usb-to-install-os-x/
