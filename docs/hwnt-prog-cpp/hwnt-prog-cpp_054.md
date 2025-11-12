## 程序 52：消失矩形的案例

我们样本的面积是多少？

```
  1 /************************************************
  2  * Demonstration of the rectangle class.        *
  3  ************************************************/
  4 #include <iostream>
  5
  6 /************************************************
  7  * rectangle -- hold constant information about *
  8  *              a rectangle.                    *
  9  *                                              *
 10  * Members:                                     *
 11  *      area -- Area of the rectangle.          *
 12  *      width -- width of the rectangle.        *
 13  *      height - length of the rectangle.       *
 14  ************************************************/
 15 class rectangle
 16 {
 17     public:
 18         const int area;   // Rectangle's Area
 19         const int width;  // Rectangle's Width
 20         const int height; // Rectangle's Height
 21
 22     public:
 23         // Create a rectangle and assign the
 24         // initial values
 25         rectangle(
 26             const int i_width,  // Initial width
 27             const int i_height  // Initial height
 28         ) : width(i_width),
 29             height(i_height),
 30             area(width*height)
 31         {}
 32         // Destructor defaults
 33         // Copy constructor defaults
 34         // Assignment operator defaults
 35 };
 36
 37 int main()
 38 {
 39     // Rectangle to play with
 40     rectangle sample(10, 5);
 41
 42     std::cout << "Area of sample is " <<
 43         sample.area << std::endl;
 44     return (0);
 45 }

```

(下一提示 210。答案 93。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

**人侵计算机**

一所大型大学的系统管理员负责维护数百台 DEC 机器的运行。他很快学会了如何诊断损坏的机器并找出哪块板子坏了。为了获得稳定的备件供应，他不得不购买服务合同。这意味着 DEC 服务代表应该下来，诊断机器，找出哪块板子坏了并更换它。实际上，大学员工有严格的命令，永远不要让 DEC 服务代表接近机器。

典型的服务通常从管理员告诉 DEC 哪块板子坏了开始。服务代表会下来，在管理员的桌子上找到损坏的板子，并用一块好的板子替换它。不需要运行诊断或其他工作，这一切都为他准备好了。

几年后，DEC 实施了“智能备件”计划。这个想法是，你有一个受过培训的人在现场，可以找出哪块板子坏了，并从 DEC 订购替换件。当然，这对大学来说很合适，因为它已经这样做了好几年。

但问题是，你必须参加 DEC 的课程，学习如何诊断系统。系统管理员抓住了这个机会。他需要休假。他几乎在两天的课程中睡着了。第三天是实验室。讲师设置了三台机器。学生们被分成小组，应该花整个上午的时间找出他们的机器出了什么问题。

我们的英雄打开了他的机器，看了一分钟闪烁的灯光，然后说：“硬盘卡坏了。”讲师有点惊讶。“你怎么知道？”

“灯光不对。”

管理员走到下一台机器前，看了看，然后说：“内存接口卡坏了。”在下一台机器上，“处理器卡坏了。”

三台机器已经检查完毕。这个练习原本需要三个小组整个上午的时间，而这个家伙只用了两分钟就诊断了所有机器。（我之前和他谈过，他告诉我如果他知道自己在计时，他会工作得更快。）

毫不畏惧，讲师走到下午问题预留的机器旁。这台机器是由现场服务设置的，应该有一个非常困难的问题，几乎不可能找到。

讲师知道问题不能通过查看灯光来识别，他等待看看我们的英雄会做什么。这个人打开后盖，甚至在按下“开启”开关之前就指向一块板子说：“这块板子坏了；芯片 U18。它会导致间歇性的数据总线奇偶校验错误。”

现在讲师知道这个人很厉害，但即使没有打开机器也能找出坏板？这是不可能的。

“你怎么知道它是坏的？”他问。

管理员指向角落里一个小小的蓝色标签。“看到那个点了吗？我把它放在那里是为了确保 DEC 不会把我的板子还给我。这块板子是大学的。是我最初发现了问题并向 DEC 现场服务展示了它。”

上午和下午的问题现在在约 10 分钟内得到了解决，班级决定是时候出去吃披萨和喝啤酒了。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
