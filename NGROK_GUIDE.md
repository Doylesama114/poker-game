# 🚀 ngrok 互联网联机配置指南

## 为什么使用 ngrok？

Sakura FRP 要求备案域名或强制 HTTPS，对于个人项目来说配置复杂。ngrok 是一个国际服务，无需域名，免费版本就能满足需求。

---

## 📝 配置步骤（5分钟）

### 步骤1：获取 authtoken

1. 访问 [ngrok 官网](https://dashboard.ngrok.com/)
2. 注册/登录账号
3. 在 Dashboard 页面找到 "Your Authtoken"
4. 复制 authtoken（格式类似：`2abc...xyz`）

### 步骤2：配置 authtoken

打开命令行，运行：

```bash
ngrok config add-authtoken 你的authtoken
```

例如：
```bash
ngrok config add-authtoken 2abc123def456ghi789jkl
```

看到 "Authtoken saved" 表示成功。

---

### 步骤3：启动游戏服务

**终端1 - 启动后端**：
```bash
cd server
node index.js
```

看到 "服务器运行在端口 3001" 表示成功。

**终端2 - 启动前端**：
```bash
pnpm run dev
```

看到 "Local: http://localhost:5173/" 表示成功。

---

### 步骤4：启动 ngrok 隧道

**终端3 - 后端隧道**：
```bash
ngrok http 3001
```

**终端4 - 前端隧道**：
```bash
ngrok http 5173
```

---

### 步骤5：记录地址

每个 ngrok 终端会显示类似这样的信息：

```
Forwarding  https://abc123.ngrok-free.app -> http://localhost:3001
```

记录两个地址：
- **后端地址**：`https://abc123.ngrok-free.app`（终端3）
- **前端地址**：`https://def456.ngrok-free.app`（终端4）

⚠️ **注意**：
- 地址每次启动都会变化（免费版）
- 使用 HTTPS 地址，不要用 HTTP

---

### 步骤6：更新配置

打开 `src/config/multiplayer.ts`，找到这一行：

```typescript
frp: 'http://localhost:3001',
```

替换为你的后端 ngrok 地址：

```typescript
frp: 'https://abc123.ngrok-free.app',
```

保存文件，前端会自动重新加载。

---

### 步骤7：测试连接

1. 访问前端 ngrok 地址：`https://def456.ngrok-free.app`
2. 第一次访问会看到 ngrok 警告页面，点击 "Visit Site"
3. 进入游戏，点击"联机对战"
4. 点击服务器地址旁的"配置"按钮
5. 选择"Sakura FRP"模式（会使用配置的 ngrok 地址）
6. 检查连接状态（应显示"✅ 已连接"）

---

### 步骤8：邀请朋友

1. 创建房间，记住房间ID
2. 告诉朋友访问：`https://def456.ngrok-free.app`
3. 朋友第一次访问也会看到警告页面，点击 "Visit Site"
4. 朋友加入房间（输入房间ID）
5. 开始游戏！🎮

---

## 🔧 故障排除

### 问题：显示"未连接"

**检查清单**：
- [ ] 后端服务正在运行（终端1应显示"服务器运行在端口 3001"）
- [ ] 前端服务正在运行（终端2应显示"Local: http://localhost:5173/"）
- [ ] 后端 ngrok 隧道正在运行（终端3应显示 Forwarding 信息）
- [ ] 前端 ngrok 隧道正在运行（终端4应显示 Forwarding 信息）
- [ ] 配置文件中的地址正确（使用 HTTPS，不是 HTTP）

**解决方法**：
1. 检查所有4个终端是否都在运行
2. 确认 ngrok 地址是 HTTPS 开头
3. 刷新浏览器（Ctrl+Shift+R）

---

### 问题：ngrok 地址每次都变

**原因**：免费版 ngrok 每次启动都会生成新地址

**解决方法**：
1. 每次启动 ngrok 后，更新配置文件中的地址
2. 或者升级到 ngrok 付费版（可以使用固定域名）

---

### 问题：朋友无法访问

**可能原因**：
1. ngrok 隧道未启动
2. 地址输入错误
3. 朋友的网络问题

**解决方法**：
1. 确认 ngrok 隧道运行中
2. 让朋友尝试访问：`https://你的前端地址`
3. 检查浏览器控制台是否有错误

---

### 问题：连接很慢

**原因**：ngrok 免费版有速度限制

**优化建议**：
1. 使用局域网模式（如果在同一网络）
2. 升级到 ngrok 付费版
3. 考虑部署到云服务器

---

## 📊 配置对照表

| 项目 | 本地开发 | ngrok |
|------|---------|-------|
| 前端访问 | http://localhost:5173 | https://def456.ngrok-free.app |
| 后端地址 | http://localhost:3001 | https://abc123.ngrok-free.app |
| 访问范围 | 仅本机 | 全球互联网 |
| 配置难度 | 简单 | 简单 |
| 需要域名 | 否 | 否 |
| 需要备案 | 否 | 否 |

---

## 💡 提示

1. **保持终端运行**：游戏期间，所有4个终端必须保持运行
2. **记录地址**：把 ngrok 地址保存下来，方便分享
3. **免费限制**：ngrok 免费版有连接数限制（40次/分钟），足够小规模使用
4. **安全性**：不要分享你的 authtoken

---

## 🎉 完成！

现在你可以邀请任何地方的朋友一起玩了！

---

## 📌 快速命令参考

```bash
# 1. 启动后端
cd server && node index.js

# 2. 启动前端（新终端）
pnpm run dev

# 3. 启动后端隧道（新终端）
ngrok http 3001

# 4. 启动前端隧道（新终端）
ngrok http 5173
```

---

**需要帮助？** 查看 ngrok 官方文档：https://ngrok.com/docs
