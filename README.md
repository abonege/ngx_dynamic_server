# ngx_dynamic_service
support create/update/delete nginx server dynamically

1.WHY

为什么需要这样一个东西呢？这是云环境的一个需求。对于普通的nginx使用者来说，都是固定的站，固定的bind ip和port。但是如果是面对云上的客户的话，默认的做法就不够了。因为云上的客户会不断的创建、更新自己的站配置，虽然这些配置可以写nginx.conf文件，然后reload。但这样的方式显然是不妥当的。因为面对极为频繁的更新，reload的方式实在是难于接受。所以我们需要这样的一个功能，即动态的更新server。

2.HOW

有了这样的想法，那么我们采用什么样的思路，去实现这个呢？这里我们可以参考tengine的dynamic upstream的做法。把配置转为一段一段的server block，然后每个进程去解析，然后加载到自己的worker进程中。
对于配置这样的东西，nginx的原生方案是：master加载配置，然后clone出来的进程天然继承这些配置。但如果想改动配置，两个思路：
1）这些配置放到共享内存里，每个worker都可见
2）这些配置放到msg queue里，每个worker同步到自己的进程中，然后销毁msg
dynamic upstream采用的方案2，这样有2个好处：1.锁很少，甚至没有 2.复用原有逻辑，调用原来的parser即可。
在dynamic service项目中，我们也采用方案2

3.使用场景

先说一下我们当前的nginx适用的场景
1）nginx直接对外
2）nginx前面有一个类似LVS的L4分流设备

对于1）来说只最简单的，nginx直接提供服务。但对于更多的云环境来说，为了安全等因素，前面一般加上L4的分流设备。那么从L4到nginx就只有内网的连接了。那么在这种情况下，该如何区分租户，身份识别呢？方式有2个：a)通过域名。b)通过虚拟vs透传。a这样的方式最简单，也是最常见的。但基于域名，还是有些限制的，比如想限制用户per-listener的L4的quota，基于per-listener的统计，或者对于虚拟主机服务商来说，他们的域名太多了，配置管理过于繁琐。我觉得b，即后者更炫酷一点，就是通过virtual listener这样的东西来区分，nginx上就像看到client连接我的公网ip一样。意不意外？惊不惊喜？

不管怎么样，这些都是用户的选型时候的均衡，对于我们dynamic service来说，都是一样的。

