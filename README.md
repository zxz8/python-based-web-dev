# 基于Python后端的网络开发调研

石守珂（电子科技大学），田超（傲势科技）

## 概述

本项目旨在调研基于Python的网络开发框架机制，充分利用现有资源，调研使用Python作为后端，丰富变换各类前端的Web开发方式。

现代互联网开发中，前后端分离开发会带来很多很多好处，通过前端工程师与后端工程师商讨好开发API接口，提取数据的规约与协议，就可以实现各自分工双线并行协作。尽管JavaScript目前是Web开发的主流生力军，同时由于NodeJs的崛起，导致JavaScript逐步开始蚕食后端开发领域市场，老牌的Java、PHP、Ruby等后台开发占比逐步下降，但是就目前JavaScript语言本身所携带的局限性（无法面向对象、类型转换过于弱化、设计模式实施困难、作用域设计不合理等），JavaScript很难形成长期稳定的后端开发代码迭代，后端可持续性较差。

既然互联网开发是前端和后端是分离的，前端开发由于目前浏览器的历史发展以来，只能使用JavaScript/Html/CSS作为技术栈，后端的选择则是变化无穷的Java、PHP、Ruby、Python都可以选择。傲势科技的技术发展血统是以科学计算为先导，傲势公司的创始人有一部分都来自于或从事过数值分析和科学计算领域相关的问题，为了后续技术栈的可延续性，后端开发的调研工作首先集中在开发语言的选择上。

备选语言有：Java、PHP、Ruby、Go、C++、Python、JavaScript/NodeJs，调研工作需要具备如下几个特点：

1. 后端开发语言必须是一门严肃的语言；（Java、PHP、Ruby、Go、C++、Python等都可以，JavaScript私以为并非一门严肃的开发语言，稍微上规模的后端服务都会非常慎重的选择JavaScript/NodeJs）；
2. 可以方便的开发后端Web服务；（Java、PHP、Ruby、Go、Python，不得不承认C++开发Web应用并非易事，但是C++在开发一些后端服务中间件时有不可替代的优势）；
3. 具备成熟的后端Web服务开发框架；（Java、PHP、Ruby、Go、Python）
4. 面向对象或者具备严整的面向对象机理；（Java、PHP、Ruby、Go、Python，其中Go语言本身并不面向对象，但是可以通过接口来抽象方法接口）
5. 擅长科学计算（只剩下了Python，其他几个都在科学计算方面效率较低）

Python作为傲势科技后端数字化服务的首先语言，首先因为其上述特点，当然后端服务和未来的微服务架构下，后端的服务颗粒化、弹性化，各种服务中间件都会选择最为擅长的语言实现，而Python会作为主要的后端服务开发语言。



## 前端框架与后端框架

一门语言是否可以迎来较大的发展，除了语言本身是否符合时代需要符合现代主流的开发方法论之外，另一个非常重要的催化机制在于“是否有繁荣社区维护的成熟框架”。使用语言必谈其相关的框架，框架的生命力才是语言最终的生命力来源。

- C++有Qt框架，基本垄断非手机类嵌入式设备开发市场，Qt框架的包罗万象和繁荣社区是C++于现代生命力的重要因素；
- JavaScript有Express.js作为老牌的前端框架统据一方，而其在前端的Vue.js、React.js、Koa.js更是令JavaScript在前端具有无可争议的地位；
- Java有Spring MVC框架，令其在全栈开发市场仍然割据一方；
- Python的后端框架有Django和Tornado，二者在设计哲学上稍有不同，但由于其跟科学计算、人工智能和大数据都有密不可分的关系，因而使用Python作为后端框架仍然很有市场；

每一种语言的主要应用领域主要是由**其对应的主流框架**在前端和后端分别擅长的方向所决定。

