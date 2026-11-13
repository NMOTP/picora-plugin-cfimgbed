# CloudFlare ImgBed · Picora 插件

![插件图标](assets/icon.svg)

**让 Picora 一键接入你自建的 CloudFlare ImgBed 图床**

![作者 NMOTP](https://img.shields.io/badge/作者-NMOTP-8A2BE2?style=flat-square)
![版本 1.1.0](https://img.shields.io/badge/版本-1.1.0-F6821F?style=flat-square)
![协议 http-v2](https://img.shields.io/badge/协议-http--v2-F6821F?style=flat-square)
![Picora 1.0.7+](https://img.shields.io/badge/Picora-%E2%89%A5%201.0.7-blue?style=flat-square)
![许可 MIT](https://img.shields.io/badge/许可-MIT-green?style=flat-square)

为 [Picora](https://github.com/Alnitak44/Picora) 桥接 [MarSeventh/CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed) —— 无论是部署在 Cloudflare Pages / Workers 还是 Docker，只要图床在线，写作时就能随手传图、立刻拿到链接。

---

## ✨ 插件特色

| | |
| :---: | :--- |
| 🎫 | **双鉴权自由选** —— API Token（推荐）或上传密码 authCode，留空的请求头不会发送，两种都不开也能用 |
| 📁 | **上传目录随心放** —— 指定 `uploadFolder`，站点侧会做路径安全清洗，分类整理图片不打架 |
| 🔗 | **链接绝对、直出直用** —— `returnFormat=full` 返回完整 URL，不受反向代理改写影响 |
| 🔒 | **权限最小化** —— 网络权限只绑定你填写的图床地址，不申请通配主机、不枚举本地文件 |

---

## 🚀 三步上手

> **1 · 装插件**
> 在 Picora「插件中心 → 模块仓库」搜索「CloudFlare ImgBed」安装；
> 或下载 [Release](releases) 中的 `cfimgbed-<版本>.picora-plugin.zip`，从「安装插件」手动导入。

> **2 · 备站点**
> 确保你的 CloudFlare ImgBed 已部署、后台配置了至少一个存储渠道（Telegram / R2 / S3 / WebDAV / Hugging Face 等），并选定一种鉴权方式（见下方）。

> **3 · 填配置**
> 打开「仓库 → 添加配置」，填入图床地址与鉴权信息，完成。同一站点可建多组配置（不同 Token、不同上传目录）。

### 🔑 鉴权模式

| | 方式 | 去哪里拿 | 备注 |
| :---: | :--- | :--- | :--- |
| ⭐ | **API Token** | 后台 → 安全设置 → **API Token 管理** → 创建新 API Token | 权限勾选「上传」；Token 只显示一次 |
| 🔑 | **上传密码 authCode** | 后台 → 安全设置 → 用户端认证 → **上传密码** | 简单场景常用 |
| 🌐 | **不鉴权** | 上述两处都未设置 | 插件里两项留空即可 |

---

## ⚙️ 可配置项

| 配置项 | 填什么 |
| :--- | :--- |
| **图床地址** | 站点根地址，如 `https://img.example.com`；不要带 `/upload` 或查询参数 |
| **API Token（推荐）** | 粘贴完整 Token，带不带 `Bearer ` 前缀都能识别；不用时留空 |
| **上传密码 authCode（可选）** | 用 Token 时留空；只开了上传密码的站点填这里 |
| **上传目录（可选）** | 如 `blog/2026`，留空为根目录 |

---

## 📤 上传过程

- **请求**：`POST {图床地址}/upload?returnFormat=full`，multipart，文件字段 `file`
- **鉴权头**：`Authorization: {API Token}` 与 `authCode: {上传密码}`，留空不发送
- **判定成功**：HTTP 200 且响应数组第一项带 `src`；失败时读取 `error` / `code`

成功响应形如：

```json
[
  {
    "src": "https://img.example.com/file/2026/abc123.png"
  }
]
```

若站点配置了「公开访问链接」（`urlPrefix`），响应里的 `src` 仍是站点自身的 `/file/…` 地址，同样可直接访问。

---

## ☁️ 不支持云端删除

> 💡 上游提供 `DELETE /api/manage/delete/<fileId>`，但上传接口只返回图片链接（`/file/<fileId>`），Picora 插件模板无法从链接截取 `fileId`，因此**本插件不提供云端删除**。
> 需要删图时，请到站点后台的文件管理中操作。

---

## ❓ 常见问题

**❌ 401 / 鉴权失败**

检查 API Token 是否勾选「上传」权限、是否已过期；或上传密码（authCode）是否填对。两者都填时 **Token 优先**。

**🔍 404 找不到 /upload**

多半是图床地址填错了 —— 多了 `/upload`、少了子路径，或者 Cloudflare 侧部署没有成功。

**🚫 400 文件格式不允许**

站点限制了允许的扩展名，到后台「上传设置」里调整白名单。

**🐘 413 / 图片过大**

站点或存储渠道有大小限制；Telegram 渠道建议单张 ≤ 20 MB。

**💥 500 / 502**

多为存储渠道（Telegram Bot、R2、S3）配置或网络问题，查看站点日志定位。

**🔗 链接打不开**

检查站点「文件访问」配置，以及自定义域名 / CDN 前缀是否正确。

> 失败详情可在「设置 → 诊断日志」中查看；日志不会记录 Token 与密码原文。

---

## 🛡️ 权限与安全

- 网络权限只绑定「图床地址」配置项的协议、主机与端口
- 允许明文 HTTP，便于内网 / Docker 自建环境；公网环境建议 HTTPS
- 仅读取用户主动选择的上传文件，不枚举本地文件、不执行脚本
- API Token 建议只勾选「上传」权限，不要用管理员账号密码代替

---

## 🛠️ 文件结构

```text
.
├── plugin.json       # 插件元信息
├── uploader.json     # 程序入口：配置字段、权限、上传请求
├── readme.md         # 本文档（同时打进插件包）
├── LICENSE
└── assets/icon.svg   # 图标
```

插件协议详见 [Picora 插件开发文档](https://github.com/Alnitak44/Picora/blob/main/docs/PLUGINS.md)。

---

## 🙏 致谢 & 许可

图床程序：[MarSeventh/CloudFlare-ImgBed](https://github.com/MarSeventh/CloudFlare-ImgBed)（MIT）

本仓库：[NMOTP/picora-plugin-cfimgbed](https://github.com/NMOTP/picora-plugin-cfimgbed) · 代码以 [MIT](LICENSE) 许可发布 · 由 **NMOTP** 维护

