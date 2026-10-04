# STATUS（本仓库对齐状态 · v0.1）

> 维护约定：每完成一项改状态并在“证据”列挂测试/回执路径。
> 标记：✅ 对齐 ｜ ⚠️ 偏离/待校准 ｜ ❌ 未实现 ｜ ➕ 本机创新 ｜ ➖ 层级替代

## 内核（A1/A2/A3）

| 项 | 状态 | 证据 |
| --- | --- | --- |
| τ=200（可改） | ✅ | `lean/patches/0001`、`config.py`、tests |
| c_AND=64 | ✅ | `lean/patches/0002`、`mcts/search.py`、`test_c_and...` |
| 未访问惩罚=32 | ✅ | 同上、`test_c_and...` |
| -40 兜底 | ✅ | `lean/patches/0003`、`config.py` |
| 渐进采样不重置（G2） | ✅ | `test_g2_reexpansion_never_resets_children` |
| prior 归一（G3） | ✅ | `test_prior_normalization_g3` |
| value target 官方语义（G1） | ✅ | `test_compute_value_target_minimax`、`test_extract_transitions...` |
| Q 变换 / AND 选择 / min 回传 | ✅ | `tests/test_mcts_parity.py` |
| 树规模 / 自适应预算 | ⚠️ | max_nodes/steps=64/64；官方预算模型后置 |

## 网络（A4）

| 项 | 状态 | 证据 |
| --- | --- | --- |
| 64 桶价值头（唯一） | ✅ | `tests/test_value_head64.py` |
| two-hot / 期望解码 | ✅ | 同上、`test_value_targets.py` |
| s18 头可加载性 | ⚠️ | 云探针进行中（`/mnt/workspace/alphaproof-aligned/probe/`） |
| 7B + LoRA 联合前向 | ⚠️ | 云端 real smoke 脚本已备（`scripts/cloud_real_smoke.py`） |

## 更新（A5）

| 项 | 状态 | 证据 |
| --- | --- | --- |
| 离线 CE 模式（policy+value） | ✅ | `tests/test_offline_learner.py`、`run_expert_iteration` 回执 |
| 反证样本进训练 / timeout 剔除 | ✅ | 同上、smoke 输出 `replay={'proof':4,'disproof':1,...}` |
| SFT 10% 混比 | ✅ | `OfflineBatchBuilder`、`sft_mix=0.1` |
| 在线 batch=1 / =N | ✅ | `tests/test_online_learner.py`、smoke 输出 4 次 / 1 次 |
| 正奖励须 terminal_verified | ✅ | `test_positive_reward_requires_verification` |
| value 目标不回落 r | ✅ | `test_value_target_optional_but_no_r_fallback` |
| value 权重 1e-3 重标定 | ⚠️ | 配置已切；真实训练校准曲线待跑 |

## 环境与测试

| 项 | 状态 | 证据 |
| --- | --- | --- |
| Lean 4.28 环境（Gloway） | ⏳ | 安装中：`/mnt/gloway/projects/lean-4.28-reap/state/` |
| 纯 CPU 测试（my-new-linux） | ✅ | `8 passed, 3 skipped`（`~/venvs/alpha104`） |
| 含 torch 全量测试（云 ROCm） | ✅ | `18 passed in 5.00s` |
| Pell Mock 端到端冒烟 | ✅ | 搜索 7 节点 / 8 模拟；两种更新回执 |
| 真实 Lean + 单道 Pell 全流程 | ⏳ | 等 Lean 环境（`docs/real_lean_next.md`） |
| 真实 7B 云端冒烟 | ⏳ | `scripts/cloud_real_smoke.py` |
