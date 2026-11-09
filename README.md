# DY 德宇 | 外贸工厂展示官网

**Shandong Deyu Hair Products Factory · 山东德宇发制品厂**

纯展示型外贸 B2B 官网：深咖 + 金色、大留白、响应式、英文默认 + 一键切换中文。
**无商城、无购物车、无在线支付、无询价表单。**

技术形态：**静态网站 + JSON 内容层 + Decap CMS 后台**，无构建步骤。
后台里改文字 / 传图片 / 传视频 / 增删产品 → 提交到 Git → Netlify 自动发布 → 前台即时生效，全程不用碰代码。

---

## 一、目录结构

```
dy-deyu-site/
├── index.html              ★ 站点外壳（双击也能完整浏览）
├── 启动本地预览.bat          ★ Windows 双击启动本地服务器（同 start-preview.bat）
├── 404.html                深链接兜底（GitHub Pages 等主机用）
├── .nojekyll               GitHub Pages 必需：禁用 Jekyll 构建
├── _redirects              Netlify SPA 回退规则（支持 /products/wigs/2 这类干净网址）
├── netlify.toml            Netlify 发布目录 + 缓存头
├── vercel.json             Vercel 回退规则（可选）
├── robots.txt / sitemap.xml SEO 文件（上线后把域名换成自己的）
├── admin/
│   ├── index.html          Decap CMS 后台入口（/admin/）
│   └── config.yml          后台字段定义（5 个内容集合）
├── css/style.css           全部样式（深咖 + 金 + 响应式）
├── js/app.js               路由、双语切换、页面渲染
├── content/                ★ 后台可编辑的全部数据
│   ├── site.json           品牌 / 地址 / 邮箱 / WhatsApp / 社交 / 版权 / SEO
│   ├── ui.json             所有按钮、标签、分页、规格名等界面文案（EN/CN）
│   ├── pages.json          首页、产品中心、工厂、关于、联系的全部文案
│   ├── categories.json     6 个分类卡片
│   ├── products.json       360 款产品（6 分类 × 60 款，每页 12 款 → 5 页）
│   └── offline-data.js     ★ 双击预览用的数据快照（由 gen_offline_data.py 生成）
├── my-site-info.json        ★ 上线信息填写表（本地用，已加入 .gitignore）
├── images/                 占位图（SVG，后台可随时替换）
│   └── uploads/            后台上传的图片 / 视频会提交到这里
│       └── products/       批量替换产品照片时把图片放这里
└── tools/
    ├── setup.py            ★ 一键替换联系方式 / 社交 / 地图 / 域名（含 sitemap）
    ├── batch_images.py     ★ 按文件名批量把真实照片挂到产品上
    ├── gen_offline_data.py ★ 刷新「双击预览」的数据快照
    ├── serve.py            本地预览服务器（带 SPA 回退）
    ├── smoke-test.js       无浏览器冒烟测试：44 项（HTTP/双击/子目录/数据兜底）
    ├── gen_placeholders.py 生成 SVG 占位图
    └── gen_catalog.py      生成示例产品目录（⚠️ 见下方警告）
```

> ⚠️ **不要在上后台编辑过之后再运行 `tools/gen_catalog.py`**，它会用示例数据覆盖 `content/products.json`。`gen_placeholders.py` 只覆盖 `images/placeholder-*.svg`，不会影响后台上传到 `images/uploads/` 的文件。

---

## 二、怎么打开（三种方式）

> ⚠️ **先把 zip 完整解压到一个文件夹**，不要在压缩软件里直接双击压缩包内的 `index.html`。那样旁边的 `css/`、`js/`、`content/`、图片都不在，页面会一直停在 "Loading…"。

### 方式 1：双击 `index.html`（最省事，不用装任何东西）

双击解压后的 `index.html` 即可浏览：导航、6 个分类、分页、产品详情、图集放大、EN/CN 切换都能用。

原理：浏览器禁止 `file://` 页面用 `fetch()` 读本地 JSON，但允许 `<script src>` 加载本地 JS。所以双击时站点会读取 `content/offline-data.js` 这份**数据快照**，网址自动变成 `#/products/wigs` 形式的哈希路由，图片改用相对路径。

> 快照是生成那一刻的内容。在后台 **Publish** 或手动改过 `content/*.json` 后，运行一次 `python tools/gen_offline_data.py` 刷新即可。通过本地服务器或线上访问时永远读取实时 JSON，与此快照无关。

### 方式 2：双击 `启动本地预览.bat`（推荐日常使用，实时数据）

