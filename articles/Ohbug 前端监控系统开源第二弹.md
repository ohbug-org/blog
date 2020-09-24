# Ohbug 前端监控系统开源第二弹

这里是 *Ohbug* 开源计划第二弹。第一弹的 [Ohbug SDK](https://github.com/ohbug-org/ohbug) 我们已经可以收集到数据，这一弹聊一聊数据的处理。

依据数据的流向，我大概绘制了一个架构图：

![core_process](https://raw.githubusercontent.com/ohbug-org/blog/master/images/core_process.drawio.png)

这里解释一下几个核心模块的功能：
- SDK：数据采集 [SDK](https://github.com/ohbug-org/ohbug)
- Transfer: 数据接收、预处理
- Manager：任务管理，包括 *Event* 聚合、存取 *Event*、存取 *Issue*、通知任务的生成
- Notifier：通知任务的执行，包括 email、[webpush](https://github.com/web-push-libs/web-push)、webhook
- Dashboard：控制台 API 的实现

可能图片解释的不够清楚，这里还是系统介绍一下：

## Transfer

首先看一眼 *Event* 的数据结构：

```typescript
interface OhbugEvent<D> {
  apiKey: string
  appVersion?: string
  appType?: string
  releaseStage?: OhbugReleaseStage
  timestamp: string
  category?: OhbugCategory
  type: string
  sdk: OhbugSDK

  detail: D
  device: OhbugDevice
  user?: OhbugUser
  actions?: OhbugAction[]
  metaData?: any
}
```

*Event* 通过 [SDK](https://github.com/ohbug-org/ohbug) 收集上来以后，通过 *Nginx* 转发给 *Transfer*。

为了方便存储，*Transfer* 需要对不确定数据类型的字段预处理为字符串，这里转化 `detail` `actions` `metaData` 三个字段。
具体转化的细节可见源码 [formatter](https://github.com/ohbug-org/ohbug-server/blob/master/packages/transfer/src/api/report/report.service.ts#L27)。

之后将预处理后的 *Event* 给到 *Manager*。

## Manager

*Manager* 收到 *Transfer* 传来的 *Event* 后，首先对 *Event* 进行聚合，聚合的目的是将相同的 *Event* 合并，毕竟数据展示时不希望看到一堆相同的问题占据视野。

聚合的思路是根据数据结构对 *Event* 进行解构，例如 *Fetch* 的异常就取出 *url*, *method*, *data* 这些数据，通过 md5 加密后得到聚合的依据 `intro`。

```typescript
case FETCH_ERROR:
  return {
    agg: [detail.req.url, detail.req.method, detail.req.data],
    metadata: {
      type,
      message: detail.req.url,
      others: detail.req.method,
    },
  };
```

得到了 `intro` 就可以将相同 `intro` 的 *Event* 合并到一个数据集里，我们称每一个数据集为 *Issue*，然后就可以将数据持久化存储了。

*Event* 的存储通过 *kafka* 通知到 *logstash*，*logstash* 存入 *elasticsearch*。

*Issue* 的存储则直接使用 *postgresql*，另外 *Ohbug* 使用 [typeorm](https://github.com/typeorm/typeorm) 来操作 *postgresql*，好处是方便做数据库之间的迁移。

## Notifier

*Manager* 在完成 *Issue* 的存储后，会判断当前条件是否符合[通知规则](https://ohbug.net/docs/dashboard/SettingProject#%E9%80%9A%E7%9F%A5)，再将 *Issue* 与通知内容一起传给 *Notifier*。

*Notifier* 的工作就是就收到任务后触发指定通知，包括 email、[webpush](https://github.com/web-push-libs/web-push)、webhook 的发送/触发。

![notifier_dingtalk](https://raw.githubusercontent.com/ohbug-org/blog/master/images/notifier_dingtalk.png)
![notifier_email](https://raw.githubusercontent.com/ohbug-org/blog/master/images/notifier_email.png)

## Dashboard

*Dashboard* 为控制台整体流程的提供支持。主要工作是为控制台前端提供数据，例如下图

![issues](https://raw.githubusercontent.com/ohbug-org/ohbug-website/master/static/images/dashboard-issues.png)

另外 *Issue* 对应的 *Event* 数据通过 *elasticsearch* 获取：

![event](https://raw.githubusercontent.com/ohbug-org/ohbug-website/master/static/images/dashboard-event.png)

除此之外 *Dashboard* 还实现了用户系统、团队、项目、通知、SourceMap 的增删改查。

关于 *Dashboard* 的实现其实还有很多东西可以聊，以后可以单独拿出一篇单聊。

## 结语

目前的实现还是属于比较基础的数据采集分析产品，以后还有许多可拓展的方向，如果你出于平台数据安全的考虑，Ohbug 也支持[本地部署](https://github.com/ohbug-org/ohbug-server)。

一个人的力量有限，我们拥抱开源一方面也是为了集中力量办事，希望有更多开发者加入进来，完善 *Ohbug*。

上述所提及的内容均以开源 [Ohbug Server](https://github.com/ohbug-org/ohbug-server)，如果你想抢先体验可以试用 [Ohbug](https://ohbug.net) 官方服务。
更多内容可见 [Ohbug Docs](https://ohbug.net/docs/integration/Installation)。

---

![wechat](https://raw.githubusercontent.com/ohbug-org/blog/master/images/wechat.jpg)
