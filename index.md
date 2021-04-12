# Windows 环境题解与解析

## DCServer

- 配置服务器的主机名，IP 地址，创建要求的用户名与密码

此题需与后面的域控制器部署结合进行

配置 IP 可在服务器管理器中直接点击网卡 IP 处快速跳转修改

使用 `ifconfig` 查看网卡 IP 配置

![Network IP Configuration](images/7e3c87a868a96e93a1a6f5bfeec1c70fa12b27ba9798820bf051fb6a9d83a005.png)  

- 防火墙配置

Win+S 搜索 `高级安全 Windows Denfencer 防火墙`，入站规则中查找 `文件和打印机共享(回显请求 - ICMPv4-In)` 选择阻止连接.

![Firewall Block ICMP Echo](images/9ff54688aaedd22ac164a292fbaf5c5d150cd0fa162da42f7ac2e30f1a361744.png)  

- 安装 Zabbix-agent

正常下一步安装即可，填写服务器 IP 地址的部分如果题中没有，也没办法填写，所以暂时填一个 `127.0.0.1` 就好。

安装结束后，Win+S 搜索服务，找到 Zabbix-Agent 截图即可

![Zabbix-Agent Service](images/01254ea50e245b7c220da5d688117e9dd3f0e291780e949232691721639a6ad5.png)  

- NIC Teaming (链路聚合)

添加网卡后，在`服务器管理器 -> 本地服务器` 找到 `NIC 组合 (NIC Teaming)` 进入，右键网卡,选择`添加到新组`，勾选成员设备，展开高级选项，成组方式选择`静态成组` (即 `IEEE 802.3ad draft v1`) ，此操作无法在虚拟机中完成，这是正常的。

![NIC Teaming](images/1131d034d434d4653b8b353ae62d3962159ef12bb8ce4cad0294e05e76680ef1.png)  

- Active Directory Service 主域控活动目录配置工作任务

安装 AD DS 与 DNS 服务器角色，按要求正常部署域控即可，添加新的名为 ChinaSkills.cn 的 Forest

![systeminfo](images/5cdaf66c95678f568468a98698b5f789e6688c9ca6cbc95c4dffb0c5636e3f60.png)  

英文版请 `systeminfo | find “Domain”`

批量创建新用户，打开记事本，写入 4 行 for 循环脚本

```bat
for /L %%a in (1,1,100) do net user sales%%a ChinaSkills20 /add /domain
for /L %%a in (1,1,5) do net user IT%%a ChinaSkills20 /add /domain
for /L %%a in (1,1,10) do net user F%%a ChinaSkills20 /add /domain
for /L %%a in (1,1,5) do net user Manage%%a ChinaSkills20 /add /domain
```

保存为 useradd.cmd (任意文件名) 在保存位置按住 Shift 右键`在此处打开 Powershell`

执行脚本 `.\useradd.cmd`

在服务器管理器中选择 `AD DS`，右键服务器打开 `Active Directory 用户和计算机` 管理工具

进入 `Users` 组织单位，新建 4 个组，并将用户加入组

加入域后使用`组策略管理`批量修改组策略

修改域控的组策略使用默认创建的 `Default Domain Controllers Policy`，右键编辑

依次展开 `计算机配置 -> 策略 -> Windows 设置 -> 安全设置 -> 本地策略 -> 用户权限分配`

找到 `允许本地登录` ( `Allow log on locally` ),增加 `IT` 组

![Log on policy](images/3ef2d93026f8abb2496d6f1eca7b990e7da1d8b3d87aa645bd283a8d7076629d.png)  

创建 `ChinaSkills20` 用户，但密码不满足复杂度要求，因此修改组策略关闭密码复杂度要求

位置为`计算机配置 -> 策略 -> Windows 设置 -> 安全设置 -> 账户策略 -> 密码策略` 将密码复杂度要求关闭

使用 `gpupdate /force` 刷新组策略

打开 `Active Directory 用户和计算机` 管理工具，添加 ChinaSkills20 用户进 `Users` 组织单位，并在 `隶属于` 选项卡中将其所属组添加 `Enterprise Admin` 与 `Domain Admins` 组，即可自动拥有 GPO 管理权限

![User Group](images/dc4ec33a9d114d17795dac07aa9164e7dde71cf4e739f140989a74b3ba9c691d.png)  

- 设置漫游文件

在 `Active Directory 用户和计算机` 中，双击要修改的用户，在 `配置文件` 选项卡中，修改用户配置文件路径

![User Profile](images/47aa890f4c6efa863cbdc258886e782a30ea51bd263922072a9d62644177b464.png)  

![Verify the Configuration](images/4e6a96248798a2ab9767a6c0763591d2eacb319924709642d11f394cbdb0d2e7.png)  

当然一个一个修改是不行的

- 开启本地与域用户登录操作日志审计记录

对所有用户修改组策略，修改 `Default Domain Policy`

`计算机设置 -> 策略 -> Windows 设置 -> 安全设置 -> 本地策略 -> 审核策略`

开启 `审核登录时间` 与 `审核账户登录事件` ，将成功与失败都勾选。