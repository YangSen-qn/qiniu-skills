# Qiniu Skills

七牛云操作的 Skills。

## 项目结构

```
qiniu-skills/
└── skills/
    └── qshell/
        ├── SKILL.md          # Skill 定义（触发条件、命令速查、安全规则）
        ├── bin/qshell         # qshell CLI 二进制
        └── references/
            └── install.md     # 安装指南
```

## qshell Skill

通过 `qshell` CLI 操作七牛云 KODO 对象存储资源，覆盖以下场景：

| 功能 | 说明 |
|------|------|
| 账号与 Bucket 管理 | 查看账号、列出/创建 Bucket、查看绑定域名 |
| 文件查询 | stat 单文件信息、listbucket2 批量列举、按前缀/时间/存储类型过滤 |
| 文件上传 | 表单上传 (fput)、分片上传 (rput)、批量上传目录 (qupload2) |
| 文件下载 | 单文件下载 (get)、批量下载 (qdownload2) |
| 文件操作 | 复制、移动、重命名、删除、抓取远程资源，均支持批量 |
| 属性修改 | 修改 MIME 类型、存储类型、过期时间、禁用/启用访问、解冻归档 |
| CDN 操作 | URL 缓存刷新、目录刷新、URL 预取 |
| 数据处理 | 触发持久化处理 (pfop)、查询处理状态 (prefop) |
| 私有链接与签名 | 生成私有访问链接、saveas、批量签名 |
| 工具命令 | 解码 reqid、计算 qetag、Base64 编解码、IP 查询、时间转换 |

## 使用方式

### 1. 克隆项目

```bash
git clone <repo-url> /path/to/qiniu-skills
```

### 2. 配置 Claude Code

将 skill 路径添加到 Claude Code 设置中：

```jsonc
// ~/.claude/settings.json
{
  "skills": [
    "/path/to/qiniu-skills/skills/*"
  ]
}
```

### 3. 配置 qshell 账号

首次使用时，Claude Code 会自动检测 qshell 是否已安装并引导配置。也可手动配置：

```bash
# qshell 二进制已内置在 skills/qshell/bin/ 目录下，无需额外安装
# 配置七牛账号（AccessKey / SecretKey 可在七牛控制台 > 密钥管理中获取）
/path/to/qiniu-skills/skills/qshell/bin/qshell account <AccessKey> <SecretKey> <Name>
```

### 4. 在对话中使用

配置完成后，在 Claude Code 对话中用自然语言描述你的需求即可。

> **提示：** 建议在对话中包含「七牛」「qshell」「kodo」等关键词，以便准确触发此 skill。

**文件查询**
- "查一下七牛 `my-bucket` 里有哪些文件"
- "用 qshell stat 一下 `my-bucket` 的 `path/to/file.jpg`"
- "列出七牛 `my-bucket` 里前缀为 `logs/2024-03` 的文件"

**文件上传 / 下载**
- "把 `./data.csv` 上传到七牛 `my-bucket`"
- "从七牛下载 `my-bucket` 里的 `report.pdf` 到本地"
- "批量上传 `./images/` 目录到七牛 `my-bucket` 的 `img/` 前缀下"

**文件管理**
- "在七牛把 `my-bucket` 的 `old.txt` 重命名为 `new.txt`"
- "七牛上复制 `src-bucket/file.jpg` 到 `dst-bucket`"
- "把七牛这个文件改成低频存储"
