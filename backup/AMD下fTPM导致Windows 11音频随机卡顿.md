1. Win + R 运行 tpm.msc，查看 TPM 当前是否启用。如果 TPM 已启用，请先在 Windows 设置搜索 "设备加密"，确保没有启用 Bitlocker 硬盘加密功能；然后移除登录帐户所用的 Pin 码。
2. 在设备管理器找到 "受信任的平台模块2.0" ，双击打开属性页，转到"驱动程序" 选项卡，依次点击 "更新驱动程序"，"浏览我的电脑以查找驱动程序"，"让我从计算机上的可用驱动程序列表中选取"，取消勾选"显示兼容硬件"，然后选中 "Advanced Micro Devices Inc."
![image1](https://pub-d3faa1947eb448819cde832afc98290f.r2.dev/blog/2024/08/d7642425d24651988cb50bc6d5168381.jpg)
3. 重启后再次运行 tpm.msc，应当看到 TPM 已经被禁用。
4. 如果需要从错误驱动切换回正确驱动，改回“受信任的平台模块2.0”即可
![image2](https://pub-d3faa1947eb448819cde832afc98290f.r2.dev/blog/2024/08/7b312c66b1b934809cd66e198d66180e.png)

值得一提的是，关闭TPM会导致拳头(Roit Games)的Vanguard出错，酌情考虑使用
参考原贴：https://tieba.baidu.com/p/7731759367?pn=1
卡顿现象参考：https://www.bilibili.com/video/BV1hK4y1H7aJ/