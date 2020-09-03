```json
{
  "updated_at": "2020-06-29",
  "updated_by": "KelipuTe",
  "tags": "Stellaris"
}
```

---

## Stellaris —— 群星

### 控制台

按下键盘 `~` 键呼出控制台。

[开关] 表示命令每输入一次，就切换一次开关状态。

| 命令                                                  | 效果                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| add_trait_leader [leader_id] [trait_name/trait_id]    | 添加领袖特性。                                               |
| remove_trait_leader [leader_id] [trait_name/trait_id] | 移除领袖特性。                                               |
| activate_ascension_perk [ap_name]                     | 激活飞升天赋。                                               |
| debugtooltip                                          | [开关] 查看 ID、效果等信息。                                 |
| event [event_name]                                    | 执行一个事件。[事件 ID.目标 ID/event_id.target_id]           |
| finish_research                                       | 瞬间完成当前科技。                                           |
| research_technology [tech_name/tech_id]               | 瞬间研究某项科技(2.0.2-未来版本）。                          |
| research_all_technologies [数值1] [数值2]             | 瞬间研究所有科技。[数值1] 为事件武器和天灾武器和飞升武器，1 为研究，0 为不研究，[数值2] 为循环科技等级。 |
| grow_pops [数值]                                      | 添加成长中的人口至选中的星球，[数值] 不填，默认为 1。        |
| help                                                  | 显示一个指令的帮助文档。                                     |
| instant_build                                         | [开关] 瞬间建造，建造费用为0，适用于AI。                     |
| invincible                                            | 玩家的舰队将不会受到伤害。                                   |
| resource [resource_name/resource_id] [数值]           | 添加资源，[数值] 不填，默认为 5000。                         |
| unity [数值]                                          | 添加凝聚力，[数值] 不填，默认为 5000。                       |
| influence [数值]                                      | 添加影响力，[数值] 不填，默认为 5000。                       |
| yesmen                                                | [开关] AI 总是同意玩家提出的条约。                           |

#### 领袖特性

打开 debugtooltip，只输入 `add_trait_leader [leader_id]` 不输入具体特性时，控制台会输出领袖特性列表里面会有特性的 ID。

#### 资源

`resource [resource_name/resource_id] [数值]`，添加资源，[数值] 不填，默认为 5000。

| 资源名称   | 资源代码(resource_name) | 资源 ID(resource_id) |
| ---------- | ----------------------- | -------------------- |
| 活体金属   | sr_living_metal         | 14                   |
| 泽珞       | sr_zro                  | 15                   |
| 暗物质     | sr_dark_matter          | 16                   |
| 纳米机器人 | nanites                 | 17                   |
| 稀有文物   | minor_artifacts         | 18                   |

##### 科技

`research_technology [tech_name/tech_id]`，瞬间研究某项科技。

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

#### 飞升

`activate_ascension_perk [ap_name]`，激活飞升天赋。

| 科技名称             | 科技代码(tech_name)                     | 科技 ID(tech_id) |
| -------------------- | --------------------------------------- | ---------------- |
| 巨像计划             | ap_colossus                             | 2                |
| 世界塑性师           | ap_world_shaper                         | 9                |
| 星河卫士             | ap_defender_of_the_galaxy               | 11               |
| 戒心永存             | ap_eternal_vigilance                    | 14               |
| 逐鹿星河             | ap_galactic_contender                   | 15               |
| 太空子民             | ap_voidborn                             | 24               |
| 建筑大师             | ap_master_builders                      | 25               |
| 星河奇迹（乌托邦）   | ap_galactic_wonders_utopia              | 26               |
| 星河奇迹（寰宇企业） | ap_galactic_wonders_megacorp            | 27               |
| 星河奇迹（乌+寰）    | ap_galactic_wonders_utopia_and_megacorp | 28               |

#### 事件

| 事件ID     | 描述                         |
| ---------- | ---------------------------- |
| distar.170 | 神经共生体研究               |
| wsg.3105   | 我认为列克星敦就是世界的主宰 |
|            |                              |
|            |                              |

### MOD

#### 舰R MOD

| 命令                            | 效果                                         |
| ------------------------------- | -------------------------------------------- |
| **wg_crisis**                   | **联合舰队事件**                             |
| event wg_crisis.2               | 消失的银河系们，同时触发堕落妈妈跑路或者觉醒 |
| event wg_crisis.3               | 联合舰队第一个传送门                         |
| event wg_crisis.6               | 星系能源波动                                 |
| event wg_crisis.7               | 来自另一个次元的我们                         |
| event wg_crisis.8               | 看起来我们有大麻烦了                         |
| event wg_crisis.22              | 反抗军介绍界面                               |
| event wg_crisis.23              | 反抗军外交界面                               |
| event wg_crisis.33              | 反抗军抵达                                   |
| event wg_crisis.34              | 反抗军抵达                                   |
| event wg_crisis.54              | 联合舰队第二个传送门                         |
| event wg_crisis.63              | 反抗军组建好舰队                             |
| event wg_crisis.95              | 联合舰队小型传送门                           |
| **wg_fe**                       | **幻梦余烬事件**                             |
| event wg_fe.22 [幻梦余烬国家id] | 烛幽誓约觉醒                                 |
| event wg_fe.23                  | 堕落妈妈觉醒为烛幽誓约                       |
| sofe_crisis.35                  | 堕落妈妈评价星辰之子是伪神                   |