Windows 下双击根目录的 `启动本地预览.bat`（同内容的英文名为 `start-preview.bat`）：自动启动服务器并打开 `http://localhost:8000/`，服务器在最小化窗口运行，关掉该窗口即停止。需要 Python 3。

也可以手动启动（Python 3，无需 npm install，无需构建）：

```bash
cd dy-deyu-site
python tools/serve.py          # 默认 http://localhost:8000
python tools/serve.py 8080     # 自定义端口
```

### 方式 3：部署到 Netlify

见第三节。线上由 Netlify 提供 HTTP，自动使用实时 JSON 与干净的 `/products/wigs/2` 网址。

本地服务器带 SPA 回退，所以 `/products/wigs/2`、`/product/DY-WG-001` 这类地址直接打开也能正常显示。

站点是**位置无关**的：放在域名根（`https://xxx.netlify.app/`）或子目录（`https://user.github.io/repo/`）都能自动识别资源根路径，图片、JSON、链接全部使用相对路径 + 动态 `<base>`。若某个主机连 `content/*.json` 都没正确发布，页面会自动改用 `content/offline-data.js` 快照渲染，不会白屏。

验证渲染：

```bash
node tools/smoke-test.js             # 44 项：HTTP/双击 × 10 路由 × EN/CN + 子目录 + 数据兜底
node tools/smoke-test.js --offline   # 只测双击模式
```

---

## 三、部署到 Netlify（推荐，Decap CMS 的标准搭配）

> **两个压缩包，别拿错**
> | 压缩包 | 解压后 | 用途 |
> | --- | --- | --- |
> | `dy-deyu-site.zip` | 得到一个 `dy-deyu-site/` 文件夹 | 本机开发、GitHub 导入（在这个文件夹里 `git init`） |
> | `dy-deyu-upload.zip` | **直接就是 `index.html` + `css/` + `js/` + `content/` + `images/`**，没有外层文件夹 | Netlify Drop 等「拖拽文件夹上传」的平台，避免多套一层目录 |
>
> 判定标准只有一条：**服务器上 `index.html` 必须和 `css/`、`js/`、`content/`、`images/` 在同一层**。诊断面板里列出的 4 个文件只是「体检项目」，不需要单独上传，也不要只上传这 4 个 —— 要上传的是整个站点目录的全部文件。

### 1. 把源码放进 Git 仓库

> ⚠️ **更新版本时**：必须把新版解压后的 **57 个文件全部覆盖**到仓库根目录，再 `git add . && git commit && git push`。只替换 `index.html` 是不行的——旧版 `js/app.js` 不认识新的资源路径，会继续报错。
>
> 部署前建议先跑一次 `python tools/gen_offline_data.py`，让「双击预览」和「数据兜底」用的快照与当前 `content/*.json` 一致（后台每次 Publish 后也建议重跑）。

```bash
cd dy-deyu-site
git init
git add .
git commit -m "DY Deyu factory showcase site"
git branch -M main
git remote add origin https://github.com/<你的账号>/<仓库名>.git
git push -u origin main
```

> 默认分支必须是 `main`。如果你的仓库默认分支叫 `master`，请改 `admin/config.yml` 里的 `backend.branch`。

### 2. 在 Netlify 导入站点

1. 登录 <https://app.netlify.com> → **Add new site → Import an existing project**。
2. 选择 **GitHub**，授权后选中刚才的仓库。
3. **Build settings**（**部署失败 90% 出在这一步**）：
   - Build command：**留空**（本站点不需要构建；填了 `npm run build` 之类会因没有 package.json 而部署一个空目录）
   - Publish directory：填 `.`（仓库根目录就是发布目录；**必须是直接包含 `index.html`、`css/`、`js/`、`content/`、`images/` 的那一层**）
   - 如果你的仓库里多套了一层 `dy-deyu-site/`，Publish directory 就要填 `dy-deyu-site`，而不是 `.`
4. 点 **Deploy site**。完成后会得到 `https://<随机名>.netlify.app`。
5. 部署日志里看 **Published files** 数量，正常应有 **50+ 个文件**；只有 1–3 个文件说明目录选错了。

### 3. 开启后台登录（关键两步）

1. **Site configuration → Identity → Enable Identity**
   - Registration preferences 选 **Invite only**（只允许自己人进后台，重要）。
2. 同一页面 **Services → Git Gateway → Enable**（让后台无需 GitHub 账号也能写仓库）。
3. **Identity → Invite users** → 填自己的邮箱 → 收邀请邮件 → 点链接设置密码。
4. 打开 `https://<你的站点>.netlify.app/admin/` → 用刚才的邮箱密码登录。

