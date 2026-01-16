# LINUX DO 社区域名分级配置指南

本文档详细说明如何配置 LINUX DO 社区外部链接分级的域名规则。

## 📄 配置文件说明

域名规则存储在 `links.txt` 文件中，该文件使用简单的文本格式：

- 每个域名/规则占一行
- 以 `#` 开头的行为注释或分类标记
- 空行会被自动忽略
- 域名和路径会自动转换为小写（正则表达式除外）

**重要提示**：
1. 匹配时 **不区分 http 和 https 协议**，只要域名和路径匹配即可（正则表达式除外）
2. 匹配时 **不区分大小写**，域名和路径都不区分（正则表达式除外）
3. 例如配置 `example.com/Admin`，则 `http://example.com/admin`、`https://example.com/ADMIN`、`https://example.com/Admin` 都会匹配

## 📋 配置项说明

系统提供 6 种域名分类，每种分类对应不同的安全级别和用户体验：

### 1. **内部域名** (Internal Domains)
- **用途**：您自己的论坛域名和 CDN 域名
- **行为**：不显示任何图标，不弹出确认窗口
- **示例**：`example.com|cdn.example.com`

### 2. **受信任域名** (Trusted Domains)
- **用途**：知名的、安全的外部网站
- **行为**：显示 🔒 锁形图标，跳过确认弹窗
- **示例**：`youtube.com|github.com|wikipedia.org`

### 3. **风险域名** (Risky Domains)
- **用途**：可能存在风险的网站（如短链接服务）
- **行为**：显示 ⚠️ 警告图标，弹出警告样式的确认窗口
- **示例**：`bit.ly|tinyurl.com|t.co`

### 4. **危险域名** (Dangerous Domains)
- **用途**：已知的钓鱼、诈骗网站
- **行为**：显示 ☠️ 危险图标，弹窗仅允许复制 URL，禁止直接访问
- **示例**：`known-phishing-site.com`

### 5. **屏蔽域名** (Blocked Domains)
- **用途**：垃圾网站、完全禁止访问的域名
- **行为**：链接被替换为 `[已屏蔽链接]` 文本，完全无法点击
- **示例**：`spam-site.com`

### 6. **普通域名** (Normal Domains)
- **用途**：除以上 5 种分类外的其他域名
- **行为**：显示外部域名图标，弹出外部域名访问确认窗口
 
---

## 🎯 匹配模式详解

系统支持 7 种强大的匹配模式：

### 1️⃣ **精确域名匹配**
```
example.com
```
- **说明**：只匹配域名，不限制路径，不论 Query 参数
- **匹配**：
  - `http://example.com`
  - `https://example.com`
  - `https://example.com/` 
  - `https://example.com/any/path`
  - `https://example.com?query=123`
- **不匹配**：
  - `https://sub.example.com` （子域名不匹配）
  - `https://another-example.com` （不同域名）

---

### 2️⃣ **子域名通配符匹配** (`*.`)
```
*.example.com
```
- **说明**：匹配所有子域名（包括主域名本身），不限制路径，不论 Query 参数
- **匹配**：
  - `https://example.com` （主域名也会匹配）
  - `https://www.example.com`
  - `https://api.example.com`
  - `https://sub.domain.example.com` （多级子域名）
  - `https://sub.example.com/any/path` （任意路径）
- **不匹配**：
  - `https://example.org` （不同的顶级域名）
  - `https://notexample.com` （不同域名）

---

### 3️⃣ **精确域名 + 精确路径匹配**
```
example.com/admin
```
- **说明**：必须同时匹配精确域名和完整路径（不区分大小写，不论 Query 参数），'/' 也算路径，匹配根路径请使用 `example.com/`
- **匹配**：
  - `https://example.com/admin`
  - `http://example.com/admin`
  - `https://example.com/Admin` （不区分大小写）
  - `https://example.com/ADMIN` （不区分大小写）
- **不匹配**：
  - `https://example.com` （缺少路径）
  - `https://example.com/` （路径不完全匹配）
  - `https://example.com/admin/` （末尾多了斜杠）
  - `https://example.com/admin/users` （路径更长）
  - `https://sub.example.com/admin` （子域名不匹配）

---

