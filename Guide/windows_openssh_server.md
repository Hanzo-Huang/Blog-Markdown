# Windows安装OpenSSH服务并启用

> Environment: Windows Server 2022

1. 在`开始`菜单点击`设置`，选择应用，点击`可选功能`

![windows_openssh_server_1](./windows_openssh_server.assets/windows_openssh_server_1.png)

2. 点击`添加功能`

![windows_openssh_server_2](./windows_openssh_server.assets/windows_openssh_server_2.png)

3. 搜索`openssh`，点击`OpenSSH服务器`并安装

![windows_openssh_server_3](./windows_openssh_server.assets/windows_openssh_server_3.png)

4. 在`开始`菜单栏搜索`服务`，点击打开

![windows_openssh_server_4](./windows_openssh_server.assets/windows_openssh_server_4.png)

5. 找到`OpenSSH SSH Server`，双击进入

![windows_openssh_server_5](./windows_openssh_server.assets/windows_openssh_server_5.png)

6. 将`启动类型`改为`自动`，并点击`启动`按钮

![windows_openssh_server_6](./windows_openssh_server.assets/windows_openssh_server_6.png)

配置成功，接下来便用其他设备远程ssh连接即可
