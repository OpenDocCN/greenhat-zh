# 第一章。通用工具

![通用工具](img/00001.jpg)

在任何编程语言中，脚本都是解决频繁执行任务的解决方案。如果你发现自己在想：“难道不能让一个机器人或训练有素的猴子来做这项工作吗？”那么使用 Ruby 进行脚本编写可能就是下一个最佳选择。为频繁执行的任务编写脚本可以使你的工作和计算体验尽可能高效。谁不想用更少的时间和精力完成工作呢？当你浏览这些示例时，我鼓励你写下自己脚本的想法。一旦你完成这本书，你可能会有一份想要编写的脚本清单，或者至少是一些对我脚本的有用修订。你准备好了吗？让我们开始吧！

# 检查已更改的文件

## 检查已更改的文件

### changedFiles.rb

此脚本的目的是验证文件完整性。虽然听起来用途很普通，但其应用范围很广：如果你无法信任你电脑上的文件内容，那么你也无法信任你的电脑。你能知道是否有恶意蠕虫或病毒修改了你的系统中的文件吗？如果你认为你的杀毒软件已经为你提供了保护，那么再想想——大多数杀毒软件只检查已知的病毒及其签名。文件完整性验证在日常的实际任务中得到了广泛应用，例如数字取证和追踪恶意逻辑的行为。下面展示了一种跟踪文件完整性的方法。

## 代码

``require 'find' require 'digest/md5'   unless ARGV[0] and File.directory?(ARGV[0])     puts "\n\n\n 您需要指定一个根目录：  changedFiles.rb <directory>\n\n\n"     exit end  ![](img/00002.jpg) root = ARGV[0]  oldfile_hash = Hash.new  newfile_hash = Hash.new  file_report = "#{root}/analysis_report.txt"  file_output = "#{root}/file_list.txt"  oldfile_output = "#{root}/file_list.old"  ![](img/00003.jpg) if File.exists?(file_output)    File.rename(file_output, oldfile_output)    File.open(oldfile_output, 'rb') do |infile|      while (temp = infile.gets)        line = /(.+)\s{5,5}(\w{32,32})/.match(temp)        puts "#{line[1]}  --->  #{line[2]}"        oldfile_hash[line[1]] = line[2]      end    end end  ![](img/00004.jpg) Find.find(root) do |file|      next if /^\./.match(file)      next unless File.file?(file)      begin          newfile_hash[file] = Digest::MD5.hexdigest(File.read(file))      rescue          puts "Error reading #{file} --- MD5 hash not computed."      end  end   report = File.new(file_report, 'wb')  changed_files = File.new(file_output, 'wb')   newfile_hash.each do |file, md5|    changed_files.puts "#{file}     #{md5}"  end  ![](img/00005.jpg)  newfile_hash.keys.select { |file| newfile_hash[file] == oldfile_hash[file]  }.each do |file|     newfile_hash.delete(file)     oldfile_hash.delete(file)  end  ![](img/00006.jpg) newfile_hash.each do |file, md5| **`    report.puts "#{oldfile_hash[file] ? "Changed" : "Added"} file: #{file}`** **`#{md5}"`**     oldfile_hash.delete(file)  end  ![](img/00007.jpg) oldfile_hash.each do |file, md5|     report.puts "Deleted/Moved file: #{file}     #{md5}"  end   report.close  changed_files.close``

## 运行代码

通过以下命令执行此脚本：

``**`ruby changedFiles.rb`** *`/path/to/check/`*``

您可以添加多个目录进行爬取，但子目录将自动进行验证。脚本将自动确定目录是否存在，并将其添加到爬虫的队列中。

## 结果

该脚本最初将生成两个独立的文件（*changed.files 和 file_report.txt*）。这两个文件都将包含脚本扫描的所有文件的名称和 MD5 哈希：

`添加文件: fileSplit.rb d79c592af618266188a9a49f91fe0453 添加文件: fileJoin.rb 5aedfe682e300dcc164ebbdebdcd8875 添加文件: win32RegCheck.rb c0d26b249709cd91a0c8c14b65304aa7 添加文件: changedFiles.rb c2760bfe406a6d88e04f8969b4287b4c 添加文件: encrypt.rb 08caf04913b4a6d1f8a671ea28b86ed2 添加文件: decrypt.rb 90f68b4f65bb9e9a279cd78b182949d4 添加文件: file_report.txt d41d8cd98f00b204e9800998ecf8427e 添加文件: changed.files d41d8cd98f00b204e9800998ecf8427e 添加文件: test.txt a0cbe4bbf691bbb2a943f8d898c1b242 添加文件: test.txt.rsplit1 35d5b2e522160ce3b3b98d2d4ad2a86e 添加文件: test.txt.rsplit2 a65dde64f16a4441ff1619e734207528 添加文件: test.txt.rsplit3 264b40b40103a4a3d82a40f82201a186 添加文件: test.txt.rsplit4 943600762a52864780b9b9f0614a470a 添加文件: test.txt.rsplit5 131c8aa7155483e7d7a999bf6e2e21c0 添加文件: test.txt.rsplit6 1ce31f6fbeb01cbed6c579be2608e56c`

