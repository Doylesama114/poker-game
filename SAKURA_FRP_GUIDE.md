# 🌸 Sakura FRP 配置指南

## 📋 概述

使用 Sakura FRP（樱花内网穿透）可以让互联网上的朋友访问你本地运行的游戏服务器。

## 🎯 需要穿透的服务

| 服务 | 本地端口 | 用途 | 隧道类型 |
|------|---------|------|---------|
| 后端服务器 | 3001 | WebSocket通信 | TCP 或 HTTP |
| 前端服务器 | 5173 | 网页访问 | HTTP |

## 🔧 详细配置步骤

### 第一步：登录 Sakura FRP

1. 访问 [Sakura FRP 官网](https://www.natfrp.com/)
2. 登录你的账号
3. 进入管理面板

### 第二步：创建隧道

#### 隧道1：后端服务器（WebSocket）

**推荐配置**：
```
隧道名称：card-game-backend
本地IP：127.0.0.1
本地端口：3001
隧道类型：TCP（推荐）或 HTTP
节点：选择延迟低的节点
远程端口：自动分配或自定义
```

**为什么选TCP？**
- WebSocket需要持久连接
- TCP隧道更稳定
- 支持双向通信

**配置示例**：
```
本地地址：127.0.0.1:3001
隧道类型：TCP
节点：上海节点
远程端口：12345（示例，实际由系统分配）
```

完成后你会得到：
```
访问地址：cn-sh-bgp-1.natfrp.cloud:12345
或：frp.example.com:12345
```

#### 隧道2：前端服务器（网页）

**推荐配置**：
```
隧道名称：card-game-frontend
本地IP：127.0.0.1
本地端口：5173
隧道类型：HTTP
节点：选择延迟低的节点（最好和后端同一节点）
域名绑定：自动分配或自定义
```

**配置示例**：
```
本地地址：127.0.0.1:5173
隧道类型：HTTP
节点：上海节点
域名：自动分配
```

完成后你会得到：
```
访问地址：http://abc123.natfrp.cloud
或：http://your-custom-domain.natfrp.cloud
```

### 第三步：启动 Sakura FRP 客户端

#### 方式1：使用启动器（推荐）

1. 下载 [Sakura FRP 启动器](https://www.natfrp.com/tunnel/download)
2. 安装并登录
3. 在隧道列表中找到你创建的两个隧道
4. 点击"启动"按钮启动两个隧道
5. 查看状态，确保显示"运行中"

#### 方式2：使用命令行

```bash
# 下载客户端
# 访问 https://www.natfrp.com/tunnel/download

# 启动后端隧道
frpc -f <后端隧道token>

# 启动前端隧道（新终端）
frpc -f <前端隧道token>
```

### 第四步：记录分配的地址

启动成功后，记录下分配的地址：

**示例**：
```
前端地址：http://abc123.natfrp.cloud
后端地址：http://cn-sh-bgp-1.natfrp.cloud:12345
或
后端地址：ws://cn-sh-bgp-1.natfrp.cloud:12345
```

### 第五步：更新游戏配置

打开 `src/config/multiplayer.ts`，更新FRP配置：

```typescript
export const SERVER_CONFIG = {
  // ... 其他配置 ...
  
  // Sakura FRP - 替换为你的实际地址
  frp: 'http://cn-sh-bgp-1.natfrp.cloud:12345',
  
  // 如果使用WebSocket协议
  // frp: 'ws://cn-sh-bgp-1.natfrp.cloud:12345',
}
```

**重要提示**：
- 如果后端使用TCP隧道，地址格式为：`http://节点域名:端口`
- 如果后端使用HTTP隧道，地址格式为：`http://分配的域名`
- 端口号是Sakura FRP分配的远程端口，不是3001

### 第六步：测试连接

#### 本地测试

1. 确保本地服务运行：
   ```bash
   # 后端（已在运行）
   cd server && node index.js
   
   # 前端
   pnpm run dev
   ```

2. 确保FRP隧道运行中

3. 访问FRP分配的前端地址：
   ```
   http://abc123.natfrp.cloud
   ```

4. 在联机大厅：
   - 点击服务器地址
   - 选择"🌸 Sakura FRP"模式
   - 检查连接状态（应该显示"已连接"）

#### 远程测试

1. 让朋友访问你的前端地址：
   ```
   http://abc123.natfrp.cloud
   ```

2. 你创建房间，朋友加入

3. 开始游戏！

## 🎮 完整使用流程

### 你（主机）的操作

1. **启动本地服务**：
   ```bash
   # 终端1：后端
   cd server
   node index.js
   
   # 终端2：前端
   pnpm run dev
   ```

2. **启动FRP隧道**：
   - 打开Sakura FRP启动器
   - 启动两个隧道（前端+后端）
   - 确认状态为"运行中"

3. **访问游戏**：
   - 访问FRP分配的地址：`http://abc123.natfrp.cloud`
   - 或访问本地：`http://localhost:5173`

4. **创建房间**：
   - 点击"联机对战"
   - 选择"Sakura FRP"模式
   - 创建房间
   - 记住房间ID

5. **分享地址**：
   - 告诉朋友访问：`http://abc123.natfrp.cloud`
   - 告诉朋友房间ID

### 朋友（客户端）的操作

1. **访问游戏**：
   - 打开浏览器
   - 访问：`http://abc123.natfrp.cloud`

2. **加入房间**：
   - 点击"联机对战"
   - 应该自动连接（显示"已连接"）
   - 点击"加入房间"
   - 输入房间ID

3. **开始游戏**！

## 🔍 常见问题

### Q1: 显示"未连接"？

**可能原因**：
1. FRP隧道未启动
2. 后端服务未运行
3. 配置的地址不正确

**解决方法**：
```bash
# 1. 检查后端是否运行
# 应该看到"服务器运行在端口 3001"

# 2. 检查FRP隧道状态
# 在Sakura FRP启动器中查看状态

# 3. 检查配置
# 打开 src/config/multiplayer.ts
# 确认 frp 地址正确
```

### Q2: 连接很慢或不稳定？

**可能原因**：
1. 节点选择不当
2. 网络延迟高
3. 免费节点限制

**解决方法**：
- 选择延迟低的节点
- 前后端使用同一节点
- 考虑升级到付费节点

### Q3: WebSocket连接失败？

**可能原因**：
- 使用了HTTP隧道而不是TCP

**解决方法**：
- 后端隧道改用TCP类型
- 或确保HTTP隧道支持WebSocket升级

### Q4: 朋友无法访问？

**检查清单**：
- [ ] FRP隧道正在运行
- [ ] 本地服务正在运行
- [ ] 地址正确（包括端口）
- [ ] 朋友的网络可以访问FRP节点

### Q5: 如何查看FRP日志？

**启动器**：
- 右键隧道 → 查看日志

**命令行**：
```bash
# 日志会直接显示在终端
```

### Q6: 端口号是多少？

**重要**：
- 本地端口：3001（后端）、5173（前端）
- 远程端口：由Sakura FRP分配（例如12345）
- 配置时使用**远程端口**，不是本地端口

## 📊 配置对照表

| 项目 | 本地开发 | Sakura FRP |
|------|---------|-----------|
| 前端访问 | http://localhost:5173 | http://abc123.natfrp.cloud |
| 后端地址 | http://localhost:3001 | http://节点域名:远程端口 |
| 连接方式 | 直连 | 通过FRP中转 |
| 访问范围 | 仅本机 | 全球互联网 |

## 🎯 最佳实践

### 1. 节点选择
- 选择离你近的节点（延迟低）
- 前后端使用同一节点
- 避免使用过载的节点

### 2. 隧道配置
- 后端使用TCP隧道（更稳定）
- 前端使用HTTP隧道（支持域名）
- 设置合理的带宽限制

### 3. 安全建议
- 不要分享你的FRP Token
- 定期更换隧道配置
- 使用复杂的房间ID

### 4. 性能优化
- 保持FRP客户端运行
- 使用有线网络
- 关闭不必要的后台程序

## 🔄 快速配置模板

### 配置文件更新

**src/config/multiplayer.ts**：
```typescript
export const SERVER_CONFIG = {
  local: 'http://localhost:3001',
  lan: 'http://192.168.1.7:3001',
  
  // 替换为你的实际FRP地址
  frp: 'http://cn-sh-bgp-1.natfrp.cloud:12345',
  
  internet: 'https://your-server.com:3001'
}
```

### 启动脚本

创建 `start-frp.bat`：
```batch
@echo off
echo 启动游戏服务器（FRP模式）
echo ================================

echo [1/3] 启动后端服务器...
start "后端服务器" cmd /k "cd server && node index.js"
timeout /t 2 /nobreak

echo [2/3] 启动前端服务器...
start "前端服务器" cmd /k "pnpm run dev"
timeout /t 2 /nobreak

echo [3/3] 请手动启动Sakura FRP隧道
echo ================================
echo 完成！
echo.
echo 前端地址：http://abc123.natfrp.cloud
echo 后端地址：http://节点域名:端口
echo.
pause
```

## 📝 配置检查清单

### 启动前
- [ ] 安装了Sakura FRP客户端
- [ ] 创建了两个隧道（前端+后端）
- [ ] 记录了分配的地址
- [ ] 更新了配置文件

### 启动时
- [ ] 后端服务运行中（端口3001）
- [ ] 前端服务运行中（端口5173）
- [ ] FRP前端隧道运行中
- [ ] FRP后端隧道运行中

### 测试时
- [ ] 本地可以访问FRP地址
- [ ] 联机大厅显示"已连接"
- [ ] 可以创建和加入房间
- [ ] 朋友可以访问FRP地址

## 🎉 总结

使用Sakura FRP的优势：
- ✅ 无需公网IP
- ✅ 无需购买服务器
- ✅ 配置简单快速
- ✅ 支持全球访问

现在你可以邀请任何地方的朋友一起玩了！

---

## 📞 需要帮助？

如果遇到问题：
1. 检查Sakura FRP官方文档
2. 查看FRP客户端日志
3. 确认本地服务正常运行
4. 测试网络连接

祝你游戏愉快！🎮✨
