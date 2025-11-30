# Ranked Bedwars Bot

Ranked Bedwars 是一个基于 Discord 的竞技 Bedwars 游戏平台，允许玩家进行排名匹配、查看排行榜、注册账户等操作。

## 功能特性

### 核心功能
- **匹配系统**：可处理高达 10,000 名并发用户的强大匹配系统
- **排名系统**：基于 ELO 的排名机制，具有多个段位等级
- **统计数据追踪**：详细的游戏统计数据跟踪（击杀、死亡、胜负场次等）
- **自动语音频道创建**：游戏开始时自动创建临时语音频道
- **工单帮助系统**：带有归档日志的支持工单系统
- **分布式架构**：通过 Socket.IO 实现多机器人协同工作

### 段位系统
Ranked Bedwars 使用基于 ELO 的段位系统，包含以下等级：
- **Coal（煤炭）**：0-400 ELO
- **Iron（铁）**：400-800 ELO
- **Gold（金）**：800-1200 ELO
- **Platinum（铂金）**：1200-1600 ELO
- **Emerald（绿宝石）**：1600-1800 ELO
- **Diamond（钻石）**：1800-2000 ELO
- **Obsidian（黑曜石）**：2000+ ELO

## 指令列表

### 用户指令
- `/register [IGN]` - 将您的 Discord 账户与 Minecraft 账户链接并验证所有权
- `/info [player]` - 查看已注册玩家的统计数据
- `/leaderboard` - 查看 Ranked Bedwars 排行榜，包含多个分类：
  - `elo` - 查看 ELO 最高的玩家
  - `games` - 查看游戏场次最多的玩家
  - `wins` - 查看胜场最多的玩家
  - `losses` - 查看败场最多的玩家
  - `wl` - 查看胜率最高的玩家
  - `kills` - 查看击杀数最多的玩家
  - `kd` - 查看 KD 比率最高的玩家
  - `winstreak` - 查看连胜纪录最高的玩家
  - `bedsbroken` - 查看破坏床数最多的玩家
  - `bblr` - 查看破坏床/失败比率最高的玩家

## 技术架构

### 后端技术栈
- **Node.js**：JavaScript 运行时环境
- **TypeScript**：类型安全的 JavaScript 超集
- **Discord.js**：用于与 Discord API 交互的库
- **MongoDB**：用于数据持久化的 NoSQL 数据库
- **Socket.IO**：实现实时、双向通信的库
- **Canvas**：用于动态生成图像（如用户资料卡）

### 数据模型

#### 玩家（Player）
```typescript
interface Player {
  _id: ObjectId;              // 数据库中的玩家 ID
  discord: string;            // Discord 用户 ID
  minecraft: {                // 链接的 Minecraft 账户
    uuid: string;             // Minecraft UUID（无破折号）
    name: string;             // Minecraft 用户名
  }
  registeredAt: number;       // 注册时间戳
  wins?: number;              // 胜利场次
  losses?: number;            // 失败场次
  bedsBroken?: number;        // 破坏床数
  bedsLost?: number;          // 床被破坏数
  kills: number;              // 击杀数
  deaths: number;             // 死亡数
  elo?: number;               // ELO 分数
  banExpires?: number;        // 封禁到期时间
  strikes?: number;           // 警告次数
  games?: number;             // 总游戏场次
  winstreak?: number;         // 当前连胜
  bedstreak?: number;         // 当前破坏床连续数
}
```

#### 游戏（Game）
```typescript
interface Game {
  _id: ObjectId;              // 数据库中的游戏 ID
  textChannel?: string;       // 文字频道 ID
  voiceChannel?: string;      // 语音频道 ID
  team1?: Team;               // 队伍 1
  team2?: Team;               // 队伍 2
  state?: GameState;          // 游戏状态
}
```

### 游戏状态管理
```typescript
enum GameState {
  PRE_GAME,     // 游戏准备阶段
  STARTING,     // 游戏即将开始
  ACTIVE,       // 游戏进行中
  SCORING,      // 得分计算中
  FINISHED,     // 游戏已完成
  VOID          // 游戏已取消
}
```

## 系统设计

### 匹配系统
1. 玩家使用 `/register` 命令注册其 Minecraft 账户
2. 系统从 Hypixel API 获取玩家数据以验证 Discord 链接
3. 玩家加入队列等待匹配
4. 系统根据 ELO 分数智能匹配对手
5. 自动创建临时语音和文字频道供游戏使用

### ELO 计算算法
ELO 系统根据比赛结果动态调整玩家分数：
- 胜利获得 ELO 分数
- 失败扣除 ELO 分数
- 分数变化取决于对手的实力（强敌胜利获得更多分数）
- 每破坏一张床额外奖励 10 ELO 分数

### 分布式架构
系统采用分布式设计，允许多个机器人实例协同工作：
- 主服务器负责协调和数据库操作
- 游戏服务器负责实际的游戏管理
- 通过 Socket.IO 实现实时通信

## 安装与部署

### 环境要求
- Node.js v14 或更高版本
- MongoDB 数据库
- Discord 机器人令牌
- Hypixel API 密钥
- Socket.IO 通信密钥

### 安装步骤
1. 克隆仓库或下载项目文件
2. 在项目根目录执行 `npm install` 安装依赖
3. 配置环境变量（.env 文件）：
   ```
   TOKEN=your_discord_bot_token
   GUILD=your_guild_id
   DB_URL=mongodb_connection_string
   HYPIXEL_KEY=your_hypixel_api_key
   SOCKET_KEY=your_socket_io_key
   ```
4. 构建项目：`npm run build`
5. 启动应用：`npm start`

### 开发命令
- `npm run build` - 编译 TypeScript 代码
- `npm run watch` - 监听文件变化并自动编译
- `npm start` - 启动编译后的应用

## API 集成

### Hypixel API
- 用于验证玩家的 Discord 和 Minecraft 账户链接
- 获取玩家的 Bedwars 统计数据

### Discord API
- 处理用户指令和交互
- 管理服务器成员和频道
- 发送消息和嵌入内容

### Visage API
- 用于获取 Minecraft 玩家皮肤的前端视图
- 在用户资料卡上显示 3D 角色模型

## 安全措施

### 账户验证
- 强制要求玩家在 Minecraft 中链接正确的 Discord 账户
- 通过 Hypixel API 验证链接的有效性

### 反作弊系统
- 实时监控游戏中的异常行为
- 自动检测并报告可疑活动
- 支持手动封禁和警告系统

### 权限管理
- 基于角色的访问控制（RBAC）
- 不同指令仅对特定角色开放
- 管理员指令受到严格保护

## 贡献指南

我们欢迎社区贡献！如果您想为项目做出贡献，请遵循以下步骤：

1. Fork 项目仓库
2. 创建您的功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交您的更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 开启 Pull Request

请确保您的代码符合项目的编码标准，并添加适当的测试。

## 许可证

本项目采用 ISC 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 联系方式

如有任何问题或建议，请通过 Discord 联系我们的开发团队。