在脚本运行第二次之后，根目录下将出现三个文件。其中两个文件，*changed.files* 和 *old_changed.files*，是存储 MD5 哈希的地方；第三个文件，*file_report.txt*，是一个显示结果的文本文件。脚本将比较 *changed.files* 中列出的所有文件的 MD5 哈希与 *old_changed.files* 中的哈希值，并返回任何找到的差异。以下是一个示例：

`已更改文件: old_changed.files 45de547aef9366eeaeb1b565dff1e1a3 删除/移动文件: test.txt.rsplit4 943600762a52864780b9b9f0614a470a 删除/移动文件: test.txt.rsplit5 131c8aa7155483e7d7a999bf6e2e21c0 删除/移动文件: test.txt.rsplit6 1ce31f6fbeb01cbed6c579be2608e56c 删除/移动文件: test.txt.rsplit1 35d5b2e522160ce3b3b98d2d4ad2a86e 删除/移动文件: test.txt.rsplit2 a65dde64f16a4441ff1619e734207528 删除/移动文件: test.txt.rsplit3 264b40b40103a4a3d82a40f82201a186`

## 它是如何工作的

这个脚本非常适合验证您硬盘驱动器的内容，并确保它们没有被篡改。脚本首先确认用户提供的参数是否包含在内，并且提供了一个有效的目录。接下来是初始化脚本中使用的变量。`root` 变量包含要扫描的根目录，创建了两个哈希值，将用于比较文件及其 MD5 哈希，最后指定了要使用的文件名 ![图片](img/00002.jpg)。根据脚本是否之前已经运行过，脚本输出将保存在两个或三个文件中。主文件 *file_report.txt* 用于读取输出，其他两个文件用于存储 MD5 哈希列表。

接下来，脚本会检查是否之前已经运行过，通过查找*file_list.txt* ![图片](img/00003.jpg)。如果找不到文件，脚本将继续执行。如果找到*file_list.txt*，脚本会立即重命名该文件。重命名的文件随后被打开，并读取其内容。对于文件中的每一行，脚本都会读取一个文件名和 MD5 哈希值，并将这些存储在`oldfile_hash`中以供后续比较。一旦`oldfile_hash`被填充，脚本就准备好开始计算新的 MD5 哈希值并比较结果。

当脚本遍历目录树时，它将遍历每个对象 ![图片](img/00004.jpg)。`Find.find`方法是一种强大的递归方式，用于检索目录及其子目录中的文件。代码块将在找到的每个文件上运行。第一条语句是查找"."和".."，出于明显的原因，这些将被跳过。如果对象是目录，脚本将对其进行跳过处理并继续。如果项目是文件，则生成哈希值并存储以供后续使用。哈希过程被包围在一个`begin`/`rescue`块中，以防出现严重错误。

信息收集的大部分工作现在已经完成。剩下要做的就是确定每个文件的状态。如果一个文件具有相同的名称和 MD5 哈希值，则表示文件未更改，脚本将移除输出哈希中的文件名。除了*未更改*之外，文件还可以归入以下三个类别之一。第一个是*已删除或移动*，这是通过检查文件在过去的扫描中存在但当前扫描中不存在来确定的 ![图片](img/00005.jpg)。接下来是*已更改*类别。如果文件名存在，但 MD5 哈希值与之前的扫描不同，则表示文件已被更改 ![图片](img/00006.jpg)。在这个阶段，为了代码的可读性，我使用了*三元运算符*，它是`if`/`then`/`else`语句的缩写。所以，这表示如果文件存在于`oldfile_hash`中，*则*将其标记为*已更改，否则*标记为*新增；*因为文件之前不存在，所以自上次扫描以来已添加 ![图片](img/00007.jpg)。所有数据都已保存，并生成报告，以便用户了解每个文件的状态。如果出现任何异常情况，则需要进一步分析。

有几个软件包执行类似的计算以用于安全目的，但上述方法是很好的替代方案，而且价格也合适。为了增强安全性，可以将输出文件存储在单独的介质上，但出于简单起见，我通常将它们留在顶级目录中。

## 操纵脚本

此脚本可以被修改以使用任何数量的哈希算法。我选择了 MD5，因为它在检查文件完整性方面最为流行（尽管其哈希值容易受到碰撞攻击）。此脚本在 Microsoft Windows 和类 Unix 系统上都能运行。跨平台脚本始终是一个加分项！

其他对脚本的潜在更改包括对哈希文件进行加密以增加保护或将其结果接口到数据库中。该脚本有许多潜在用途，我将留给你们去进一步调查。如果你对加密感兴趣，请查看下一脚本。

# 加密文件

## 加密文件

### encrypt.rb