> 邀请邮件里的链接会先落到首页，首页已内置跳转脚本自动送到 `/admin/`。

### 4. 绑定自己的域名

1. Netlify → **Domain management → Add a domain** → 输入你的域名（如 `www.deyuhair.com`）。
2. 按 Netlify 提示配置 DNS：
   - 子域名：添加 `CNAME` 指向 `<你的站点>.netlify.app`；
   - 主域名（裸域）：按面板给的 A 记录填写，或直接使用 Netlify DNS 托管。
3. DNS 生效后在 **HTTPS** 面板启用证书（Let's Encrypt 自动签发），建议打开 **Force HTTPS**。
4. 上线后运行一条命令，把示例域名 `https://your-domain.com` 换成真实域名：

   ```bash
   python tools/setup.py --domain https://www.deyuhair.com --yes
   ```

   它会自动同步 4 处：`content/site.json` → `site_url`、`admin/config.yml` → `site_url`、`robots.txt` → Sitemap、`sitemap.xml` → 全部页面地址（并按当前分类重新生成）。
   `site_url` 也能在后台 `Site Settings → Live site URL` 改，但 robots / sitemap 仍需这条命令或手动编辑。
5. `git commit && git push`，Netlify 自动重新发布。

### 5. GitHub Pages 直接发布

> ⚠️ **重要：Decap CMS 后台在 GitHub Pages 上用不了。** `/admin/` 依赖 Netlify Identity + Git Gateway 做登录和写仓库，GitHub Pages 只是静态托管。如果你需要后台自己传图、改产品，请走第 2 步「GitHub 仓库 → Netlify 导入」——代码照样放 GitHub，后台照常可用。GitHub Pages 只适合纯展示。

**仓库结构（最常见的错）**：仓库根目录必须**直接**是 `index.html`、`404.html`、`.nojekyll`、`_redirects`、`css/`、`js/`、`content/`、`images/`、`admin/`。不要多套一层 `dy-deyu-site/`，也不要只把 zip 传上去。用 `dy-deyu-upload.zip` 解压后全选上传最省事。

**开启 Pages**：
1. 仓库 → **Settings → Pages**；
2. Source 选 **Deploy from a branch**；
3. Branch 选 `main`，文件夹选 **`/ (root)`**（不是 `/docs`）；
4. Save，等 1–3 分钟，打开面板给出的网址，`Ctrl+F5` 强刷。

**网址规律**：仓库名是 `<用户名>.github.io` 时网址为 `https://<用户名>.github.io/`；其它仓库名则是 `https://<用户名>.github.io/<仓库名>/`。站点会**自动识别这个子路径**并注入 `<base href="/<仓库名>/">`，链接、图片、JSON 全部自动带前缀，不用改代码；绑定自定义域名后回到根路径，同样自动适配。

- 根目录的 `.nojekyll` 已放好，防止 GitHub Pages 用 Jekyll 构建时忽略下划线开头的文件或改写静态资源。
- 深链接 `/<仓库名>/products/wigs/2` 由 `404.html` 兜底渲染：页面能正常浏览，但 GitHub Pages 返回的是 **404 状态码**，不利于收录。要干净的 200 状态码请用 Netlify。
- 自检：打开 `https://<用户名>.github.io/<仓库名>/js/app.js` 能看到代码文字即正确；显示 404 或跳回首页，说明仓库结构或 Pages 文件夹选错。

**其它主机**

| 主机 | 说明 |
| --- | --- |
| Netlify | 首选，`_redirects` 已配好，且 Decap CMS 的 Git Gateway 依赖它 |
| Cloudflare Pages | 支持 `_redirects`，但后台需另配 Git Gateway / OAuth |
| Vercel | `vercel.json` 已给出回退规则，后台同样需另配认证 |
| GitHub Pages | 见上，`404.html` 兜底深链接，**后台不可用** |

> 若不用 Netlify，Decap CMS 需要换成 GitHub / GitLab OAuth 后端并自建认证服务，建议直接用 Netlify 省事。

---

## 四、后台怎么用

登录后左侧有 5 个集合：

