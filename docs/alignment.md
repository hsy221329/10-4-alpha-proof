# 对齐映射（N33 → 本仓库实现位置）

> 官方可信度顺序：补充材料表格 Table 1–7 > `pseudocode.py` > 档案推导 > `(S)` 建议值。
> 本仓库只实现“语义/公式/关键超参”的对齐；规模不复刻。

## A1 · MCTS 超参

| 项目 | 官方 | 本仓库 | 位置 |
| --- | --- | --- | --- |
| K 采样数 | 6 | 6 | `alphaproof/config.py::SearchConfig.num_samples` |
| c_base / c_init | 3200 / 0.001 | 同 | 同上 |
| γ | 0.99 | 同 | `visit_discount` |
| 先验温度 τ | 200 | **200（可改）** | `prior_temperature`；Lean 补丁 0001 |
| 渐进采样 C / α | 0.01 / 0.6 | 同 | `ps_c / ps_alpha` |
| c_AND | 64 | **64** | `c_and`；Lean 补丁 0002 |
| 未访问惩罚 | 32 | **32** | `unvisited_penalty`；Lean 补丁 0002 |
| 无合法动作兜底 | -40 | **-40** | `no_legal_actions_value`；Lean 补丁 0003 |
| 每步奖励 | -1 | focus 边 0 / 其余 1（等价细分） | `mcts/tree.py::Edge.step_cost` |
| Q 变换 | γ^(-1-V) | 同 | `mcts/search.py::puct_scores` |
| 根 Dirichlet 噪声 | 无 | 无 | — |

## A2/A3 · 算法逻辑

| 伪码 | 本仓库 | 位置 | 测试 |
| --- | --- | --- | --- |
| `run_mcts` | `MCTS.search` | `mcts/search.py` | `test_full_pell_mock_search...` |
| `select_child` + `ucb_score` | `_select` + `puct_scores` | 同上 | `test_prior_normalization_g3`、`test_c_and...` |
| `expand_node` | `_expand`（去重、AND focus、不重置） | 同上 | `test_g2_reexpansion_never_resets_children` |
| `progressive_sample` | `_progressive_sample`（仅 OR） | 同上 | `test_progressive_sampling_condition` |
| `backpropagate` | `_backprop` | 同上 | 端到端断言 |
| `backprop_value_towards_min` | `backprop_towards_min` | `mcts/tree.py` | 端到端断言 |
| `compute_value_target` | `compute_value_target` | `targets/value_targets.py` | `test_compute_value_target_minimax` |
| `extract_transitions` | `extract_transitions`（只取最优路径） | 同上 | `test_extract_transitions_only_optimal_path` |
| `final_check` | Lean 侧保留（`checkProof`）；Mock 用脚本复验替代 | `env/base.py` | — |

## A4 · 网络

| 项目 | 官方 | 本仓库 | 说明 |
| --- | --- | --- | --- |
| 参数规模 | 3B enc-dec | REAL-Prover 7B decoder-only | ➖ 有理由的替代（规模不复刻） |
| hidden | 1024/256 | 3584 | 实测值 |
| value 形态 | 64 桶 categorical | **仅 64 桶**（tanh 标量版已放弃） | `net/value_head.py` |
| value 解码 | `-Σ p_b·b`（V=-d̂） | 同 | `expected_distance` |
| 标签 | d(s) | two-hot（1..64，terminal 0 → 桶 1） | `two_hot_target` |
| 共享 backbone | 一次 forward 两路 | 同 | `net/policy_value.py` |
| 微调 | 全参 | LoRA r16/α32 | `update_offline/make_hf_encode_fn` 配套 |

## A5 · 参数更新

| 项目 | 官方 | update_offline | update_online |
| --- | --- | --- | --- |
| 范式 | 离线专家迭代（replay 批量） | ✅ | 在线 RTTT（本机 ➕） |
| policy 损失 | CE(搜索选中动作) | ✅ | -r·Δ + βΔ²（本机） |
| value 损失 | 64 桶 CE，权重 1e-3 | ✅ 1e-3 | ✅ 1e-3（显式 d(s)，不回落 r） |
| 样本 | 证明+反证；timeout 剔除 | ✅（可开关） | 已验证奖励（正奖励须验证） |
| batch | 4096（可配） | `--batch-size` | `learn_batch_size` |
| SFT 混比 | 10% | ✅ `sft_mix=0.1` | 不适用 |
| 重标定说明 | — | 历史 0.5 → 1e-3 后需按 64 桶 CE 重新校准（保留配置项） | 同 |

## 尚未实现（后续里程碑）

- Matchmaker 自适应预算官方化（250×1.17^f, cap 16000；trust 8/12）；
- 候选成本函数 μ|a|+t_exec；
- TTRL 变体生成 + 聚焦 RL（官方第二阶段）；
- auto-formalization 课程（teacher + 三闸门接入）；
- 评测协议（solve@B、等计算、3 seeds、独立 Lean 复验的自动化流水）。