你有多少次听说有人在拍卖网站上出售他们的电脑，结果后来发现他们的敏感信息已经在互联网上泄露了？还有企业间谍活动，或者所有那些丢失的政府笔记本电脑呢？如果你和网络安全专家交谈，他们给出的第一个建议之一就是加密敏感信息。你总是可以购买一个为你做这件事的程序，但这没有乐趣。让我们编写自己的加密脚本！有许多加密算法可供选择，它们具有不同的强度级别。在这个例子中，我将使用 Blowfish，这是一种非常快、对称的分组密码。

## 代码

`![](img/00002.jpg) require 'crypt/blowfish'   unless ARGV[0]      puts "用法：ruby encrypt.rb <文件名.ext>"      puts "示例：ruby encrypt.rb secret.stuff"      exit  end   #接收要加密的文件名作为参数  filename = ARGV[0].chomp  puts filename ![](img/00003.jpg) c = "Encrypted_#{filename}" ![](img/00004.jpg) if File.exists?(c)      puts "文件已存在。"      exit  end ![](img/00005.jpg) print '请输入您的加密密钥（1-56 字节）：'  kee = gets.chomp ![](img/00006.jpg) begin ![](img/00007.jpg)     blowfish = Crypt::Blowfish.new(kee) ![](img/00008.jpg)     blowfish.encrypt_file(filename.to_str, c)      puts '加密成功！' ![](img/00009.jpg) rescue Exception => e      puts "加密过程中发生错误：\n #{e}"  end`

## 运行代码

你必须在你的系统上安装 Ruby gem *crypt*——在控制台使用命令`gem install crypt`来安装加密库。此加密脚本通过命令提示符访问。要运行，请输入：

``**`ruby encryption.rb`** *`/path/of/file/to/encrypt`*``

您将被提示输入密码：

`请输入您的加密密钥（1-56 个字符）：`

* * *

### 警告

*记住你的密码，否则你将无法解密你的文件！*

* * *

现在按回车键，如果加密成功，你将看到这条消息：

`加密成功！`

查看此脚本所在的文件夹；你会看到新的加密文件，命名为*Encrypted_<filename>*。

## 结果

对于上面的示例，我使用了一个包含以下内容的纯文本文件：

`Wicked Cool Ruby Scripts`

在脚本完成文件加密后，它将输出一条成功消息。然后你可以尝试查看文件。如果你忘记了密码，祝你好运解密：

`qo".1>°<|šã_8tÃhÞí}"f-%◦1ð»=ðrþ¡.,`

正如你所见，结果与原始明文完全不相似。

## 工作原理

