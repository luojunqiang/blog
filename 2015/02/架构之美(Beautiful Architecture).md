# Beautiful Architecture
 
- Functionality
- Changeability
- Performance
- Capacity
- Ecosystem
- Modularity
- Buildability
- Producibility
- Security
 
好的架构会影响组织架构。
 
架构分层的指导原则：下层比上层的执行速度快9倍，执行频率高9倍。
 
开发团队中的健康工作关系将直接有益于软件设计。不健康的关系和个性膨胀会导致不健康的软件。
 
不良架构的影响不仅限于代码。它会进一步影响到人、团队、过程和时间表。
 
项目的早期确定采用的类库、顶层文件结构、如何对事物命名、编码风格、单元测试框架、基础设施（版本控制、构建系统、持续集成）。
 
 
## DarkStar
 
### DataService
 
### TaskService
 
所有的数据操作在一个Transaction中完成，可以进行冲突检测。
 
### SessionService
 
在登陆和认证后，客户端和服务器间建立起一个会话。
 
### ChannelService
 
一对多的通信机制。

## 记忆留存项目

- 应该由客户（而不是提供者）确定接口。
- 复制原始对象而不要引用，可以保留一个到原始对象的连接信息以便后续能根据原始对象进行更新。
- RESTful方式要求请求是无状态的。生成响应需要的所有信息都要放在请求中。

- jakarta.apache.org/bcel
- asm.objectweb.org/
- gnu classpath

plugin 要作为第一等公民存在，其开发应尽量简单。如果可能应做到隔离plugin问题影响主应用。

考虑对未来的兼容和影响，不要让使用者依赖于不必要的细节。