前端框架的选型是一项非常繁杂的工作，但是前端开发中为了提升代码的复用性，提升代码的维护效率，前端的选择往往会更倾向于模块化自动化维护的工具链。由于前端的框架实在是太多，如何学习前端成为一个很有技巧性的开放命题。在[专治前端焦虑的学习方案](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551432&idx=1&sn=749f99e25de7a3c451a42f6d157d7d40&chksm=8025a109b752281f39fda3ab07d9997bca42408bad991663a6229b5b2fdee9bec20239ca1a03&mpshare=1&scene=1&srcid=1217enOnGmoiEUabZCs3SHjK&pass_ticket=YWbmLG4aRyF0kFS02U2OD7NtK3qoB76HbpDKyJns2vc%3D#rd)中，作者总结了目前主流的前端框架所采用的技术点。

后端框架的选型需要做到服务接口和API的通畅，满足前端工程师对于有效数据的筛选的便利性。前端和后端的框架的融合整体上需要实现[一个高可用可扩展的分层架构](https://mp.weixin.qq.com/s?__biz=MjM5ODYxMDA5OQ==&mid=2651959728&idx=1&sn=933227840ec8cdc35d3a33ae3fe97ec5&chksm=bd2d046c8a5a8d7a13551124af36bedf68f7a6e31f6f32828678d2adb108b86b7e08c678f22f&mpshare=1&scene=1&srcid=0703Rito1HXSnoWVrJuNcs6D&pass_ticket=YWbmLG4aRyF0kFS02U2OD7NtK3qoB76HbpDKyJns2vc%3D#rd)，这一架构通常包含有如下几个层次：

1. 客户端（浏览器或移动设备）；
2. DNS域名服务器；
3. 负载均衡（Nginx服务器反向代理）；
4. Web服务器（Web容器）；
5. 业务服务器（微服务、业务中间件、消息队列、计算中心、日志系统、流程引擎等等）；
6. 数据库缓存（缓存热数据）
7. 数据库（读写分离、主从分离）

一般而言，网站服务的基础架构是逐步演化的，在最开始原型截断，DNS服务器是由云服务商提供的，负载均衡可以暂时不考虑。

由上可知，前端和后端的分离，发生在Web服务器层，客户端的请求会直接发送给Web服务器，返回Web服务器渲染好的页面交给客户端浏览器，浏览器收到的数据只有一系列的Html、JavaScript、CSS数据，具体可以参见：[聊聊浏览器的渲染机制](https://mp.weixin.qq.com/s?__biz=MzAxODE2MjM1MA==&mid=2651551666&idx=1&sn=9288432c1383366fadacec55941517db&chksm=8025a073b75229655b746f92f5b0d2e511d75bebf8de2063b0ea619d492e3681036e0039432b&mpshare=1&scene=1&srcid=07033cIhqDtaYjVmdurWGV7I&pass_ticket=YWbmLG4aRyF0kFS02U2OD7NtK3qoB76HbpDKyJns2vc%3D#rd)和[浏览器的渲染原理及流程](http://www.cnblogs.com/slly/p/6640761.html)。既然Web服务器返回的数据基本上只有Html、JavaScript、CSS三种数据，那么如何在Web服务器端将此三种数据合理组装并发送回浏览器，直接影响了前端界面的展示效果和用户体验。那么前端框架是通过何种方式访问到后端的数据的？API是如何协作的？前端框架如何保证只刷新客户端需要改变的部分而不是完全更新浏览器所有的页面数据？这都是本次调研的基本问题。

## 技术路线

本次调研主要技术路线如下：

1. 在本地搭建并运行[Django+React.js](https://github.com/Seedstars/django-react-redux-base) ，研究这个代码仓库中前端框架中所涉及的每个模块的功能和工具链条中的作用，针对Django+React.js前后端协作的API接口进行调研，给出说明报告；
2. 在本地搭建并运行[Django+Vue.js](https://github.com/NdagiStanley/vue-django) ，研究这个代码仓库中前端框架中所涉及的每个模块的功能和工具链条中的作用，针对Django+Vue.js前后端协作的API接口进行调研，给出说明报告；
3. 上述两个使用的后端框架是Python-Django，如果更换了后端框架，使用方法上是否存在区别？请参考[python-react](https://github.com/markfinger/python-react)以及[tornado-bootstrap](https://github.com/paulocheque/python-tornado-bootstrap)等项目，针对tornado与Django的不同给出调研报告；（Tornado python web服务器是nodejs的竞品，都是非阻塞I/O事件驱动的后端框架，而Django是基于信号的python后端框架）。
4. 对比分析Tornado与Django后端框架的不同。