在第一行，我包含了用于加密的库：crypt/blowfish ![图片。请注意，您可以将此更改为使用其他算法，例如 Rijndael 或 GOST。![图片](img/00003.jpg) 行开始创建我们的加密文件。在 Ruby 中创建文件非常简单。如您所见，我使用了一个快捷方式来命名文件，通过在字符串中包含变量（`filename`）*在同一行*中，命名为 `Encrypted_#{filename}`。我喜欢有在文本字符串中包含变量的选项，所以您会看到我在整本书中都会使用它们。

接下来，我们检查加密的文件名是否已经存在。我们不希望脚本随意覆盖文件——这样很容易丢失数据。如果没有冲突，脚本将继续 ![图片](img/00004.jpg)。现在脚本知道加密文件尚未创建，需要用户提供一个*加密密钥*或密码。脚本要求提供一个 1 到 56 个字符的密钥 ![图片](img/00005.jpg)。一旦收集到所有必要的信息，脚本开始一个 `begin`/`rescue` 错误处理块 ![图片](img/00006.jpg)。脚本最后也是最重要的部分是实际的数据加密。使用传递给参数的加密密钥创建一个新的加密对象 ![图片](img/00007.jpg)。然后文件被传递到 `encrypt_file` 方法，*嗖*——文件被加密 ![图片](img/00008.jpg)。如果在加密阶段遇到任何错误，rescue 块会捕获它们并优雅地退出脚本，报告具体的错误 ![图片](img/00009.jpg)。

## 漏洞脚本

您可以通过多种方式修改此脚本。例如，您可以将其作为另一个程序的模块化部分，更改加密算法，分层加密，在加密后自动删除明文文件，或加密整个目录。

接下来，我们将探讨如何逆向这个过程并恢复我们的信息。

# 解密文件

## 解密文件

### decrypt.rb

此代码的结构与加密算法非常相似，所以我将重点放在两者之间的区别上。我使用与加密时相同的算法进行解密。如前所述，您可以使用任意数量的加密算法——只需确保使用相应的解密算法。不要忘记您的密码，否则如果您想再次看到您的数据，您将不得不编写自己的暴力破解脚本！

## 代码

` require 'crypt/blowfish'   unless ARGV[0]   puts "用法：ruby decrypt.rb <加密文件名.ext>"   puts "示例：ruby decrypt.rb Encrypted_secret.stuff"   exit   end   ![](img/00002.jpg) filename = ARGV[0].chomp   puts "正在解密 #{filename}。"   p = "Decrypted_#{filename}"   ![](img/00003.jpg) if File.exists?(p)   puts "文件已存在。"   exit   end   ![](img/00004.jpg) print '输入你的加密密钥：'   kee = gets.chomp   begin   ![](img/00005.jpg)     blowfish = Crypt::Blowfish.new(kee)       blowfish.decrypt_file(filename.to_str, p)       puts '解密成功！'   ![](img/00006.jpg)     rescue Exception => e       puts "解密过程中发生错误：\n #{e}"   end`

## 运行代码

代码执行简单；只需输入解密脚本的名称，然后输入你希望解密的文件名：

``**`ruby decrypt.rb`** *`encrypted_filename.ext`*``

Ruby 脚本将提示你输入加密密钥。记住，你必须有用于加密文件的密钥才能解密它。如果没有，那么除了暴力破解之外，没有其他方法可以恢复文件，而这可能比你愿意花费的时间要长得多。

## 结果

`文件内容之前：qo".1>°<|šã_8tÃhÞí}"f-%◦1ð»=ðrþ¡., 文件内容之后：Wicked Cool Ruby Scripts`

如预期的那样，解密脚本将密文干净地转换回明文。如果你有时间，尝试使用错误的密钥并检查输出。它看起来会像密文一样晦涩难懂。

## 工作原理

脚本首先从命令行参数中获取文件名，并初始化将要使用的变量！[。每当创建一个文件时，你应该检查是否已经存在同名文件！[](../Images/00003.jpg)。在初始化算法之后，脚本将要求输入一个密钥！[](../Images/00004.jpg)。

到目前为止，脚本中的所有内容看起来都与加密脚本相同。即使你输入了错误的加密密钥，脚本也会根据该错误的密钥解密文件，结果与之前一样晦涩难懂。如果一切顺利，你将能够使用之前加密的文件。

实际的解密是通过使用 crypt 库中的 `decrypt` 方法！[](../Images/00005.jpg)来完成的，这仅仅是加密的逆过程。

如果没有错误或异常，输出将显示`解密成功！`然后程序将退出。如果有问题，我们的`begin`/`rescue`块将捕获错误并进入我们的`rescue`情况。`rescue`情况将显示错误消息并通知用户文件尚未被解密！[](../Images/00006.jpg)。

对加密脚本所做的任何修改都必须对解密脚本进行相同的修改。如果你在加密脚本中执行了一个任务，却忘记在解密脚本中撤销它，你的数据将不复存在。

# 文件分割

## 文件分割

### fileSplit.rb

Ruby 脚本的一个酷用是将大文件分割成几个较小的、对称的文件。我写这个脚本是为了一个朋友，他因为网络管理员不允许传输超过一定大小的文件（可能是出于带宽原因）而无法将文件发送到他的公司网络内部。这个脚本效果非常好。

## 代码

`![](img/00002.jpg) if ARGV.size != 2      puts "Usage: ruby fileSplit.rb <filename.ext> <size_of_pieces_in_bytes>"      puts "Example: ruby fileSplit.rb myfile.txt 10"      exit  end   filename = ARGV[0]  size_of_split = ARGV[1]  ![](img/00003.jpg) if File.exists?(filename)      file = File.open(filename, "r")      size = size_of_split.to_i       puts "The file is #{File.size(filename)} bytes."  ![](img/00004.jpg)     temp = File.size(filename).divmod(size)      pieces = temp[0]      extra = temp[1]       puts "\nSplitting the file into #{pieces} (#{size} byte) pieces and 1  (#{extra} byte) piece"  ![](img/00005.jpg)     pieces.times do |n|          f = File.open("#{filename}.rsplit#{n}", "w")          f.puts file.read(size)      end ![](img/00006.jpg)     e = File.open("#{filename}.rsplit#{pieces}", "w")      e.puts file.read(extra)  else      puts "\n\nFile does NOT exist, please check filename."  end`

## 运行代码

在一个全新的目录中运行此脚本，并包含你想要分割的文件，是最简单的方法。像之前的脚本一样，首先输入：

``**`ruby fileSplit.rb`** *`path/to/file size_of_pieces_in_bytes`*``

如果你想要将一个 10KB 的文件分割成 1,000 字节（或 1KB）的片段，脚本将会创建 10 个单独的文件，分别标记为*<filename>.rsplit<#1-10>*。为此，请输入以下命令：

``**`ruby fileSplit.rb test.txt 1000`**``

## 结果

在本例中使用的初始文件名为 *test.txt*，结果如下所示：

`test.txt.rsplit0 test.txt.rsplit1 test.txt.rsplit2 test.txt.rsplit3 test.txt.rsplit4 test.txt.rsplit5 test.txt.rsplit6 test.txt.rsplit7 test.txt.rsplit8 test.txt.rsplit9`

## 工作原理

如果你面临着一个棘手的公司网络策略，限制了允许传输的文件大小，或者如果你正在寻找一种更可靠的传输大文件的方法，这个实用程序将能帮上大忙。我遇到了公司环境中的这种情况，我知道文件大小的限制，因此我能够硬编码文件大小。然而，你可以使用你需要的任何大小，或者将其作为脚本中的一个选项。

脚本首先从 ARGV 数组中读取前两个项目：要分割的文件名和每个部分的尺寸。如果这两个变量，`filename` 和 `size`，没有指定，脚本将显示 `正确用法`![](img/00002.jpg)。

接下来，脚本确保你正在尝试分割一个真实文件 ![图片](img/00003.jpg)。除以零很困难，分割一个不存在的文件则更加困难。如果找不到文件，脚本将退出并显示错误信息，告知用户文件名有问题。希望文件能被找到，脚本开始为分割做准备。

正如你所知，文件可以是任何大小，而且很少能被你选择的字节数完美分割。为了处理动态文件大小，脚本使用`divmod`——`divmod`将两个数字相除，返回一个包含商和余数的数组。在这个脚本中，`pieces`是商，而`extra`是余数 ![图片](img/00004.jpg)。

为了保持文件的完整性，分割片段是通过一次读取一个字节并将二进制写入输出创建的。这一部分是魔法发生的地方 ![图片](img/00005.jpg)。首先写入整个片段，然后写入额外的片段 ![图片](img/00006.jpg)。

## 漏洞脚本

如果你想要扩展代码，一个完美的补充就是在分割文件之前添加一个压缩程序。我稍后会更多地讨论压缩。另一个使脚本更具灵活性的方法是为分割文件到特定数量的片段添加一个选项，无论大小如何。你也可以修改这个脚本以创建符合你选择的媒体格式的文件片段，无论是 700MB 的光盘还是 2.88MB 的软盘。

# 文件连接

## 文件连接

### fileJoin.rb

这个脚本也是为了我的朋友而写的，知道如果他无法重建文件，他会非常沮丧。这是文件分割脚本的配套脚本，如果你愿意，可以将这两个脚本放在一个包装器中。（包装器是将两个脚本结合在一起的一个工具。）我这里将它们分开是为了教学目的。这个文件连接脚本只适用于之前分割过的文件（参见文件分割中的“#4 文件分割”）；然而，你可以调整它以满足你的需求。

## 代码

`![图片](img/00002.jpg) if ARGV.size != 1

## 运行代码

脚本不支持更改目录，所以请确保它位于你想要连接的文件所在的同一目录中。要运行脚本，请输入：

``**`ruby fileJoin.rb`** *`filename.ext`*``

## 结果

使用文件分割脚本输出的文件，输入应该是要重新组装的文件名，如下所示：

`读取文件：test.txt.rsplit0 读取文件：test.txt.rsplit1 读取文件：test.txt.rsplit2 读取文件：test.txt.rsplit3 读取文件：test.txt.rsplit4 读取文件：test.txt.rsplit5 读取文件：test.txt.rsplit6 读取文件：test.txt.rsplit7 读取文件：test.txt.rsplit8 读取文件：test.txt.rsplit9 成功！ 文件已重建。`

脚本运行后，组装的文件将被命名为*New.test.txt*。

被连接的新文件将在脚本所在的同一目录中找到。每个*.rsplit*片段仍然存在，以防在重建文件时出现任何错误。一旦找到文件并打开它，内容应该与分割文件之前完全相同。你可以比较旧的和新的 MD5 散列值来亲自查看（见检查更改的文件中的“#1 检查更改的文件”）。

## 工作原理

脚本首先获取被分割的文件的原始文件名。如果没有提供命令行参数，脚本将报错，并且你必须再次尝试！ ![](img/00002.jpg)。如果提供了文件名，脚本将检查是否有与该文件名对应的片段！ ![](img/00003.jpg)。如果没有，它将再次报错，表示找不到文件。

找到第一块后，脚本创建输出文件 ![](img/00004.jpg)。接下来，使用 while 循环确保只将下一个连续的片段附加到主体 ![](img/00005.jpg)。只要存在“下一个片段”，脚本就会继续将数据附加到输出文件。由于每个分割片段的数据末尾都有一个换行符，我们使用`chomp`方法确保只流式传输原始数据 ![](img/00006.jpg)。

将所有片段附加到输出文件后，输出文件将被关闭。将显示一个成功的消息，并且脚本退出。现在你可以检查新文件以验证它是否完美恢复。

## 漏洞脚本

如果你信任脚本，你可以对其进行调整以清理自身，删除所有的 .*rsplit* 片段。你还可以在分割前后计算文件的 MD5 散列值以验证其真实性。

# Windows 进程查看器

## Windows 进程查看器

### listWin Processes.rb

Windows 任务管理器中的进程查看器可能会非常令人沮丧，因为信息不足。如果你曾在类 Unix 系统中使用过 `ps` 命令，你就会知道除了进程名称、CPU/内存使用情况和进程所有者之外，还有多少信息可用。一些应用程序在进程查看器中创建了详细条目，这些任务很容易识别，但其他应用程序有一些模糊的名称，对你没有任何帮助。有另一种方式查看进程很方便，因为你可以根据需要自定义脚本，以显示对你来说真正重要的信息。这个脚本演示了如何检索每个可用的进程属性。

## 代码

`![win32ole 库](img/00002.jpg) require 'win32ole' ![Windows 内部方法](img/00003.jpg) ps = WIN32OLE.connect("winmgmts:\\\\.") ![连接到 WMI](img/00004.jpg) ps.InstancesOf("win32_process").each do |p| ![遍历进程](img/00005.jpg) puts "进程: #{p.name}" puts "\tID: #{p.processid}" puts "\t 路径:#{p.executablepath}" puts "\t 线程数: #{p.threadcount}" puts "\t 优先级: #{p.priority}" puts "\t 命令行参数: #{p.commandline}" end`

## 运行代码

脚本编写得可以独立运行，并显示每个进程的信息。根据需要添加和删除脚本中的属性。

## 结果

`进程: winlogon.exe    ID: 1296    路径:C:\WINDOWS\system32\winlogon.exe    线程数: 22    优先级: 13    命令行参数: winlogon.exe 进程: services.exe    ID: 1348    路径:C:\WINDOWS\system32\services.exe    线程数: 15    优先级: 9    命令行参数: C:\WINDOWS\system32\services.exe 进程: explorer.exe    ID: 1240    路径:C:\WINDOWS\Explorer.EXE    线程数: 14    优先级: 8    命令行参数: C:\WINDOWS\Explorer.EXE 进程: svchost.exe    ID: 3836    路径:C:\WINDOWS\System32\svchost.exe    线程数: 8    优先级: 8    命令行参数: C:\WINDOWS\System32\svchost.exe -k HTTPFilter 进程: firefox.exe    ID: 2140    路径:C:\Program Files\Mozilla Firefox\firefox.exe    线程数: 7    优先级: 8    命令行参数: "C:\Program Files\Mozilla Firefox\firefox.exe" 进程: cmd.exe    ID: 1528    路径:C:\WINDOWS\system32\cmd.exe    线程数: 1    优先级: 8    命令行参数: "C:\WINDOWS\system32\cmd.exe" 进程: ruby.exe    ID: 244    路径:c:\ruby\bin\ruby.exe    线程数: 4    优先级: 8    命令行参数: ruby ListWinProcesses.rb`

## 工作原理

对于与 Windows 操作系统的大多数交互，我使用 win32ole 库 ![win32ole 库](img/00002.jpg)。这个库非常有用，我将在后面的章节中用它演示更多的自动化。脚本的第一个部分是初始化 *winmgmts*，这使得脚本可以与 Windows 内部方法交互 ![Windows 内部方法](img/00003.jpg)。*Winmgmts* 是 *Windows Management Interface (WMI)*。WMI 有很多有用的工具，如果你对 Windows 脚本编写感兴趣，可以进一步探索。我把我实例化的 WMI 命名为 `ps`，因为它让我想起了 Unix 风格系统中的 `ps` 方法。

接下来，脚本迭代所有 `win32_process` 的实例。这是找到所有进程并提取信息的地方 ![](img/00004.jpg)。我用于脚本的属性包括 `进程名称、ID、路径、运行的线程`、`优先级` 和 `命令行参数`。我发现知道命令行参数在我想从其他脚本或命令行调用程序时很有用 ![](img/00005.jpg)。

## 破解脚本

如果你想查看每个进程的所有信息，你可以从 WMI 属性类中包含以下属性。有很多可能的配置可以满足你的需求。

| WMI 进程类属性 |  |  |
| --- | --- | --- |
| `Caption` | `OSCreationClassName` | `QuotaPeakPagedPoolUsage` |
| `CommandLine` | `OSName` | `ReadOperationCount` |
| `CreationClassName` | `OtherOperationCount` | `ReadTransferCount` |
| `CreationDate` | `OtherTransferCount` | `SessionId` |
| `CSCreationClassName` | `PageFaults` | `Status` |
| `CSName` | `PageFileUsage` | `TerminationDate` |
| `Description` | `ParentProcessId` | `ThreadCount` |
| `ExecutablePath` | `PeakPageFileUsage` | `UserModeTime` |
| `ExecutionState` | `PeakVirtualSize` | `VirtualSize` |
| `Handle` | `PeakWorkingSetSize` | `WindowsVersion` |
| `HandleCount` | `Priority` | `WorkingSetSize` |
| `InstallDate` | `PrivatePageCount` | `WriteOperationCount` |
| `KernelModeTime` | `ProcessId` | `WriteTransferCount` |
| `MaximumWorkingSetSize` | `QuotaNonPagedPoolUsage` | `WorkingSetSize` |
| `MinimumWorkingSetSize` | `QuotaPagedPoolUsage` |  |
| `Name` | `QuotaPeakNonPagedPoolUsage` |  |

虽然上述列表包含了 WMI 进程类的所有属性，但还有几个其他的操作系统类——每个类都有其自己的属性。要使用与不同操作系统类相同的脚本，请将 ![](img/00004.jpg) 中的 `win32_process` 替换为另一个 WMI 类。例如，`registry` 将会是 `win32_registry`。

# 文件压缩器

## 文件压缩器

### compress.rb

当你开始谈论数据存储时，能够有效地压缩文件是一个重要的资产。压缩越高效，在相同的空间内可以存储的信息就越多。目前有两个流行的 Ruby 压缩库正在使用。第一个是 ruby-zlib，第二个是 rubyzip。它们各有优缺点，我将留给您选择适合您目的的压缩算法。在下面的脚本中，我将使用 rubyzip。

## 代码

` require 'zip/zip'  ![图片](img/00002.jpg) unless ARGV[0]      puts "Usage: ruby compress.rb <filename.ext>"      puts "Example: ruby compress.rb myfile.exe"      exit  end   file = ARGV[0].chomp  ![图片](img/00003.jpg) if File.exists?(file)      print "Enter zip filename:"      zip = "#{gets.chomp}.zip" ![图片](img/00004.jpg)     Zip::ZipFile.open(zip, true) do |zipfile| ![图片](img/00005.jpg)        begin             puts "#{file} is being added to the archive." ![图片](img/00006.jpg)        zipfile.add(file,file) ![图片](img/00007.jpg)        rescue Exception => e             puts "Error adding to zipfile: \n #{e}"         end      end  else      puts "\nFile could not be found."  end`

## 运行代码

此脚本允许用户压缩不同类型的文件，无论是为了节省空间还是为了便于存档。使用以下命令调用脚本：

``**`ruby compress.rb`** *`/path/to/file`*``

## 结果

脚本将使用用户在提示中提供的名称，通过命令行指定的文件创建一个压缩存档。在这个例子中，我将`chapter1.odt`压缩成了`nostarch.zip`。在压缩之前，`chapter1.odt`的大小是 29.1KB，压缩后变成了 26.3KB。文件将存储在脚本执行的同一目录中。

## 工作原理

当脚本运行时，首先进行错误处理检查以确保用户已提供要压缩的文件 ![图片](img/00002.jpg)。如果已提供文件名，则会检查文件是否可用。始终，如果我们想要操作的物体不可用，就没有继续下去的意义。如果文件不存在，脚本会提醒用户并立即退出 ![图片](img/00003.jpg)。如果文件存在且已验证，则会要求用户命名 Zip 文件。在用户输入文件名并按下 ENTER 键后，脚本继续执行。请注意，按下 ENTER 键时，会将`\n`（换行字符）添加到字符流中，并将其作为用户输入发送到脚本。您会注意到使用了`chomp`来移除用户按下 ENTER 时添加的`\n`。

用于压缩文件的代码很简单。如上所示，在![图片](img/00004.jpg)行，该部分将打开一个现有的 Zip 文件（如果可用且第二个参数设置为`true`），如果文件不存在，则会创建一个新的 Zip 文件。这些选项与 File 库中的`open`方法类似。

有时会发生错误。在这个脚本中，最脆弱的地方是在压缩过程中向 Zip 文件添加文件时。![图片](img/00005.jpg)处的`begin`/`rescue`块用于处理意外错误并向用户提供有关任何问题的信息。如果发生错误，`rescue`块将被执行，脚本将退出，并显示错误信息![图片](img/00007.jpg)。

每个被添加到 Zip 文件中的文件都使用`add`方法保存 ![图片 6](img/00006.jpg)。你可以从这一部分在 Zip 文件中创建目录，或者即时写入全新的文件。基本上，Zip 文件系统可以像你的计算机上的任何正常目录一样处理。语法略有不同，但结果相同。

rubyzip 库非常出色，因为它允许你打开 Zip 文件并操作内容，而无需解压整个存档。此外，与 tar 和 gz 将文件分组然后压缩不同，rubyzip 只需一个命令就能完成所有这些操作。

# 文件解压

## 文件解压

### decompress.rb

此脚本展示了如何解压文件的基本方法。rubyzip 库为你完成所有工作。在标准的类 Unix 系统中，你将需要手动解压文件，执行任务，然后重新压缩文件。使用 rubyzip，你可以使用一个无缝的库来处理存档中的文件。此脚本将存档完全解压到用户指定的目录。

## 代码

` require 'zip/zip'  require 'fileutils'   unless ARGV[0]      puts "用法: ruby decompress.rb <zipfilename.zip>"      puts "示例: ruby decompress.rb myfile.zip"      exit  end   archive = ARGV[0].chomp  ![](img/00002.jpg) if File.exists?(archive)      print "输入保存文件的路径（\'.\'表示同一目录）: " ![](img/00002.jpg)     extract_dir = gets.chomp ![](img/00004.jpg)     begin ![](img/00005.jpg)         Zip::ZipFile::open(archive) do |zipfile|              zipfile.each do |f| ![](img/00006.jpg)                 path = File.join(extract_dir, f.name)                  FileUtils.mkdir_p(File.dirname(path))                  zipfile.extract(f, path)              end          end ![](img/00007.jpg)     rescue Exception => e          puts e      end  else      puts "解压过程中发生错误：\n #{e}"  end`

## 运行代码

解压脚本的使用方式与压缩脚本类似，需要将需要解压的文件作为命令行参数传入。

``**`ruby decompress.rb`** *`/path/to/compressed/file`*``

## 结果

所有最初放入 Zip 文件中的文件都将按照压缩前的结构进行解压。例如，我将*nostarch.zip*解压到*chapter1.odt*。压缩后的 Zip 文件*chapter1.odt*大小为 26.3KB，解压后文件恢复到原始的 29.1KB。

## 它是如何工作的

与压缩脚本类似，此脚本期望将压缩文件作为命令行参数提供。如果无法找到存档文件，脚本将向用户显示错误消息 ![图片 2](img/00002.jpg)。这两个脚本的主要区别在于，解压脚本不是要求输入要创建的 Zip 文件名，而是要求输入解压文件的目标路径 ![图片 3](img/00003.jpg)。

下一步是`begin`/`rescue`块的开始 ![图片](img/00004.jpg)。与压缩脚本一样，解压缩是代码中的一个脆弱部分。解压缩的第一部分是打开压缩文件 ![图片](img/00005.jpg)。之后，每个文件都会被解压缩。解压缩例程会重新创建与压缩前相同的目录结构 ![图片](img/00006.jpg)。因此，如果压缩前有两个子文件夹，那么在脚本完成后也会有两个文件夹。只要没有遇到错误，脚本就会将每个文件输出到用户指定的目录。脚本的最后一部分是`rescue`块，它会捕获并报告解压缩过程中发生的任何错误 ![图片](img/00007.jpg)。

# 抵押贷款计算器

## 抵押贷款计算器

### mortgageCalc.rb

我最近开始看房子。作为一个首次购房者，这项任务似乎很艰巨，尤其是当我考虑融资选项时。所以我决定写一个脚本来帮助我了解抵押贷款利率——至少这样，我可以估算我的月供。尽管 Ruby 没有解决与购房相关的所有问题，但这个脚本帮助我了解了我的融资选项。

## 代码

` print "Enter Loan amount: "  loan = gets.chomp.to_i  print "Enter length of time in months: "  time = gets.chomp.to_i  print "Enter interest rate: "  rate = gets.chomp.to_f/100  ![](img/00002.jpg) i = (1+rate/12)**(12/12)-1 ![](img/00003.jpg) annuity = (1-(1/(1+i))**time)/i  ![](img/00004.jpg) payment = loan/annuity  ![](img/00005.jpg) puts "\n$%.2f per month" % [payment]`

## 运行代码

这个脚本交互式运行，因此无需任何参数。它会引导用户输入所需的所有信息以得出正确的月供。不需要命令行参数。

## 结果

``输入贷款金额：*`250000`* 输入月数：*`360`* 输入利率：*`6.5`*  每月$1580.17``

## 它是如何工作的

对于我来说，抵押贷款计算似乎有点神秘，我以为我需要一堵学位墙来理解这些公式。幸运的是，计算抵押贷款支付并不像解微分方程！一旦你理解了基本公式，它就简单多了。抵押贷款支付的计算分为两个主要公式（如果你特别大胆，可以合并成一个公式）。第一个公式使用以下方程计算每月的*利率* ![图片](img/12)**(12/12)-1`

我们需要的下一项信息是年金因子 ![图片](img/00003.jpg)。基本上，*年金因子*是每个时间段内 1 美元的当前价值。*时间*以月份为单位。因此，计算如下：

`annuity = (1-(1/(1+i))**time)/i`

现在年金因子已经计算出来，我们真正追求的是月付款额。将贷款金额除以年金因子就可以得到最终答案 ![图片](img/00004.jpg)。剩下的只是进行一些格式调整，以便更容易阅读信息。与其他编程语言一样，Ruby 为程序员提供了指定输出格式的能力。在这种情况下，对于货币值，我感兴趣的是除了整数或整美元值之外，还有两位小数的分值 ![图片](img/00005.jpg)。

## 操纵脚本

操纵这个脚本的一种方法是为利率或贷款金额提供一些变动，这样输出就可以显示几个可能的月付款额，而不仅仅是其中一个——可能是±0.05%。通常，当你寻找抵押贷款时，你会比较大量的财务信息。你能在同一个界面上提供的信息越多，你做出的决定就越好。