| 集合 | 管什么 |
| --- | --- |
| **1 · Site Settings** | 品牌名、地址、邮箱、WhatsApp、电话、工作时间、社交链接、版权、SEO、谷歌地图嵌入链接 |
| **2 · Interface Text** | 所有按钮 / 导航 / 分页 / 规格名等界面小文案，逐条 EN + CN |
| **3 · Page Copy** | 首页、产品中心、工厂、关于、联系的全部段落（分 EN / CN 两栏填写） |
| **4 · Product Categories** | 6 张分类卡片的名称、副标题、简介、图片、排序 |
| **5 · Products** | 全部产品：增删改、图片、视频、描述、规格 |

每次改完点 **Publish** → 内容写入 Git → Netlify 自动发布 → 刷新前台即可看到（`/content/*` 已设 `no-store`，不用清缓存）。

### 替换占位图

- 后台任意图片字段 → 点开直接 **Upload new**，或先到顶部 **Media** 上传再选择。
- 上传的文件保存在 `images/uploads/`，引用路径形如 `/images/uploads/xxx.jpg`。
- 首页大图：`Page Copy → Home → Hero → Hero background image`。
- 车间实拍：`Page Copy → Factory → Workshop photos`，每条可换图 + 改中英说明。
- 产品图：`Products → 某产品 → Product images`，**第一张就是列表页卡片图**，建议放 4–6 张（详情页大图 + 缩略图）。

**产品图太多、不想逐款点？用批量工具：**

1. 把照片放进 `images/uploads/products/`，文件名以产品编号开头：
   `DY-WG-001.jpg`（主图）、`DY-WG-001_2.jpg`、`DY-WG-001_3.jpg` …
2. 先预览（不写文件）：`python tools/batch_images.py`
3. 确认无误后写入：`python tools/batch_images.py --yes`

同一编号的多张按文件名排序，第一张自动成为卡片图；只改动匹配到的产品，其他产品与字段不动。
若照片是按 `0001.jpg、0002.jpg…` 顺序拍的，可用 `python tools/batch_images.py --by-order --yes` 按目录顺序依次分配。
> 文件名建议只用英文、数字、下划线，不要空格和中文，避免网址转义问题。

### 上传短视频

产品字段里有三格：

- `Product video (upload MP4)` — 直接上传 mp4；
- `Video URL` — 粘贴 YouTube 链接或外部 MP4 地址，**优先级高于上传的文件**；
- `Video cover image` — 视频封面。

三者都留空时，详情页显示带播放按钮的占位框。

### 新增 / 修改产品

1. `Products → All products → Add`（列表默认折叠，点条目展开）。
2. 必填建议：`id`（唯一编号，同时是详情页网址，如 `DY-WG-061`）、`category`（分类 slug）、中英名称、材质。
3. 其余字段按需要填；**每条都有 EN / CN 两栏**，前台一键切换时对应显示。
4. Publish 后，该分类列表自动按 **每页 12 款** 重新分页，页码 `1 2 3 4 5…` 与上一页 / 下一页自动更新，不需要改代码。

### 新增分类（也可以不改代码）

1. `Product Categories → Add`，填一个新的 `slug`（如 `toppers`）、中英名称、图片、排序。
2. 在 `Products` 里新增产品，`Category slug` 填同一个 slug。
3. 首页、导航下拉、页脚、产品中心会自动出现新分类。

### 双语

前台右上角 `EN / CN` 一键切换，选择记在浏览器本地。所有页面文字、产品名、材质标注、按钮、分页、规格名都是两套文案；产品详情和工厂简介还额外提供**中英对照**排版。

---

## 五、上线前替换清单（照着改就行）

分三类：**① 后台点几下**（最常用，不用碰代码）、**② 本地向导一次填完**（联系方式 + 域名）、**③ 批量工具**（几百张产品图）。

### ① 后台 `/admin/` 里替换