### 4️⃣ **精确域名 + 路径前缀通配符匹配** (`/*`)
```
example.com/api/*
```
- **说明**：匹配精确域名和以指定路径开头的所有 URL（不区分大小写，不论 Query 参数）。
- **匹配**：
  - `https://example.com/api/`
  - `https://example.com/api/users`
  - `https://example.com/api/v1/data`
  - `https://example.com/api/anything/nested`
- **不匹配**：
  - `https://example.com` （缺少路径）
  - `https://example.com/` （路径不匹配）
  - `https://example.com/admin` （路径前缀不匹配）
  - `https://example.com/apiabc` （必须以 `/api/` 开头）
  - `https://sub.example.com/api/users` （子域名不匹配）

**关键点**：
- `*` 只能放在路径末尾
- 路径前缀匹配也不区分大小写
- 如果使用 `/api*` 则匹配：`/api` 和 `/apiabc` 等

---

### 5️⃣ **子域名通配符 + 路径通配符组合匹配** (`*.` + `/*`)
```
*.example.com/api/*
```
- **说明**：同时使用子域名通配符和路径通配符，匹配所有子域名下的特定路径（不区分大小写，不论 Query 参数）。
- **匹配**：
  - `https://example.com/api/` （主域名也会匹配）
  - `https://www.example.com/api/users`
  - `https://api.example.com/api/v1`
  - `https://sub.domain.example.com/api/data`
  - `https://any.example.com/api/anything/nested`
- **不匹配**：
  - `https://example.com` （缺少路径）
  - `https://example.com/admin` （路径前缀不匹配）
  - `https://sub.example.com` （缺少路径）
  - `https://example.org/api/users` （域名不匹配）

**应用场景**：
- `*.github.com/api/*` - 匹配 GitHub 所有子域名的 API 路径
- `*.example.com/download/*` - 匹配所有子域名的下载路径
- `*.cdn.com/assets/*` - 匹配 CDN 和子域名的所有资源路径
- `*.cdn.com/user*` - 匹配 CDN 和子域名所有`/user`开头的路径

---

### 6️⃣ **URL 关键词匹配** (`**`)
```
**phishing
```
- **说明**：在整个 URL（包括协议、域名、路径、查询参数）中搜索关键词
- **特点**：**不区分大小写**
- **匹配**：
  - `https://phishing.com` （域名包含关键词）
  - `https://example.com/phishing-page` （路径包含）
  - `https://example.com?redirect=phishing-site.com` （参数包含）
  - `https://safe-site.com/warning-about-PHISHING` （不区分大小写）
- **用途示例**：
  - `**aff=` - 检测联盟链接（如 `?aff=123`）
  - `**affiliate` - 检测联盟营销链接
  - `**download` - 检测下载链接

---

### 7️⃣ **正则表达式匹配** (`^`)
```
^.*\.(tk|ml|ga|cf)(/|$)
```
- **说明**：`^` 后面的内容作为正则表达式进行匹配，匹配整个 URL（包括协议、域名、路径、查询参数）
- **特点**：**区分大小写**，功能最强大
- **匹配示例**：
  - `https://example.tk`
  - `https://site.ml/page`
  - `https://test.ga`
  - `https://demo.cf/`
- **不匹配**：
  - `https://example.com`
  - `https://example.tk.com` （正则不匹配）

**高级示例**：

1. **匹配 IP 地址形式的 URL**：
   ```
   ^https?://[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}
   ```
   匹配：`http://192.168.1.1`、`https://10.0.0.1/path`

2. **匹配多个可疑 TLD**：
   ```
   ^.*\.(tk|ml|ga|cf|gq)(/|$)
   ```
   匹配：`.tk`、`.ml`、`.ga`、`.cf`、`.gq` 结尾的域名

3. **匹配包含数字用户 ID 的 URL**：
   ```
   ^.*/user/\d{6,}
   ```
   匹配：`/user/123456`、`/user/999999999`

---

## 📝 域名和路径的区别总结

