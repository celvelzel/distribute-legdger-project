# Distribute Ledger Project

基于 Solidity 的两阶段课程项目：
- **Stage1**：自定义 Token 合约（严格 API）
- **Stage2**：双人掷骰对战合约（Commit-Reveal + 奖励池）

> 当前仓库以 `contracts/Stage1.sol`、`contracts/Stage2.sol` 为唯一源码真相。

## 核心能力

- 双人等额 ETH 对局（A 建局，B 加入）
- Commit-Reveal 揭示机制
- 结算奖励：赢家获得全部 ETH + Stage1 代币奖励（默认每局最多 100）
- 超时处理：`checkTimeout()`（30 分钟）
- 等待加入期间可取消：`cancelGame()`（仅 A，30 分钟内）
- Stage1 代币支持 `mint / transfer / sell / close`

## 项目结构

```text
.
├─ contracts/
│  ├─ Stage1.sol
│  └─ Stage2.sol
├─ docs/
│  ├─ README.md
│  ├─ user/
│  ├─ deployment/
│  ├─ internal/
│  ├─ project/
│  └─ final_submit/
├─ scripts/
├─ tests/
└─ remix.config.json
```

## 快速开始（Remix）

1. 在 Remix 打开 `contracts/Stage1.sol` 并部署（构造参数：`name`, `symbol`）
2. 部署 `contracts/Stage2.sol`（构造参数为 Stage1 地址）
3. 用 Stage1 给 Stage2 预充代币奖励池（`transfer(stage2, amount)`）
4. 游戏流程：
   - A：`createDiceGame(fingerPrintForA)`（附带 ETH）
   - B：`joinGame(fingerPrintForB)`（ETH 必须等额）
   - A/B：`revealA`、`revealB`
   - 超时场景：`checkTimeout()`

## 关键接口说明

### Stage1（当前实现）

- `getName()` / `getSymbol()` / `getPrice()`
- `totalSupply()` / `balanceOf(account)`
- `transfer(to, value)`
- `mint(to, value)`（仅 owner）
- `sell(value)`（600 wei / token）
- `close()`（仅 owner，销毁并转出剩余 ETH）

> 注意：Stage1 为课程要求的**严格自定义 API**，并非完整 ERC20（无 `approve/allowance/transferFrom`）。

### Stage2（当前实现）

- `createDiceGame(bytes32)`
- `joinGame(bytes32)`
- `revealA(bytes32)` / `revealB(bytes32)`
- `checkTimeout()`
- `cancelGame()`
- `getRemainingTimeout()` / `getRemainingCancelTime()`

## 文档导航

- [文档总览](./docs/README.md)
- [用户使用指南](./docs/user/user-guide.md)
- [功能与接口说明](./docs/user/functionality.md)
- [部署指南](./docs/deployment/deployment-guide.md)
- [部署记录（实际交易）](./docs/deployment/record.md)
- [变更摘要（Git）](./docs/internal/update-report.md)

## 许可证

MIT

---

最后更新：2026-04-24
