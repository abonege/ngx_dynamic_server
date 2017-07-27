# ngx_dynamic_service
support create/update/delete nginx server dynamically

1.WHY
为什么需要这样一个东西呢？这是云环境的一个需求。对于普通的nginx使用者来说，都是固定的站，固定的bind ip和port。但是如果是面对云上的客户的话，默认的做法就不够了。
因为云上的客户会不断的创建、更新自己的站配置，虽然这些配置可以写nginx.conf文件，然后reload。但这样的方式显然是不妥当的。因为面对极为频繁的更新，reload的方式实在是难于接受。所以我们需要这样的一个功能，即动态的更新server。
2.HOW
有了这样的想法，那么我们采用什么样的思路，去实现这个呢？这里我们可以参考tengine的dynamic upstream的做法。把配置转为一段一段的server block，然后每个进程去解析，然后加载到自己的worker进程中。
对于配置这样的东西，nginx的原生方案是：master加载配置，然后clone出来的进程天然继承这些配置。但如果想改动配置，两个思路：
1）这些配置放到共享内存里，每个worker都可见
2）这些配置放到msg queue里，每个worker同步到自己的进程中，然后销毁msg
dynamic upstream采用的方案2，这样有2个好处：1.锁很少，甚至没有 2.复用原有逻辑，调用原来的parser即可。
在dynamic service项目中，我们也采用方案2

3.DESIGN