| 要换的内容 | 左侧集合 → 具体字段 | 数据落在哪（不用手改，供核对） |
| --- | --- | --- |
| 邮箱、WhatsApp、电话、地址、工作时间 | `1 → Site Settings → Global Settings` → Email / WhatsApp display number / WhatsApp digits only / Telephone / Address (EN,CN) / Business hours | `content/site.json` |
| 社交链接 | 同上 → **Social links**（每行选平台 + 填主页网址，不用的删掉该行） | `site.json → social` |
| 谷歌地图 | 同上 → **Google Maps embed URL**；大陆访客多改用 **Static map image** | `site.json → map_embed_url / map_image` |
| 版权行、SEO 标题与描述 | 同上 → Copyright line / SEO title / SEO description | `site.json` |
| 首页大图 | `3 → Page Copy → HOME → Hero → Hero background image` | `pages.json → home.hero.image` |
| 规模数字（20+ 年 / 40+ 国家 / 6,000 m² / 月产能…） | `3 → Page Copy → HOME → Numbers strip → Items → Value`；`FACTORY → Factory numbers → Items → Value` | `pages.json` |
| 工厂环境图、车间实拍图 | `3 → Page Copy → FACTORY → Factory environment gallery / Workshop photos → Photos`（每条可换图 + 改中英说明） | `pages.json` |
| 三种发源图、关于页配图 | `3 → Page Copy → HOME → Hair origins → Origins → Image`；`ABOUT US → Story image` | `pages.json` |
| 发展历程年份与事件 | `3 → Page Copy → ABOUT US → Milestones → Items → Year / Text` | `pages.json` |
| 任意页面段落（EN + CN 两栏） | `3 → Page Copy` 下 HOME / PRODUCTS HUB / FACTORY / ABOUT US / CONTACT | `pages.json` |
| 导航、按钮、分页、规格名等小字 | `2 → Interface Text → Interface labels` | `ui.json` |
| 6 张分类卡片的图片与文案 | `4 → Product Categories → Category cards` | `categories.json` |
| 产品名称 / 材质 / 描述 / 规格 / 图片 / 视频 | `5 → Products` → 展开某款逐项改 | `products.json` |
| 360 款示例产品 | `5 → Products` → 删除不需要的、Add 新增真实产品；**每页 12 款与页码自动重算** | `products.json` |

改完点 **Publish** → 内容写入 Git → Netlify 自动发布 → 刷新前台生效。

### ② 联系方式 + 域名：用向导，不用逐个找文件

```bash
python tools/setup.py --show        # 先看当前（示例）值
python tools/setup.py              # 交互式：逐项问，直接回车 = 保留原值
```

也可以编辑根目录的 `my-site-info.json`（只填要改的项，`social` 里写 `remove` 表示删除该图标；若从 Git 克隆后没有这个文件，先运行 `python tools/setup.py --template` 生成），然后：

```bash
python tools/setup.py --config my-site-info.json          # 列出改动并确认
python tools/setup.py --config my-site-info.json --yes   # 不询问直接写入
```

绑定域名时单独一条命令即可：

```bash
python tools/setup.py --domain https://www.deyuhair.com --yes
```

向导会自动同步 `content/site.json`、`admin/config.yml`、`robots.txt`、`sitemap.xml`，并按当前分类重建 sitemap；WhatsApp 图标的 `wa.me` 链接会跟随纯数字号码自动更新。
写之前会列出全部改动让你确认，随时可用 Git 回滚。`my-site-info.json` 已在 `.gitignore` 中，真实联系方式不会提交到公开仓库。

> 谷歌地图嵌入链接怎么拿：Google Maps 搜到地址 → **分享 → 嵌入地图** → 复制 `src="..."` 里的整条 URL，粘进 `Google Maps embed URL` 字段。

### ③ 图片与视频

- **少量图片**：后台任意图片字段 → **Upload new**，或先到顶部 **Media** 上传再选择；文件保存在 `images/uploads/`。
- **几百张产品图**：放进 `images/uploads/products/`，按产品编号命名，运行 `python tools/batch_images.py --yes`（详见第四节「替换占位图」）。
- **视频**：`5 → Products` 里 `Product video (upload MP4)` 上传 MP4，或 `Video URL` 粘贴 YouTube 链接（后者优先），`Video cover image` 设封面；都留空时显示占位框。
- 建议产品图压到 200 KB 内、宽约 1200 px，优先 WebP/JPG；文件名只用英文数字下划线。
- 占位图清单：`images/placeholder-hero.svg`（首页）、`placeholder-category-*.svg`（6 分类）、`placeholder-product-1..6.svg`（产品，循环使用）、`placeholder-factory-1..8.svg`（工厂/车间）、`placeholder-material-1..3.svg`（发源）、`logo-dy.svg`（Logo，在 Site Settings → Logo 里换）。

**仍然需要人工确认的**：360 款示例产品的名称与规格、首页/工厂的规模与产能数字、关于页的发展历程年份。这些是示例文案，建议按真实情况改完再上线。

---

## 六、常见问题

**Q：一直显示 "Loading…" / 打开是空白？**
A：按顺序排查 —
1. 是否**在压缩软件里直接双击**了 `index.html`？必须先完整解压整个文件夹。
2. `css/`、`js/`、`content/`、`images/` 是否与 `index.html` 在同一目录（不要只复制单个 HTML 文件）。
3. 双击模式需要 `content/offline-data.js`；若缺失，运行 `python tools/gen_offline_data.py`。
4. 仍不行就双击 `启动本地预览.bat`，用实时数据 + HTTP 打开。