| 配置格式 | 匹配域名 | 匹配路径 | 示例 |
|---------|---------|---------|-----|
| `example.com` | 精确匹配 `example.com` | **所有路径** | `example.com/anything` ✅ |
| `*.example.com` | `example.com` 及所有子域名 | **所有路径** | `sub.example.com/anything` ✅ |
| `example.com/admin` | 精确匹配 `example.com` | **精确匹配** `/admin` | `example.com/admin` ✅<br>`example.com/Admin` ✅<br>`example.com/admin/` ❌ |
| `example.com/api/*` | 精确匹配 `example.com` | **前缀匹配** `/api/` 开头 | `example.com/api/users` ✅<br>`example.com/API/users` ✅ |
| `*.example.com/api/*` | `example.com` 及所有子域名 | **前缀匹配** `/api/` 开头 | `sub.example.com/api/v1` ✅ |

**关键要点**：
1. **域名部分**自动转换为小写，不区分大小写
2. **路径部分**也自动转换为小写，不区分大小写
3. **协议部分**（http/https）完全忽略，不影响匹配
4. **单独的域名**会匹配该域名下的所有路径
5. **域名 + 路径**必须同时匹配域名和路径规则
6. **只有正则表达式**（`^` 开头）才区分大小写

---

## ⚙️ 配置格式说明

### 文件格式

`links.txt` 文件遵循以下格式规则：

1. **每行一个域名/模式**：每个域名或匹配模式占用一行
2. **注释行**：以 `#` 开头的行用于分类标记和注释
3. **空行忽略**：空白行会被自动忽略
4. **自动小写转换**：域名和路径会自动转换为小写（正则表达式除外）
5. **空格处理**：前后空格会被自动去除

### 示例文件结构

```text
# internal - Internal domains
mysite.com
*.mysite.com
cdn.mysite.com

# trusted - Trusted external domains
github.com
*.github.io
youtube.com

# risky - Potentially risky domains
bit.ly
tinyurl.com

# danger - Dangerous domains
**aff=
^.*\.(tk|ml|ga)(/|$)

# blocked - Completely blocked domains
spam-site.com
```

**注意事项**：
- 域名和路径会自动转换为小写（正则表达式除外）
- 前后空格会被自动去除
- 空白条目会被自动过滤

---

## 💡 实用配置示例

### 📂 lists.txt 文件结构

配置文件使用注释行（以 `#` 开头）来分隔不同的分类：

```text
# internal - Internal domains
mysite.com
*.mysite.com
cdn.mysite.com

# trusted - Trusted external domains
github.com
*.github.io
youtube.com
youtu.be

# risky - Potentially risky domains
bit.ly
tinyurl.com
t.co

# danger - Dangerous domains
**aff=
^.*\.(tk|ml|ga|cf)(/|$)

# blocked - Completely blocked domains
spam-site.com
```

---

### ✅ 内部域名配置示例（Internal）

**用途**：您自己运营的网站和 CDN

```text
# internal - Internal domains
mysite.com
*.mysite.com
cdn.mysite.com
static.mysite.com
img.mysite.com
```

**效果**：这些链接不显示任何图标，像内部链接一样处理

---

### 🔒 受信任域名配置示例（Trusted）

**用途**：知名的、安全的外部网站

```text
# trusted - Trusted external domains
# 代码托管平台
github.com
*.github.io
*.githubusercontent.com
gitlab.com
*.gitlab.io

# 视频网站
youtube.com
*.youtube.com
youtu.be

# 知识网站
wikipedia.org
*.wikipedia.org
stackoverflow.com
*.stackoverflow.com

# 社交媒体（受信任的官方域名）
twitter.com
x.com
reddit.com
*.reddit.com
```

**效果**：显示 🔒 锁形图标，点击后直接跳转，不弹出确认窗口

---

### ⚠️ 风险域名配置示例（Risky）

**用途**：可能有风险的服务（主要是短链接）

```text
# risky - Potentially risky domains
# 短链接服务
bit.ly
tinyurl.com
t.co
goo.gl
ow.ly
j.mp
short.link
*.short.link

# 文件分享（可能有风险）
adf.ly
linkvertise.com
```

**效果**：显示 ⚠️ 警告图标，点击后弹出警告样式的确认窗口

---

### ☠️ 危险域名配置示例（Dangerous）

**用途**：检测已知的钓鱼、诈骗或可疑模式

