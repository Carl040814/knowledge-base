# 以太坊前史 (Prehistory of Ethereum)

> “英雄之所以是英雄，是因为他们的行为英勇，而不是因为他们的输赢。”
> —— 纳西姆·尼古拉斯·塔勒布 (Nassim Nicholas Taleb)

本文探索了以太坊 (Ethereum) 的血脉传承，向那些用勇气、创造力和纯粹的叛逆精神影响了它的英雄们致敬。

以太坊根植于早期互联网的开放精神，其设计哲学呼应了 Unix 关于“做好一件事并把它做好”的理想。以 GNU/Linux 为代表的自由和开源软件运动的兴起，重新确立了软件领域的开放标准。与此同时，公钥密码学 (public key cryptography) 的突破以及密码朋克 (cypherpunks) 的倡导，为比特币 (Bitcoin) 等安全、透明和去中心化 (decentralized) 的系统奠定了基础，这最终激发了以太坊建立一个无国界、自主主权的数字经济 (self-sovereign digital economy) 平台的愿景。

> “如果你看看早期参与比特币空间的人，他们早期的履历（如果他们有履历的话）都在开源领域——Linux、Mozilla 和密码朋克邮件列表。”
> —— *维塔利克·布特林 (Vitalik Buterin)，以太坊联合创始人。*

---

## 信息高速公路 (The information super highway)

