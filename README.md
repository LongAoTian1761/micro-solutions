# 经济学的思维方式 · 课后讨论题与同学解答

课程教材：**Heyne, Boettke & Prychitko, _The Economic Way of Thinking_, 13th ed.**
（中译本《经济学的思维方式》第 13 版）

网站收录 **16 章、300 道**课后讨论题（Questions for Discussion），中英对照。
同学打开网页填完就能提交 —— 不需要注册、不需要登录，答案和附件一起存下来，
展示在对应题目页。

- 线上地址：https://longaotian1761.github.io/micro-solutions/
- 代码仓库：https://github.com/LongAoTian1761/micro-solutions

## 双语是怎么做的

每道题在 `data/course.json` 里存两份题面：`problem`（英文，取自原版 PDF）
与 `problemZh`（中文，取自中译本）。页面把两份都写进 HTML，靠 `<html data-lang-mode>`
属性决定显示哪份，所以切换是瞬时的、不重新请求。

界面右上角三个按钮：**中文 / English / 中英对照**，选择记在浏览器 localStorage，
首次绘制前就生效，不会闪。

题面是自动提取的，复杂公式、上下标可能有偏差，**以教材为准**。
发现错误改 `data/course.json` 重新构建即可。

## 三步接入接收服务器

同学端之所以能做到「零账号」，是因为答案和附件直接写进 Supabase
（免费额度足够一个班用很多年）。配置一次，大约 5 分钟。

**第一步**　到 [supabase.com](https://supabase.com) 注册（免费），新建一个 Project。
本项目用**独立的 Supabase 项目**，不要和《国际宏观经济学》那个站共用。

**第二步**　左侧 **SQL Editor → New Query**，把 [`supabase/schema.sql`](supabase/schema.sql)
整个文件粘贴进去，点 **Run**。它会建好表、权限和存放附件的存储桶。

**第三步**　左侧 **Project Settings → API**，复制两个值填进 `data/course.json`：

```json
"submission": {
  "backend": "supabase",
  "supabase": {
    "url": "https://xxxxxxxxxxxx.supabase.co",
    "anonKey": "sb_publishable_...",
    "table": "submissions",
    "bucket": "answer-uploads",
    "maxFileMb": 20,
    "autoApprove": true
  }
}
```

然后重新构建并推送：

```bash
npm run build
```

体检：

```bash
node scripts/check-backend.mjs
```

它用和浏览器完全相同的公钥逐项验证：能否匿名提交、能否传附件、有没有把整张表
暴露出去、有没有办法绕过前端给自己「加精」。全绿就说明同学可以用了。

> `anonKey` 本来就是公开的，写进网页没有安全问题 —— 真正拦人的是 `schema.sql`
> 里的行级权限（RLS）。**绝对不要**把 `service_role` key 写进网页，
> 那个只在你自己的电脑上用。

### 不想注册 Supabase，只想先看看效果

```bash
npm run mock-backend        # 本地假后端 → http://localhost:4174

# 另开一个终端
$env:SUPABASE_URL="http://localhost:4174"
$env:SUPABASE_ANON_KEY="local-test-key-000000000000000000"
npm run build
```

## 同学是怎么交作业的

1. 打开「上传答案」，选习题编号、填姓名（可填「匿名」）、写正文 —— 支持 Markdown 与 LaTeX
2. 需要的话把图表、手写照片、PDF、代码拖进附件框，单个 20 MB 以内
3. 点提交，答案立刻出现在对应题目页的「同学答案」里
4. 写错了不用慌：提交页下方「我的提交」里可以随时修改或撤回

图片和 PDF 会**直接显示**在解答里，不会被藏在一个下载链接后面。
正文里写 `![说明](fig.png)`，只要附件里有同名文件，会自动换成上传后的地址。

## 教师 / 管理员操作

### 1. 改站点信息

编辑 `data/course.json` 顶部的 `site`：

```json
"site": {
  "title": "The Economic Way of Thinking",
  "titleZh": "经济学的思维方式 · 课后讨论题与同学解答",
  "subtitle": "Heyne, Boettke & Prychitko, 13th ed.",
  "description": "……",
  "repo": "LongAoTian1761/micro-solutions",
  "showProblemText": true,
  "submission": { "…": "见上文" }
}
```

`repo` 填了真实仓库地址后，导航栏的「讨论区」（指向 GitHub Issues）和页脚的
「GitHub 仓库」才会出现；留空或写成 `your-github-name/...` 时这两处直接不渲染。

`showProblemText: false` 可以关掉题面展示，只留提交框。

### 2. 改题面 / 补参考思路

每道题在 `chapters[].exercises[]` 里，字段：

| 字段 | 说明 |
| --- | --- |
| `id` | 题号，如 `3.12`；章节页与上传页的下拉都按它排序 |
| `slug` | 页面文件名，把 `id` 的点换成下划线，如 `3_12` |
| `title` / `titleZh` | 列表页显示的短标签 |
| `problem` / `problemZh` | 英文、中文题面，支持 Markdown 与 LaTeX |
| `problemSource` | 填 `book` 时页面底部提示「题面由教材自动提取」 |
| `reference` | 可选的教师参考思路：`{ "note": "标题", "body": "Markdown 正文" }` |
| `tags` | 字符串数组，显示在题号后面 |

改完跑 `npm run build`，再提交推送。

### 3. 加一道新题

复制一段 `exercises` 里的对象，改 `id` / `slug` / 题面即可。
`id` 的数字部分必须连续，章节页的「上一题 / 下一题」按数组顺序走。

### 4. 管理同学交上来的作业

直接在 Supabase 的 Table Editor 里改 `submissions` 表：

- `status` 改成 `withdrawn` 即撤下（公开页面不显示）
- `feedback`、`grade` 写批语和等级，会显示在答案下方
- 想先审后放：把 `data/course.json` 里的 `autoApprove` 改成 `false`，
  新提交默认 `pending`，审过再改成 `verified`

## 目录结构

```
assets/          站点样式与脚本（改界面看这里）
  css/site.css
  js/site.js     渲染、双语开关、附件、搜索
  js/submit.js   上传页
  js/backend.js  Supabase 读写
chapters/        构建产物：每章目录 + 每道题一页
data/course.json 题库与站点配置（唯一需要手工维护的数据）
scripts/build.mjs 生成静态页面
supabase/schema.sql 数据库结构与权限
tools/           本地小工具
```

## 本地预览与构建

```bash
npm run build        # 生成 index.html / answers.html / submit.html / chapters/**
npm run dev          # 本地预览 → http://localhost:4173
```

改完 `data/course.json` 或 `scripts/`、`assets/` 里的东西，都要重新 build
再提交，否则线上还是旧的。浏览器缓存较狠，本地预览记得 `Ctrl + F5`。

## 部署到 GitHub Pages

仓库 **Settings → Pages**：Source 选 `Deploy from a branch`，
分支 `main`，目录 `/ (root)`，Save。等一两分钟即可。

以后更新：改完 → `npm run build` → GitHub Desktop 里 Commit → Push。

## 已知限制

- 题面由教材 PDF 自动提取，公式与上下标可能有偏差；以教材为准。
- 内嵌 PDF 预览依赖浏览器的阅读器，个别内置浏览器（如代码编辑器的内置预览窗）
  显示不出内容，用 Chrome/Edge 打开正常。
- 讨论题没有唯一答案，站上展示的都是同学自己的解答，不是标准答案。

## 版权

题目文字版权归原作者与出版社所有，本站仅供本课程教学使用。
建议不要把站点提交给搜索引擎收录。
