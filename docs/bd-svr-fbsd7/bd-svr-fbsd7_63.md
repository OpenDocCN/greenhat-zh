## 第二十八章。SPAMASSASSIN 3.2.4

#### HTTP://SPAMASSASSIN.APACHE.ORG

### 28.1. 摘要

SpamAssassin 是一个高效的开源电子邮件分类器。它通过使用一系列不同的测试来检查电子邮件，以确定一条消息是否为垃圾邮件。通过使用关键词、历史数据（例如，贝叶斯过滤）和指纹识别方法（例如，Vipul's Razor 和 DCC 数据库）来评分，这些测试旨在通过利用每种测试类型的好处来最大化其有效性。SpamAssassin 将这些测试的结果存储在每个电子邮件的标题中。邮件投递代理（MDA）如 Procmail（参见 "Procmail 3.22"）可以使用这些标题来路由或对消息进行进一步处理。

通常，SpamAssassin 作为守护进程或后台进程被调用。邮件传输代理（MTA）如 Postfix 被配置为将电子邮件通过 SpamAssassin 进行分析。如果确定一条消息是垃圾邮件，SpamAssassin 可以配置为以多种方式修改该消息。默认情况下，垃圾邮件被重新编码为附件，并且消息的正文显示触发积极结果的测试列表。然后垃圾邮件和正常邮件（合法电子邮件）被投递到用户的邮箱中。

许多商业反垃圾邮件软件包都将其产品中的 SpamAssassin 集成。这些包括 McAfee 的 SpamKiller、Kerio 的 Kerio MailServer 和 SmarterTools 的 SmarterMail。

SpamAssassin 由爱尔兰软件开发者 Justin Mason 在 2001 年编写。它基于 Mark Jeftovic 在 1997 年用 Perl 编写的 spam 过滤器 filter.plx。Justin 为 Jeftovic 的 filter.plx 贡献了补丁，后来决定从头开始重写代码（同样是用 Perl）。这次重写成为了 SpamAssassin，现在它是 Apache 软件基金会的一个项目。Mason 目前作为 Apache 软件基金会的副总裁负责 SpamAssassin 的发展。
