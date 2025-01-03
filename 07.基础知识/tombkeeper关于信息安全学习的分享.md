## 前言

这是我整理的tombkeeper关于信息安全学习研究相关的经验聚合文章。

以下内容均来自tombkeeper的微博或知乎。

## 如何做事（编者加）

中国每年大概有五百万理工科毕业生。其中堪称优秀技术人才的自然是少数。然而，技术好又会做事的更是极少数。

什么是“做事”？就是确立目标、分析路径、规划资源、制定方案、迭代执行。优秀技术人才，自然是能把技术做好的人。但从做技术到做事，中间还有很远的距离。大部分人可能终其一生都不会做事。

前阵子一个同学跟我说想去BlackHat演讲。我说你是怎么规划这件事的？他一愣，半天说不出话。他定了目标，但没去想后面的事情。

然后我问他有没有注意到BlackHat有些演讲水平很高，但也有一些不太行的。他说确实是这样。然后我说你有没有分析过为什么会这样？他说没有。我说如果你分析一下那些不太行的演讲为什么也能上BlackHat，是不是会对你达成目标有帮助？他恍然大悟。

我说这只是一个小点，还不算完。你知道怎么才能上BlackHat？他想了一下说是投稿被接受。我说那么是不是应该收集既往演讲的内容摘要，分析被接受的投稿有什么特点。

然后我说你有没有想过BlackHat演讲者和主办方之间是什么关系？他说不知道。我说本质是商业合作关系。主办方给演讲者提供曝光机会和机票酒店，演讲者用自己的内容帮主办方把票卖出去。所以投稿要让别人看了觉得是有利于卖票的。

我又问你有没有想过Review Board里的人是怎么挑选演讲的？有没有想过如果你在Review Board你会看重哪些？有没有了解过Review Board里的那些人？其中和你技术方向相关的人有哪些？这些人历史上做过什么？他们可能会对哪些东西更青睐？

说了这么多，还只是分析路径这一步。后面规划资源、制定方案、迭代执行都还需要更多的思考。当然，这个同学刚毕业，不会做事很正常，后面还有时间慢慢学。但如果你毕业已经很久了，发现自己还不会做事，就要开始留心培养这方面的能力了。在学校里我们的成长主要靠学习知识。毕业后，更重要的成长是学会做事。

## 大学专业选择（编者加）

前阵子一个朋友忽然说要谢我，感谢我当初在他儿子考大学选专业时给的建议。我都不太记得这事了，想了好一会才回忆起来。

这个朋友一直觉得我们实验室特别好，所以 2019 年他表弟还在读大四的时候，他就想让表弟跟我学网络安全。我当时看了他表弟的简历，说这孩子将来的前途必然在 AI 上，没必要强行来搞安全。

几年后，这位朋友告诉我，他表弟读研究生了，方向就是 AI。后来他又告诉我，他表弟已经是学校人工智能实验室事实上的负责人了。而且他表弟在听说了我的遍历文档的学习方法后，也用到了对 AI 的研究上。不光自己这么做，还要求实验室的同学都这样。据说效果出乎意料地好。

前两年他儿子要考大学了，他来问我应该报什么专业。他当时还是想让儿子将来搞网络安全，于是问我是读网络安全专业还是读计算机专业。

我说将来干什么还得看兴趣，孩子未必就喜欢网络安全。听了他儿子的情况后，我建议他让儿子本科学数学。我说数学底子打好了，如果愿意搞网络安全，我可以教他怎么学。而如果喜欢搞别的，比如 AI，那么数学底子好也很有优势，别的都可以自学。

于是他就听我的建议让儿子读了数学系。最近他儿子去和他表弟学 AI，学的特别快，一下体现出数学底子好的优势。他很惊喜，所以特别来谢我。

刚才仔细回忆了一下这几年的事情，感觉我就是他家的定制版张雪峰。

## 遍历文档学习法

2005.10.02

这两天论坛上又有人开始抱怨世风日下，大家都现实了，都不开放了，不交流了。对这种“月经贴”，我基本上已经习惯了，不过因为吃了粉皮炖鸡，心情比较好，于是就说了两句。

三四年前，当时我对人性的看法还不像现在这样。有几个人加了我的QQ，说想学Windows，我居然就好为人师起来，自不量力地教人学Windows。我很天真地把自己的经验告诉他们：

一、先把Windows的帮助文件从头到尾看一遍。
二、在Windows目录下搜索*.txt、*.htm?、*.log、*.ini，把每一个文件内容都看一遍。
三、把注册表浏览一遍。

没有诀窍，也不用花钱买书。任何人把这三步做完之后，只要不是傻子，在Windows应用方面都可以非常熟练。并且如果想进一步学，也自然知道应该去看什么了。

结果甚至没有一个人能看完Windows帮助文件，看完三分之一的都没有，都说看不下去。我很奇怪，我看Windows的帮助文件就像看金庸小说一样愉快，怎么会有人觉得辛苦？

后来我想明白了：因为我爱她，而他们不爱她，只是想占有她而已。

他们要的不是交流，不是开放，甚至也不是想找个人“拜师”，他们想要的不是郭靖遇到的洪七公，而是虚竹遇到的无涯子。

再后来，一个偶然的机会，我看到了小四同学写的那篇《你尽力了么？》，才知道原来这不只是我一个人的看法。

这两天在家，在笔记本上折腾Linux，遇到了很多问题，我就把内核每一个编译选项的说明都细细看了一遍，反复编译上二十多遍——然后，所有问题的答案都找到了。显然，学Linux和学Windows的方法并没有什么不同。

## 为什么很多高中生都能比大学生强

