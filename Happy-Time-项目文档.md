# Happy Time · 快乐时光
## 项目结构与操作手册

> 最后更新：2026年5月5日

---

## 一、项目概述

一个用于记录朋友圈聚会、出游活动的私人相册网站，支持照片和视频展示，国内可直接访问。

**网站地址**
- 🌐 国内访问：`深圳活动群.cn`
- 🌐 国外访问：`sunshine-memories.vercel.app`
- ⚙️ 管理后台：`深圳活动群.cn/admin.html`（国内）或 `sunshine-memories.vercel.app/admin.html`（需VPN）

---

## 二、整体架构

```
你的电脑
    ↓ 上传照片/视频
广州 COS（媒体存储）sunshine-1343282399
    ↓ 文件上传触发
腾讯云函数 sync-memories
    ↓ 同时推送
    ├── GitHub 仓库 GitComeOn/sunshine-memories（data.json）
    │       ↓ 自动部署
    │   Vercel（国外访问）
    └── 香港 COS sunshine-web-1343282399（data.json）
            ↓ 域名解析
        深圳活动群.cn（国内访问）
```

---

## 三、各平台账号与资源

| 平台 | 资源名称 | 用途 |
|------|----------|------|
| 腾讯云 COS | `sunshine-1343282399`（广州） | 存储所有照片和视频 |
| 腾讯云 COS | `sunshine-web-1343282399`（香港） | 托管网站文件，国内访问 |
| 腾讯云函数 | `sync-memories`（广州） | 自动同步数据 |
| GitHub | `GitComeOn/sunshine-memories` | 网站代码仓库，触发Vercel部署 |
| Vercel | `sunshine-memories` | 网站托管（国外） |
| 域名 | `深圳活动群.cn` | 国内访问域名，DNS指向香港COS |

---

## 四、文件结构

### GitHub 仓库（GitComeOn/sunshine-memories）
```
sunshine-memories/
├── index.html      # 主展示页（时间线+相册视图）
├── admin.html      # 管理后台（添加活动、推送GitHub）
├── data.json       # 活动数据文件（自动生成）
└── README.md
```

### 广州 COS（媒体存储）
```
sunshine-1343282399/
├── 2026-02-23_聚会_春节同学相聚_广东深圳/
│   ├── 微信图片_001.jpg
│   ├── 微信图片_002.jpg
│   └── 视频.mp4
├── 2026-05-02_出游_粤赣三日自驾_广东韶关·江西崇义/
│   ├── 微信图片_001.jpg
│   └── ...
└── ...
```

### 香港 COS（网站文件）
```
sunshine-web-1343282399/
├── index.html      # 与GitHub同步
├── admin.html      # 与GitHub同步
└── data.json       # 由云函数自动更新
```

### 腾讯云函数
```
sync-memories/
├── src/
│   ├── index.js        # 主函数代码（v2，实际执行文件）
│   ├── package.json
│   └── node_modules/
├── index.js            # 与src/index.js保持一致
├── package.json
└── package-lock.json
```

> ⚠️ **重要**：云函数实际执行的是 `src/index.js`，修改代码时两个文件都要同步更新。

---

## 五、COS 文件夹命名规则

上传活动照片时，**文件夹命名必须严格遵守以下格式**，云函数才能正确解析：

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

## 六、媒体文件命名建议

云函数按**文件名字母顺序**排列，视频自动排在所有照片后面（代码层面已处理）。

**照片**：保持原始文件名即可，无需改名。

**视频**：无需特殊处理，云函数自动识别视频后缀并排到最后。

**支持的格式：**
- 图片：`.jpg` `.jpeg` `.png` `.gif` `.webp` `.heic` `.heif`
- 视频：`.mp4` `.mov` `.avi` `.mkv` `.m4v`

---

## 七、日常操作流程

### 新增活动（完整流程）

