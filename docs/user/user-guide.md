# 使用文档

本指南将帮助您快速部署和使用分布式账本智能合约系统。

> 说明：本指南对应当前仓库 `contracts/Stage1.sol` 与 `contracts/Stage2.sol`。

## 目录

1. [部署准备](#部署准备)
2. [部署步骤](#部署步骤)
3. [游戏流程](#游戏流程)
4. [常见问题](#常见问题)

---

## 部署准备

### 1. 环境检查

确保您的开发环境满足以下要求：

| 工具 | 版本要求 |
|------|----------|
| Node.js | >= 16.0 |
| npm/yarn | 最新版本 |
| MetaMask | 最新版本 |

### 2. 获取测试网ETH

在部署到主网前，建议先在测试网进行测试：

- **Sepolia Testnet**: 访问水龙头获取测试ETH
- **Goerli Testnet**: 访问水龙头获取测试ETH

### 3. 准备钱包

1. 安装 MetaMask 浏览器扩展
2. 创建或导入钱包
3. 获取测试网ETH
4. 记录所有者账户的私钥（用于部署）

---

## 部署步骤

### 方式一：使用 Remix IDE（推荐新手）

1. **打开 Remix IDE**
   访问 https://remix.ethereum.org

2. **导入合约**
   - 在 File Explorer 中创建新文件
   - 将 `Stage1.sol` 和 `Stage2.sol` 的代码粘贴进去

3. **编译 Stage1**
   - 选中 Stage1.sol
   - 点击 "Compile" 按钮
   - 确保编译成功无错误

4. **部署 Stage1**
   - 切换到 "Deploy & Run Transactions" 面板
   - 选择 "Injected Provider - MetaMask"
   - 在 CONTRACT 下拉菜单选择 "Stage1"
   - 在 constructor 参数中输入：`"GameToken", "GT"`
   - 点击 "Deploy"
   - 在 MetaMask 中确认交易

5. **复制 Stage1 地址**
   - 部署成功后，在 Deployed Contracts 面板复制合约地址
   - 例如：`0x1234...abcd`

6. **部署 Stage2**
   - 编译 Stage2.sol
   - 选择 "Stage2" 合约
   - 在 constructor 参数中粘贴 Stage1 地址
   - 点击 "Deploy"

### 方式二：使用 Hardhat

当前仓库默认是 Remix 工作流，未提供完整 Hardhat 工程配置（仅保留 `scripts/` 示例文件）。
如需 Hardhat，请先自行初始化后再迁移 `contracts/` 目录。

---

## 游戏流程

### 阶段一：创建游戏

**玩家A（庄家）操作：**

1. 生成随机数作为秘密：
   ```javascript
   // 生成随机秘密
   const secretA = crypto.randomBytes(32).toString('hex');
   ```

2. 计算哈希承诺：
   ```javascript
   const fingerPrintForA = web3.utils.keccak256(secretA);
   ```

3. 在前端调用：
   ```javascript
   // 下注 ETH 创建游戏
   await stage2Contract.createDiceGame(fingerPrintForA, {
     value: web3.utils.toWei("0.01", "ether")
   });
   ```

4. 分享游戏链接和指纹给玩家B

### 阶段二：加入游戏

**玩家B操作：**

1. 获得玩家A的指纹后，生成自己的秘密
   ```javascript
   const secretB = crypto.randomBytes(32).toString('hex');
   const fingerPrintForB = web3.utils.keccak256(secretB);
   ```

2. 调用加入函数：
   ```javascript
   await stage2Contract.joinGame(fingerPrintForB, {
     value: web3.utils.toWei("0.01", "ether")  // 必须与A下注金额相同
   });
   ```

### 阶段三：揭示秘密

**玩家A揭示：**
```javascript
await stage2Contract.revealA(secretABytes32);
```

**玩家B揭示：**
```javascript
await stage2Contract.revealB(secretBBytes32);
```

注意：`revealA/revealB` 入参是 `bytes32` 原始秘密值，
合约内部校验 `keccak256(abi.encodePacked(secret)) == fingerprint`。

### 阶段四：结算

双方都揭示后，系统自动结算：

- **随机数生成**: 使用双方的秘密、块号、时间戳、地址生成
- **胜负判定**: 1-3点 = A胜，4-6点 = B胜
- **奖金分配**: 赢家获得全部ETH + Stage1代币奖励

---

## 常见问题

### Q1: 交易失败怎么办？

**常见原因：**
- Gas不足 → 增加Gas限制
- 余额不足 → 检查钱包余额
- 参数错误 → 确认函数参数

**解决方案：**
```javascript
// 增加Gas限制
await contract.functionName(args, {
  gasLimit: 500000
});
```

### Q2: 如何查看游戏状态？

```javascript
// 查看当前游戏状态
await stage2Contract.diceGameState();  // 0=None, 1=WaitingForB, 2=BothBetted, 3=Settled

// 查看玩家信息
await stage2Contract.gamblerA();
await stage2Contract.gamblerB();
await stage2Contract.betAmount();
```

### Q3: 游戏超时了怎么办？

游戏创建后30分钟为超时时限。如果：
- 一方已揭示，另一方超时 → 已揭示方获胜
- 双方都未超时 → 等待揭示

可以由任意用户调用 `checkTimeout()` 触发超时处理。

### Q4: 如何取消游戏？

只有游戏创建者（玩家A）可以在以下条件下取消：
- 游戏状态为 WaitingForB
- 在创建后30分钟内

调用：
```javascript
await stage2Contract.cancelGame();
```

### Q5: 合约中没有足够的ETH支付奖金？

确保相关合约有足够的 ETH：
1. 游戏参与者的ETH会注入合约
2. 如果调用 `Stage1.sell()`，需确保 Stage1 合约先被充值 ETH

---

## 安全提示

1. **保护私钥**: 不要将私钥提交到代码仓库
2. **测试优先**: 先在测试网验证功能
3. **小额测试**: 首次使用少量ETH测试
4. **检查权限**: 确保只有合约所有者可以调用管理函数

---

*文档更新时间: 2026-04-24*
