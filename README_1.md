# PXM Help Center - 协作编辑平台

🚀 一个完整的团队知识库管理系统，支持实时协作、权限控制和版本管理。

## 功能特性

✨ **核心功能**
- 📝 实时协作编辑（所有编辑者同时看到更新）
- 👥 权限管理（编辑者/查看者分离）
- 📊 表格和富文本编辑
- 🖼️ 图片和媒体支持
- 📋 版本历史跟踪
- 🔍 全文搜索
- 🎯 分类和标签系统

🔐 **认证和安全**
- Google OAuth 登录
- 邮箱 OTP 登录
- 行级权限控制 (RLS)

☁️ **云端部署**
- 实时数据库（Supabase）
- 免费托管（Vercel）
- 无服务器架构

## 快速开始（10分钟）

### 前置需求
- GitHub 账号（用于Vercel）
- Google 账号（可选，用于OAuth）
- Node.js 16+（本地开发）

### 第一步：创建 Supabase 项目

1. 访问 [supabase.com](https://supabase.com)
2. 注册账号并创建新项目：
   - **Project Name**: `pxm-help-center`
   - **Region**: Asia Singapore（或最近的地区）
   - 设置强密码
3. 项目创建完成后，打开 **SQL Editor**

### 第二步：初始化数据库

复制以下 SQL 代码到 Supabase SQL Editor 并执行：

```sql
-- 创建文章表
CREATE TABLE articles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT DEFAULT '',
  category TEXT DEFAULT '未分类',
  tags TEXT[],
  status TEXT DEFAULT 'draft',
  created_by TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  published_at TIMESTAMP,
  CONSTRAINT articles_created_by_check CHECK (created_by != '')
);

-- 创建编辑者表
CREATE TABLE editors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  name TEXT,
  role TEXT DEFAULT 'editor',
  created_at TIMESTAMP DEFAULT NOW()
);

-- 创建版本历史表
CREATE TABLE article_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  article_id UUID NOT NULL REFERENCES articles(id) ON DELETE CASCADE,
  version_num INT NOT NULL,
  title TEXT NOT NULL,
  content TEXT,
  changed_by TEXT NOT NULL,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 启用行级安全
ALTER TABLE articles ENABLE ROW LEVEL SECURITY;
ALTER TABLE editors ENABLE ROW LEVEL SECURITY;
ALTER TABLE article_history ENABLE ROW LEVEL SECURITY;

-- 创建 RLS 策略
-- 任何人可读已发布的文章
CREATE POLICY "Anyone can read published articles" ON articles
  FOR SELECT USING (status = 'published');

-- 编辑者可读所有文章
CREATE POLICY "Editors can read all articles" ON articles
  FOR SELECT USING (
    EXISTS (
      SELECT 1 FROM editors 
      WHERE email = auth.jwt() ->> 'email'
    )
  );

-- 编辑者可创建文章
CREATE POLICY "Editors can insert articles" ON articles
  FOR INSERT WITH CHECK (
    created_by = auth.jwt() ->> 'email' AND
    EXISTS (
      SELECT 1 FROM editors 
      WHERE email = auth.jwt() ->> 'email'
    )
  );

-- 编辑者可修改自己的或任何文章（管理员）
CREATE POLICY "Editors can update articles" ON articles
  FOR UPDATE USING (
    created_by = auth.jwt() ->> 'email' OR
    EXISTS (
      SELECT 1 FROM editors 
      WHERE email = auth.jwt() ->> 'email' AND role = 'admin'
    )
  );

-- 编辑者可删除自己的文章
CREATE POLICY "Editors can delete own articles" ON articles
  FOR DELETE USING (
    created_by = auth.jwt() ->> 'email' OR
    EXISTS (
      SELECT 1 FROM editors 
      WHERE email = auth.jwt() ->> 'email' AND role = 'admin'
    )
  );

-- 只读历史记录
CREATE POLICY "Anyone can read history" ON article_history
  FOR SELECT USING (true);
```

### 第三步：获取 API 密钥

1. 在 Supabase Dashboard 中点击 **Settings** > **API**
2. 复制以下信息：
   - **Project URL** (例如: `https://xxx.supabase.co`)
   - **anon public** key (以 `eyJ` 开头的长字符串)

### 第四步：添加编辑者

1. 在 Supabase 中打开 **editors** 表
2. 点击 **Insert** 按钮
3. 添加你的邮箱地址和你想邀请的编辑者：

```json
{
  "email": "your-email@example.com",
  "name": "Your Name",
  "role": "admin"
}
```

### 第五步：配置 Google OAuth（可选）

1. 访问 [Google Cloud Console](https://console.cloud.google.com)
2. 创建新项目
3. 启用 Google+ API
4. 创建 OAuth 2.0 Client ID：
   - 类型：Web application
   - 授权 JavaScript 来源：`https://xxx.supabase.co`
   - 授权重定向 URI：`https://xxx.supabase.co/auth/v1/callback`
5. 获得 Client ID 和 Secret
6. 在 Supabase Dashboard 中：
   - **Authentication** > **Providers**
   - 启用 Google 并填入 Client ID 和 Secret

### 第六步：部署到 Vercel

**选项 A：通过 GitHub（推荐）**

1. 上传代码到 GitHub
2. 访问 [vercel.com](https://vercel.com)
3. 用 GitHub 账号登录
4. 点击 **Add New** > **Project**
5. 导入你的 GitHub 仓库
6. 添加环境变量：
   ```
   VITE_SUPABASE_URL=你的_Project_URL
   VITE_SUPABASE_KEY=你的_anon_public_key
   ```
7. 点击 **Deploy** 完成

**选项 B：使用 Vercel CLI（快速）**

```bash
# 安装 Vercel CLI
npm i -g vercel

# 进入项目目录
cd pxm-help-center

# 部署
vercel --prod

# 配置环境变量后重新部署
vercel env pull
vercel --prod
```

### 完成！🎉

你的应用现在应该可以访问了！你会获得一个类似 `https://pxm-help-center-xxx.vercel.app` 的链接。

## 本地开发

```bash
# 安装依赖
npm install

# 启动开发服务器
npm run dev

# 构建生产版本
npm run build

# 预览构建结果
npm run preview
```

## 使用指南

### 对于编辑者

1. **登录** - 使用 Google 或邮箱登录
2. **创建文章** - 点击 "+ 新建" 按钮
3. **编辑内容** - 在编辑器中撰写文章
4. **实时保存** - 内容会自动保存到云端
5. **发布** - 点击 "发布" 按钮让内容对所有人可见

### 对于查看者

1. **浏览** - 访问网站即可浏览所有已发布的文章
2. **搜索** - 使用搜索框找到需要的内容
3. **分享** - 复制文章链接分享给他人

## 管理员任务

### 添加新编辑者

在 Supabase 的 **editors** 表中添加新行，设置邮箱和权限。

### 管理权限

- **editor**: 可以创建和编辑自己的文章
- **admin**: 可以编辑所有文章，管理其他编辑者

### 备份数据

在 Supabase Dashboard 中：
1. 点击 **Settings** > **Backups**
2. 下载备份文件

## 常见问题

**Q: 支持多少个编辑者？**  
A: Supabase 免费套餐支持无限用户。实时连接数限制为 200。

**Q: 可以自定义域名吗？**  
A: 可以！在 Vercel 中添加自定义域名，修改 DNS 指向即可。

**Q: 如何删除文章？**  
A: 编辑者可以在文章编辑页面点击 "删除" 按钮。

**Q: 支持离线编辑吗？**  
A: 当前版本不支持。内容会自动同步到云端。

**Q: 如何导出文章？**  
A: 可以在预览页面复制内容到文本编辑器，或者直接从数据库导出。

## 成本估算

| 服务 | 免费额度 | 价格 |
|------|--------|------|
| Supabase 数据库 | 500MB 存储 | $25/月（超出部分） |
| Vercel 托管 | 无限次部署 | 免费 |
| 总计 | - | 建议：完全免费试用 |

## 支持和反馈

遇到问题？

- Supabase 文档：https://supabase.com/docs
- Vercel 文档：https://vercel.com/docs
- GitHub Issues：提出 bug 报告

## 许可证

MIT

---

**开心编辑！🚀**