前几天一个老朋友找我，他需要找一个某方向的技术合作伙伴。这个方向需要对操作系统达到专家级的熟悉程度，做起来并不容易。我认识的能做这个的人都已经在大公司里有很好的职位了，于是我给他介绍了一个相关的开源项目。他联系了那个开源项目的作者，想拉他入伙。结果发现作者今年九月刚上大一，代码是高中时写的。

任何领域，是否能学有所成，除了天分原因外，还需要：

1. 能接触到所需的学习资源
2. 投入了足够的时间精力

文科的学习资源主要是书，相对容易获取，所以我们可以看到很多自学成才的作家。但多数理工科的重要学习资源都不太容易获取。比如你想学生物，就很难在家建立起实验室。

计算机领域（不止信息安全）是个例外。只需要有一台能联网的计算机，就可以学很多东西。那么这时候就主要看是否投入了足够的时间精力。

你们可以回忆一下自己的大学生涯，身边有多少人每天能投入 12 个小时在学习上？投入 8 小时的有多少？哪怕投入 4 小时的有多少？四年下来读了多少行代码？写了多少行代码？

所以被热爱技术、投入很多时间学习的高中生超过有什么奇怪的。

## 想读某个导师的博士应该如何与之沟通

有个学电子工程的同学想去读一个网络安全方向导师的博士，问我该怎么找那个导师说。这类情况，我建议先把导师团队过去几年做的东西都看看，再把其他相关研究者的工作也看看。然后对这个方向进行一些思考，争取能有点自己的想法。然后再去找那个导师聊聊自己对安全的热爱，和对相关领域的学习和理解。

## 学习网络安全的最佳路线是什么

“怎么学XX”这样的问题，有点类似问路。别人只能给你指个方向。但多数人想要的都是 GPS，给他规划好详细路线，到哪儿该变道，实时路况怎么样。多学一点额外的东西，就感觉自己吃亏了。

“从西直门到天安门怎么走？”

“坐地铁二号线到复兴门再转一号线。”

“我看地图这样是走了一个直角，感觉绕远了。有没有直接去的路？”

“有啊，你飞过去。别飞太高，不然起飞降落要多花不少力气。”

## 如何才能系统地学习逆向

我也是先学WEB后学逆向的。从我的经验看，首先应该抛弃“系统的学习”这个想法。因为逆向需要用人的自然语言逻辑去适应程序的机器指令逻辑，痛苦指数比较高。要拮抗这个痛苦，需要持续有正反馈。但“系统的学习”相对难以持续提供正反馈。

所以比较好的方式是给自己找一些难度中等，又比较有意思的目标来入手，比如给 notepad.exe 的菜单里增加一个功能。在这个入手过程中，先不用想系统不系统的问题，搞到哪儿算哪儿。入手之后，完成了最痛苦的那个门槛翻越，后面就和学别的差不多了。

## 大学培养计划的问题

最近几年帮一些大学的网络安全专业评审过几次人才培养计划，本硕博都有。感觉存在一些共性问题。

其中一个是部分学校的网络安全专业本科课程没有数电和模电。软工不开数电模电也就算了，但网安专业在未来的工作中可能需要面对从芯片、信号到代码、数据的各种问题，真的还是需要学这两门课的。尤其是在当前的技术发展形势下。

当然，如果本科没有这两门课，研究生阶段如果是硬件安全或无线安全的方向，再来学这两门课也不是不可以。但这就要求本硕培养计划能衔接上。而且我看到一般硕士培养计划里似乎也没有考虑这个问题。

## 行业观察（编者加）

每当网络安全行业发展陷入停滞，行业里的第一阶级就会新造一个词。然后第二阶级打开文档，用新词进行查找替换。第三阶级则开始思考拿什么产品改一改能贴上这个新词。

## 在重复的工作中有所收获（编者加）

我刚参加工作的时候，做了很长一段时间“低级”“无意义”的工作。就是给 IDS 加一些针对古老攻击工具的规则。

那些木马和漏洞有些甚至古老到连运行环境都不好找了。所以加那些规则对检测能力来说确实没有意义，永远都不会发挥作用。但竞争对手有这个规则你就也得有。你看，其实连网络安全产品也是“内卷”的。

工作的步骤就是搜索、下载、运行、抓包、看特征、写规则。然后再重复，再重复，重复很多遍。有时候为了应付一个投标前的测评，每天要加接近一百条规则。就这么重复，干到十一二点。

但通过做这些重复的、“低级”的、“无意义”的工作，我熟悉了各类漏洞的原理，学会了很多分析工具的使用，熟悉了大量通用的和私有的网络协议，看到了各种系统可能出问题的地方。如果 2002 年没有做过这些“低级”“无意义”的工作，我大概也不会对网络协议有那么深刻的理解，不会在 2016 年发现 BadTunnel 漏洞。

当然，如果只是机械地去重复工作过程，而不在工作中学习和思考，那不管做什么工作，都不会有长进。对这类人来说，工作确实只是消耗过程，而不能成为增益过程。

## 一个女生怎样才能进入腾讯的玄武实验室？

『本人是一位热爱网络安全技术的女生，大二开始接触网络安全，因为热爱，大部分时间都在自学网络安全方面的知识技能，有学习主动信息收集，被动信息收集，无线攻击，密码破解，web渗透,metasploit,linux等渗透测试所需技能，正学习python。不断地学习中发现，可能因为网络安全涉及面较广，所以需要掌握的知识较多但不深，而心中有大牛的目标，向往玄武这种顶级的国内安全研究实验室，查看各种资料发现玄武对技术的要求非常专一且深入，再加上玄武好像只有一位女生，有些迷茫不知朝哪个方向努力，才能离玄武更近一步。』