```
1. 在广州 COS 创建文件夹
   命名格式：2026-06-01_聚餐_六一聚餐_广州天河

2. 把照片/视频上传到该文件夹
   ↓ 自动触发云函数（约1-2分钟）
   ↓ 自动更新 GitHub 和香港 COS 的 data.json
   ↓ 网站自动更新

3. （可选）打开管理后台补充活动简介
   地址：深圳活动群.cn/admin.html
   - 连接 GitHub Token
   - 找到对应活动，在 JSON 编辑器里修改 desc 字段
   - 点「推送到 GitHub」保存
```

### 手动触发同步

如果上传照片后网站没有自动更新，可以手动触发：

腾讯云控制台 → 云函数 → `sync-memories` → 函数代码 → 点「测试」

---

## 八、云函数环境变量

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
| `TZ` | `Asia/Shanghai` | 时区 |

---

## 九、管理后台使用说明

**地址：** `深圳活动群.cn/admin.html`

**首次使用需连接 GitHub：**
1. 打开后台，顶部有「连接 GitHub」输入框
2. 粘贴 GitHub Personal Access Token
3. Token 存储在浏览器本地，换设备需重新输入

**Token 权限要求：** 勾选 `repo` 权限，设置为永不过期

**后台主要功能：**
- 查看所有活动列表
- 为活动添加/修改简介（desc 字段）
- 直接编辑 JSON 数据
- 一键推送到 GitHub

> ⚠️ **注意**：后台修改的媒体顺序会被云函数下次同步覆盖，媒体顺序由文件名决定，视频由代码自动排最后。

---

## 十、data.json 数据结构

```json
{
  "events": [
    {
      "id": "cos_2026_05_02_出游_粤赣三日自驾_广东韶关_江西崇义",
      "title": "粤赣三日自驾",
      "date": "2026-05-02",
      "type": "出游",
      "location": "广东韶关·江西崇义",
      "desc": "深圳出发，途经珠玑古巷、梅关古道、丹霞山。",
      "cover": "",
      "cosFolder": "2026-05-02_出游_粤赣三日自驾_广东韶关·江西崇义",
      "media": [
        {
          "type": "photo",
          "url": "https://sunshine-1343282399.cos.ap-guangzhou.myqcloud.com/...",
          "thumb": "https://sunshine-1343282399.cos.ap-guangzhou.myqcloud.com/...",
          "caption": ""
        }
      ]
    }
  ]
}
```

**云函数保留的手工字段：** `title`、`desc`、`cover`（其余字段每次同步自动生成）

---

## 十一、已知注意事项

1. **域名备案**：`深圳活动群.cn` 尚未完成 ICP 备案，目前通过香港 COS 绕过备案要求。如需将媒体存储也迁移到香港，需重新规划。

2. **GitHub Token 安全**：Token 存储在浏览器 localStorage，不要在公共设备上使用管理后台，使用后可在浏览器设置中清除。

3. **云函数代码位置**：实际执行文件是 `src/index.js`，修改时需同步更新根目录 `index.js` 和 `src/index.js`，然后在在线编辑器部署。

4. **浏览器缓存**：如果网站更新后看不到新内容，电脑按 `Ctrl+Shift+R` 强制刷新，手机在地址栏加 `?v=数字` 参数访问。

5. **COS 触发器**：已配置广州 COS 的全部创建事件触发云函数，上传照片后约 1-2 分钟网站自动更新。

6. **费用预估**：
   - 广州 COS 存储：约 ¥0.1/GB/月
   - 香港 COS 存储：略高于广州
   - 云函数：免费额度内（50万次/月）
   - Vercel：免费
   - 域名：¥33/年

---

## 十二、待完善功能

- [ ] 活动封面手动指定功能（在后台选择某张照片作为封面）
- [ ] 活动简介编辑更友好的 UI（目前需要编辑 JSON）
- [ ] 域名 ICP 备案（完成后可使用国内 CDN 加速）
- [ ] 网站访问统计

---

*文档生成时间：2026年5月5日*
