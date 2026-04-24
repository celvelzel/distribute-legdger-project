# 更新摘要（基于近期 Git 记录）

本文档按近期提交整理，聚焦“新增/变更了什么”，不展开长篇实现细节。

## 2026-04-17

- **文档归档整理**（`docs: final_submit`）
  - 更新 `docs/final_submit/` 下报告与归档合约副本
  - `docs/internal/report_unfilled.tex` 重命名为 `docs/internal/report.tex`
- **新增部署交易记录**（`交易记录`）
  - 新增 `docs/deployment/record.md`
- **Stage2 结算路径修复**（`重入取消`）
  - 更新 `contracts/Stage2.sol`
  - 关键点：修复结算流程中状态清理与转账顺序相关问题
- **作业命名回归到仓库标准文件名**（`main`）
  - `contracts/` 目录恢复为 `Stage1.sol` / `Stage2.sol`

## 2026-04-16

- **修复 Stage2 重入锁死锁问题**（`fix: 修复重入锁死锁...`）
  - 移除导致内部流程锁冲突的路径
  - 调整 `joinGame` 错误类型处理
- **报告体系补齐（中英文/LaTeX/PDF）**
  - 更新 `docs/internal/report.md`、`report_zh.md`、`report.tex`
  - 更新 `docs/final_submit/report/*.pdf`
- **课程提交材料归档补齐**
  - 新增 `docs/final_submit/contracts/*Stage1.sol`、`*Stage2.sol`
- **文档目录重组**（`docs: 整理文档目录结构`）
  - 用户、部署、内部报告分层到 `docs/user`、`docs/deployment`、`docs/internal`

## 2026-04-09 ~ 2026-03-20

- **Stage1/Stage2 核心能力完成并强化安全性**
  - Stage1：严格 API、自定义 token、`mint/transfer/sell/close`
  - Stage2：双人对战、Commit-Reveal、超时处理、取消游戏、奖励池发放
- **代码文件清理**
  - 删除历史教学示例合约（如 `1_Storage.sol`、`2_Owner.sol`、`3_Ballot.sol`、`demo.sol`）

## 本次文档对齐（2026-04-24）

- README 改为与当前真实实现一致：
  - 明确 Stage1 为课程严格 API（非完整 ERC20）
  - 修正文档链接到 `docs/` 新结构
- 用户与功能文档修正：
  - 去除不存在的 `approve/allowance/transferFrom/withdraw/emergencyWithdrawToken`
  - 补充 `close()`、`DuplicateParticipant`、真实函数签名
  - 修正 reveal 参数说明（需传原始 `bytes32 secret`）
- 新增 `docs/README.md` 作为文档导航入口

---

最后更新：2026-04-24
