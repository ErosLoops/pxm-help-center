# PXM Help Center - 部署指南

## 快速开始（5分钟）

### 第一步：创建 Supabase 项目

1. 访问 [supabase.com](https://supabase.com)
2. 点击 **Start your project** 注册账号
3. 创建新项目：
   - Project Name: `pxm-help-center`
   - Database Password: 设置强密码
   - Region: 选择最近的地区（新加坡/东京）
4. 等待项目初始化完成

### 第二步：创建数据库表

1. 进入 Supabase Dashboard
2. 点击 **SQL Editor**
3. 新建 Query，运行以下 SQL：

```sql
-- 文章表
CREATE TABLE articles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  content TEXT,
  category TEXT,
  tags TEXT[],
  status TEXT DEFAULT 'draft',
  created_by TEXT,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  published_at TIMESTAMP,
  CONSTRAINT articles_pkey PRIMARY KEY (id)
);

-- 编辑权限表
CREATE TABLE editors (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT NOT NULL UNIQUE,
  name TEXT,
  role TEXT DEFAULT 'editor',
  created_at TIMESTAMP DEFAULT NOW()
);

-- 版本历史表
CREATE TABLE article_history (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  article_id UUID REFERENCES articles(id) ON DELETE CASCADE,
  version_num INT,
  title TEXT,
  content TEXT,
  changed_by TEXT,
  created_at TIMESTAMP DEFAULT NOW()
);

-- 启用 RLS (行级安全)
ALTER TABLE articles ENABLE ROW LEVEL SECURITY;
ALTER TABLE editors ENABLE ROW LEVEL SECURITY;

-- 创建 RLS 策略 (任何人可读，只有编辑者可写)
CREATE POLICY "Anyone can read articles" ON articles
  FOR SELECT USING (status = 'published' OR auth.jwt() ->> 'email' = created_by);

CREATE POLICY "Editors can insert articles" ON articles
  FOR INSERT WITH CHECK (
    EXISTS (
      SELECT 1 FROM editors WHERE email = auth.jwt() ->> 'email'
    )
  );

CREATE POLICY "Editors can update own articles" ON articles
  FOR UPDATE USING (
    created_by = auth.jwt() ->> 'email' OR
    EXISTS (SELECT 1 FROM editors WHERE email = auth.jwt() ->> 'email' AND role = 'admin')
  );
```

### 第三步：获取 API 密钥

1. 在 Supabase Dashboard 点击 **Settings** > **API**
2. 复制：
   - **Project URL** → 保存为 `VITE_SUPABASE_URL`
   - **anon public key** → 保存为 `VITE_SUPABASE_KEY`

### 第四步：部署到 Vercel

1. 访问 [vercel.com](https://vercel.com)
2. 用 GitHub 账号登录
3. 创建新项目 → 导入下面提供的项目代码
4. 在环境变量中添加：
   ```
   VITE_SUPABASE_URL=你的_Project_URL
   VITE_SUPABASE_KEY=你的_anon_public_key
   ```
5. 点击 **Deploy** 完成部署

### 第五步：邀请编辑者

1. 回到 Supabase Dashboard
2. 打开 **editors** 表
3. 点击 **Insert** 添加编辑者：
   ```json
   {
     "email": "editor@example.com",
     "name": "Editor Name",
     "role": "editor"
   }
   ```

## 功能清单

✅ 实时协作编辑  
✅ 权限控制（编辑者/查看者分离）  
✅ 完整的表格和图片支持  
✅ 版本历史跟踪  
✅ 全文搜索  
✅ 一键分享链接  

## 常见问题

**Q: 需要付费吗？**  
A: 不需要！Supabase 和 Vercel 都有免费套餐，足以支持中小型团队。

**Q: 如何添加更多编辑者？**  
A: 在 Supabase 的 editors 表中添加邮箱即可。

**Q: 支持自定义域名吗？**  
A: 支持！在 Vercel 中可以绑定自己的域名（需要修改 DNS）。

**Q: 数据安全吗？**  
A: Supabase 提供企业级数据加密和备份，数据存储在你的账户中。

## 需要帮助？

- Supabase 文档: https://supabase.com/docs
- Vercel 文档: https://vercel.com/docs