从 1969 年作为冷战项目（[阿帕网 / ARPANET](https://en.wikipedia.org/wiki/ARPANET)）的简陋起点开始，互联网已演变成一种前所未有的全球现象。

> “互联网的普及速度令之前的所有技术都相形见绌。收音机诞生 38 年后才拥有 5000 万听众；电视用了 13 年才达到这一基准。在第一台个人电脑套件问世 16 年后，有 5000 万人正在使用它。而互联网一旦向公众开放，仅用四年就跨过了这一门槛。”
> —— [《新兴的数字经济》，(1998年7月)。](https://www.commerce.gov/sites/default/files/migrated/reports/emergingdig_0.pdf)

![A map of internet cables from 1989 to 2021.](img/overview/information-superhighway.gif)
**1989 年至 2021 年互联网海底光缆图。[来源：纽约时报](https://www.nytimes.com/interactive/2019/03/10/technology/internet-cables-oceans.html)**

最初只是少数机构的研究工具，现在连接了全球数十亿人，瓦解了地理边界，促进了曾经难以想象的人类交互。

> “在信息高速公路上，国家边界只是减速带。”
> —— 蒂莫西·梅 (Timothy May)，密码朋克。

---

## Unix 与贝尔实验室 (Unix & Bell Labs)

Unix 诞生于简化 [MULTICS 操作系统](https://en.wikipedia.org/wiki/Multics)（1960 年代一个庞大而雄心勃勃的操作系统项目）复杂性的努力中。随着 MULTICS 变得笨重，包括 AT&T 贝尔实验室 (AT&T Bell Labs) 的 [肯·汤普森 (Ken Thompson)](https://en.wikipedia.org/wiki/Ken_Thompson) 和 [丹尼斯·里奇 (Dennis Ritchie)](https://en.wikipedia.org/wiki/Dennis_Ritchie) 在内的一个小组试图创建 Unix——一个更具模块化、更简单且更可组合的替代方案：

> “在某个时刻，我意识到我距离一个操作系统只有三周的时间。我需要一个编辑器、汇编器和内核叠加层——管它叫操作系统。一周、一周、又一周，我们便拥有了 Unix。”
> —— [*肯·汤普森在一次采访中*](https://www.youtube.com/watch?v=EY6q5dv_B-o)

1972 年，丹尼斯还编写了具有深远影响的 [C 语言 (C language)](<https://en.wikipedia.org/wiki/C_(programming_language)>)。

![Ken Thompson and Dennis Ritchie](img/overview/ken-thompson-dennis-ritchie.jpg)
**肯·汤普森和丹尼斯·里奇。**

贝尔实验室是本世纪最决定性的技术基石的无双孵化器：

> “你无法去商店购买贝尔实验室的创新，但它深深蕴藏在其他事物之中；它是构成通信基础设施必不可少的平台创新。”
> —— 乔恩·格特纳 (Jon Gertner)，《创意工厂》

> 🎦 观看：[乔恩谈论贝尔实验室的创新](https://www.youtube.com/watch?v=OJsKgiGGzzs)

在许多方面，[以太坊基金会](https://ethereum.foundation/infinitegarden) 的运作方式就像一个开放的贝尔实验室。

Unix 引入了诸如分层文件系统、作为命令行界面 (command-line interface) 的 shell、以及可以组合起来执行复杂任务的单一用途实用工具等概念。
这些基本原则为后来被称为 UNIX 哲学 (UNIX philosophy) 的思想奠定了基础——在软件设计中偏爱简单性、灵活性和可复用性。

今天，UNIX 及其衍生品继续支撑着现代计算的大部分，影响着从 Linux 和 macOS 等操作系统到永恒的软件开发原则的一切。

> 🎦 观看：[Unix 纪录片](https://www.youtube.com/watch?v=tc4ROCJYbm0)

Unix 的遗产证明了一小群人通过软件对世界产生的深远影响。

---

## 我们能保守秘密吗？(Can we keep a secret?)

自文明诞生以来，保密传递信息一直是人类恒久的追求。从隐瞒商业秘密的商人，到传递关键信息的间谍和军队，密码学一直发挥着至关重要的作用。早期的加密方法通常对加密和解密使用相同的密钥，这使安全的密钥分发成为一场噩梦：

> “对于没有军事通信经验的个人来说，制作、登记、分发和注销密钥的问题可能显得微不足道，但在战时，通信流量之大甚至会让通信参谋们目瞪口呆。”
> —— [大卫·卡恩 (David Kahn) 在《破译者》中写道](https://en.wikipedia.org/wiki/The_Codebreakers)

如果密钥落入敌人手中，信息就会受到威胁。这在第二次世界大战中显而易见，数学家 [艾伦·图灵 (Alan Turing)](https://en.wikipedia.org/wiki/Alan_Turing) 及其团队破解了德国精密的密码机 [恩尼格玛密码机 (Enigma machine)](https://en.wikipedia.org/wiki/Enigma_machine)。他们的成功极大改变了战争的进程。

![A statue of Alan Turing and the Enigma machine.](img/overview/alan-turing.jpg)
**艾伦·图灵雕像和恩尼格玛密码机。**

如何在从未谋面的人们之间、在漫长的距离上安全地交换密钥？批评者曾认为密码学注定要依赖于信任：

> “很少有人会相信，发明一种能挫败调查的秘密书写方法不是一件很容易的事。然而，可以断言，人类的聪明才智无法编造出人类的聪明才智无法破解的密码。”
> —— 埃德加·爱伦·坡 (Edgar Allan Poe)

坡的观点被 1974-1978 年间的一系列发明证明是错误的。

1974 年，[拉尔夫·默克尔 (Ralph Merkle)](https://en.wikipedia.org/wiki/Ralph_Merkle) 发明了 [默克尔谜题 (Merkle's Puzzles)](https://en.wikipedia.org/wiki/Merkle%27s_Puzzles)——这是一种初始方法，允许双方通过交换信息来商定一个共享的秘密，即使他们事先没有任何共同的秘密。

两年后的 1976 年，默克尔的工作启发了 [惠特菲尔德·迪菲 (Whitfield Diffie)](https://en.wikipedia.org/wiki/Whitfield_Diffie) 和 [马丁·赫尔曼 (Martin Hellman)](https://en.wikipedia.org/wiki/Martin_Hellman) 发表了他们具有历史意义的论文 [《密码学新方向》("New directions in cryptography")](https://ee.stanford.edu/~hellman/publications/24.pdf)，引入了 [迪菲-赫尔曼密钥交换算法 (Diffie–Hellman key exchange algorithm)](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange)。这种方法在数学上比默克尔谜题要健壮得多——孕育了无需信任 (trustless) 的密码学。

![Whitfield and Martin published "New directions in cryptography"](img/overview/new-direction-in-cryptography.jpg)
**惠特菲尔德和马丁发表《密码学新方向》。**

在接下来的 1977 年，计算机科学家 [罗纳德·李维斯特 (Ronald Rivest)](http://amturing.acm.org/award_winners/rivest_1403005.cfm)、[阿迪·萨莫尔 (Adi Shamir)](http://amturing.acm.org/award_winners/shamir_2327856.cfm) 和 [伦纳德·阿德曼 (Leonard Adleman)](http://amturing.acm.org/award_winners/adleman_7308544.cfm) 在一篇题为 [《获得数字签名和公钥密码系统的方法》](https://people.csail.mit.edu/rivest/Rsapaper.pdf) 的论文中开发了 [RSA 加密系统 (RSA cryptosystem)](<https://en.wikipedia.org/wiki/RSA_(cryptosystem)>)——这是公钥密码学的第一个可行实现。Rivest 将该论文的副本寄给了数学家马丁·加德纳 (Martin Gardner)。加德纳对此留下了深刻的印象，以至于打破了他通常提前几个月规划专栏的规则，迅速撰写并 [发表在《科学美国人》1977年8月刊](https://web.archive.org/web/20230728001717/http://simson.net/ref/1977/Gardner_RSA.pdf) 上：

![Len, Adi, Ron at CRYPTO '82, and the now-famous article by Martin Gardner](img/overview/rsa-in-scientific-american.jpg)
**Len、Adi、Ron 在 CRYPTO '82 会议上，以及马丁·加德纳发表在《科学美国人》上的[那篇如今闻名遐迩的文章](https://web.archive.org/web/20230728001717/http://simson.net/ref/1977/Gardner_RSA.pdf)。**

在文章中，加德纳包含了一个 RSA-129 密码，并向第一个破解它的人悬赏 100 美元：

![MIT's RSA Challenge](img/overview/rsa-challenge.jpg)
**麻省理工学院 (MIT) 的 RSA 挑战。**

1994 年，一群计算机科学家和志愿者 [破解了该密码](https://en.wikipedia.org/wiki/The_Magic_Words_are_Squeamish_Ossifrage)，并将这笔钱捐赠给了 [自由软件基金会 (Free Software Foundation)](https://www.fsf.org/)。这一努力凸显了一个至关重要的观点：密码学中的绝对安全只是一种幻想。加密方法（如 RSA）正在不断演进，特别是为了应对 [量子计算机](/wiki/Cryptography/post-quantum-cryptography.md) 的威胁。

尽管如此，现代 RSA 加密（1024 到 4096 位）在信息高速公路上创造了一条安全的通道，使银行和信用卡公司能够保护金融交易。这培育了信任，并促进了电子商务和在线银行的增长。

![Inventors of modern cryptography](img/overview/inventors-of-modern-cryptography.jpg)
**现代密码学的发明者们：Adi Shamir、Ron Rivest、Len Adleman、Ralph Merkle、Martin Hellman 和 Whit Diffie 在 Crypto 2000 会议上。**

1997 年，英国政府解密了 1970 年的类似 [研究成果](https://cryptocellar.org/cesg/possnse.pdf)。

---

## 自由如风 (Free as in freedom)

在 1950 年代至 60 年代硬件和操作系统蓬勃发展的浪潮中，早期的软件往往很原始，需要修改，而软件源码并非秘密；事实上，分享源码是常态。这孕育了提倡探索和知识交流的业余爱好者 [“黑客文化” (hacking culture)](https://en.wikipedia.org/wiki/Hacker_culture)。任何人都可以检查、修改源码并提供反馈。计算机杂志甚至会刊登印刷的 [键入式程序 (type-in programs)](https://en.wikipedia.org/wiki/Type-in_program)，鼓励用户亲手编写软件：

![Type-in program from compute magazine](img/overview/type-in-program.jpg)
**《Compute!》杂志上用于备份数据的键入式程序。**

随着软件规模的增长和存储成本的下降，软件开始以磁带形式分发，通常由 IBM 等制造商与计算机硬件捆绑销售。这一做法由于 1969 年 [美国诉 IBM 反垄断诉讼](https://www.justice.gov/atr/case-document/united-states-memorandum-1969-case) 而宣告停止，该诉讼辩称用户被迫购买硬件才能使用捆绑的软件。尽管该诉讼后来被撤销，但它产生了反作用——公司抓住了机会，开始对软件单独收费。软件变成了一种商品。

Unix 是这一趋势的另一个牺牲品。最初免费分发给政府和学术研究人员，但到了 1980 年代初，随着 Unix 变得更加普及，AT&T 停止了免费分发，并开始对系统补丁收费。由于切换到其他架构面临重重挑战，许多研究人员选择购买商业许可。

为了增加收入，不分发源码成为大势所趋。一些公司甚至不遗余力地阻止软件分发。在一封臭名昭著的 [公开信](https://en.wikipedia.org/wiki/An_Open_Letter_to_Hobbyists) 中，[比尔·盖茨 (Bill Gates)](https://en.wikipedia.org/wiki/Bill_Gates) 要求爱好者们停止分享 BASIC 源码：

> “这是为什么？因为大多数业余爱好者必须意识到，你们中的大多数人都偷窃了软件。硬件必须付钱购买，但软件是可以分享的东西。谁会在乎开发它的人是否得到了报酬呢？”
> —— 比尔·盖茨，[《致爱好者的公开信》。](https://en.wikipedia.org/wiki/An_Open_Letter_to_Hobbyists)

在关于软件所有权的日益激烈的争论中，麻省理工学院 (MIT) 人工智能实验室的研究助理 [理查德·斯托曼 (Richard Stallman)](https://en.wikipedia.org/wiki/Richard_Stallman) 陷入了一场个人的战斗。由于无法修改新安装的施乐 (Xerox) 打印机的源码，他感到十分沮丧。他认为这种限制是“反人类的罪行”：

> “如果你做饭，你可能会交流食谱并与你的朋友分享，他们可以根据自己的喜好自由修改。想象一个世界，你无法修改你的食谱，因为有人不遗余力地对其进行设置，以至于无法更改。如果你尝试与朋友分享食谱，他们会称你为**海盗 (pirate)** 并把你送进监狱。”
> —— 理查德·斯托曼，在一部[纪录片](https://www.youtube.com/watch?v=XMm0HsmOTFI)中谈到

在 1983 年的一封 [电子邮件](https://groups.google.com/g/net.unix-wizards/c/8twfRPM79u0) 中，他宣布了他致力于开发名为 [GNU](https://www.gnu.org/) 的 Unix 自由替代方案的雄心：

![GNU announcement](img/overview/gnu-announcement.jpg)
**理查德·斯托曼以及他宣布 GNU 项目的[邮件](https://groups.google.com/g/net.unix-wizards/c/8twfRPM79u0)。**

GNU 是对 Unix 的公然挑战，并 [递归地](https://en.wikipedia.org/wiki/Recursion) 代表 “GNU's Not Unix (GNU 不是 Unix)”。他决定让该操作系统与 Unix 兼容，因为整体设计已经过验证且易于移植，并且兼容性可以让 Unix 用户很容易地从 Unix 切换到 GNU。

正如理查德所解释的，自由软件不仅仅关乎价格：

> “我需要解释的是，‘自由软件’ (Free software) 指的是自由 (freedom)，而不是价格 (price)。很遗憾，在英语中，‘free’ 这个词具有歧义——它有许多不同的含义。其中之一意思是‘零价格’，但另一个意思是‘自由’。因此，请想想‘言论自由’ (free speech)，而不是‘免费啤酒’ (free beer)。”

> 🎦 观看：[理查德·斯托曼谈论自由软件及其对社会的影响](https://www.youtube.com/watch?v=Ag1AKIl_2GM)

GNU 始于 1984 年 1 月。作为这项工作的一部分，理查德编写了 [GNU 通用公共许可协议 (GPL)](https://en.wikipedia.org/wiki/GNU_General_Public_License)。到 1990 年，除了一个组件——内核之外，GNU 已经找到或编写了该操作系统的所有主要组件。

无独有偶，计算机科学专业的学生 [林纳斯·托瓦兹 (Linus Torvalds)](https://en.wikipedia.org/wiki/Linus_Torvalds) 正在开发一个名为 Linux 的内核：

![Linux announcement](img/overview/linux-announcement.jpg)
**林纳斯·托瓦兹以及他发布 Linux 的[邮件](https://groups.google.com/g/comp.os.minix/c/dlNtH7RRrGA/m/SwRavCzVE7gJ)。**

最初的回复在几个小时内就送达了，在接下来的一年里，有数百人加入了开发。Linux 在 GPL 协议下发布，这完善了整个 GNU/Linux 操作系统。

在此过程中，Linux 实际上奠定了基于社会共识的软件开发蓝图：

> “在 1992 年底的非常早期，突然之间，我不再认识每一个人了。这不再只是我和几个朋友。而是我和成百上千的人。那是一个巨大的飞跃。”
> —— 林纳斯·托瓦兹

这些范围广泛的贡献者（从个人爱好者到大型企业）共同协作来改进内核、修复漏洞并实现新功能。它实际上为后来塑造开源软件运动奠定了蓝图。

开源运动与自由软件运动有所分歧，它更侧重于开放源码的实用性优势。这种方法在社区驱动的创新与商业可行性之间达成了平衡，从而带来了广泛的商业采用。自由和开源软件 (FOSS) 是自由软件和开源软件的包容性统称。

GNU/Linux 见证了这样一种理念：软件应该赋予用户权利，而不是限制用户。

> 🎦 观看：[《操作系统革命》：关于 GNU/Linux 的纪录片](https://www.youtube.com/watch?v=k0RYQVkQmWU)

---

## 密码朋克写代码 (Cypherpunks write code)

自第二次世界大战结束以来，政府一直对密码学的进步进行着铁腕控制，并据此予以防范。在美国，加密技术受 [军火清单 (Munitions List)](https://en.wikipedia.org/wiki/United_States_Munitions_List) 的控制。这意味着 [国家安全局 (National Security Agency, NSA)](https://en.wikipedia.org/wiki/National_Security_Agency) 对密码学的进展极为关注。

当 NSA 收到麻省理工学院寄来的 RSA 论文副本时，他们试图将该研究列为机密，但最终允许发表：

![NSA cryptology debate](img/overview/nsa-cryptology-debate.jpg)
**NSA 对[2009年依据《信息自由法》提出请求](https://cryptome.org/2021/04/Joseph-Meyer-IEEE-1977.pdf)的回复。**

当 NSA 员工 Joseph Meyer 写给 IEEE 的一封私人信件（指出密码学出版物需要政府批准）被公开时，NSA 的做法引发了公众的极大批评。

这标志着政府与密码学倡导者之间 [密码战争 (Crypto Wars)](https://en.wikipedia.org/wiki/Crypto_Wars) 的开端。

![NSA cryptology debate](img/overview/nsa-crypto-wars.jpg)
**《科学》杂志关于密码学辩论的[文章](https://www.science.org/doi/10.1126/science.197.4311.1345)。**

政府削弱密码学的企图被视为监视公众通信的手段。

![NSA cryptology debate](img/overview/wire-tap-surveillance.jpg)
**《科学》杂志关于窃听的[文章](https://www.science.org/doi/10.1126/science.199.4330.750)，以及班克西 (Banksy) 在英国的相关街头艺术。**

将密码学作为保障通信安全手段的研究在整个 1980 年代继续演进。

1985 年，密码学家 [大卫·查姆 (David Chaum)](https://en.wikipedia.org/wiki/David_Chaum) 发表了突破性的论文 [《无需身份识别的安全性：使老大哥变得多余的交易系统》](https://dl.acm.org/doi/pdf/10.1145/4372.4373)，在文中他描述了提供安全和隐私的交易方案。他还提出了利用密码学为个人建立“数字假名”的激进想法。

![David Chaum](img/overview/david-chaum.jpg)
**大卫·查姆及其论文。**

面向大众的密码技术普及是由 Phil Zimmermann 在 1991 年开发的加密程序 [Pretty Good Privacy (PGP)](https://en.wikipedia.org/wiki/Pretty_Good_Privacy) 所推动的。PGP 允许个人使用强加密来保护他们的通信和数据。

密码战争在 1993 年继续升级，当时 Zimmermann 因涉嫌违反密码学软件出口限制而成为美国海关总署刑事调查的对象。

在一项标志性的举动中，他将 PGP 的全部源码出版在一本 [精装书](https://philzimmermann.com/EN/essays/BookPreface.html) 中，辩称书籍的出口受 [第一修正案 (First Amendment)](https://en.wikipedia.org/wiki/First_Amendment_to_the_United_States_Constitution) 的保护。这些书籍按照美国出口法规从美国出口，然后在欧洲将这些页面扫描并通过光学字符识别 (OCR) 还原，使源码在电子形式上可用。为了表示支持，一些活动人士将源码印在 T 恤上。

该案于 1996 年被撤销。

> “我以前觉得自己就像霸王龙背上的一只跳蚤。现在我觉得自己可能是霸王龙背上一只不断吠叫的小贵宾犬。”
> —— 菲尔·齐默曼 (Phil Zimmermann)

![Phil Zimmermann ](img/overview/phil-zimmermann.jpg)
**Phil Zimmermann、印有 RSA 源码的 T 恤，以及一位扫描 PGP 书籍的欧洲志愿者。来自全欧洲的 [70 多人付出了 1000 多个小时，才使 PGP 在美国以外顺利发布。](https://www.pgpi.didisoft.com/pgpi/project/scanning/)**

1992 年初，在 PGP 2.0 发布的同一周，三位湾区工程师—— [埃里克·休斯 (Eric Hughes)](<https://en.wikipedia.org/wiki/Eric_Hughes_(cypherpunk)>)、[蒂莫西·C·梅 (Timothy C. May)](https://en.wikipedia.org/wiki/Timothy_C._May) 和 [约翰·吉尔摩 (John Gilmore)](<https://en.wikipedia.org/wiki/John_Gilmore_(activist)>) 聚在一起，发起了一个名为 [密码朋克 (Cypherpunk)](https://mailing-list-archive.cryptoanarchy.wiki/)（密码 cipher + 赛博朋克 cyberpunk）的邮件列表。

密码朋克演变成一场决定性的运动，拥有包括 Zimmermann 在内的 700 多名准备用代码进行反击的活动人士和反叛者：

> “密码朋克写代码。我们知道必须有人编写软件来保护隐私，我们就要去写它。
> ……
> 因此，密码朋克致力于密码学。密码朋克希望学习它、教授它、实现它并使之更加普及。密码朋克知道密码协议构成了社会结构。密码朋克知道如何攻击系统以及如何防范。密码朋克深知制作优秀的密码系统有多么艰难。”
> —— 埃里克·休斯

![Phil Zimmermann ](img/overview/cypherpunks-write-code.jpg)
**Tim、Eric 和 John（上）。Eric 的密码朋克[邮件](https://mailing-list-archive.cryptoanarchy.wiki/archive/1992/09/fdf9c19e77ec3f1a9bbc6bc19266d565b89d19dbd0ad369f5a2e800af3fc9558/)（下）。[《密码朋克宣言》 (The Cypherpunk's Manifesto)](https://www.activism.net/cypherpunk/manifesto.html)（右）。**

在 1994 年的一次会议上，Tim 描述了密码朋克的核心信念：

> 虽然没有官方规定，但大多数列表成员似乎持有以下一系列浮现的、一致的信念：
> - 政府不应该能够窥探我们的事务
> - 保护对话和交流是一项基本权利
> - 这些权利可能需要通过*技术 (technology)* 而不是通过法律来得到保障
> - 技术的力量往往会创造新的政治现实（因此有了该列表的座右铭：“密码朋克写代码”）

在他 1988 年的 [《加密无政府主义宣言》("Crypto Anarchist Manifesto")](https://groups.csail.mit.edu/mac/classes/6.805/articles/crypto/cypherpunks/may-crypto-manifesto.html) 中，Tim 引入了“加密无政府主义”的政治哲学，它反对一切形式的权威，并且除了由密码学描述并由代码执行的规则之外，不承认任何法律。

![Crypto Anarchist Manifesto](img/overview/crypto-anarchy.jpg)
**无政府主义以及 Tim May 的《加密无政府主义宣言》。**

该宣言将匿名数字交易视为个人自由的基石。

遗留的拼图：**一种加密原生 (crypto-native) 的 [数字货币 (digital currency)](https://en.wikipedia.org/wiki/Digital_currency)。**

> 🎦 观看：[Tim 思考加密无政府主义的 30 年](https://www.youtube.com/watch?v=TdmpAy1hI8g)

---

## 寻找遗失的拼图 (Search for the missing piece)

在整个 90 年代，密码朋克们做出了几次创建数字货币的尝试。

1990 年，大卫·查姆引入了 [DigiCash](https://en.wikipedia.org/wiki/DigiCash)，首次展示了匿名数字经济的曙光。然而，它依赖于现有的金融基础设施，并且在很大程度上是中心化的。最终，DigiCash 于 1998 年申请破产。

![DigiCash Homepage](img/overview/digicash.jpg)
**DigiCash 网页主页。**

E-gold 随后于 1996 年出现，由储备的实物黄金提供支持。在鼎盛时期，e-gold 拥有 [350 万个注册账户](https://web.archive.org/web/20061109161419/http://www.e-gold.com/stats.html)，每年促进价值数十亿美元的交易。然而，在 2009 年，由于法律问题，转账被暂停。

后来的方案专注于摆脱黄金等抵押物，而是通过数字化来控制稀缺性。1998 年，[戴伟 (Wei Dai)](https://en.wikipedia.org/wiki/Wei_Dai) 提出了由密码学函数驱动以创造货币的 [B-money](https://web.archive.org/web/20220303184029/http://www.weidai.com/bmoney.txt)。2005 年，[尼克·萨博 (Nick Szabo)](https://en.wikipedia.org/wiki/Nick_Szabo) 设计了 [比特黄金 (BitGold)](https://web.archive.org/web/20240329075756/https://unenumerated.blogspot.com/2005/12/bit-gold.html)，但从未付诸实现。这两者都没有成功获得主流采用，但它们的设计影响了最终使数字货币成为现实的事物——比特币 (Bitcoin)。

![Wei Dai and Nick Szabo](img/overview/wei-dai-nick-szabo.jpg)
**戴伟和尼克·萨博。**

---

## 比特币 (Bitcoin)

2008 年的金融危机重新激发了人们对数字货币实验的兴趣，尤其是将比特黄金 (BitGold) 带回了人们的视野。

一位署名为 [中本聪 (Satoshi Nakamoto)](https://en.wikipedia.org/wiki/Satoshi_Nakamoto) 的匿名作者在 2008 年一篇题为 [《比特币：一种点对点的电子现金系统》("Bitcoin: A Peer-to-Peer Electronic Cash System")](https://bitcoin.org/bitcoin.pdf) 的论文中，引入了关于如何实现无领导共识 (leaderless consensus) 的开放问题的解决方案。比特币将自己确立为一个分布式账本系统，其中数据在时间上以密码学方式链接到区块中。它也成为了第一个去中心化的数字货币，在没有底层抵押物的情况下运行，并消除了对银行等受信任的第三方中介机构的需求。

![A statue dedicated to Satoshi, and Bitcoin announcement post.](img/overview/satoshi-and-bitcoin.jpg)
**献给中本聪的雕像以及比特币的发布公告。**

比特币也是世界上见过的最大的社会经济学实验：

> “当中本聪在 2009 年 1 月首次启动比特币区块链时，他同时引入了两个激进且未经验证的概念。第一个是‘比特币’，这是一种去中心化的、点对点的在线货币，它在没有任何支持、内在价值或中央发行人的情况下维持着价值。到目前为止，作为货币单位的‘比特币’占据了公众的大部分注意力。
>
> 然而，中本聪的宏伟实验还有另一个同样重要的部分：基于工作量证明的区块链概念，以允许公众对交易顺序达成一致。”
> —— 维塔利克·布特林

为了利用新创建的数字货币，人们做出了[几次](https://web.archive.org/web/20230404234458/https://www.etoro.com/wp-content/uploads/2022/03/Colored-Coins-white-paper-Digital-Assets.pdf) [尝试](https://en.wikipedia.org/wiki/Namecoin) 在比特币网络之上构建应用程序。然而，为此目的，比特币网络被证明是十分原始的，并且这些应用程序是使用复杂且扩展性不强的临时方案构建的。

以太坊的出现是为了应对这些挑战。

---

## 以太坊世界计算机 (The Ethereum world computer)

2012 年，[维塔利克·布特林 (Vitalik Buterin)](https://en.wikipedia.org/wiki/Vitalik_Buterin) 和 Mihai Alisie 创立了 [《比特币杂志》 (Bitcoin Magazine)](https://en.wikipedia.org/wiki/Bitcoin_Magazine)——第一本致力于数字货币的严肃出版物。维塔利克很快发现了比特币的局限性，并 [提出了一个平台](https://web.archive.org/web/20150627031414/http://vbuterin.com/ultimatescripting.html) 以支持泛化的金融应用程序。

2014 年，在 [加文·伍德 (Gavin Wood)](https://en.wikipedia.org/wiki/Gavin_Wood) 的帮助下，[以太坊的设计得到了正式确立](https://ethereum.github.io/yellowpaper/paper.pdf)。

![Vitalik, Jeff, and Gavin working on Ethereum.](img/overview/ethereum-launch.jpg)
**维塔利克、杰夫和加文在开发以太坊。**

2015 年 7 月 30 日，以太坊 [正式上线](https://etherscan.io/block/1)，作为一个旨在利用数字货币为自主主权经济构建工具的平台。

截至撰写本文时，以太坊的市值已达 **4000 亿美元**。

> 📄 阅读：[维塔利克关于以太坊起源的文章](https://vitalik.eth.limo/general/2017/09/14/prehistory.html)
> 
> 🎦 观看：[Mario Havel 谈论以太坊哲学](https://streameth.org/ethereum_protocol_fellowship/watch?session=65d77e4f437a5c85775fef9d)
> 
> 📄 阅读：[以太坊的演进](/wiki/protocol/history.md)
> 
> 📄 阅读：[以太坊原始开发计划](https://hudsonjameson.com/files/Ethereum-Dev-Plan-preview.pdf)，以及讨论以太坊原始愿景的 [推特讨论](https://xcancel.com/hudsonjameson/status/1973786503659626893)。

---

## 资源 (Resources)

- 📄 计算机历史博物馆, ["The history of Computer Networking"](https://www.computerhistory.org/timeline/networking-the-web/)
- 📄 维基百科, ["ARPANET"](https://en.wikipedia.org/wiki/ARPANET)
- 📘 Brian K., ["Unix: A History and a Memoir"](https://www.amazon.com/dp/1695978552)
- 📄 CryptoCouple, ["A History of The World’s Most Famous Cryptographic Couple"](https://cryptocouple.com/)
- 📄 Steven E., ["The Day Cryptography Changed Forever"](https://medium.com/swlh/the-day-cryptography-changed-forever-1b6aefe8bda7)
- 📄 GNU, ["Overview of the GNU System"](https://www.gnu.org/gnu/gnu-history.en.html)
- 📄 Steven V., ["A look back at 40 Years of GNU and the Free Software Foundation"](https://www.zdnet.com/article/40-years-of-gnu-and-the-free-software-foundation/)
- 📄 David C., [“Security without Identification: Transaction Systems to Make Big Brother Obsolete”](https://dl.acm.org/doi/pdf/10.1145/4372.4373)
- 📄 Steven L., ["Wired: Crypto Rebels"](https://web.archive.org/web/20160310165713/https://archive.wired.com/wired/archive/1.02/crypto.rebels_pr.html)
- 📄 Arvind N., ["What Happened to the Crypto Dream?"](https://www.cs.princeton.edu/~arvindn/publications/crypto-dream-part1.pdf)
- 📄 Satoshi N., ["Bitcoin: A Peer-to-Peer Electronic Cash System"](https://bitcoin.org/bitcoin.pdf)
- 📄 Harry K. et al, ["An empirical study of Namecoin and lessons for decentralized namespace design"](https://www.cs.princeton.edu/~arvindn/publications/namespaces.pdf)
- 📄 Nick S,, ["Formalizing and Securing Relationships on Public Networks"](https://web.archive.org/web/20040228033758/http://www.firstmonday.dk/ISSUES/issue2_9/szabo/index.html)
- 📄 Nick S., ["The Idea of Smart Contracts"](https://web.archive.org/web/20040222163648/https://szabo.best.vwh.net/idea.html)
- 📄 Vitalik B., ["Ethereum Whitepaper"](https://ethereum.org/content/whitepaper/whitepaper-pdf/Ethereum_Whitepaper_-_Buterin_2014.pdf)
- 📄 Vitalik B., ["Ethereum at Bitcoin Miami 2014"](https://www.youtube.com/watch?v=l9dpjN3Mwps)
- 🎥 Gavin Wood, ["Ethereum for Dummies"](https://www.youtube.com/watch?v=U_LK0t_qaPo)
