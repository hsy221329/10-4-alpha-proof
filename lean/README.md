# lean/ · Reap 内核对齐补丁

对上游 `IQuestLab/reap@0090d73c5f739e4d74000e053b00fd0148ff46aa`（v1 训练容器同版本）的**最小对齐补丁**：

| 补丁 | 文件 | 内容 |
| --- | --- | --- |
| `0001-align-options.patch` | `Options.lean` | τ 默认 200；新增 `reap.c_and=64`、`reap.unvisited_penalty=32`、`reap.no_legal_actions_value=-40` |
| `0002-align-puct.patch` | `Tactic/TreeSearch.lean` | `computePUCTScores` 支持未访问惩罚与 AND 探索乘子；`SearchHyperparameters` 扩展 |
| `0003-align-fallback.patch` | `Tactic/Generator.lean` | 值服务失败兜底 -1000 → -40 |

## 应用顺序（在 v1 训练补丁 0001–0003 之后）

```bash
cd <reap-repo>
patch -p1 --dry-run -i 0001-align-options.patch && patch -p1 -i 0001-align-options.patch
patch -p1 --dry-run -i 0002-align-puct.patch    && patch -p1 -i 0002-align-puct.patch
patch -p1 --dry-run -i 0003-align-fallback.patch && patch -p1 -i 0003-align-fallback.patch
```

安装好的环境（Gloway `/mnt/gloway/projects/lean-4.28-reap`）说明见 `docs/real_lean_next.md`。

## 语义对照

- 未访问子节点：官方 “V(s,a) = V_net(s) - c_pen（Table 3: 32）”，本补丁按 `node.data.valueSum / numVisit` 作为 `V(s)`；
- AND 节点：探索项 `U` 整体乘 `c_AND=64`；
- 其余语义（Q 变换、focus 代价、min 回传、渐进采样）保持上游不动（本就与官方一致）。
