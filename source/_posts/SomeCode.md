---
title: SomeCode
date: 2024-12-25 15:47:19
abstract: 一些场景下的需求代码。有所思考的内容。
tags:
    - Working
categories:
    - Blog
---

## 场景记录

### 累积活跃度奖励

#### 场景概述

给定某 N 个槽位用于领奖（比如累积活跃度奖励），支持按周增量更新配置奖励内容及奖励条件。

#### 配置结构

| reward_id | from_week_id | need_active_num | rewards |
| :-: | :-: | :-: | :-: |
| 1 | 1 | 2 | aaa |
| 2 | 1 | 4 | bbb |
| 3 | 1 | 8 | ccc |
| 4 | 1 | 16 | ddd |
| 5 | 1 | 32 | eee |
| 6 | 1 | 64 | fff |
| 2 | 5 | 4 | bbb' |

#### 详细说明

有 6 个奖励项（N = 6），每个奖励项从第一周开始存在奖励配置。例如第一周开始，要求 reward_id(1) 的奖励项达成条件是 `need_active_num` 达到 2；reward_id(3) 的奖励达成条件是 `need_active_num` 达到 8。从第 5 周开始，奖励项 reward_id(2) 的奖励条件由 `need_active_num` 的 4 保持为 4，而奖励发生了变更。如何支持配置的索引结构来高效支持这样的频繁查询？需要支持至少以下两种功能接口：

- `const WeeklyActiveRewardConf *GetWeeklyActiveRewardConf(std::uint32_t reward_id, std::uint32_t week_id)` 按照给定周的给定奖励项，获取得到相应的奖励项配置。

- `const std::vector<const WeeklyActiveRewardConf *> &GetWeeklyActiveRewardConf(std::uint32_t week_id)` 按照给定周来获取当前周所有的奖励项配置。比如上述从第五周开始，调用此方法得到的配置项的奖励列表应当是 `aaa`、`bbb'`、`ccc`、`ddd`、`eee` 和 `fff`。

#### 实现代码

```cpp
    static const std::uint32_t max_reward_num = 6U;
    for (const auto &reward_conf : WeeklyActiveRewardConfVector::GetInst().GetConfAll())
    {
        if (weekly_active_reward_conf_.find(reward_conf.from_week_id) == weekly_active_reward_conf_.end())
        {
            weekly_active_reward_conf_[reward_conf.from_week_id] =
                std::vector<const WeeklyActiveRewardConf *>(max_reward_num, nullptr);
        }
        CHECK_EXP_ELOG(reward_conf.reward_id <= 0U || reward_conf.reward_id > max_reward_num, continue,
                       "invalid reward_id|must from 0 to %u|reward_id:%u", max_reward_num, reward_conf.reward_id);
        weekly_active_reward_conf_[reward_conf.from_week_id][reward_conf.reward_id - 1UL] = &reward_conf;
    }
    for (auto iter = weekly_active_reward_conf_.begin(); iter != weekly_active_reward_conf_.end(); ++iter)
    {
        for (std::size_t i = 0UL; i < iter->second.size(); ++i)
        {
            CHECK_EXP(iter->second[i] != nullptr, continue);
            if (iter == weekly_active_reward_conf_.begin())
            {
                ERR_LOG(0UL, "no reward config|week_id:%u|reward_id:%zu", iter->first, i + 1UL);
                return false;
            }
            else
            {
                iter->second[i] = std::prev(iter)->second[i];
            }
        }
    }
```

利用这样的缓存的 `weekly_active_reward_conf_` 数据，可以满足上述需求。对此的两个接口定义为：

```cpp
    const WeeklyActiveRewardConf *GetWeeklyActiveRewardConf(std::uint32_t reward_id, std::uint32_t week_id) const
    {
        COND_RET(weekly_active_reward_conf_.empty(), nullptr);
        auto iter = weekly_active_reward_conf_.upper_bound(week_id);
        COND_RET(iter == weekly_active_reward_conf_.begin(), nullptr);
        const auto &conf_vec = std::prev(iter)->second;
        COND_RET(reward_id <= 0 || reward_id > conf_vec.size(), nullptr);
        return conf_vec[reward_id - 1U];
    }
```

```cpp
    const std::vector<const WeeklyActiveRewardConf *> &GetWeeklyActiveRewardConf(std::uint32_t week_id) const
    {
        static std::vector<const WeeklyActiveRewardConf *> empty;
        COND_RET(weekly_active_reward_conf_.empty(), empty);
        auto iter = weekly_active_reward_conf_.upper_bound(week_id);
        COND_RET(iter == weekly_active_reward_conf_.end(), empty);
        return std::prev(iter)->second;
    }
```
