---
title: BT 下载、盗版和流媒体
date: 2019-10-27T21:02:13+08:00
indent: false
slug: piracy
tags: ["internet"]
comments: true
---

最近再次折腾了一下 aria2，按照网上的教程进行配置，手动添加 tracker 和 DHT，终于算是能用了，热门资源的下载速度可以达到带宽。

<!--more-->

[aira2](https://aria2.github.io/) 是一款覆盖全平台、占用系统资源极小并且支持多种下载协议（支持 HTTP、HTTPS、FTP、Bittorrent 和 Metalink）的下载器。但为了做到占用资源小，aria2 极简到连 UI 界面也没有，只有命令行界面，需要用户自己创建配置文件进行设置，可以说非常用户不友好。曾经我按照网上教程配置过一次 aria2，配置完以后我试验下载一个文件，发现几乎没有速度，就放弃了。这两天再次折腾才知道，原来需要手动添加最近热门的 tracker，tracker 可以理解为最近下载者的服务器信息，通过它可以找到可能在上传资源的用户，DHT 也类似，具体原理我也不是非常理解，可以参考[这篇文章](http://www.senra.me/solutions-to-aria2-bt-metalink-download-slowly/)。

其实我平时使用 [qbittorrent](https://www.qbittorrent.org/) 配合 [IDM](https://www.internetdownloadmanager.com/) 进行下载已经足够了，折腾 aira2 原本是以为它可能比 qibittorrent 下载 BT 资源更快。实际上 BT 下载的原理已经决定了，下载速度和下载器的选择基本没什么关系，只和资源的热度有关。这就涉及到 BT 下载或者说 P2P 下载的原理了，国内的科普团队 「回形针 Paperclip」做过一个非常好的[科普视频](https://www.youtube.com/watch?v=jp0bF9Qu2Jw)，讲解了 BT 下载的原理。简单来说，BT(BitTorrent) 是一种 P2P（peer to peer）的下载协议，初始上传者将自己的电脑作为服务器发布一个资源生成一个 `.torrent` 文件，也就是我们俗称的「种子」，其中包含了这个资源的地址和一系列协议，其他下载者通过这个 `.torrent` 文件包含的信息就可以找到这个资源进行下载。不仅如此，下载过这个资源的用户，自己也会变成一个服务器，将已经下载的资源部分开放给其他下载者下载。这样，一个资源下载的人数越多，下载速度反而会越快，因为这时可以从多个下载者的服务器上获取资源。

尽管原理听起来有点陌生，但曾经应该人人都用过 BT 或者其他类似的 P2P 下载服务，比如大家更加熟悉的迅雷。当然迅雷不止支持 P2P 协议，迅雷是一个真正的全协议支持下载器，甚至用自己的服务器存储大量资源以提高用户下载速度，即所谓的离线下载，这已经不是去中心化的 P2P 下载了。然而时至今日，使用 P2P 的方式下载资源的用户越来越少了，普通用户提到下载可能第一反映是百度网盘。网盘本质是云存储，也就是中心化的存储方式，把资源存在百度的中央服务器，所有的用户都从百度的服务器上进行下载。更多的用户连下载都不会下，要听音乐或看视频都会选择在线的流媒体服务，可能只有在找一些被封禁的资源或盗版软件时，会不情愿地用百度云或迅雷下载，并骂一句「狗日的百度又限我速」。

早年间网速不快的时候，大家都在用 P2P 下载，后来带宽升级，出现了各种各样的视频网站，再到现在移动网络普及，流媒体服务已经成为主流，全民下载的日子一去不复返。流媒体的确很好，得益于网速的提升，在线看视频、听音乐几乎都可以做到即点即播（尽管品质比不上下载下来的高质量文件），但流媒体服务也带来了很多麻烦。

首当其冲的就是无休止的版权问题。比如流媒体音乐平台，在规范版权后，用户发现自己常用的平台大片的歌曲变成灰色，又或者很多歌曲必须付费或成为会员才能听。其实付费理所当然，歌手写歌、发行都要花钱，理应给他们付费，但即便愿意付费，由于平台之间的竞争，很多歌手或歌曲常常变成某某平台独占。比如我最常用的音乐平台是「网易云音乐」，因为我喜欢它的界面（尽管最近越来越臃肿了）和推荐算法，在长期使用中积累了大量的红心歌曲，并且购买了它的会员，这让我很难完美迁移到别的平台。但是一些我很喜欢的歌手，比如周杰伦，又是 QQ 音乐独占。我不会为了周杰伦单独再开一个 QQ 音乐的会员，因此我将周杰伦的歌曲都下载到了电脑里（主要是留作回忆）。视频平台情况类似，有些电影、电视剧或者综艺，往往是某个平台独占，如果用户喜欢的内容是多个平台分别独占的，就会陷入一个很尴尬的处境。

YouTube 的科技主播 Austin Evans 出过一期视频讲这个问题，叫 [Netflix SAVED Piracy](https://www.youtube.com/watch?v=3PCs_jcOFdI)，即 Netflix 拯救了盗版。这不是说大家都在下载 Netflix 的盗版视频，而是指随着 Netflix 这种流媒体平台的兴起，越来越多的流媒体平台开始出现，其中包括迪士尼和 CBS 这样拥有大量 IP 的老牌媒体。理所当然，这些流媒体平台会将很多自制 IP 作为独占内容，迪士尼这样的老牌媒体还会将已有的经典 IP 作为独占以吸引用户订阅他们会员服务。这就带来了上一段中的问题，即版权越来越分散了，用户如果想要享受他们喜欢的所有内容，就不得不订阅多个平台，这显然是很不划算的一笔投入。怎么办呢？答案就是盗版，也就是 BT 下载。国外 BT 下载盗版资源是违法的，因此这种行为有一定风险，真的有因为下载过多盗版资源被判刑的例子。国内理论上下载盗版资源也是违法的，但实际上如果不作商用进行牟利，基本上也没有人管（的确也很难管）。

流媒体的另一个大问题可以说很有中国特色，那就是——审核。流媒体是中心化的服务，上面的资源是面向所有公众的，当然逃不过审核。也许相比于大荧幕上的电影和电视节目，网络内容的审核会宽松一点，但是审核的存在始终是一把达摩克利斯之剑，你不知道自己喜欢的作品能不能通过审核被引入国内。甚至，你不知道自己喜欢的作品是否会在哪一天因某种奇怪的理由被强制下线。

这里就不得不提一下非常具有奉献和自由精神的字幕组们了，有人称赞他们是网络时代的普罗米修斯，盗取和传递了文明的火种。单纯制作和分享字幕应该不算是侵犯版权的行为，但字幕组常常还顺带压制和分享资源，更不用说其片源的获取常常都是以盗版的手段来进行的，这使得字幕组绝对是盗版资源的重要推动者，尽管他们并不完全为了利益（事实上几乎都是无偿奉献，并且靠传播盗版资源来获利是非常严重的违法行为）。我无意指责字幕组，相反，我也非常感激和依赖他们。试想在国内，如果没有字幕组和盗版资源，你可能有正规的途径收看 Netflix 的最新剧集么？我这并非为盗版站台，我也想付费观看 Netflix 等平台的剧集，为自己喜欢的公司和作品用钱投票，只不过总有不可抗力在阻拦。希望有一天这种不可抗力会消失不见，尽管这种希望似乎变得越来越渺茫。

然而 BT 也并非无所不能，去中心化存储意味着你不能要求每个用户在下载完成后都在电脑里保存资源并持续做种，下完就删或停止分享的行为屡见不鲜。如果是一些年代久远的、或非常冷门的资源，你很难指望自己能将它完整下载下来。流媒体为代表的中心化存储的优势就体现在这里，只要中央服务器还安全存在，这个资源就永远存在并能获得。

显然，盗版和 BT 下载不会消亡，即便是在版权法律更为严格、审查相对宽松的国家，也存在着大量下载盗版资源的行为，因为尽管羞于承认，盗版除了自由以外，最大的优点就是免费。但盗版又必须被控制，否则人人都依赖盗版，那内容制作者就得不到收入来源，也无法投入金钱去做出优质的内容了，这反过来会使得你没有盗版资源可下载，形成恶性循环。所以长期来看，付费流媒体服务一定会是主流（也的确已经成为了主流），而以 BT 下载为代表的盗版则会变得越来越小众。

至于我，对于能够付费观看的作品尽量选择付费观看（去电影院或购买各大视频网站会员），而大量无法付费观看的内容，也不得不选择盗版资源。另外希望大家不要丧失 BT 下载的技能，毕竟真的涉及到分享违禁资源的时候，BT 比百度网盘靠谱多了。
