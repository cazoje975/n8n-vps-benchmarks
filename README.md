# n8n VPS 推荐：2 核 4G 够不够？一年花多少钱？续费会不会涨？（附 RackNerd 全套餐对比与实测吞吐数据）

n8n 自托管，**2 vCPU + 4 GB 内存**是大多数人该买的配置，年付一百美元以内能拿下；要跑面向客户的 webhook，4 vCPU 是性价比拐点，再往上加核基本是浪费钱。

按这个标准，RackNerd 的 4 GB KVM VPS 是 2 vCore / 4 GB / 60 GB SSD / 7 TB 流量，**$59.99 一年**，折合每月 5 美元。下面把三件事讲清楚：n8n 到底吃多少资源、2 核和 4 核差多少、以及为什么第二年才是真正该看的账。

> **选购速览**：只跑自己的自动化，选 2 GB KVM VPS（$35.99/年）；有对外 webhook 或小团队生产环境，选 4 GB KVM VPS（$59.99/年）；要 PostgreSQL 加多个 AI 工作流同居，选 6 GB（$89.99/年）。1 GB 那档能装 n8n，但内存会先顶到上限，只适合先试水。
> 👉 [看 4 GB KVM VPS 当前价格（$59.99/年）](https://my.racknerd.com/aff.php?aff=11397&pid=954)

## n8n 到底吃多少 CPU 和内存

n8n 的官方最低配置是 1 vCPU、2 GB 内存、20 GB SSD。1 GB 内存能启动，但稍微复杂点的工作流就会内存报错，不建议。推荐档是 2–4 vCPU、4–8 GB 内存、40–80 GB NVMe，系统用 Ubuntu 24.04 LTS，用 Docker 部署，带宽 1 TB 起步、生产环境按 2 TB 算。

装完之后实际占多少，有具体数字。2026 年 11 月 5 日在一组 Hetzner 云服务器上跑的压测，用的是 n8n 2.46.6 + PostgreSQL 16 + Docker Compose，赫尔辛基机房：n8n 容器空闲时 379.7 MiB，PostgreSQL 69.6 MiB，整机空闲 1036 MiB；50 并发打满时 n8n 容器峰值 **1049.6 MiB**，到此为止没再涨。安装本身占 4.7 GB 磁盘。

内存这条线到 4 GB 就到顶了，剩下的瓶颈全在 CPU。

## 实测：2 核、4 核、8 核差多少

同一个 webhook 工作流（触发一个 Code 节点并返回结果），ApacheBench 打 300–2000 次请求，并发 1 / 10 / 25 / 50：

| 机型 | 配置 | 月价 | 峰值 executions/s | 10 并发 p95 | 50 并发 p95 |
| --- | --- | --- | --- | --- | --- |
| CX23 | 2 vCPU / 4 GB | 6.64 € | 15.04 | 893 ms | 3954 ms |
| CAX11（ARM） | 2 vCPU / 4 GB | 7.25 € | 13.9 | 934 ms | 4114 ms |
| **CX33** | 4 vCPU / 8 GB | 10.27 € | **31.93** | **438 ms** | 1846 ms |
| CAX21（ARM） | 4 vCPU / 8 GB | 12.69 € | 25.77 | 528 ms | 2197 ms |
| CX43 | 8 vCPU / 16 GB | 19.35 € | 36.14 | 341 ms | 1682 ms |

这组数字里有三件事值得记住。

**2 vCPU 的 15 次/秒对个人用户够用。** 一天 86400 秒，15 次/秒是理论上限，实际你一天可能就几百次触发。2 核机器从 10 并发开始吞吐就停在 15 附近不动了，延迟继续爬，50 并发时中位延迟 3445 ms、p95 3954 ms。

**4 vCPU 买到的是延迟。** 从 2 核换到 4 核，峰值吞吐翻倍到 31.93 次/秒，同样 10 并发下 p95 从 893 ms 掉到 438 ms。如果你的 webhook 是同步等结果的场景——表单提交、聊天机器人回复、给前端返回数据——4 秒和 0.4 秒是两个产品。这台的每欧元吞吐也是五台里最高的，0.32 € 换一次/秒。

**8 vCPU 只多 13%，价格贵 88%。** CX43 跑到 36.14 次/秒，比 CX33 高 13%，月费从 10.27 € 涨到 19.35 €。原因是常规模式下单个 n8n 主进程既收 webhook 又跑执行，吃不满 8 个核。要突破得换架构：queue mode，PostgreSQL + Redis + worker，可选再加 webhook processor 和负载均衡。持续跑超过 30 次/秒的轻工作流，加核不如换架构。

ARM 机型在这个场景里不划算：CAX11 比更便宜的 CX23 慢，CAX21 比便宜 2.42 € 的 CX33 慢。n8n 的 Node 运行时在 x86 上更吃得开。

## RackNerd 全部 KVM VPS 套餐

RackNerd 目前在售的 KVM VPS 特价机型是年付制，全部带 RAID-10 纯 SSD、1Gbps 端口、10Gbps DDoS 防护、完整 root 权限，机房覆盖洛杉矶、圣何塞、西雅图、达拉斯、亚特兰大、芝加哥、新泽西、纽约、阿什本、阿姆斯特丹和法国。

| 套餐 | vCPU / 内存 | 存储 / 流量 | 价格（年付） | 购买 |
| --- | --- | --- | --- | --- |
| 1 GB KVM VPS | 1 vCore / 1 GB | 20 GB SSD / 3 TB | $21.99/年 | [ 1 GB 年付 $21.99](https://my.racknerd.com/aff.php?aff=11397&pid=952) |
| 2 GB KVM VPS | 2 vCore / 2 GB | 35 GB SSD / 5 TB | $35.99/年 | [ 2 GB 年付 $35.99](https://my.racknerd.com/aff.php?aff=11397&pid=953) |
| 4 GB KVM VPS | 2 vCore / 4 GB | 60 GB SSD / 7 TB | $59.99/年 | [ 4 GB 年付 $59.99](https://my.racknerd.com/aff.php?aff=11397&pid=954) |
| 6 GB KVM VPS | 6 vCore / 6 GB | 100 GB SSD / 12 TB | $89.99/年 | [ 6 GB 年付 $89.99](https://my.racknerd.com/aff.php?aff=11397&pid=955) |
| 8 GB KVM VPS | 7 vCore / 8 GB | 150 GB SSD / 20 TB | $119.99/年 | [ 8 GB 年付 $119.99](https://my.racknerd.com/aff.php?aff=11397&pid=956) |

特价机型的库存和价格会变，下单前以购物车页面为准。RackNerd 的 ctrl 面板里有一键应用，n8n、Coolify、Docker + Portainer 都在列表里，装完凭据直接显示在面板中；也可以 SSH 上去自己装。官方说明里写着 n8n 从 2 GB 内存起比较舒服，OpenClaw 这类要 8 GB，AnythingLLM、Ollama、LibreChat 要 4 GB。

注意 4 GB 那档和 6 GB 那档的核数差得有点大：2 vCore 到 6 vCore。对单进程 n8n 来说，6 核的收益按上面的实测是递减的；这台的真正用处是把 PostgreSQL、Redis、第二个应用一起塞进去，或者上 queue mode 后跑多个 worker。

## 按场景选：四档怎么对号入座

**只跑自己的自动化。** 日程同步、表格读写、Slack 通知、几十条工作流，一天触发几百次。2 GB KVM VPS，$35.99/年。2 GB 是 n8n 的最低线，配 SQLite 更稳；想同时上 PostgreSQL 会有点紧。

👉 [用 2 GB 套餐先跑起来（$35.99/年）](https://my.racknerd.com/aff.php?aff=11397&pid=953)

**有对外 webhook、要给用户实时返回。** 订单事件、CRM 回调、客服机器人。4 GB KVM VPS，$59.99/年。多花的 24 美元买的是 p95 从 893 ms 到 438 ms 这一档延迟。

**PostgreSQL + 多个 AI 工作流 + 别的应用同居。** 6 GB KVM VPS，$89.99/年。AI 推理走 OpenAI、Claude 的 API 时不需要 GPU，但多个工作流并发时内存和数据库连接会一起涨。

**多租户、要跑 Dify 或本地模型。** 8 GB KVM VPS，$119.99/年。真要在这台机器上跑 Ollama，内存得按模型大小另算，8 GB 只是入场券。

1 GB 那一档 $21.99/年，价格确实低。但 n8n 容器峰值就要 1 GB，操作系统本身还要占几百 MB，跑起来会频繁碰到内存上限。想先熟悉界面可以，别拿它接生产流量。

## 续费价才是你第二年要付的钱

RackNerd 这些套餐标的是年付价，续费是否同价，下单时在购物车里核对账单周期和金额，别只看页面上那个数字。

行业里的常规做法是首期促销、续费回到原价，差距可以很大。已核验的两个例子：Hostinger KVM 2 首期 **$8.79/月**，续费 **$14.99/月**，而它是 2 vCPU / 8 GB——2 核正好是吞吐瓶颈，8 GB 内存在 n8n 场景里用不上，续费价还高过 4 vCPU 的 Hetzner CX33（10.27 €/月）。ABLENET 的 L2 套餐契约时特别价 1,536 日元/月，更新后 1,706 日元/月。

算法很简单：把年付总额除以 12 得到月成本，再跟续费价、跟同类机型的月付价横向比。$59.99/年 = $5/月，$119.99/年 = $10/月。

顺便放一个对照：n8n Cloud 的 Starter 是 20 €/月（年付），2500 次执行、5 并发；Pro 是 50 €/月（年付），10000 次执行。自托管没有执行次数上限，代价是机器、域名、备份、升级全归你。

👉 [按年付价开通 4 GB 套餐](https://my.racknerd.com/aff.php?aff=11397&pid=954)

## 五个容易踩的坑

**数据库一开始就用 PostgreSQL。** 小规模用 SQLite 没问题，但 SQLite 不支持分布式部署，将来想上 queue mode 得迁移。queue mode 需要 PostgreSQL + Redis + worker，可选再加 webhook processor。

**一键应用不等于托管运维。** RackNerd 的 VPS 是非托管的，ctrl 面板负责开关机、重装系统、VNC 控制台、一键应用和网络防火墙，OS 补丁、防火墙规则、n8n 版本升级、备份都归你。应用层的问题去 n8n 官方文档和社区找，不是找 VPS 客服。

**HTTPS 不是可选项。** n8n 里存着 API key、数据库密码、客户数据。对外只开 443（申请证书时临时开 80），SSH 的 22 端口限制来源 IP。

**备份放到 VPS 之外。** RAID-10 防的是磁盘故障，防不了你误删工作流或者升级翻车。异地存一份，成本几美元一年。

**执行历史会长胖。** 安装只占 4.7 GB，60 GB 的盘起步够用，但执行记录会一直增长，把 pruning 打开。

## 2026 年选机器要看的两件事

**n8n 3.0 起自托管只支持 Docker。** 据 3.0 的发布信息，3.0.0 在 2026 年 10 月 27 日发布 beta，稳定版计划 12 月初；从 3.0 开始 npm 上不再有可运行的 n8n 包，`npx n8n` 那套老办法会失效。选机器时确认能跑 Docker——RackNerd 支持 ctrl 里一键装 Docker + Portainer，也可以 SSH 自己装，两条路都通。这也意味着 1 GB 内存的机器更没有余地了，容器化的开销躲不掉。

**云厂商的价格在动。** 上面那组 Hetzner 数字是 2026 年 11 月 5 日抓取的，Hetzner 在 2026 年重新定过价，别拿两三年前的文章比价。同样，RackNerd 的特价机型价格随库存调整，表格里的数字是当前页面的口径。

## 下单前的最后两个确认

按需求挑：个人自动化 → 2 GB（$35.99/年）；对外 webhook 和小团队生产 → 4 GB（$59.99/年）；PostgreSQL 加多应用 → 6 GB（$89.99/年）。

下单时确认两件事：购物车里的计费周期和续费金额，以及机房位置。RackNerd 有十余个机房，美国西海岸（洛杉矶、圣何塞、西雅图）对国内访问和亚太区 API 的延迟通常比东海岸低，如果你的工作流主要打国内或亚洲的服务，优先选西海岸。

👉 [挑一台 RackNerd KVM VPS 开始装 n8n](https://bit.ly/RacKnerd)