页面内置 6 秒诊断：资源没加载出来时会**逐个列出** `js/app.js`、`css/style.css`、`content/site.json`、`content/offline-data.js` 的 ✅/❌、当前网址、资源根路径和对应修复建议，不会再无限停在 Loading。

**Q：部署后出现「页面资源没有加载成功」？（最常见）**
A：看诊断面板里哪个是 ❌，然后按对应项处理：

| 诊断结果 | 原因 | 怎么做 |
| --- | --- | --- |
| ❌ `js/app.js` 或 `css/style.css` | 发布目录不对 / 只上传了 `index.html` / 填了 Build command | Netlify → **Build settings**：Build command **留空**，Publish directory 指向**直接包含 index.html 的那一层**（`.` 或 `dy-deyu-site`），重新 Deploy；手动上传时把**整个文件夹**拖进去，不要只拖一个 HTML |
| ❌ `content/site.json` | `content/` 没提交，或主机把 JSON 请求回退成首页 HTML | 确认 `git add .` 把 `content/` 提交了；检查托管平台是否对静态目录做了 rewrite |
| 四项全 ✅ 但仍报错 | 脚本被 CDN/CSP 拦截或浏览器缓存 | 按 F12 看控制台红色报错；`Ctrl+F5` 强刷；换 Chrome / Edge；GitHub Pages 需确认 `404.html` 在站点根 |

> 即使 `content/*.json` 全部读不到，页面现在也会自动改用 `content/offline-data.js` 快照正常显示（内容可能不是最新，但不会白屏）。所以**只有连 `js/app.js` 都没发布时**才会看到这个诊断页 —— 那一定是发布目录/上传文件的问题，改后台或改代码都没用。

**Q：部署在子目录（如 GitHub Pages 的 `/repo/`）会白屏吗？**
A：不会。首页会自动计算资源根并注入 `<base href="/repo/">`，链接自动带 `/repo/` 前缀，图片与 JSON 用相对路径。已用真实浏览器验证过根部署、深链接与子目录三种情况。

**Q：后台 `/admin/` 打不开或登录失败？**
A：按顺序检查 — Identity 是否启用、Git Gateway 是否启用、是否已邀请自己并设置密码、GitHub 仓库默认分支是否与 `backend.branch: main` 一致、浏览器是否拦截了第三方脚本（Decap CMS 与 Identity 走 CDN）。

**Q：后台 Publish 了但前台没变？**
A：确认Publish 成功、Netlify 的 deploy 已完成（通常几十秒）、刷新页面。若仍不变，检查 `netlify.toml` 的 `/content/*` 缓存头是否生效。

**Q：想在本机用后台？**
A：`config.yml` 已开 `local_backend: true`。在项目根目录另开一个终端运行 `npx decap-server`，再用 `python tools/serve.py` 打开 `http://localhost:8000/admin/`，即可在本地 Git 代理模式下编辑（需要目录是 Git 仓库）。正式线上仍走 Netlify Identity。

**Q：图片太大加载慢？**
A：产品图建议压缩到 200 KB 以内、宽度 1200 px 左右，优先 WebP/JPG；视频建议外链 YouTube 或压缩后 MP4。

**Q：网址里的 `#` 号？**
A：线上（HTTP）是干净的 path 网址（`/products/wigs/2`），靠 `_redirects` 回退实现，方便分享与收录；只有**双击离线预览**时才用 `#/products/wigs` 哈希路由，因为 `file://` 无法做路径重写。

---

## 七、维护命令

```bash
python tools/serve.py          # 本地预览
python tools/setup.py          # ★ 上线信息替换向导（联系方式 / 社交 / 地图 / 域名）
python tools/batch_images.py   # ★ 批量替换产品图片（先预览，加 --yes 写入）
node tools/smoke-test.js       # 冒烟测试：HTTP/双击 × 10 路由 × EN/CN + 子目录 + 数据兜底（44 项）
python tools/gen_offline_data.py   # 后台 Publish 或改过 JSON 后，刷新双击预览快照
python tools/gen_placeholders.py   # 重新生成占位图（不会动后台上传的文件）
python tools/gen_catalog.py        # ⚠️ 会覆盖 products.json，编辑过后台后请勿运行
```

---

© DY | 山东德宇发制品厂 · 本站点仅用于产品展示。
