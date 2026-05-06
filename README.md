# Happy Time · 快乐时光
## 项目结构与操作手册

> 最后更新：2026年5月6日

---

## 一、项目概述

一个用于记录朋友圈聚会、出游活动的私人相册网站，支持照片和视频展示，国内可直接访问。

**网站地址**
- 🌐 国内访问：`深圳活动群.cn`
- 🌐 国外访问：`sunshine-memories.vercel.app`
- ⚙️ 管理后台：`深圳活动群.cn/admin.html`

---

## 二、整体架构

```
你的电脑
    ↓ 上传照片/视频
广州 COS（媒体存储）sunshine-1343282399
    ↓ 文件上传自动触发
腾讯云函数 sync-memories v3
    ↓ 同时推送
    ├── GitHub 仓库 GitComeOn/sunshine-memories（data.json）
    │       ↓ GitHub Actions 自动触发
    │       ↓ 调用云函数同步香港COS
    │   Vercel（国外访问，自动部署）
    └── 香港 COS sunshine-web-1343282399（data.json）
            ↓ 域名解析
        深圳活动群.cn（国内访问）

admin.html 修改活动信息
    ↓ 推送到 GitHub
    ↓ GitHub Actions 自动触发云函数
    ↓ 云函数同步香港 COS
    ↓ 网站自动更新（约2分钟）
```

---

## 三、各平台账号与资源

| 平台 | 资源名称 | 用途 |
|------|----------|------|
| 腾讯云 COS | `sunshine-1343282399`（广州） | 存储所有照片和视频 |
| 腾讯云 COS | `sunshine-web-1343282399`（香港） | 托管网站文件，国内访问 |
| 腾讯云函数 | `sync-memories`（广州） | 自动同步数据 |
| GitHub | `GitComeOn/sunshine-memories` | 网站代码仓库，触发自动化流程 |
| Vercel | `sunshine-memories` | 网站托管（国外） |
| 域名 | `深圳活动群.cn` | 国内访问域名，DNS指向香港COS |

---

## 四、文件结构

### GitHub 仓库（GitComeOn/sunshine-memories）
```
sunshine-memories/
├── .github/
│   └── workflows/
│       └── sync-cos.yml    # GitHub Actions：推送后自动触发云函数
├── index.html              # 主展示页（时间线+相册视图+分类标签）
├── admin.html              # 管理后台（密码保护，编辑活动信息）
├── data.json               # 活动数据文件（自动生成，勿手动编辑）
└── README.md               # 本文档
```

### 广州 COS（媒体存储）结构
```
sunshine-1343282399/
├── 2026-05-02_出游_粤赣三日自驾_广东韶关·江西崇义/
│   ├── 人物/
│   │   ├── 微信图片_001.jpg
│   │   └── 微信图片_002.jpg
│   ├── 风景/
│   ├── 足迹/
│   └── 花絮/
├── 2026-02-23_聚会_春节同学相聚_广东深圳/
│   ├── 合影/
│   ├── 美食/
│   └── 花絮/
└── ...
```

### 香港 COS（网站文件）
```
sunshine-web-1343282399/
├── index.html      # 与GitHub同步
├── admin.html      # 与GitHub同步
└── data.json       # 由云函数自动更新（含no-cache头）
```

### 腾讯云函数
```
sync-memories/
├── src/
│   ├── index.js        # 主函数代码 v3（实际执行文件）
│   ├── package.json
│   └── node_modules/
└── index.js            # 与 src/index.js 保持一致
```

> ⚠️ **重要**：云函数实际执行的是 `src/index.js`，修改代码后需同时更新 `src/index.js` 和根目录 `index.js`，然后在控制台重新部署。

---

## 五、COS 文件夹命名规则

上传活动照片时，**顶层文件夹命名必须严格遵守以下格式**：

```
日期_类型_活动名称_地点
```

**示例：**
```
2026-05-02_出游_粤赣三日自驾_广东韶关·江西崇义
2026-06-01_聚餐_六一聚餐_广州天河
2026-07-15_日常_周末爬山_深圳梧桐山
```

**类型可选值：**
- `出游`：户外旅行、景点游览
- `聚餐`：饭局、聚会
- `日常`：日常活动

---

## 六、子分类文件夹规则

在活动文件夹下建子文件夹，云函数自动识别并在网站显示分类标签：

| 子文件夹名 | 适用场景 | 图标 |
|-----------|---------|------|
| `人物` | 人像、合照 | 👥 |
| `风景` | 自然风光、建筑 | 🏔️ |
| `足迹` | 景点打卡、地标 | 🗺️ |
| `花絮` | 路上随拍、搞笑瞬间 | 🎬 |
| `合影` | 集体照（聚餐用） | 📸 |
| `美食` | 菜品、餐厅环境（聚餐用） | 🍜 |

**旧数据兼容**：根目录的照片 category 为空，在网站归入「全部」，无需整理。

---

## 七、媒体文件说明

**支持格式：**
- 图片：`.jpg` `.jpeg` `.png` `.gif` `.webp` `.heic` `.heif`
- 视频：`.mp4` `.mov` `.avi` `.mkv` `.m4v`

