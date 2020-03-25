#  认证授权，SSO，分布式，基础速记
1. 认证 (Authentication) 和授权 (Authorization)的区别是什么？
- 认证：身份验证
- 授权：权限验证
2.  什么是Cookie ? Cookie的作用是什么?如何在服务端使用 Cookie ?
- 什么是Cookie？ 网站为了辨别用户身份而储存在用户本地终端上的数据
- Cookie的作用是什么？ Cookie 存放在客户端，一般用来保存用户信息
- 如何在服务端使用 Cookie？ request.getCookies()、@CookieValue、response.addCookie()
3. Cookie 和 Session 有什么区别？如何使用Session进行身份验证？
- Cookie一般存储用户数据，存放在客户端，安全性较低
- Session一般用于服务器跟踪状态，存放在服务段，安全性较高
- 可以使用SessionID。用户登录成功后，服务器生成一个SessionID，存储到Redis，然后返回客户端带有SessionID的Cookie。客户端下一次请求就会带上该SessionID，这样子后端就知道用户身份了。
4. 如果没有Cookie的话Session还能用吗？
- 可以。比如把SessionID放在url里面，或者新增一个sessionID的header 等等
5. 为什么Cookie 无法防止CSRF攻击，而token可以？
- CSRF（Cross Site Request Forgery），跨站请求伪造
- Cookie 中的 SessionId 是由浏览器发送到服务端的，这是个特性，而且无法取消
- 而 Token 一般存储在 LocalStorage，是需要程序员编码才能发送给服务端，黑客就没办法伪造请求，因为没有token
6.  什么是 Token?什么是 JWT?如何基于Token进行身份验证？
- 什么是Token：服务端生成的访问令牌。用户第一次登录成功后，服务器生成并返回token，客户端以后的请求只需带上token而无需用户名和密码。
- 什么是JWT：一段签名的JSON格式的数据
- 如何基于Token进行身份验证？
    - Header：描述 JWT 的元数据。定义了生成签名的算法以及 Token 的类型。
    - Payload：实际需要传递的数据
    - Signature：服务器通过Payload、Header和一个密钥(secret)使用 Header 里面指定的签名算法（默认是 HMAC SHA256）生成。
    - 过程：服务其生产并返回token，客户端存储到localStorage，以后的请求都带上这个token，放在 Authorization 请求头即可。
7. 什么是 SSO?
- 单点登录，说的是用户登陆多个子系统的其中一个就有权访问与其相关的其他系统
8. SSO与OAuth2.0的区别
- OAuth2 是一个行业的标准授权协议，主要用来授权第三方应用获取权限。比如：我们系统对接的 美团外卖、饿了么外卖、配送平台、微信公众平台、华为云市场、电信云市场、
- SSO 解决的是一个公司多个相关的子系统的之间的登陆问题。比如：我们系统里的 商户后台、ERP后台、小程序后台、赚餐后台
9. 注销登录等场景下 token 还有效，token 的续签问题
- todo
10. CAP定理
- 一致性（Consistence） :所有节点访问同一份最新的数据副本
- 可用性（Availability）:每次请求都能获取到非错的响应——但是不保证获取的数据为最新数据
- 分区容错性（Partition tolerance） : 分布式系统在遇到某节点或网络分区故障的时候，仍然能够对外提供满足一致性和可用性的服务
11. 聊聊分布式事务，再说说解决方案
- 事务的参与方分布在不同的服务器上，分布式事务需要保证参与方要么全部成功，要么全部失败
- 解决方案：
    - 2PC（基于XA协议）：性能不好，调用链过长时可用性很低
    - TCC（try/confirm/cancel）：要写很多补偿代码，而且可能会出现confirm或cancel回滚不了的情况
    - 本地消息表：消息表和业务表放在同一个事务，失败消息重新发送。也需要写挺多额外代码
    - 消息事务 + 最终一致性：发消息+做业务+确认消息发送。需要发送方实现check方法，来决定回滚还是继续。java 中可以采用 RocketMQ
    - Sagas 微服务架构设计模式一书中有提到，分为两种：协同式 和 编排式，推荐使用 编排式，这个比较新颖也比较难，折腾了2天后决定还是不引入项目
12. 什么是 RPC?RPC原理是什么?
- RPC（Remote Procedure Call）—远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。
- 原理：4个角色 Client、Client Stub、Server Stub、Server 的时序图

补充：

20. 中心化和去中心化
- 中心化：leader主备；领导安危问题；k8s
- 去中心化：leader选举；“脑裂”问题；ZK、etcd、
20. 分布式和集群区别
- 分布式：一个业务分拆多个子业务，部署在不同的服务器
- 集群：同一个业务，部署在多个服务器上
20. BASE定理
- Basically Available（基本可用）：允许损失部分可用性
- Soft-state（软状态）：允许系统中的数据存在中间状态
- Eventually Consistent（最终一致性）：系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态
21. 一致性协议/算法
- Paxos 没看过
- Raft算法 看过，说不清楚
- Zab算法 Zookeeper的自研算法