---
order: 3
icon: ph:sword-bold
---

# 战斗流程协议

`resource/copilot/*.json` 的使用方法及各字段说明

::: tip
请注意 JSON 文件是不支持注释的，文本中的注释仅用于演示，请勿直接复制使用
:::

## 完整字段一览

```json
{
    "stage_name": "暴君", // 关卡名，必选。关卡中文名、code、stageId、levelId等，只要能保证唯一均可。
    "opers": [
        // 指定干员
        {
            "name": "重岳", // 干员名
            "skill": 3, // 技能序号。可选，默认为 1，取值范围 [1, 3]
            "skill_usage": 2, // 技能用法。可选，默认为 0
            // 0 - 不自动使用（依赖 "actions" 字段）
            // 1 - 好了就用，有多少次用多少次（例如干员 棘刺 3 技能、桃金娘 1 技能等）
            // 2 - 使用 X 次（例如干员 山 2 技能用 1 次、重岳 3 技能用 5 次，通过 "skill_times" 字段设置）
            // 3 - 自动判断使用时机（画饼.jpg）
            // 如果是全自动的技能，填 0
            "skill_times": 5, // 技能使用次数。可选，默认为 1
            "requirements": {
                // 练度要求。保留接口，暂未实现。可选，默认为空
                "elite": 2, // 精英化等级。可选，默认为 0, 不要求精英化等级
                "level": 90, // 干员等级。可选，默认为 0
                "skill_level": 10, // 技能等级。可选，默认为 0
                "module": 1, // 模组编号。可选，默认为 0
                "potentiality": 1 // 潜能要求。可选，默认为 0
            }
        }
    ],
    "groups": [
        {
            "name": "任意正常群奶", // 群组名，必选
            // 自己随便取名字，与下面 deploy 中的 name 对应起来就行
            "opers": [
                // 干员任选其一，无序，会优先选练度高的，用法同上述 opers 字段
                {
                    "name": "夜莺",
                    "skill": 3,
                    "skill_usage": 2 // 若是指定时刻，不同的干员技能 cd 可能不同，请留意
                },
                {
                    "name": "白面鸮",
                    "skill": 2,
                    "skill_usage": 2
                }
            ]
        }
    ],
    "actions": [
        // 战斗中的操作。有序，执行完前一个才会去执行下一个。必选
        {
            "type": "部署", // 操作类型，可选，默认为 "Deploy"
            // "Deploy" | "Skill" | "Retreat" | "SpeedUp" | "BulletTime" | "SkillUsage" | "Output" | "SkillDaemon" | "MoveCamera"
            // "部署"   |  "技能"  |  "撤退"   | "二倍速"   |  "子弹时间"  |  "技能用法"   | "打印"  |  "摆完挂机" | "移动镜头"
            // 中英文皆可，效果相同
            // 若为 "部署", 当费用不够时，会一直等待到费用够（除非 timeout）
            // 若为 "技能", 当技能 cd 没转好时，一直等待到技能 cd 好（除非 timeout）
            // "二倍速" 是可切换的，即使用一次变成二倍速，再次使用又变回一倍速
            // "子弹时间" 即点击任意干员后的 1/5 速度，再进行任意 action 会恢复正常速度
            //      "name" 或 "location" 必填一项，即点哪个干员进入子弹时间，战场上的干员或待部署区的干员均可（会自动判断）
            //      若下一个动作是 "技能" / "撤退"，需要填与下一个动作相同的 "name" 或 "location"
            //      若下一个动作是 "部署"，随便填谁（但最好不要填待部署的那个人，头像被点了会影响识别）
            // "打印" 界面不显示这条步骤，仅用于输出 doc 里的内容（用来做字幕之类的）
            // "摆完挂机" 仅使用 "好了就用" 的技能，其他什么都不做，直到战斗结束
            // "移动镜头" 用于 “引航者试炼” 模式，还需要填写 distance 字段
            // 目前下面四个条件是且的关系，即 &&
            "kills": 0, // 击杀数条件，如果没达到就一直等待。可选，默认为 0，直接执行
            "costs": 50, // 费用条件，如果没达到就一直等待。可选，默认为 0，直接执行
            // 费用受潜能等影响，可能并不完全正确，仅适合对时间轴要求不严格的战斗。
            // 否则请使用下面的 cost_changes
            // 另外仅在费用是两位数的时候识别的比较准，三位数的费用可能会识别错，不推荐使用
            "cost_changes": 5, // 费用变化量条件，如果没达到就一直等待。可选，默认为 0，直接执行
            // 注意是从开始执行本 actions 开始计算的（即前一个 action 结束时的费用作为基准）
            // 支持负数，即费用变少了（例如“孑”等吸费干员使得费用变少了）
            // 另外仅在费用是两位数的时候识别的比较准，三位数的费用可能会识别错，不推荐使用
            "cooling": 2, // CD 中干员数量条件，如果没达到就一直等待。可选，默认为 -1，不识别
            // TODO: 其他条件
            // TODO: "condition_type": 0,    // 执行条件间的关系，可选，默认为 0
            //                        // 0 - 且； 1 - 或
            "name": "棘刺", // 干员名 或 群组名， type 为 "部署" 时必选，为 "技能" | "撤退" 时可选
            "location": [
                5,
                5
            ], // 部署干员的位置。
            // type 为 "部署" 时必选。
            // type 为 "技能" | "撤退" 时可选，
            // "技能"：仅推荐场地上自动的装置等，不填写 name，并使用 location 开启技能。正常部署的干员推荐使用 name 开启技能
            // 撤退"：仅推荐有多个同名召唤物时，不填写 name, 并使用 location 进行撤退。正常部署的干员推荐 name 进行撤退
            "direction": "左", // 部署干员的干员朝向。 type 为 "部署" 时必选
            // "Left" | "Right" | "Up" | "Down" | "None"
            // "左"   |  "右"   | "上"  | "下"   |  "无"
            // 中英文皆可，效果相同
            "skill_usage": 1, // 修改技能用法。当 type 为 "技能用法" 时必选
            // 举例：刚下淘金娘需要她帮忙打几个怪，不能自动开技能，中后期平稳了需要她自动开技能
            // 则可以在对应时刻设置为 1
            "skill_times": 5, // 技能使用次数。可选，默认为 1
            "pre_delay": 0, // 前置延时。可选，默认为 0, 单位毫秒
            "post_delay": 0, // 后置延时。可选，默认为 0, 单位毫秒
            // "timeout": 999999999,   // 保留字段，暂未实现。
            // 超时时间。当 type 为 "部署" | "技能" 时可选。默认 INT_MAX, 单位毫秒
            // 等待超时则放弃当前动作, 转而执行下一个动作
            "distance": [
                4.5,
                0
            ], // type 为 "移动镜头" 时必选
            // [ x 移动格子数，y 移动格子数 ]，可为小数
            // 注意 "移动镜头" 时是识别不到正在站场中的，需要用 sleep 完整覆盖整个移动动画
            "doc": "下棘刺了！", // 描述，可选。会显示在界面上，没有实际作用
            "doc_color": "orange" // 描述文字的颜色，可选，默认灰色。会显示在界面上，没有实际作用。
        },
        // 举例 1
        {
            "name": "任意正常群奶",
            "location": [
                5,
                6
            ],
            "direction": "右"
        },
        // 举例 2
        {
            "name": "史尔特尔",
            "location": [
                4,
                5
            ],
            "direction": "左",
            "doc": "你史尔特尔奶奶来啦！",
            "doc_color": "red"
        },
        // 举例 3
        {
            "type": "二倍速"
        }
    ],
    "minimum_required": "v4.0", // 最低要求 maa 版本号，必选。保留字段，暂未实现
    "doc": {
        // 描述，可选。
        "title": "低练度高成功率作业",
        "title_color": "dark",
        "details": "对练度要求很低balabala……", // 建议在这里写上你的名字！（作者名）、参考的视频攻略链接等
        "details_color": "dark"
    }
}
```

## 举例

[OF-1](https://github.com/MaaAssistantArknights/MaaAssistantArknights/blob/master/resource/copilot/OF-1_credit_fight.json)