很多人以为我是专门研究二进制安全的，所以偏向二进制方向。其实去年发现的 BadTunnel 就不属于二进制方向：《BadTunnel：跨网段劫持广播协议》[BadTunnel：跨网段劫持广播协议](https://mp.weixin.qq.com/s?__biz=MzA5NDYyNDI0MA==&mid=2651953600&idx=1&sn=7779613d4886c7739a9bbf076b7195f9&scene=0)。你上面提到的东西，我基本也都搞过。

这是很早以前一份简历里的内容：

- 2002年1月，发现了CheckPoint MetaInfo Sendmail管理接口执行命令漏洞。
- 2002年1月，发现了Snitz Forums 2000 Version 3.1 SR4的管理界面认证绕过漏洞。
- 2002年4月，和一个朋友共同发现了PhpBB等多个论坛程序的BBCode函数拒绝服务漏洞（bugtraq ID 4432、4434）

前几年研究浏览器的二进制漏洞时也同时研究过 Web 前端相关的问题：《浏览器和本地域》[网页链接](https://hitcon.org/2013/download/[I1] Tombkeeper - 瀏覽器和本地域_HITCon2013.pdf)

你说的无线攻击我也研究过：《来自空中的威胁》[网页链接](https://www.cert.org.cn/upload/cncertcc06/0330/ct1/5.060330-YuYang-ThreatsFromAir.pdf)

渗透测试做的就更多了。以前在乙方公司，我是渗透测试主力。还记得最忙的一个项目，我和部门领导一起，两个月做了 184 台服务器的渗透测试。

所以我可能是最不会对各类安全技术有偏见的人。

无论哪类技术，背后的需要的东西都是类似的。AFL 这个革命性的二进制漏洞 Fuzz 工具的作者，也是《The Tangled Web: A Guide to Securing Modern Web Applications》这本 Web 安全圣经的作者。

我们实验室有几个做 Windows 方向的同学， 毕业前都没接触过 Windows 安全。但在实验室工作半年后，就开始能做出一些成果。一年多之后，就已经能做出很不错的东西。当然，这并不意味着干这行很容易，人人可为。这几位同学虽然之前没接触过 Windows 安全，但在校期间，在其它方向上都做出了一些东西。我记得其中一个同学，毕业前硬啃了几个月 Apache 源码，发现了一个漏洞。在 Apache 里还能搞出东西来，是不容易的。

工作并不意味着学习的结束，而是另一个开始。我们大多数人至少都要工作三十年，这三十年里还会有很多东西要学。所以不用太在意之前学了什么，更重要的是看你还能学会什么。无论之前学过什么，将来都要再学新东西。无论之前学的是什么，将来也都会有用。艺无止境，功不唐捐。

## 答一位热爱网络安全的同学的来信

谢谢对玄武实验室的关注和对我的信任。

实验室对实习同学的要求，其实主要就两点：

1. 对信息安全有比较整体的认识；
2. 至少钻研过一两项工业界的技术，有深入的了解和一定的成果。

读了来信，我感觉你是一个勤奋好学的人，这很难得。大学时代是最适合学习的时候——这听起来有点像废话，但在这段时间里，你有很多时间，而没有生存压力，你的记忆力处于一生中的巅峰状态，你即使一夜不睡也可以很快恢复精力。毕业后可能再也不会有同时具备这些条件的时间了。

有了勤奋，接下来主要是在什么方向上勤奋的问题。

技术可以分为学术界和工业界，这两者都有意义，互有交叉，但其实区别也很大。企业需要的技术主要是工业界的技术，而在学校的环境里，不少人学术上不错，但对工业界的方向和需求了解比较少。前阵子有位高校的青年教师给我写信，谈了他做的研究。我发现他的技术很好，但做过的东西离工业界比较远。北美华人安全学术界的宋晓东（Dawn Song）教授、蒋旭宪（XuxianJiang）教授都是把学术和应用结合的比较好的代表。你们可以读一读他们的研究，体会一下。

还有一个比较好的方法是阅读工业界安全会议的资料。从这些资料中也许学不到什么技术细节，但可以看看大家在做些什么，哪些自己比较感兴趣，然后选一两个方向，寻找相关资料深入学习。

## 信息安全走向漫谈

村长<airsupply#[0x557.org](http://0x557.org/)>邀我来B105沙龙和大家闲扯。而我近来的工作是拉磨居多，接客其次，实在没有什么新货。村长说：不必讲技术，可以谈谈“信息安全的现状和未来”。我思前想后，觉得这个题目纲领性太强，我这点资历讲起来显然自不量力。还是改称“信息安全走向漫谈”显得比较低调。漫谈漫谈，就是漫天乱谈，谈错了不要紧。万一谈得对，就算蒙上了。

### 1. 学什么技术不会过时？

常有人跟我发牢骚，说搞技术太累，总要学新东西。还总问，安全技术未来的方向是什么，学什么技术不会过时，五年十年之后还能混饭？

每到这时我都很尴尬，不知道应该说什么。

有些朋友知道，我读了五年医科大学。很多人认为医生是一个稳定的职业，医学是保值的知识，学完了就可以躺在上面吃一辈子。其实恰恰相反。稍大一点的医院都有自己的图书馆。年轻医生就不用说了，很多六十多岁的老医生上门诊，抽屉里还放着一本专业书，有病人的时候就给人看病，没病人就拉开抽屉看书。以前儿科的老主任对我说过：干医生这一行，半个月不去图书馆读文献，就落后了。毛主席说“三天不学习，赶不上刘少奇。”医生这一行就是这样。

我们家乡有句俗话，叫“看别人吃豆腐觉得牙齿快”。很多技术人员都觉得销售这个活儿好干，工作就是吃喝玩乐混关系，挣钱又多，还不费脑子。我只能说“那你去试试看吧”。先甭说销售绝对不是不费脑子的活儿，也绝对不是光靠“吃喝玩乐混关系”就行的。就算是，这“吃喝玩乐混关系”七个字岂是简单的？谁刚从台上下来满怀落寞但还有一些重要关系？谁虽然只是个普通教授但是诸多弟子都身居要职？谁手里有指标但自己不能完全做主？谁行政级别高但没有实权？新上来一把手是爱人民币爱高尔夫爱燕鲍翅还是爱制服捆绑？京城里何处灯最红何处酒最绿？——这些信息都是动态的，变化的，而且靠订阅邮件列表和看BBS是得不到的。不收集，不学习，咋整？

公交车站牌上贴的那些招聘职位，没有学历要求，只要“形象好，气质佳，思想开放”就可以“日薪1500以上”，这个钱挣起来算是容易又轻松了吧？其实即便从事这种地球上最古老的职业，学和不学那也是大大不同。苏小小、李师师的时代，要上头牌都得会琴棋书画，填词唱曲。到了十里洋场上海滩，根据才情高低也要分出“书寓”、“长三”、“幺二”来，啥都不会就只能混“野鸡堂子”。现在没那么多讲究了，不过“一剑穿心毒龙钻，冰火红绳空中飞”这些基本业务总还得学，要不然也还是“野鸡堂子”、Street-Walker的命。

那究竟有没有什么是学了不会过时的呢？学会学习的方法，学会从学习中获得快乐，这是永不过时的。如果享受不了汲取知识的快乐，那就不适合做任何需要脑力的工作。

### 2. 信息安全的未来如何？

最近一两年，大家感觉信息安全形势比前两年要好些了，不再像2002、2003年的时候，漏洞满天飞，蠕虫遍地爬。于是有人开始担心：漏洞少了的确有利于信息安全，但这样下去，最终会不会导致我们失业？

对此我是这样看的：信息安全技术的发展将来一定会有技术方向上的变化，但不会有前途上的问题。

电影《笑傲江湖》中任我行说：“有人就有江湖 ”。从有马帮的那一天起，就有马贼；从有海船的那一天起，就有海盗。有盗贼怎么办？理论上靠官府，实际上靠自己。自己搞不定怎么办？花钱找镖局。几千年来，什么时候这个格局改变过？过去镖局保的是金银，今天我们保的，归根结底也还是金银。从这个意义上讲，我们这个行业其实是镖局发展进入信息时代后，出现前面说的“技术方向上的变化”，而化生出来的。

所以，只要人类社会还存在信息交换行为，只要这些信息交换涉及到利益，就会有人试图改变这些利益的分配规则，就会有对信息安全的需求。百川归大海，这是一个根本法则，不管中间怎么九曲十八弯，最终，这个根本法则是不会有什么变化的。

大家感觉信息安全形势比前两年要好，可能一个主要原因就是看到软硬件厂商对安全越来越重视，安全措施越来越多。而看起来，比较严重的安全漏洞似乎有减少的趋势。互联网上几乎每台机器都有防火墙保护。新的Fedora Core默认开启了Linux的很多安全特性，而且看起来以后会一直这样下去。微软将要发布的Vista也似乎是一个强健无比的系统。这一切仿佛都在暗示信息安全会成为一个历史阶段性的事物，随着安全形势的进一步好转，这个行业也会逐渐淡去。

下面我们具体来谈谈这些问题。

Vista是个纸老虎

很多搞Windows安全的人最近都着实被微软的Vista给吓着了，觉得以后Windows就安全了，没什么可搞了。微软号称这个系统比前代大大增强了安全性。不过大家别忘了，微软推出Windows 2000的时候是这么宣传的，推出Windows XP的时候是这么宣传的，推出Windows 2003的时候也是这么宣传的。

当然，实事求是地说，从我们最近一段时间对Vista Beta版的研究来看，这个新系统的确采取了很多新的安全特性，大大增加了传统漏洞的利用难度；新的开发过程和开发工具也的确降低了漏洞发生的几率，比起之前的产品在安全上可以说有一个大飞跃。

但关键问题是：人们真的会接受这样一个用大量确认窗口和限制措施来虐待用户的操作系统么？更别提可怕的资源占用和乌龟般的速度了。至少我是绝对不会用这个东西的。估计在Vista正式上市后，各种Windows优化软件肯定会立即提供关闭这些安全特性的功能。

信息安全，信息为肉，安全为骨。肉无骨则不立，骨无肉则不活。蚯蚓之类，只有肉没有骨，尚可以慢慢蠕动，可以不太精彩地活下去；但是没有肉，光剩骨头，什么动物也活不了。安全措施对用户的扰动越小，就越容易被接受。时刻发挥作用，却几乎感觉不到它的存在，这就是安全工作的至善境界，也是最难达到的目标。要不然为什么杜雷斯的超薄型卖得贵还那么受欢迎。

在我看来，Vista就是个至少一厘米厚的杜雷斯。

任何企业的目标都是挣钱，只有当维护用户安全和挣钱这个目标恰好吻合时，它就会设法增强用户安全，如果维护用户安全影响了挣钱这个目标，它一定会考虑重新调整两边的砝码。

今天我在这里关起门来做个大胆的预言：Vista终将成为一个类似Windows ME那样没什么人愿意用的失败产品。微软甚至可能会在Vista后续的Service Pack或者下一代操作系统中取消或者减弱一些影响用户体验的强制安全特性。

另外，这个一厘米厚的杜雷斯是否真的就比0.03mm的超薄型安全333.33倍？值得怀疑。Windows 95只有1500万行代码，Windows 98有1800万行代码，Windows XP 有3500万行。Vista 则有5000万行，比XP多出了40%。新代码带来新功能，同样，新代码也必然会引入新漏洞。

毛主席说过，“一切反动派都是纸老虎。看起来，反动派的样子是可怕的，但是实际上并没有什么了不起的力量。从长远的观点看问题，真正强大的力量不是属于反动派，而是属于人民。”在我看来，Vista也很可能是个纸老虎。假以时日，纸老虎的软裆必然会被一一发现。

退一步讲，即便Vista、Fedora Core等为代表的新一拨操作系统真的是一个漏洞都没有了，从整体上看，信息安全态势仍然不乐观。

未来，各行各业各领域都将数字化，生活的方方面面都离不开信息技术。各种不同的信息技术交织在一起，构成巨大的宏信息世界，比我们现在的互联网大得多，复杂得多。安全问题也会随之复杂和严重。现在个人电脑上那一点点漏洞到那时候看就很渺小了。想想看，以前是你的电脑被入侵，以后你的电饭锅被入侵；过去是让你上网慢，以后让你吃夹生饭。

既然这样，为什么我们还要用信息技术呢？这一页PPT的标题叫做“信息技术是个狐狸精”。狐狸精，大家知道是什么吧，就是你知道她可能会害你，但身不由己。为什么呢？还是因为好，足够好到让你愿意冒这个险。

上高中时候，学摄影，用胶片，大夏天也要自己闷在暗房里冲洗。现在都是数码相机，随拍随看，多么方便。上大学的时候，大家都用“Walk-Man”，阔气一点的听随身CD、MD。现在一个火柴盒大小的MP3就都搞定了，价钱也便宜。

我在网上买过一个二手CF卡。拿回来，用数据恢复软件一看，里面有几张刑事案件的现场照片，血迹斑斑。还拍了受害人。这个卡不知道是哪个派出所淘汰下来的。如果追究起来，肯定要有人受到不好的影响。这种事情，在用胶片的时代不会有。但今天，新问题就出现了。

这些是什么啊？是狐狸精。狐狸精太好了，你很难舍弃。但糖衣好吃，炮弹难挡。信息技术发展的这么快，配套的安全很难跟得上。大家想想，汽车发明多少年后才有的安全带、安全气囊？有了安全带、安全气囊，每年还要车祸死多少人？每年车祸死这么多人，大家是不是还一样开车？这就是狐狸精，死都离不开。

今年世界杯，就有人提出，要在每个球员身上装RFID，对球员进行实时精确定位，用来作为裁判的辅助依据。虽然最后没有这么干，但这一天迟早会到来（注：后来才知道，马拉松比赛已经开始用RFID来跟踪选手的位置）。

（此处省略几页对RFID安全问题科普讲解的PPT内容）

刚才讲的比较科幻。落到现实中来，你我能看到的未来若干年中，可能的新热点有：Wi-Fi、蓝牙、WiMax、UWB等无线通信技术的安全问题，RFID的安全问题，消费类个人电子产品的安全问题，数字家电的安全问题，等等。

人类社会用信息技术越多，意味着越多的的财富和荣誉将由数字介质承载，越多的风险将是信息安全风险，信息安全技术就越重要，这个职业就越重要。

现在国外有种职业叫保安顾问，保障你现实世界的安全。有为企业服务的，也有为个人服务的。我瞎猜一下：在未来，可能要有类似这样的信息安全顾问。而且可能比较普及，就像现在国外的私人医生、私人律师一样。 退一万步说：信息安全企业也许会消失，但信息安全这个职业永远不会消失——只要人类还使用信息技术。

谢谢大家。

## 安全研究者的个人成长

### 个人成长

- 确立个人方向，结合工作内容，找出对应短板
  - 该领域主要专家们的工作是否都了解？
  - 相关网络协议、文件格式是否熟悉？
  - 相关的技术和主要工具是否看过、用过？
- 阅读只是学习过程的起点，不能止于阅读
  - 工具的每个参数每个菜单都要看、要试
  - 学习网络协议要实际抓包分析，学习文件格式要读代码实现
  - 学习老漏洞一定要调试，搞懂别人代码每一个字节的意义，之后要完全自己重写一个Exploit
  - 细节、细节、细节，刨根问底

### 建立学习参考目标

- 短期参考什么？比自己优秀的同龄人
  - 阅读他们的文章和其他工作成果，从细节中观察他们的学习方式和工作方式
- 中期参考什么？你的方向上的业内专家
  - 了解他们的成长轨迹，跟踪他们关注的内容
- 长期参考什么？业内老牌企业和先锋企业
  - 把握行业发展、技术趋势，为未来做积累

### 推荐的学习方式

- 以工具为线索
  - 一个比较省事的学习目录：Kali Linux
  - 学习思路，以Metasploit为例：
    - 遍历每个子目录，除了Exploit里面还有什么？
    - 每个工具分别有什么功能？原理是什么？涉及哪些知识？
    - 能否改进优化？能否发展，组合出新的功能？
- 以专家为线索
  - 你的技术方向里有哪些专家？
  - 他们的邮箱、主页、社交网络帐号是什么？
  - 他们在该方向上有哪些作品？发表过哪些演讲？
  - 跟踪关注，一个一个学

### 处理好学习、工作和生活

- 学习、工作和生活是矛盾统一的
- 三者都需要时间，你一天只有24小时
  - 调和矛盾的关键：提高效率
- 对没有一个好爸爸的人来说，你的学习、工作会影响你能不能追求诗和远方
- 有好爸爸也要学习，因为能力之外的资本等于零：

![image-20240228192842870](https://blog.wohin.me/posts/tombkeeper-sec-learning/image-20240228192842870.png)

### 如何提高效率

- 做好预研，收集相关前人成果，避免无谓的重复劳动
- 在可行性判断阶段，能找到工具就不写代码，能有脚本语言写就不要用编译语言，把完美主义放在最终实现阶段
- 做好笔记并定期整理，遗忘会让所有的投入都白白浪费
- 多和同事交流，别人说一个工具的名字可能让你节约数小时
- 咖啡可以提高思维效率，而且合法
- 无论怎么提高效率，要成为专家，都需要大量的时间投入

## 如何学习

学校教育的方式是：由浅入深，先理论再实践，多门基础课一起平面推进。这种方式的好处是学得扎实，适合批量培养人。缺点是出活儿慢，没有利用人的内驱力。

师傅带徒弟或者自学的情况下就不一定要按批量教学的方式来。

我个人的经验是不管会不会，先动手搞起来。而且不搞太入门的，要难度中等，这样才有成就感，能形成正反馈，调动内驱力。过程中会遇到很多不懂的东西，没关系，遇到什么就去学什么。这个阶段不求多求全，以把手头的东西搞起来为目标。搞成了再设定一个更难的新目标。新老目标之间要有继承性。最后等高难度目标也能搞定了，再转过头系统性地去看看相关技术资料，加固一下地基。

## 如何研究

十几年前流行过一篇文章叫《把信带给加西亚》。前些年又流行过批判这篇文章。研究工作大多和那位送信者面临的情况差不多：有一封信交给你，让你送给加西亚。加西亚在哪儿？没人知道。你得自己找，把信送到。

做研究工作，第一步是资料查找。这是从事研究探索最基础的能力，非常重要。而且也没有想象的那么简单，并不是每次把想问的问题输入搜索引擎都能得到答案。所以没找到答案可能不是没有答案。具体用哪些关键词、怎么组合这些词、怎么根据第一步搜索结果中的线索再提取关键词、怎么判断搜索结果有效性，等等，这些都需要反复实践和归纳思考。

通过资料查找，就可以知道是否有人做过类似工作，是否有可直接参考的资料。有时候你会发现全世界也没人做过类似工作。这时就需要针对目标，进行路径分析，看看有哪些路可能通往目标，然后再针对这些分解出的路径再进行资料查找。

这种目标分解有时候会不止一层。也就是说那些分解出来的路径也可能没有资料。那就需要针对路径再作路径分解。然后遍历这些路径。

从历史经验看，天分高、脑子快、手速快、对技术路径有敏锐直觉的人，当然能干得快一些。而一般人只要愿意坚持遍历路径，能扛得住长期得不到正反馈，大部分也能干出成绩。就是会痛苦一些。别人乐在其中，你苦在其中。不怕苦的话也行。

## 如何成长

刚参加工作的时候，部门接到各类任务，领导问我能不能干，我一般都说让我去试试吧。然后抓紧时间查资料，学习和这项工作有关的内容。

勇敢接受工作挑战，可以让人快速成长。也会让你有更多的机会，这是个马太效应的事儿。我记得当时的领导给过我这样一个评语：“不管什么工作，交给你我就放心了，反正你总有办法搞定”。

当然也要了解清楚需求，特别是对时间的要求，评估好是否在自己跳一跳能够得着的范围内。

## 敲代码的，如何转行挖掘漏洞？

“本人写了五六年Linux C，应用层业务为主，技术含量不算太高；C语言比较熟练，C++仅做过Win下MFC的简单项目；Linux kernel大致了解，写过简单的驱动；对于理论、算法之类的比较薄弱。是否可转行Linux或Win下漏洞挖掘？还需要补充点什么知识？多谢前辈们的指点。”

给你个建议：别想那么多，先干起来再说。

我找到第一个漏洞的时候，还在医院实习，只会写几行批处理。为了写 PoC 需要学编程语言，看电脑报介绍过 Perl，就去买了本《Perl 编程 24 学时教程》。后来为了写更好的 Exploit 学了 C 语言。再后来，为了各种研究读各种 RFC、调各种程序、读各种代码、试各种工具，等等。干这行，你永远不知道未来需要会什么。所以什么都可以不会，但不能学不会。

## 程序员如何判断自己是否适合做安全研究

有人跟我说以前是程序员，问能否转干安全研究。我问他有没有自己研究过什么，他说还没有，但一直很感兴趣。然后我问他感兴趣了多少年，他就沉默了。当然，这至少说明他脑子很快，迅速知道我想表达什么：天天对着电脑，真感兴趣怎么可能从来没研究过——你跟林志玲睡一个床能好几年一直盖棉被纯聊天？

## 网络安全初学者应该看什么书

前阵子遇到一个对网络安全感兴趣的中学生，让我推荐一本书。我告诉他：

随便找一个书店，在网络安全分类下面随便找一本书，然后从第一页开始学。

如果觉得学不下去，那可能是书有问题，就换一本。

如果换一本还是学不下去，就再换一本。

如果换了三四本都还学不下去，可能就不是书的问题。

如果能学得下去，等这本书学完，接下来该学什么，你会有自己的想法，不需要再问别人了。

## 信息安全方向就业要考哪些证书

这个问题问得很笼统，如果也笼统回答的话，只能说：信息安全专业学生如果以后想从事专业对口的工作，需要信息安全专业技能，很多证书都可以提高就业机会。

具体举例子来说的话——比如在国内乙方就业，CISP 是有用的。因为安全企业如果要申请信息安全服务资质，就需要若干名有 CISP 证书的员工。而很多甲方招标会要求乙方企业有相关资质。

所以顺着这个思路，可以去查查各安全项目的招标公告，看看对企业要求了什么，再查查企业如果要具备这些要求，需要有什么证书的员工。那么这些证书就是显著有利于提高就业机会的。

## 信息安全专业有必要考研吗

如果确实对安全技术很感兴趣，自己读了很多资料，写了很多代码，做了很多研究，那未必要考研。

如果你不属于前一种人，又想干这行，那跟着导师做几年项目还是有好处的。

## 学院派的信息安全指的什么？

“在各个安全技术的群里感觉已经没办法交流了。我研究的是信息安全的框架、顶层设计、整体的解决方案，平时也花不少时间跟进技术。但是遇到过多次，我说了自己的一些观点，就招来嘲讽，说我是学院派，满是鄙夷口吻。为什么会是这样的局面？理论指导实践，实践证明理论，如果是学院派指的是研究信息安全理论的人，那这样的局面显然不利于信息安全行业的发展。”

我们常把从事信息安全技术研究的人分为学院派和工业派，分别指更偏向于理论的研究者，和更偏向于实践的研究者。

学院派和工业派之间的界限并不特别清晰。有些学院派搞的东西也很实用，工业派也常研究学院派的理论。信息安全领域很多东西都是由学院派开创、工业派完善的。理论和实践都很重要，学院派和工业派都不是贬义词。

不过需要注意，理论和空论还是要区分的。理论可以指导实践，空论只能指导扯淡。所以，上面说的两派之外，还有裘千丈派、索天响派等，这些也常被人称作“学院派”，但实际上他们不是学院派。有点像我们称呼一个姑娘为“小姐”，但她可能不是小姐，是失足妇女。

至于在沟通中被人以鄙视的语气称呼为“学院派”，可能是你的观点有问题，也可能是对方无法理解你的观点，但和学院派本身没啥关系。

## 信息安全专业的发展会受到文凭的限制吗？

我们部门有名校博士，也有本科肆业的。企业招人的目标肯定是能力而不是文凭。有些大一大二的同学已经比硕士毕业生的平均水平高出很多。至少我个人会宁可要这些大一大二的同学，而不是招一个文凭漂亮但做不了事的人。

文凭是对能力的一种背书。就像我们只看标签无需品尝，就可以认为“勃艮第葡萄酒”很大概率上会比“门头沟葡萄酒”要强。然而，虽然文凭是获得入场资格的一种主流方式，但不是唯一方式。而且信息安全行业远比葡萄酒行业有更多更简单的方式让自己的能力为人所知。全真七子是一种发展路线，杨过也是一种发展路线。具体路怎么走，还得你自己根据自己的具体情况来判断。

## 信息安全工程师有全栈一说吗？

『随着互联网的发展，应用层的花样越来越多，甚至连编程语言都层出不穷了。这样的时代背景下，一个合格的安全工程师，应该具备哪些技术栈呢？真的有人熟知渗透测试，还能对软件/APP逆向分析，而且能够挖掘内核级别的0Day吗？题主7年左右的安全学习背景，目前很困惑以后的发展方向。就我了解到的，渗透测试需要应对的环境太多了，能做好一个“脚本小子”都已实属不易。何况逆向分析和系统内核，更是需要强大的编程功底和计算机理论基础，再加上最近几年火热的工控安全和人工智能等等，到底该怎么规划安全路呢？』

信息安全工程师没有全栈一说，因为信息安全工程师默认就需要是全栈的。每个人都在往全栈的方向上走，只是走的阶段不同，最终能到达的地方不同而已。

比如你去做安全服务工程师，而“安全服务”这四个字背后可能是任何操作系统，任何网络环境，任何异常问题。比如你去做漏洞研究，而“漏洞研究”这四个字背后可能是各种不同协议，不同格式，不同系统，不同指令集。所以这不是学一点东西就能吃一辈子的行业。

具体到提问中所说的“真的有人熟知渗透测试，还能对软件/APP逆向分析，而且能够挖掘内核级别的0Day吗”，只是这三样的话，能做的人还挺多的。

## 网站越来越难渗透怎么办

『大家有没有发现，现在网站防御越来越高，漏洞也越来越少，安全在进步，网站也难渗透了，一些老牌工具很难扫出注入点了，如果弄不到后台其他的也做不了。现在的工具都是2010年前的了，现在教程也不多，以前的教程内容已经越来越不适合现在的网络了。现在也没有网站能给新手练手了。』

为什么用网上下载的工具很难扫描出安全问题了？因为你能下载别人也能下载。企业的安全部门里但凡只要有一个会用这些工具的人，就能自己把这些问题找出来，剩不到你手上。

十年前，很多企业的安全做的还很差，甚至没有网络安全部门，所以下载个工具扫一扫还能发现不少问题。现在一方面是企业自身安全能力提高了，另一方面网站开发者的安全能力也提高了。所以如果你只会用十年前的工具扫一扫，在今天自然会觉得很吃力。

然而，二十年前，随便找个工具扫一扫还可以发现很多远程管理弱口令、远程溢出漏洞。那么二十年前那些搞渗透测试的，在发现这些扫出来直接就能拿 root shell 的漏洞逐渐消失后是不是也很绝望？不是的。他们开始研究注入，研究 XSS，研究 CSRF，研究反序列化。所以他们才写出了你现在从网上下载的那些工具。

二进制安全也一样。二十年前的漏洞挖掘和利用都无比简单。二十年来，漏洞越来越难发现，越来越难利用。但二进制安全这个技术方向消失了吗？不但没有，而且发展越来越好。

对出身于普通家庭的人来说，工作有难度有门槛是好事。如果一个工作不难，那通常也不挣钱。如果又不难又还能挣钱，那人家为什么要让你来做的呢？找自己的堂侄表弟小舅子不好吗？

## 做安全研究挖不到漏洞怎么办

这两年经常有人问类似“做安全研究挖不到漏洞怎么办”这样的问题，这里统一答复一下。

早年没有信息安全专业，搞漏洞研究的全都是因为爱好这个，自己主动来搞的。不适合干这个，搞不出来的，自然也就还干本行去了。所以早年没有人抱怨为什么研究不出东西。

现在信息安全专业开设的越来越多，有些同学看别人搞漏洞研究，自己也想搞。搞不出来，就四处找人问为什么自己搞不出来，怎么才能搞出来。

首先，必须要先泼一瓢冷水：由于性格、能力等多方面原因，无论是不是学信息安全专业的，世界上大部分人都不适合做漏洞研究，就像大部分人都不适合做职业运动员一样。

你们琢磨一下漏洞是什么？漏洞是程序员犯的错误。那些著名大公司软件的漏洞是什么？是一些面试好多轮才能入职的名校毕业的程序员犯的错误。而且这些公司里还有很多面试好多轮才能入职的名校毕业的人在做安全。他们都没查出来，才能留下来让你去发现。

我曾给微软中国的 QA 做过培训，和他们连续接触了数天。这些人体现出来的平均水平肯定在国内安全行业（不特指漏洞研究）之上，其中有几个还相当出色。

所以，挖不到漏洞是正常的，挖到才不正常。信息安全工作有很多方向，学信息安全，不一定非要都做漏洞研究。

## 学习网络安全时遇到瓶颈怎么办

“基础的那些sql，xss，csrf那些基本的都会了，但是想更进一步，不知道该怎么办了，有点迷茫，然后我就开始学习python和php，但是，学了一个月，回想一下，发现现在连一个小的脚本都不会写，应聘公司想以实习的目的去学习（ps：中专毕业的渣渣），没有上大学是不是很难应聘公司啊，请大神指点我一下，谢谢”

从事任何方向的技术研究，不知道该干什么的时候，就问自己四个问题：

- 这个方向上最新进展是什么？ 都知道吗？
- 这个方向上最著名的专家有哪些？他们的研究都看过吗？
- 这个方向上最著名的技术社区有哪些？精华帖都看过一遍吗？
- 这个方向上最重要的文章、工具有哪些？文章都看过吗？工具都分析过吗？

## 高手不愿和我交流怎么办

姚期智说：我看不出你的证明有什么错误，你的证明是对的。你要的杨振宁先生和张益唐先生的邮件地址分别是：xxxxxxxx、xxxxxxxx，你自己去跟他们联系。

张益唐说：这件事还是由你自己定夺，你说你证明了黎曼猜想就证明了黎曼猜想，你说你没有证明黎曼猜想就没有证明了黎曼猜想。

梦参老和尚说：我走的地点多，有的人曾经跟我说：“能像弘一法师、慈舟法师、虚云老和尚、倓虚老法师那些大善之士，在哪里有？老法师你给我介绍一个，我去跟他学习。

我说：“就是弘一法师在，你这个样子，他理都不理你。”

弘一法师，不是那么容易亲近的。他寮房的门永远是关着的，你想跟他说几句话，他没有时间跟你说。

慈舟法师，整天披着衣，讲完课，他就围着佛堂转：阿弥陀佛！阿弥陀佛！阿弥陀佛！

倓虚老法师，他的事务多，接触的政府官吏多。我们这里有几位道友，曾经跟着他到过华南，如果你们想跟他多亲近，多说几句话，他没有那个时间，并不是他不慈悲。

虚云老和尚，他在禅堂讲开示，就是你亲近他的时候。不过一个月他才讲两次，你可以到得了他身边吗？即使他真的在你身边，你能得到他的智能吗？

歌德说：读一本好书，就是和高尚的人谈话。

## 有了量子计算信息安全还有用吗

“题主今年大一，学习信息安全专业。一开始拼劲满满，下定决心当一名学霸。但在第一节计算机导论课上，老师列举计算机类型时，提到了量子计算机和光子计算机，而且说如果这些计算机普及，就不需要网络安全员了。”

你们老师可能并不清楚量子计算机和光子计算机是什么、能干什么，但认为自己明白并向学生布道，而这可能是由于把看科技新闻作为学习新知识的主要方法而导致的。

## 有了量子通信信息安全还有用吗

量子通信（无论是量子密钥分发，还是严格意义上的量子通信）的目标是对抗窃听信道的攻击，尤其是对抗窃听光缆。

漏洞攻击、木马病毒、钓鱼欺诈……等等等等绝大多数一般人能遇上的信息安全问题和量子通信要解决的问题都没什么关系，也不是量子通信能解决的。假使你现在正在被人用漏洞攻击，刚进行到50%，有一个仙女挥动魔法棒，瞬间在全球普及了量子通信，那接下来的50%也会照常进行，无论你还是攻击者都不会感觉到任何不同。

## 有哪些Linux逆向相关的学习资料推荐

SEEDLabs是雪城大学杜文亮教授创立的，十几年来得到了全球上百所大学的认可。建议每个认为找不到合适学习资料的人都看看，之后至少可以对自己到底是不是真想学有更接近于真实的认识。