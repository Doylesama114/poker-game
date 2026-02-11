# 🚀 GitHub Pages 部署指南

## 📋 前置准备

1. GitHub 账号
2. Git 已安装
3. 项目代码已准备好

---

## 🎯 部署步骤

### 步骤1：创建 GitHub 仓库

1. 访问 [GitHub](https://github.com/)
2. 点击右上角的 "+" → "New repository"
3. 填写信息：
   - Repository name: `poker-game`（或其他名字）
   - Description: 卡牌对战游戏
   - Public（公开）
   - 不要勾选 "Add a README file"
4. 点击 "Create repository"

---

### 步骤2：推送代码到 GitHub

在项目根目录打开命令行，运行：

```bash
# 初始化 Git（如果还没初始化）
git init

# 添加所有文件
git add .

# 提交
git commit -m "Initial commit"

# 添加远程仓库（替换为你的仓库地址）
git remote add origin https://github.com/你的用户名/poker-game.git

# 推送到 GitHub
git branch -M main
git push -u origin main
```

---

### 步骤3：启用 GitHub Pages

1. 在 GitHub 仓库页面，点击 "Settings"
2. 在左侧菜单找到 "Pages"
3. 在 "Build and deployment" 部分：
   - Source: 选择 "GitHub Actions"
4. 保存

---

### 步骤4：等待自动部署

1. 点击仓库顶部的 "Actions" 标签
2. 你会看到 "Deploy to GitHub Pages" 工作流正在运行
3. 等待几分钟，直到显示绿色的 ✓
4. 部署完成后，访问：`https://你的用户名.github.io/poker-game/`

---

## 🔧 配置后端地址

部署完成后，需要配置后端服务器地址。

### 方案A：使用 ngrok（临时测试）

1. 在本地启动后端 ngrok：
   ```bash
   ngrok http 3001
   ```

2. 记录 ngrok 地址（例如：`https://abc123.ngrok-free.app`）

3. 访问你的 GitHub Pages 网站

4. 进入"联机对战"

5. 点击"配置服务器"

6. 选择"自定义"模式，输入 ngrok 地址

### 方案B：部署后端到 Render（永久）

参考 `RENDER_DEPLOY_GUIDE.md`（即将创建）

---

## 🔄 更新部署

每次修改代码后：

```bash
# 添加修改
git add .

# 提交
git commit -m "描述你的修改"

# 推送
git push
```

推送后，GitHub Actions 会自动重新部署。

---

## 📊 查看部署状态

1. 访问仓库的 "Actions" 标签
2. 查看最新的工作流运行状态
3. 点击可以查看详细日志

---

## ⚠️ 常见问题

### 问题1：部署失败

**检查**：
- Actions 标签中的错误日志
- 是否正确配置了 Pages 设置

**解决**：
- 查看错误信息
- 确保 `pnpm install` 和 `pnpm run build` 在本地能正常运行

### 问题2：访问 404

**原因**：
- base 路径配置错误
- 部署未完成

**解决**：
- 确认 Actions 显示绿色 ✓
- 等待几分钟让 DNS 生效

### 问题3：无法连接后端

**原因**：
- 后端地址未配置
- 后端服务未运行

**解决**：
- 在游戏中配置正确的后端地址
- 确保后端服务（ngrok 或 Render）正在运行

---

## 🎉 完成！

现在你的游戏已经部署到互联网上了！

- **前端地址**：`https://你的用户名.github.io/poker-game/`
- **分享给朋友**：直接发送这个链接

---

## 📝 下一步

1. 部署后端到 Render（永久在线）
2. 配置自定义域名（可选）
3. 添加 HTTPS 支持（GitHub Pages 自动提供）

需要帮助？查看 GitHub Actions 的日志或联系我！
