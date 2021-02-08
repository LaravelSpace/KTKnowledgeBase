## Stellaris（群星）

### 控制台

按下键盘`~`键呼出控制台。

开关命令表示输入命令式会切换命令生效的状态。

| 命令                                                  | 效果                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| help                                                  | 显示一个指令的帮助文档。                                     |
| debugtooltip                                          | 开关，查看ID、效果等信息。                                   |
| event [event_name]                                    | 执行一个事件。[事件ID.目标ID/event_id.target_id]             |
| instant_build                                         | 开关，瞬间建造，建造费用为0，对AI同样生效。                  |
| invincible                                            | 玩家的舰队将不会受到伤害。                                   |
| yesmen                                                | 开关，AI总是同意玩家提出的条约。                             |
| add_trait_leader [leader_id] [trait_name/trait_id]    | 添加领袖特性。                                               |
| remove_trait_leader [leader_id] [trait_name/trait_id] | 移除领袖特性。                                               |
| finish_research                                       | 瞬间完成当前科技。                                           |
| research_all_technologies [数值1] [数值2]             | 瞬间研究所有科技。数值1为事件武器和天灾武器和飞升武器，1为研究，0为不研究，数值2为循环科技等级。 |
| research_technology [tech_name/tech_id]               | 瞬间研究某项科技(2.0.2-未来版本）。                          |
| resource [resource_name/resource_id] [数值]           | 添加资源，数值不填，默认为5000。                             |
| influence [数值]                                      | 添加影响力，数值不填，默认为5000。                           |
| grow_pops [数值]                                      | 添加成长中的人口至选中的星球，数值不填，默认为 1。           |
| unity [数值]                                          | 添加凝聚力，数值不填，默认为5000。                           |

#### 领袖特性

打开debugtooltip，只输入`add_trait_leader [leader_id]`不输入具体特性ID时，控制台会输出领袖特性列表里面可以用的特性的ID。同理，只输入`remove_trait_leader [leader_id]`不输入具体特性ID时，控制台会输出领袖当前拥有的特性的ID。

#### 资源

| 资源名称   | 资源代码(resource_name) | 资源 ID(resource_id) |
| ---------- | ----------------------- | -------------------- |
| 活体金属   | sr_living_metal         | 14                   |
| 泽珞       | sr_zro                  | 15                   |
| 暗物质     | sr_dark_matter          | 16                   |
| 纳米机器人 | nanites                 | 17                   |
| 稀有文物   | minor_artifacts         | 18                   |

##### 科技

| 科技名称       | 科技代码(tech_name)         | 科技 ID(tech_id) |
| -------------- | --------------------------- | ---------------- |
| 巨像           | tech_colossus               | 0                |
| 地爆天星       | tech_pk_cracker             | 1                |
| 暗物质偏导护盾 | tech_dark_matter_deflector  | 125              |
| 暗物质能源     | tech_dark_matter_power_core | 126              |
| 暗物质推进     | tech_dark_matter_propulsion | 127              |
| 星门激活       | tech_gateway_activation     | 143              |
| 星门建造       | tech_gateway_construction   | 144              |
| 戴森球         | tech_dyson_sphere           | 145              |
| 物质解压器     | tech_matter_decompressor    | 146              |
| 环形世界       | tech_ring_world             | 147              |

#### 事件

| 事件ID      | 描述                     |
| ----------- | ------------------------ |
| distar.50   | 迷失个体，泡泡           |
| distar.170  | 神经共生体研究           |
| graygoo.400 | 安静散步，灰风获得       |
| graygoo.550 | 诡异的宁静，灰风前置事件 |

### MOD

#### 舰RMOD事件

| 事件ID          | 事件描述                     |
| --------------- | ---------------------------- |
| wsg.1100        | 异世界信号，NEO合成工艺          |
| wsg.3105        | 我认为列克星敦就是世界的主宰 |
| wsg.3066 | 神秘的AI飞船，大小姐出现 |
| wsg.3075 | 对话结束，大小姐获得 |
| wg_lady.1 | 堕落妈开漫展，大小姐改造 |
| wsg_shimakaze.1 | 奇怪的新舰娘，岛风获得            |
| wg_crisis.2  | 消失的银河系们，联合舰队事件开始。<br>同时触发堕落妈跑路或者觉醒 |
| wg_fe.22 [幻梦余烬国家id] | 觉醒烛幽誓约             |
| wg_fe.23                  | 堕落妈觉醒为烛幽誓约     |
| sofe_crisis.35            | 堕落妈评价星辰之子是伪神 |
| z_fw1_relic_koh.312 | 她回来了，良良多小鼬 |

