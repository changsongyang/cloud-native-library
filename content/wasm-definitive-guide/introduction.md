---
linktitle: 第 1 章：简介
summary: WebAssembly 简介。
weight: 2
title: WabAssembly 简介
date: '2023-01-16T00:00:00+08:00'
type: book
---

> 非凡的主张需要非凡的证据。
>
> ——Carl Sagan 博士

本章将介绍 WebAssembly 和该技术产生的背景。从某种意义上说，它是过去七十年来 Web 发展的一个顶峰。我们有相当多的历史要介绍。如果你不喜欢历史和论述，你可以跳过这一章，直接去看第二章，但我希望你不要这样做。我认为理解为什么这项技术如此重要以及它的起源是很重要的。

## WebAssembly 提供什么

我认为评估一项新技术带来什么的能力，这是工程师可以发展的最伟大的技能之一。正如北卡罗来纳大学的 Fred Brooks 博士（译者注：《人月神话》作者，2022 年去世）提醒我们的那样，没有什么 ["银弹"](https://oreil.ly/7isP5) 所有的东西都是有取有舍的。复杂性往往不会因为一项新技术而被消除，而只是被转移到别的地方。因此，当某些东西确实改变了可能的东西或我们的工作方式的积极方向时，它值得我们关注，我们应该弄清楚原因。

当我试图理解某项新事物的含义时，我通常会先试着确定其背后的动机。洞察力的另一个良好来源是替代方案的不足之处。以前有什么，它是如何影响我们试图破译的这项新技术的？就像艺术和音乐一样，我们不断地从多个来源借鉴好的想法，所以要真正理解为什么 WebAssembly 值得我们关注，以及它所提供的东西，我们必须首先看看它之前的东西，以及它是如何发挥作用的。

在正式向世界介绍 WebAssembly 的论文中，作者指出，其动机是为了满足现代 Web 交付的需求，在软件方面的应用，单靠 JavaScript 是无法做到的 [^1] 。 归根结底，这是一个提供软件的追求，即：

- 安全
- 快速
- 可移植
- 紧凑

在这个愿景中，WebAssembly 的中心是软件开发、Web、Web 的历史以及它如何在地理分布空间中提供功能的交叉点。随着时间的推移，这个想法已经大大超出了这个起点，想象出一个无处不在的、安全的、高性能的计算平台，几乎触及我们作为技术专家的职业生活的每一个方面。WebAssembly 将影响客户端 Web 开发、桌面和企业应用、服务器端功能、传统现代化、游戏、教育、云计算、移动平台、物联网（IoT）生态系统、无服务器和微服务等领域。我希望在本书中能让你相信这一点。

我们的部署平台比以往任何时候都更加多样化，因此我们需要在代码和应用层面上的可移植性。一个通用的指令集或字节码目标可以使算法在不同的环境中工作，因为我们只需要将逻辑步骤映射到它们在特定机器架构上的表达方式。程序员们使用应用程序编程接口（API），如 OpenGL [^2] POSIX [^3] 或 Win32 [^4] ，因为它们提供了打开文件、拉起子进程或在屏幕上绘制的功能。它们很方便，减少了 开发人员需要编写的代码量，但它们对提供功能的库存在依赖性。如果 API 在目标环境中不可用，应用程序将无法运行。这是微软能够利用其在操作系统市场上的优势，也是其在应用程序套件中占据主导地位的方法之一。另一方面，开放标准可以使软件更容易移植到不同的环境。

软件运行时另一个问题是，不同的主机有不同的硬件能力（CPU 核心数量，是否有 GPU）或安全限制（是否可以打开文件或发送或接收 Web 流量）。软件往往通过使用功能测试方法来适应现有的东西，以确定一个应用程序可以利用哪些资源，但这往往会使业务功能复杂化。我们根本无法承担为多个平台不断重写软件所需的时间和金钱。相反，我们需要更好的重用策略。我们还需要这种灵活性，而不需要修改代码以支持它将运行的平台的复杂性。为不同的主机环境编写不同的代码会增加其复杂性，使测试和部署策略变得复杂。

经过几十年的发展，开源软件的价值主张已经很清楚了。我们倾向于使用由其他开发者编写的有价值的、可重复使用的组件，以此来满足我们自己的需求 [^5]。然而，并非所有可用的代码都是值得信赖的，当我们执行从互联网上下载的不值得信赖的代码时，我们就会受到软件供应链的攻击。通过 Web 钓鱼攻击、数据泄露、恶意软件和勒索软件，我们容易受到不安全的软件系统的风险、商业影响和个人成本的影响。

直到现在，JavaScript 一直是解决其中一些问题的唯一方法。当它在沙盒环境中运行时，它给了我们某种程度的安全。它是无处不在的，可移植的。引擎已经变得更快。生态系统已经爆炸性地成为生产力的雪崩。然而，一旦你离开了基于浏览器的保护范围，我们仍然有安全问题。作为客户端运行的 JavaScript 代码和在服务器上运行的 JavaScript 是有区别的。单线程设计使长期运行或高度并发的任务变得复杂。由于它起源于一种动态语言，有几类优化可以用于其他编程语言，而这些优化现在和将来都不能作为最快和最现代的 JavaScript 运行时的选项。

此外，增加 JavaScript 的依赖性太容易了，而没有意识到有多少包袱和风险被拉到了这里。开发人员如果不花时间仔细考虑这些决定，最终会给上游软件测试、部署和使用的各个方面带来麻烦。每一个脚本一旦在 Web 上传输，就必须被加载和验证。这就减慢了使用的时间和使得一切都感觉很迟钝。当一个依赖包被修改或删除时，它有可能扰乱大量的部署软件 [^6]。

在有些观察者眼中 WebAssembly 是对 JavaScript 的攻击，但事实并非如此。当然，如果你愿意，你可以避免使用 JavaScript，但它主要是为你提供选择，用你选择的语言解决问题，而不需要一个单独的运行时，或不得不关心另一个软件是用什么语言写的。现在已经可以使用 WebAssembly 模块而不知道它是如何构建的。这将增加我们从软件中获得的商业价值的寿命，同时允许我们在采用新语言时进行创新，而不影响其他部分。

在过去的几十年里，我们经历了几个试图解决这些问题的工具、语言、平台和框架，但 WebAssembly 第一次把它做对。它的设计者并没有试图过度规范什么。他们正在从过去的经验中学习，拥抱 Web，并将问题空间的思维运用到最终是一个困难的多维问题上。在我们进一步深入研究之前，让我们看看对这一令人兴奋的新技术的形成性影响。

## Web 的历史

在 WebAssembly 社区有一个笑话，说 WebAssembly 是 "非 Web 也非汇编" [^7]。虽然这在某些方面是真的，但这个名字足以暗示它所提供的东西。它是一个目标平台，有一系列的指令，隐约有汇编的感觉 [^8]。事实上，WebAssembly 模块经常作为另一种类型的 URL 可寻址资源通过 Web 传递，这就证明了在名称中加入 **Web** 这个词是合理的。

"传统的软件开发" 和 "Web 开发" 之间的一个主要区别是，一旦你有了浏览器，后者实际上不需要安装。这在交付成本和面对错误和功能要求时快速转换新版本的能力方面，是一个改变游戏规则的因素。在其他跨平台的技术生态系统中，如互联网和 Web，它也使支持多种硬件和软件环境变得更加容易。

万维网的发明者 Tim Berners-Lee 爵士曾在欧洲核子研究组织（CERN）工作，在那里他提交了一份提议，关于在追求 CERN 更大的研究目标的过程中，将文件、图像和数据相互连接 [^9]。尽管事后看来，其影响是显而易见的，但在他被要求采取行动之前，他不得不在内部多次宣传他的想法 [^10]。作为一个组织，欧洲核子研究中心由世界各地的几十个研究机构代表，这些机构派科学家携带自己的计算机、应用程序和数据。没有真正的能力来强迫每个人使用相同的操作系统或平台，所以他认为需要一个技术解决方案来解决问题。

在 Web 之前，有诸如 Archie 这样的服务 [^11]、Gopher [^12] 和 WAIS [^13] 但他想象了一个更方便用户的平台，该平台最终作为互联网分层结构顶端的一个应用层面的创新而产生。他还 从标准通用标记语言（SGML）[^14] 中吸取了一些想法，并使它们 ，成为超文本标记语言（HTML）的基础。

这些设计的结果迅速成为向世界提供信息、文件和最终应用功能的主要机制。它通过定义标准的交换，在不要求各利益相关者就特定的技术或平台达成一致的情况下做到了这一点，并包括如何发出请求和返回什么。任何理解这些标准的软件都可以与任何其他理解这些标准的软件进行交流。这给了我们选择的自由，以及任何一方独立于另一方而发展的能力。

## JavaScript 的起源

Web 的交互模型被称为超文本传输协议（HTTP）。它是基于一组受限制的动词来交换基于文本的信息。虽然这是一个简单而有效的模型，易于实现，但由于不断返回服务器的固有延迟，它很快就被认为不适合交互式现代应用的任务。能够将代码下发到浏览器的想法一直很有说服力。如果它在用户的交互中运行，那么不是每一个活动都需要返回服务器。这将使 Web 应用极大地提高互动性、响应性和使用的乐趣。如何实现这一点这并不完全清楚。哪种编程语言是最有意义的？我们如何平衡表达能力和浅显的学习曲线，以便更多的人能够参与到开发过程中？哪些语言比其他语言更好，我们将如何保护客户端的敏感资源不受恶意软件的侵害？

浏览器领域的大部分创新最初是由网景公司（Netscape Communications Corp）推动的。信不信由你，Netscape Navigator 最初是一个付费软件，但该公司更大的兴趣是销售服务器端软件 [^15]。通过扩展客户端的功能，它可以创造和销售更强大和有利可图的服务器功能。

当时，Java 正从其作为计算机设备的嵌入式语言的雏形中出现，但它还没有太多的成功记录。作为 C++ 的简化版本，它是一个引人注目的想法，它运行在一个虚拟平台上，因此本质上是跨平台的。作为一个旨在运行通过 Web 下载的软件的环境，它通过语言设计、沙盒容器和细粒度的许可模型来实现安全。

在不同的操作系统之间移植应用程序是一件棘手的事情，而现在不在需要这样的前提，使人们对软件开发的未来产生了狂热的期待。太阳微系统公司发现自己处于一个令人羡慕的位置，即拥有解决各种问题和机会的完美风暴的方法。考虑到这一潜力，正在讨论将 Java 引入浏览器，但还没搞清具体如何实施，以及何时发布。

作为一种面向对象的编程（OOP）语言，Java 包含复杂的语言特性，如线程和继承。在 Netscape ，有人担心这对非专业的软件开发人员来说可能太难掌握，因此该公司雇用了 Brendan Eich 来创建一个 "浏览器的方案 [^16]"，设想一种更容易的、轻量级的脚本语言 [^17]。Brendan 可以自由决定他想在该语言中包含哪些内容，但他也面临着要尽快完成的压力。一种用于交互式应用的语言被认为是这个新兴平台向前迈进的关键一步，而且每个人都希望尽快完成。正如塞巴斯蒂安 - 佩罗特（Sebastián Peyrott）在博文 "JavaScript 简史" 中指出的那样，出现的是 "Scheme 和 Self 的早产儿，具有 Java 的外观"  [^18]。

最初，浏览器中的 JavaScript 只限于简单的交互，如动态菜单、弹出式对话框和对按钮点击的响应。与每一个动作都要往返于服务器相比，这些都是很大的进步，但与当时在台式机和工作站上所能做到的相比，它仍然是玩具。

我在 Web 早期工作的公司创造了第一个整个地球的可视化环境，涉及到 TB 级的地形信息，超光谱图像，以及从无人机视频中提取视频帧 [^19]。当然，这最初需要 Silicon Graphics 工作站，但在几年内，它能够在带有消费级图形处理单元（GPU）的 PC 上运行。在当时的 Web 上，这样的事情根本不可能发生，不过，由于 WebAssembly 的出现，这种情况已经不复存在 [^20]。

真正的软件开发和 Web 开发根本不存在混淆。正如我们所注意到的，客户端和服务器之间的关注点分离的好处之一是，客户端可以独立于服务器而发展。当 Java 和 Java Enterprise 模型开始主宰后端时， JavaScript 在浏览器中发展，并最终成为它的主导力量。

## Web 平台的演变

随着 Java 小程序和 JavaScript 在网景浏览器中的应用，开发人员开始尝试动态网页、动画和更复杂的用户界面组件。几年来，这些仍然只是玩具性的应用，但这一愿景具有吸引力，而且不难想象它最终会导致什么。

微软觉得有必要跟上，但对直接支持其竞争对手的技术没有太大兴趣。它（正确地）认为，这种 Web 发展可能最终颠覆其操作系统的主导地位。当微软发布支持脚本的 IE 浏览器时，该公司称其为 JScript 以避免法律问题，并对网景的解释器进行了反向工程。微软的版本支持与基于 Windows 的组件对象模型（COM）元素的交互，并有其他使浏览器之间很容易编写不兼容的脚本。它最初对将 JavaScript 标准化为 ECMAScript 的努力的支持在一段时间内减弱了，最终浏览器大战开始了 [^21]。这对开发者来说是一个令人沮丧的时期，最终涉及美国政府对微软的反竞争诉讼。

随着网景公司命运的衰落，IE 浏览器开始在浏览器领域占据主导地位，即使在 JavaScript 经历了标准化的过程中，跨平台的创新也有一段时间消退了。Java 小程序在一些圈子里被广泛使用，但它们在沙盒环境中运行，所以用它们作为驱动动态网页活动的基础比较棘手。你当然可以使用 Sun 的图形和用户界面 API 来做有成效的和有趣的事情，但它们在一个独立的内存空间中运行，而不是 HTML 文档对象模型（DOM）[^22]。它们是不兼容的，有不同的编程和事件模型。用户界面在沙盒元素和 Web 元素之间看起来并不一样。总的来说，这是一个完全不合适的开发经验。

其他非可移植技术，如 ActiveX，在微软 Web 开发领域开始流行。Macromedia 的 Flash 变成了 Adobe 的 Flash，并在大约十年的时间里有一个短暂但活跃的流行期。然而，所有这些次要的选择都存在问题。内存空间相互隔绝，安全模型也不如人们所期望的那样强大。这些引擎都是新的，并且在不断开发中，所以错误很常见。ActiveX 提供了代码签名保护，但没有沙盒保护，所以如果证书可以被伪造，就可能出现可怕的攻击。

火狐浏览器从 Mozilla 中脱颖而出，成为 Netscape 后继的一个可行的竞争者。谷歌的 Chrome 浏览器最终成为 IE 浏览器的合适替代品。每个阵营都有自己的追随者，但人们对解决它们之间的不兼容问题的兴趣越来越大。在浏览器领域引入选择，迫使每个供应商更加努力，做得更好，以超越对方，作为实现技术主导地位和吸引市场份额的手段。

因此，JavaScript 引擎的速度明显提高了。尽管 HTML 4 仍然很 "古怪"，在不同的浏览器和平台上使用起来也很痛苦，但现在已经开始有可能隔离这些差异了。这些发展与在基于标准的环境结构中工作的愿望相结合，鼓励了 [Jesse James Garrett](https://oreil.ly/L0DCk) 想到了一种不同的 Web 开发方法。他提出了 **Ajax** 这个术语，它代表了一组标准的组合：异步 JavaScript 和 XML。这个想法是让数据从后端系统流入前端应用程序，而前端应用程序将对新的输入作出动态响应。通过在操作 DOM 的层面上工作，而不是拥有一个单独的、沙盒式的用户界面空间，浏览器可以成为基于 Web 的客户 - 服务器架构中的通用应用程序消费者。

在这一时期，漫长的 HTML 5 标准化进程也已经开始，试图提高各浏览器的一致性，引入新的输入元素和元数据模型，在其他功能中，Ajax 是一个非常重要的元素，它提供了硬件加速的 2D 图形和视频元素。Ajax 风格的融合，ECMAScript 作为一种语言的出现和成熟，更容易的跨浏览器支持，以及功能越来越丰富的基于 Web 的环境，都引起了活动和创新的爆炸。我们看到无数基于 JavaScript 的应用程序框架来来去去，但就可能的情况而言，有一种稳定的前进势头。随着开发者们的努力，浏览器供应商也改进了他们的引擎，使其能够进一步推动发展。这是一个良性循环，为安全、可移植、零安装的软件系统的潜力带来了新的愿景。

随着其他障碍和限制的消除，这种处于核心位置的奇怪的小语言成为前进道路上一个越来越大的惯性阻力。引擎正在成为世界级的开发环境，具有更好的调试和性能分析的工具。新的编程范式，如基于承诺的风格，允许更好的模块化和异步友好的应用代码，在 JavaScript 臭名昭著的单线程环境中实现强大的结果 [^23]。但是语言本身并不能像 C 或 C++ 等其他语言那样进行优化。从语言性能的角度来看，有一些简单的限制，是可以实现的。

随着 WebGL [^24] 和 WebRTC [^25] 等技术的开发和采用，Web 平台标准继续得到推进不幸的是，JavaScript 的性能限制使它不适合在浏览器中扩展涉及低级 Web、多线程代码、图形和流媒体视频编解码的功能。

该平台的发展需要 W3C 成员组织进行痛苦的努力，以决定什么是重要的设计和构建，然后在各种浏览器的实现中推出。随着人们对使用 Web 作为一个重量级、交互式应用的平台越来越感兴趣，这个过程被认为是越来越站不住脚。一切都必须用 JavaScript 编写（或重写），或者浏览器必须对行为和界面进行标准化，这意味着要花几年时间才能实现新的进步。

正是由于这些和其他原因，谷歌开始考虑采用另一种 ，以实现安全、快速和可移植的客户端 Web 开发。

## 原生客户端 （NaCI）

2011 年，谷歌发布了一个新的开源项目，名为原生客户端（NaCl）。其想法是在浏览器中提供接近原生速度的代码执行，同时出于安全考虑在有限的权限沙盒中运行。你可以认为它有点像 ActiveX，背后有一个真正的安全模型。该技术很适合谷歌的一些大目标，如支持 ChromeOS 和将任务从桌面应用转移到 Web 应用。它最初并不是要为每个人扩展开放 Web 的能力。

这些用例主要是为了支持基于浏览器的计算密集型软件的交付，如：

- 游戏
- 音频和视频编辑系统
- 科学计算和 CAD 系统
- 模拟

最初的重点是 C 和 C++ 作为源语言，但由于它是基于 LLVM 编译器工具链的 [^26]。它有可能支持其他可以生成 LLVM 中间表示（IR）[^27] 的语言。你将看到，这将是我们向 WebAssembly 过渡的一个反复出现的主题。

这里有两种形式的可分发代码。第一种是同名的 NaCl，它产生的 "nexe" 模块将针对特定的硬件架构（例如 ARM 或 x86-64），并且只能通过 Google Play 商店分发。另一种是称为 PNaCl [^28] 的可移植形式。它将以 LLVM 的 Bitcode 格式表示，使其与目标无关。这些模块被称为 "pexe" 模块，需要在客户端的主机环境中被转换为原生架构。

该技术是成功的，因为在 浏览器中展示的性能与本地执行速度的差距很小。通过使用软件故障隔离（SFI）技术，有可能从网上下载高性能、安全的代码并在浏览器中运行。一些流行的游戏，如 **Quake** 和 **Doom** 被编译成这种格式，以显示最终可能的结果。问题是，NaCl 二进制文件需要为每个目标平台生成和维护，且只能在 Chrome 上运行的。它们还在进程外空间运行，因此它们不能直接与其他 Web API 或 JavaScript 代码互动。

虽然在有限权限的沙盒中运行是可以实现的，但它确实需要对二进制文件进行静态验证，以确保它们不会试图直接调用操作系统服务。生成的代码必须遵循某些地址边界对齐模式，以确保它不违反分配的内存空间。

如上所述，PNaCl 模块的可移植性更高。LLVM 基础设施可以生成 NaCl 原生代码或可移植的 Bitcode，而不需要修改 ，也不需要修改原始源。这是一个很好的结果，但是代码的可移植性和应用程序的可移植性之间是有区别的。应用程序要求它们所依赖的 API 可用，以便工作。谷歌提供了一个名为 Pepper API [^29] 的应用二进制接口（ABI）。用于低级别的服务，如 3D 图形库、音频播放、文件访问（通过 IndexedDB 或 LocalStorage 模拟）等等。虽然由于 LLVM 的存在，PNaCl 模块可以在不同平台的 Chrome 浏览器中运行，但它们只能在提供合适的 Pepper API 实现的浏览器中运行。虽然 Mozilla 最初表示有兴趣这样做，但他们最终决定尝试一种不同的方法，这就是后来被称为 asm.js 的方法。NaCl 在推动行业向这个方向发展方面功不可没，但它最终还是太麻烦了，而且太针对 Chrome，无法推动开放 Web 的发展。Mozilla 的尝试在这方面比较成功，即使它没有提供与本地客户端方法相同的性能水平。

## asm.js

[asm.js 项目](https://asmjs.org/) 至少有一部分动机是为了给 Web 带来一个更好的竞争故事。这很快就扩展到了一个愿望，即允许任意的应用程序安全地传递到浏览器的沙盒中，而不需要对现有的代码进行次级修改。

正如我们之前所讨论的，浏览器生态系统已经在不断进步，使 2D 和 3D 图形、音频处理、硬件加速视频等以基于标准的、跨平台的方式提供。我们的想法是，在这个环境中操作，将允许应用程序使用这些被定义为可以从 JavaScript 调用的任何功能。JavaScript 引擎是高效的，并且有强大的沙盒环境，经历了重要的安全审计，所以没有人觉得要从头开始。真正的问题仍然是无法提前选择 ，所以运行时的性能可以进一步提高。

由于其动态性质和缺乏适当的整数支持，在代码加载到浏览器之前，有几个性能障碍无法得到有意义的管理。一旦这样做了，及时优化（JIT）编译器就能很好地加速，但仍然存在固有的问题，比如缓慢的边界检查数组引用。虽然整个 JavaScript 不能被提前优化，但它的一个子集可以被优化。

这意味着什么的确切细节与我们的历史叙述并不超级相关，但最终的结果是 asm.js 也使用基于 LLVM 的 clang [^30] 前端解析器，通过 Emscripten 工具链 [^31] 编译的 C 和 C++ 代码是可以提前优化的，所以通过现有的优化通道可以使生成的指令非常快。LLVM 是一个干净的模块化架构，所以它的各个部分都可以被替换，包括机器代码的后端生成。从本质上讲，Emscripten 团队可以重复使用前两个阶段（解析和优化），然后将这个 JavaScript 子集作为一个自定义的后端发出。因为输出的都是 "单纯的 JavaScript"，它比 NaCl/PNaCl 的方法更容易移植。不幸的是，代价是性能上的。它比直接使用 JavaScript 有了很大的改进，但性能上还不如谷歌的方法。不过，这已经足够让开发者感到惊讶了。然而，除了适度的性能改进之外，你可以将现有的 C 和 C++ 应用程序部署到浏览器中，并具有合理的性能，而且几乎不需要修改代码，这一事实本身就很引人注目。虽然有一些涉及 Unity 引擎 [^32] 的极有说服力的演示。让我们看一个简单的例子。"Hello, world!" 似乎是一个好的开始。

```c
#include <stdio.h>
  int main () {printf ("Hello, world!\n");
    return 0;
}
```

请注意，这个版本的经典程序没有任何异常。如果你把它存储在一个叫做`hello.c`的文件中，Emscripten 工具链将允许你发出一个叫做`a.out.js`的文件，它可以直接在 Node.js [^33] 中运行或者通过一些脚手架，在浏览器中运行。

```bash
brian@tweezer ~/s/w/ch01> emcc hello.c
brian@tweezer ~/s/w/ch01> node a.out.js
Hello, world!
```

很酷，不是吗？

只是有一个问题：

```bash
brian@tweezer ~/s/w/ch01> ls -lah a.out.js
-rw-r--r-- 1 brian staff 119K Aug 17 19:08 a.out.js
```

119KB，这是一个非常大的"Hello, world!"程序。快速浏览一下本地可执行文件可能会让你感觉到发生了什么。

```bash
brian@tweezer ~/s/w/ch01> clang hello.c brian@tweezer ~/s/w/ch01> ls -lah a.out
-rwxr-xr-x 1 brian staff 48K Aug 17 19:11 a.out
```

为什么我们所谓的优化的 JavaScript 程序比本地版本大了近三倍？这不仅仅是因为作为一个基于文本的文件，JavaScript 的体积更大。再看一下这个程序。

```c
#include <stdio.h> ①
int main () {printf ("Hello, world!\n"); ②
  return 0;
}
```

1. 头部确定了标准 IO 相关函数定义的来源。
2. 对 `printf ()` 函数的引用将由一个在运行时加载的动态库来满足。

如果我们使用 nm 查看编译后的可执行文件中定义的符号，我们会发现二进制文件中 [^34] 不包含 `printf ()` 函数的定义。它被标记为 "U"，表示 "未定义"。

```bash
brian@tweezer ~/s/w/ch01> nm -a a.out
0000000100002008 d dyld_private
0000000100000000 T mh_execute_header
0000000100000f50 T _main
                 U _printf
                 U dyld_stub_binder
```

当 Clang 生成可执行文件时，它留下了一个占位符，指向它期望由操作系统提供的函数。对于浏览器来说，没有可用的标准库，至少在动态加载的意义上没有，所以必须提供库函数和它需要的任何东西。此外，这个版本不能直接与浏览器的控制台对话，所以它需要被赋予钩子来调用一个函数，如浏览器的`console.log ()` 功能。为了让它能在浏览器中工作，该功能必须与应用程序一起交付，这就是为什么它最终会变得如此庞大。

这很好地突出了可移植代码和可移植应用程序之间的区别，这是本书的另一个共同主题。现在，我们可以感叹它的工作原理，但本书不叫 * asm.js: The Definitive Guide * 是有原因的。asm.js 是一块了不起的垫脚石，它证明了从各种可优化的语言中生成性能合理的沙盒化 JavaScript 代码是可能的。Java 脚本本身也可以进一步优化，这是超集所不能做到的。通过基于 LLVM 的工具链和自定义的后端来生成这个子集，所付出的努力要比原来小得多。

asm.js 代表了不支持 WebAssembly 标准的浏览器的一个很好的后备位置，但现在是时候抛出本书主题了。

## WebAssembly 的崛起

通过 NaCl，我们找到了一个能提供沙盒和性能的解决方案。通过 PNaCl，我们发现了平台的可移植性，但没有浏览器的可移植性。通过 asm.js，我们发现了浏览器的可移植性和沙箱，但没有同样的性能水平。我们还被限制在处理 JavaScript，这意味着如果不首先改变语言本身，我们就不能用新的功能来扩展这个平台（例如，高效的 64 位整数）。考虑到这是由一个国际标准组织管理的，这不可能是一个快速周转的方法。

此外，JavaScript 在浏览器从 Web 上加载和验证的方式上有一定的问题。浏览器必须等到下载完所有引用的文件后才开始验证和优化它们（而进一步的优化则需要我们等到应用程序已经在运行了）。考虑到我们已经说过的关于开发者如何用大量的横向依赖来填充他们的应用程序，JavaScript 的 Web 传输和加载时间的性能是在既定的运行时间问题之外需要克服的另一个瓶颈。

在看到这些部分解决方案的可能性之后，出现了对高性能、沙盒化、可移植代码的强烈需求。浏览器、Web 标准和 JavaScript 环境中的各种利益相关者都认为需要一个在现有生态系统范围内工作的解决方案。为了让浏览器发展到今天，我们已经做了大量的工作。创建动态的、有吸引力的、交互式的、跨操作系统平台和浏览器实现的应用程序是完全可能的。只要再努力一下，似乎就可以把这些设想合并成一个统一的、基于标准的方法。

正是在这种情况下，2015 年，除了 JavaScript 的创造者 Brendan Eich 之外，其他人都宣布已经开始了 WebAssembly [^35] 的工作。他强调了这项工作的几个具体原因，并称其为 "低级安全代码的二进制语法，最初与 asm.js 共同表达，但从长远来看，能够与 JS 的语义相分离，以便最好地作为多种源级编程语言的通用对象级格式。"

他继续说："可能的长期分歧的例子：零成本异常、动态链接、调用 /cc。是的，我们的目标是开发 Web 的多语言编程语言对象文件格式。"

至于为什么这些各方对此感兴趣，他提出了这样的理由。"asm.js 很好，但一旦引擎为其优化，解析器就会成为热点 —— 在移动设备上非常热。传输压缩是需要的，可以节省带宽，但在解析前的解压却会造成伤害"。

最后，也许公告中最令人惊讶的部分是谁将参与其中：

> 一个 W3C 社区小组，WebAssembly CG，向所有人开放。从 GitHub 的日志中可以看到，WebAssembly 到目前为止是由谷歌、微软、Mozilla 和其他一些人共同完成的。我很抱歉，这项工作一开始是通过 GitHub 的私人账户完成的，但这是一个临时措施，以帮助几家大公司达成共识，并接受长期的合作游戏，必须玩这个游戏才能成功。

在很短的时间内，其他公司如苹果、Adobe、AutoCAD、Unity 和 Figma 都支持这项工作。这个在几十年前就开始的、涉及无尽的冲突和自身利益的愿景正在转变为一个统一的倡议，最终为我们带来一个安全、快速、可移植和 **Web 兼容的 ** 运行环境。

在建立这个平台的过程中，潜在的复杂问题是无止境的。我们并不完全清楚到底应该在前面指定什么。不是每一种语言都支持原生线程。并非每种语言都使用异常。C/C++ 和 Rust 就是例子，它们的运行时不需要垃圾收集。魔鬼总是在细节中，但合作的意愿是存在的。而且，正如他们所说的，有意愿的地方就有办法。

在接下来的一年多时间里，CG 变成了 W3C 工作组（WG），其任务是 ，定义实际标准。他们做出了一系列决定，以定义一个最小可行产品（MVP）的 WebAssembly 平台，它将被所有主要的浏览器供应商所支持。此外，Node.js 社区也很兴奋，因为这可以为需要用低级语言编写的 Node 应用程序的部分提供一个管理本地库的解决方案。与其依赖 Windows、Linux 和 macOS 库，Node.js 应用程序可以有一个 WebAssembly 库，它可以被加载到 V8 环境，并在运行中转换为本地汇编代码。突然间，WebAssembly 似乎已经准备好超越在浏览器中部署代码的目标，但我们不要超前于自己。我们有本书的其余部分来告诉你这部分的故事。

## 注释

[^1]: Andreas Haas 等人，"Bringing the Web Up to Speed with WebAssembly"，在 2017 年 6 月举行的第 38 届 ACM SIG- PLAN 编程语言设计与实现会议上发表。 https://dl.acm.org/doi/10.1145/3062341.3062363
[^2]: 多年来，OpenGL 一直是可移植 3D 图形应用程序的定义标准。如今，它正被 Vulkan 和 Metal 等更现代的 API 所取代，但你可以在 [OpenGL 网站上](https://www.opengl.org) 了解更多关于这些标准的信息。
[^3]: [可移植操作系统接口 （POSIX）](https://oreil.ly/H5l9c)  是一个 IEEE 标准的集合，用于定义共同的应用功能，以便在多个操作系统上运行。
[^4]: [Win32](https://oreil.ly/YACjw) 是一个更大的 API 集合的一部分，它为开发者提供了对 Windows 操作系统中可用的通用功能的访问。
[^5]: 这种技术允许决策者建立最低标准来决定他们的需求何时被满足。满足的目标不是要找到一个 [完美](https://oreil.ly/KgXx6) 的解决方案，而是要找到一个在当前情况下可以接受的解决方案。
[^6]: 维基百科上的 [关于 npm](https://oreil.ly/RRTC9) 的页面强调了几个案例，在这些案例中，破碎的依赖关系产生了巨大的影响。任
[^7]: 任何人都可以看出它是 [J.F.  BastienBastien](https://oreil.ly/l5ATA) 说的，但连他自己都不确定。
[^8]: [Assembly 语言](https://oreil.ly/Ymw8q) 是一种低级别的编程语言，通常与特定机器的处理器结构和指令集有关。
[^9]: 欧洲核子研究中心（CERN）这个名字来自于法国的欧洲核子研究委员会 **（**Conseil Européen pour la Recherche Nucléaire）。其许多令人振奋的项目详见于其 [主页](https://home.cern/)。在
[^10]: 他自己的时间！[^11]: [Archie](https://oreil.ly/kgQKI) 是一个早期的搜索引擎，帮助人们在 FTP 服务器上寻找文件。
[^12]: [Gopher](https://oreil.ly/OgxGb) 是我们现在所依赖的基于 HTTP 的 Web 的一个令人兴奋的前奏。
[^13]: [广域信息服务器 (WAIS) ](https://oreil.ly/n8XCt) 是另一个在分布式系统中搜索和请求文本信息的早期系统。
[^14]: [SGML](https://oreil.ly/Q665x) 是一个定义结构化、声明性文档的 ISO 标准，是 HTML、DocBook 和 LinuxDoc 的基础。
[^15]: 当时我买了一个 Netscape 1.0 Silicon Graphics IRIX 的许可证。由于...... 历史原因，我仍然保留着这张 CD，在某个地方吃灰。
[^16]: [Scheme](https://oreil.ly/4NmPN) 是 Lisp 的一个相当轻量级的版本。 
[^17]: [在这里](https://oreil.ly/HxiNS) 有一个对 JavaScript 的早期历史很好的总结。
[^18]: [Self](https://oreil.ly/hSGMo) 是一种面向对象的编程语言，影响了 JavaScript 的基于原型的继承。
[^19]: [Autometric](https://oreil.ly/ff3sc) 有一个疯狂的背景，涉及派拉蒙电影公司、Trinitron 电子管，以及帮助美国国家航空航天局（NASA） ，决定在哪里登陆月球！此后，它被波音公司收购。
[^20]: [谷歌地球](https://oreil.ly/sIlwP) 现在在浏览器中运行。
[^21]: 虽然现在的浏览器供应商在标准方面的合作更加紧密，但有一段时间他们的竞争非常激烈。这个时期的讨论见 [维基百科](https://oreil.ly/Rmg9i)。
[^22]: [DOM](https://oreil.ly/OpTD5) 是由浏览器呈现的网页或应用程序的树状结构。它通常以 HTML 声明性的文本形式从服务器发送到客户端，但 JavaScript 能够在浏览器中对其进行操作。
[^23]: [Promises](https://oreil.ly/lcozC)(或称 future) 允许开发者在提供具有并发功能的应用程序的同时建立相对简单的编程模型。
[^24]: [WebGL](https://oreil.ly/pM90T) 将一个类似的 3D 图形模型从 OpenGL 世界带到了 Web。
[^25]: [WebRTC](https://oreil.ly/7JRIc) 提供了建立对摄像机和麦克风的许可访问以及加密的点对点连接的机制。
[^26]: [LLVM](https://llvm.org/) 并不代表什么，但它是一个非常有影响力的工具链，你应该多了解一下。我们将在本书中经常提到它。
[^27]: 通常情况下，软件被编译成二进制的可执行形式。IR 允许它以解析的、结构化的形式存在，以达到优化的目的，还有其他原因。
[^28]: 发音为 “pinnacle”。
[^29]: 因为 NaCI，明白吗？
[^30]: [ClangClang](https://clang.llvm.org/) 是一个用于 C、C++ 和 Objective-C 的 LLVM 编译器工具套件。
[^31]: 在本书中，我们将了解更多关于 [Emscripten](https://emscripten.org/) 的信息，但如果你很好奇。
[^32]: 在浏览器中获得零安装的游戏体验是推动这种创新的主要原因。你可以 [在浏览器中](https://oreil.ly/qoDTv) 看到一个 Unity 引擎使用 WebGL 的例子。
[^33]: Node.js 是一个非常流行的服务器端 JavaScript 环境，我们将在第 8 章中进一步讨论。
[^34]: nm 是一条 Unix 命令，用于显示一个可执行文件的符号表。
[^35]: Brendan Eich，"[从 ASM.JS 到 WebAssembly](https://brendaneich.com/2015/06/from-asm-js-to-Webassembly/)"，2015 年 6 月 17 日。