**排序规则：** 照片在前，视频自动排最后（代码层面处理，无需改文件名）。

---

## 八、完整自动化流程

### 场景一：上传新照片/视频

```
1. 在广州 COS 创建活动文件夹（命名规则见上）
2. 在子分类文件夹里上传照片/视频
   ↓ 自动触发（约1-2分钟完成）
3. 云函数扫描 → 更新 GitHub data.json + 香港 COS data.json
4. 网站自动显示新内容
```

### 场景二：修改活动标题/简介/封面

```
1. 打开 深圳活动群.cn/admin.html
2. 输入密码：YJ87654321
3. 找到活动，点击展开编辑
4. 修改标题、简介，或点选封面图
5. 点「保存修改」→「推送到 GitHub」
   ↓ GitHub Actions 自动触发（约2分钟）
6. 云函数同步香港 COS，网站自动更新
```

### 场景三：手动强制同步

如果自动触发失败，可手动触发：

腾讯云控制台 → 云函数 → `sync-memories` → 函数代码 → 点「测试」

---

## 九、云函数环境变量

| 变量名 | 值 | 说明 |
|--------|-----|------|
| `COS_BUCKET` | `sunshine-1343282399` | 广州媒体存储桶 |
| `COS_REGION` | `ap-guangzhou` | 广州区域 |
| `WEB_BUCKET` | `sunshine-web-1343282399` | 香港网站存储桶 |
| `WEB_REGION` | `ap-hongkong` | 香港区域 |
| `GH_TOKEN` | `ghp_xxx...` | GitHub Personal Access Token |
| `GH_OWNER` | `GitComeOn` | GitHub 用户名 |
| `GH_REPO` | `sunshine-memories` | GitHub 仓库名 |
| `GH_BRANCH` | `main` | 分支名 |
| `TZ` | `Asia/Shanghai` | 时区（北京时间） |

---

## 十、GitHub Actions 配置

**文件位置：** `.github/workflows/sync-cos.yml`

**触发条件：** `main` 分支的 `data.json` 文件发生变更时自动触发

**所需 Secrets（在仓库 Settings → Secrets 里配置）：**

| Secret 名称 | 说明 |
|-------------|------|
| `TENCENT_SECRET_ID` | 子用户 `github-actions` 的 SecretId |
| `TENCENT_SECRET_KEY` | 子用户 `github-actions` 的 SecretKey |

**子用户权限：** `QcloudSCFFullAccess`（仅云函数调用权限）

---

## 十一、文件更新对照表

| 修改内容 | 需要更新的地方 |
|---------|--------------|
| 新增/删除照片视频 | 只操作广州 COS，自动同步 |
| 修改活动标题/简介/封面 | admin.html 操作，推送 GitHub，自动同步 |
| 修改网站 UI（index.html） | GitHub + 香港 COS 都要上传 |
| 修改管理后台（admin.html） | GitHub + 香港 COS 都要上传 |
| 修改云函数代码 | 腾讯云函数控制台（src/index.js + index.js） |

---

## 十二、管理后台说明

**地址：** `深圳活动群.cn/admin.html`

**登录密码：** `YJ87654321`（会话期间有效，关闭浏览器后需重新登录）

**GitHub Token 连接：**
- 首次使用需粘贴 GitHub Personal Access Token
- Token 存储在浏览器本地（localStorage），换设备需重新输入
- Token 权限：`repo`，永不过期

**功能说明：**
- 展开每个活动可编辑：标题、地点、简介
- 点击照片缩略图可设置封面（再次点击取消）
- 点「保存修改」保存到内存，点「推送到 GitHub」才会更新网站

---

## 十三、已知注意事项

1. **域名备案**：`深圳活动群.cn` 尚未完成 ICP 备案，目前通过香港 COS 绕过。如将来需要国内 CDN 加速，需先完成备案。

2. **GitHub Token 安全**：存储在浏览器 localStorage，不要在公共设备使用管理后台。

3. **云函数代码位置**：实际执行文件是 `src/index.js`，修改后需同步根目录 `index.js`，两个文件保持一致。

4. **浏览器缓存**：网站更新后看不到新内容时，电脑按 `Ctrl+Shift+R`，手机在地址栏加 `?v=数字` 访问。

5. **data.json 冲突**：若 admin 推送报 SHA 冲突错误，刷新 admin 页面重新加载最新数据后再推送。

6. **GitHub Actions 延迟**：admin 推送后约 2 分钟香港 COS 才会更新，属正常现象。

---

## 十四、费用预估

| 服务 | 费用 |
|------|------|
| 广州 COS 存储 | ¥0.099/GB/月 |
| 香港 COS 存储 | ¥0.156/GB/月 |
| 腾讯云函数 | 免费额度内（50万次/月） |
| GitHub Actions | 免费（公开仓库） |
| Vercel | 免费 |
| 域名 `深圳活动群.cn` | ¥33/年 |

---

## 十五、待完善功能

- [ ] ICP 备案（完成后可使用国内 CDN 加速，访问更快）
- [ ] 搜索功能（按关键词搜索活动）
- [ ] 照片数量超多时的懒加载优化

---

*文档生成时间：2026年5月6日*
