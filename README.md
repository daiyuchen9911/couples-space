# 我们的空间 · 情侣 PWA 使用指南

一个纯 HTML/JS 的情侣空间 PWA：备忘录、纪念日（自动倒计时）、照片墙，双人实时同步。

## 功能

- **首页**：在一起天数 + 纪念日列表（自动显示"还有 N 天"）
- **备忘录**：随手记事
- **照片墙**：照片自动压缩到 1080px 后同步，两人都能看到
- 离线可用（Service Worker），有网时自动同步
- 设置页支持一键导出 JSON 备份

## 文件结构

```
couples-space/
├── index.html          # 应用本体（全部逻辑都在这一个文件里）
├── manifest.json       # PWA 清单
├── sw.js               # Service Worker（离线缓存）
└── icons/              # 图标（见下方第 1 步）
```

## 部署步骤

### 1. 生成图标（一次性）

在电脑浏览器打开 `icons/make-icons.html`，下载两个 PNG，保存/移动到 `icons/` 文件夹内，文件名保持 `icon-192.png` 和 `icon-512.png`。之后 `make-icons.html` 可以删掉。

### 2. 注册 Supabase（免费，用于双人同步）

1. 打开 https://supabase.com 注册（免费版完全够个人用）
2. 新建一个项目（New project），区域选东京或新加坡
3. 项目建好后，在 **Settings → API** 页面找到 `Project URL` 和 `anon public` key
4. 在左侧 **SQL Editor** 里运行下面这段建表语句：

```sql
create table entries (
  id text primary key,
  type text not null,
  title text,
  content text,
  date text,
  note text,
  photo text,
  created_at timestamptz default now()
);

alter table entries enable row level security;

-- 个人使用：允许匿名读写（URL 和 key 不外泄即可）
create policy "public access" on entries
  for all using (true) with check (true);
```

### 3. 部署到网上（必须 HTTPS，任选其一）

**方式 A：GitHub Pages（推荐）**

1. 在 GitHub 建一个仓库，把 `couples-space` 里所有文件传上去
2. 仓库 Settings → Pages → Source 选 main 分支
3. 等一两分钟，得到地址 `https://你的用户名.github.io/仓库名/`

**方式 B：Cloudflare Pages / Vercel**，把文件夹拖进去即可。

### 4. iPhone 上安装

1. 用 **Safari**（必须是 Safari，微信里打开不行）访问部署好的地址
2. 点底部分享按钮 → 添加到主屏幕
3. 桌面上会出现粉色爱心图标，点开是全屏无浏览器栏的 app

### 5. 配置同步

1. 打开 app → 设置 → 云同步设置
2. 粘贴 Supabase 的 URL 和 anon key，保存
3. 让伴侣也按第 4 步装好 app，填**同样的** URL 和 key
4. 之后任何一方的记录都会自动同步给对方

## 注意事项

- 照片以压缩后的 base64 存在数据库里，单张约 100–300KB。免费 Supabase 项目有 500MB 数据库空间，按每张 200KB 算能存约 2000 张，个人使用足够
- anon key 是公开密钥，泄露的后果是别人也能读写这个表——不要把仓库地址发到公开场合
- 建议每月在设置页导出一次 JSON 备份
- iOS 卸载 PWA 会清掉本地缓存，但数据都在云端，重新装并登录同一配置即可恢复

## 常见问题

**改动代码后手机上没更新？** Service Worker 有缓存延迟。可以在 Safari 里长按刷新，或等一天缓存自动更新（sw.js 里的版本号 `v1` 改成 `v2` 可强制更新）。

**想改主题色/名字？** 编辑 `manifest.json` 和 `index.html` 顶部的 CSS 变量 `--pink`。