```text
# danger - Dangerous domains
# 联盟链接检测
**aff=
**affiliate
**ref=sponsor

# 可疑的免费 TLD
^.*\.(tk|ml|ga|cf|gq)(/|$)

# IP 地址形式的 URL（可能是钓鱼）
^https?://[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}

# 已知钓鱼域名
known-phishing-site.com
fake-login-page.com
```

**效果**：显示 ☠️ 危险图标，弹窗仅允许复制 URL，**禁止直接访问**

---

### 🚫 屏蔽域名配置示例（Blocked）

**用途**：垃圾网站、恶意软件站点

```text
# blocked - Completely blocked domains
spam-site.com
malware-example.com
unwanted-content.com
```

**效果**：链接被替换为 `[已屏蔽链接]` 文本，无法查看，无法点击

---

### 🎯 高级配置示例

#### 示例 1：只信任 GitHub 的特定路径
```text
# trusted - Trusted external domains
github.com/linux-do/*
```
只有 `github.com/linux-do/` 下的链接被信任，其他 GitHub 链接正常处理。

#### 示例 2：组合使用域名和子域名
```text
# trusted - Trusted external domains
*.example.com
specific.another-site.com
```
- `example.com` 和所有子域名都被信任
- 只有 `specific.another-site.com` 被信任，其他子域名不信任

#### 示例 3：检测 Telegram 频道链接
```text
# trusted - Trusted external domains
t.me/your_channel
t.me/your_group
```
只信任特定的 Telegram 频道/群组链接。

#### 示例 4：屏蔽所有包含特定关键词的 URL
```text
# blocked - Completely blocked domains
**casino
**porn
**viagra
```
任何 URL 中包含这些关键词都会被屏蔽。

---

## 🔍 匹配优先级

当一个 URL 同时匹配多个分类时，按以下优先级处理：

1. **屏蔽域名** - 最高优先级，直接替换为文本
2. **危险域名** - 显示危险警告
3. **风险域名** - 显示风险警告
4. **受信任域名** - 显示信任图标
5. **内部域名** - 不显示任何标记
6. **其他外部链接** - 显示普通外部链接图标

---

## 📝 配置建议

1. **内部域名**：配置您自己的所有域名和 CDN
2. **受信任域名**：只添加真正信任的知名网站
3. **风险域名**：添加短链接服务、文件分享站等
4. **危险域名**：使用 `**` 和 `^` 模式检测可疑关键词和模式
5. **屏蔽域名**：只添加确认的垃圾/恶意网站

---

## ❓ 常见问题

**Q: 如何配置这些域名规则？**  
A: 域名规则存储在 `lists.txt` 文件中，每行一个域名/模式。直接修改并PR该文件即可。

**Q: http 和 https 链接会区别对待吗？**  
A: 不会。除了正则表达式（`^` 开头）外，匹配时完全忽略协议部分，`http://example.com` 和 `https://example.com` 被视为相同。

**Q: 域名匹配区分大小写吗？**  
A: 不区分。除了正则表达式（`^` 开头）外，域名和路径都会自动转换为小写进行匹配。例如 `example.com/Admin` 和 `example.com/admin` 是相同的。

**Q: `example.com` 和 `example.com/` 有区别吗？**  
A: 在匹配时，单独的 `example.com` 会匹配该域名下的所有路径。`example.com/` 表示精确匹配根路径（带斜杠）。

**Q: 如何同时匹配子域名和路径？**  
A: 可以组合使用 `*.` 和路径，例如 `*.example.com/api/*` 会匹配所有子域名下的 `/api/` 路径。

**Q: 如何测试我的配置是否生效？**  
A: 在帖子中发布测试链接，查看是否显示预期的图标和行为。

**Q: 正则表达式区分大小写吗？**  
A: 是的，正则表达式（`^` 开头）是唯一区分大小写的匹配模式。如果需要不区分大小写的正则，可以使用 `(?i)` 标志。

**Q: `**` 和 `^` 有什么区别？**  
A: `**` 是简单的关键词搜索（不区分大小写，匹配速度更快），`^` 是完整的正则表达式（区分大小写，功能更强大）。

**Q: 为什么我配置的路径匹配不生效？**  
A: 请检查：1) 域名部分是否正确匹配；2) 是否需要使用 `/*` 通配符；3) 路径是否完全匹配（注意末尾斜杠）。记住 `example.com/admin` 不会匹配 `example.com/admin/`。

