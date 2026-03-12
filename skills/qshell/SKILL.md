---
name: qshell
description: |
  使用 qshell CLI 操作七牛云 KODO 对象存储资源。支持文件查询、上传、下载、复制、移动、
  删除、属性修改、CDN 刷新/预取、数据处理等全部存储操作。
  当用户想操作七牛存储、管理 bucket/文件、上传下载、CDN 操作、查看文件信息时使用此 skill。
  触发短语包括："查一下这个 bucket"、"列一下文件"、"上传文件到七牛"、"下载七牛文件"、
  "刷新 CDN"、"看看文件信息"、"stat 一下"、"qshell"、"七牛存储"、"kodo"、
  "bucket 里有什么"、"批量删除"、"改一下文件类型"、"生成私有链接"、"解冻归档文件"、
  "解码 reqid"、"算一下 qetag"。
  当用户提到 qshell 命令、七牛 bucket 名称、或任何对象存储操作时也应触发。
  当用户说「安装 qshell」、「下载 qshell」、「配置 qshell」、「qshell 怎么装」时也应触发。
---

# 七牛 KODO 资源操作 (qshell)

通过 `qshell` CLI 操作七牛云 KODO 对象存储资源。

## 前置条件

配置文件：`~/.qshell.json`

### 使用策略

按以下优先级查找 qshell，**不要提前检查是否安装**，直接用 `qshell` 执行命令：

1. **系统 PATH 中的 `qshell`**：直接执行 `qshell <子命令>`
2. **skill 目录下的 `<skill_base_dir>/bin/qshell`**：如果第 1 步返回 command not found（exit code 127），改用完整路径重试
3. **自动下载安装**：如果第 2 步也返回 command not found，阅读 `references/install.md` 下载安装到 `<skill_base_dir>/bin/`，然后重新执行

一旦确定了可用路径，后续命令直接使用该路径，无需重复探测。

### 账号未配置时

如果命令返回 `bad token` / `unauthorized` / `401` 错误，提示用户运行：
```bash
qshell account <AccessKey> <SecretKey> <Name>
```

---

## 命令速查

### 1. 账号与 Bucket 管理

```bash
# 查看当前账号
qshell account

# 列出所有 bucket
qshell buckets
qshell buckets --detail              # 带详情
qshell buckets --region z0           # 指定区域

# 查看 bucket 信息
qshell bucket <Bucket>

# 创建 bucket
qshell mkbucket <Bucket> --region z0          # 华东
qshell mkbucket <Bucket> --region z1          # 华北
qshell mkbucket <Bucket> --region z2          # 华南
qshell mkbucket <Bucket> --region na0         # 北美
qshell mkbucket <Bucket> --region as0         # 东南亚
qshell mkbucket <Bucket> --region z0 --private  # 创建私有 bucket

# 查看 bucket 绑定的域名
qshell domains <Bucket>
qshell domains <Bucket> --detail
```

### 2. 文件查询

```bash
# 查看单个文件信息
qshell stat <Bucket> <Key>

# 列举 bucket 中的文件
qshell listbucket2 <Bucket>                                # 列出所有文件
qshell listbucket2 <Bucket> -p <Prefix>                    # 按前缀列举
qshell listbucket2 <Bucket> -p <Prefix> --limit 100        # 限制数量
qshell listbucket2 <Bucket> -o result.txt                  # 输出到文件
qshell listbucket2 <Bucket> -r                              # 人类可读的文件大小
qshell listbucket2 <Bucket> -s 2024-01-01-00-00-00         # 起始时间
qshell listbucket2 <Bucket> -e 2024-12-31-23-59-59         # 结束时间
qshell listbucket2 <Bucket> --file-types 0,1               # 按存储类型过滤
qshell listbucket2 <Bucket> --min-file-size 1048576         # 最小文件大小 (1MB)
qshell listbucket2 <Bucket> --show-fields Key,FileSize,PutTime  # 指定显示字段

# 批量查看文件信息（从文件或 stdin 读取 key 列表）
qshell batchstat <Bucket> -i <KeyListFile>
```

**listbucket2 输出格式（默认 tab 分隔）：**
```
Key    FileSize    Hash    PutTime    MimeType    FileType    EndUser
```

**存储类型 FileType：**
- 0: 标准存储 (STANDARD)
- 1: 低频存储 (IA)
- 2: 归档存储 (ARCHIVE)
- 3: 深度归档 (DEEP_ARCHIVE)
- 4: 归档直读 (ARCHIVE_IR)
- 5: 智能分层 (INTELLIGENT_TIERING)

### 3. 文件上传

```bash
# 表单上传（适合小文件，< 几百 MB）
qshell fput <Bucket> <Key> <LocalFile>
qshell fput <Bucket> <Key> <LocalFile> --overwrite          # 覆盖同名
qshell fput <Bucket> <Key> <LocalFile> --file-type 1        # 上传为低频存储
qshell fput <Bucket> <Key> <LocalFile> -t image/png         # 指定 MIME 类型

# 分片上传（适合大文件）
qshell rput <Bucket> <Key> <LocalFile>
qshell rput <Bucket> <Key> <LocalFile> --overwrite
qshell rput <Bucket> <Key> <LocalFile> --resumable-api-v2   # 使用 v2 分片
qshell rput <Bucket> <Key> <LocalFile> -c 10                # 10 个并发分片

# 批量上传目录
qshell qupload2 --src-dir <LocalDir> --bucket <Bucket>
qshell qupload2 --src-dir <LocalDir> --bucket <Bucket> --key-prefix "dir/" --overwrite
qshell qupload2 --src-dir <LocalDir> --bucket <Bucket> --thread-count 10
qshell qupload2 --src-dir <LocalDir> --bucket <Bucket> --skip-suffixes .tmp,.log
qshell qupload2 --src-dir <LocalDir> --bucket <Bucket> --check-exists  # 跳过已存在
```

### 4. 文件下载

```bash
# 下载单个文件
qshell get <Bucket> <Key>
qshell get <Bucket> <Key> -o <LocalFile>                    # 指定保存路径
qshell get <Bucket> <Key> --domain <Domain>                 # 指定下载域名
qshell get <Bucket> <Key> --check-hash                      # 下载后校验 hash
qshell get <Bucket> <Key> --enable-slice                    # 分片下载大文件

# 批量下载
qshell qdownload2 --bucket <Bucket> --dest-dir <LocalDir>
qshell qdownload2 --bucket <Bucket> --dest-dir <LocalDir> --prefix "logs/"
qshell qdownload2 --bucket <Bucket> --dest-dir <LocalDir> -c 10   # 10 并发
qshell qdownload2 --bucket <Bucket> --dest-dir <LocalDir> --domain <Domain>
qshell qdownload2 --bucket <Bucket> --dest-dir <LocalDir> --key-file <KeyFile>
```

### 5. 文件操作

```bash
# 复制文件
qshell copy <SrcBucket> <SrcKey> <DestBucket> -k <DestKey>
qshell copy <SrcBucket> <SrcKey> <DestBucket>               # key 不变
qshell copy <SrcBucket> <SrcKey> <DestBucket> -k <DestKey> --overwrite

# 移动/重命名文件
qshell move <SrcBucket> <SrcKey> <DestBucket> -k <DestKey>
qshell move <Bucket> <OldKey> <Bucket> -k <NewKey>          # 同 bucket 重命名

# 删除文件
qshell delete <Bucket> <Key>

# 抓取远程资源到 bucket
qshell fetch <RemoteURL> <Bucket> -k <Key>

# 批量操作（从文件或 stdin 读取列表）
qshell batchcopy <SrcBucket> <DestBucket> -i <MapFile>       # 每行: SrcKey\tDestKey
qshell batchmove <SrcBucket> <DestBucket> -i <MapFile>
qshell batchdelete <Bucket> -i <KeyListFile>                  # 每行一个 key
qshell batchrename <Bucket> -i <OldNewKeyMapFile>
qshell batchfetch <Bucket> -i <URLKeyMapFile>                 # 每行: URL\tKey
```

> **安全提示：** 批量删除/移动操作需加 `-y` 强制执行。执行前务必确认操作范围。
> 建议先用 `--success-list` 和 `--failure-list` 记录结果。

### 6. 文件属性修改

```bash
# 修改 MIME 类型
qshell chgm <Bucket> <Key> <NewMimeType>

# 修改存储类型
qshell chtype <Bucket> <Key> <FileType>   # 0/1/2/3/4/5

# 设置文件过期时间（天数，0 表示取消过期）
qshell expire <Bucket> <Key> <DeleteAfterDays>

# 禁用/启用文件访问
qshell forbidden <Bucket> <Key>            # 禁用
qshell forbidden <Bucket> <Key> -r         # 取消禁用

# 解冻归档文件（FreezeAfterDays: 1~7 天）
qshell restorear <Bucket> <Key> <FreezeAfterDays>

# 批量操作
qshell batchchgm <Bucket> -i <KeyMimeFile>        # 每行: Key\tMimeType
qshell batchchtype <Bucket> -i <KeyTypeFile>       # 每行: Key\tFileType
qshell batchexpire <Bucket> -i <KeyDaysFile>       # 每行: Key\tDays
qshell batchforbidden <Bucket> -i <KeyListFile>
qshell batchrestorear <Bucket> -i <KeyDaysFile>    # 每行: Key\tFreezeAfterDays
```

### 7. CDN 操作

```bash
# 刷新 URL 缓存（从文件读取 URL 列表，每行一个 URL）
qshell cdnrefresh -i <URLListFile>
qshell cdnrefresh -i <URLListFile> -r        # 刷新目录

# 预取 URL（从文件读取 URL 列表）
qshell cdnprefetch -i <URLListFile>

# 也可以从 stdin 输入
echo "http://example.com/path/file.jpg" | qshell cdnrefresh
echo "http://example.com/path/file.jpg" | qshell cdnprefetch
```

### 8. 数据处理 (pfop)

```bash
# 触发持久化数据处理
qshell pfop <Bucket> <Key> <FopCommand>
qshell pfop <Bucket> <Key> <FopCommand> -p <Pipeline>       # 指定队列
qshell pfop <Bucket> <Key> <FopCommand> -u <NotifyURL>       # 通知 URL

# 查询处理状态
qshell prefop <PersistentId>
```

### 9. 私有链接与签名

```bash
# 生成私有资源访问链接
qshell privateurl <PublicURL>                    # 默认有效期
qshell privateurl <PublicURL> <Deadline>         # 指定 Unix 时间戳

# 生成带数据处理 + saveas 的 URL
qshell saveas <PublicUrlWithFop> <SaveBucket> <SaveKey>

# 批量签名（从文件读取 URL 列表）
qshell batchsign -i <URLListFile>
```

### 10. 工具命令

```bash
# 解码七牛 reqid
qshell reqid <ReqId>

# 计算本地文件的 qetag
qshell qetag <LocalFilePath>

# 校验本地文件与云端文件是否一致
qshell match <Bucket> <Key> <LocalFile>

# Base64 编解码
qshell b64encode <String>
qshell b64decode <EncodedString>

# RPC 编解码
qshell rpcencode <String>
qshell rpcdecode <EncodedString>

# 时间工具
qshell d2ts <Seconds>                  # N 秒后的时间戳
qshell tms2d <TimestampInMs>           # 毫秒时间戳转日期

# IP 查询
qshell ip <IPAddress>

# 目录缓存（用于批量上传前生成文件列表）
qshell dircache <DirPath> -o <OutputFile>
```

### 11. 异步抓取

```bash
# 异步批量抓取远程资源
qshell abfetch <Bucket> -i <URLKeyFile>

# 查询异步抓取状态
qshell acheck <Bucket> <Id> --zone <Zone>

# 从 AWS S3 抓取
qshell awslist <AwsBucket> --access-key <AK> --secret-key <SK> --region <Region>
qshell awsfetch <AwsBucket> <QiniuBucket> --access-key <AK> --secret-key <SK>
```

---

## 用户意图到操作的映射

| 用户说 | 操作 |
|--------|------|
| "列一下 bucket 里的文件" | `qshell listbucket2 <Bucket>` |
| "看看这个文件信息" | `qshell stat <Bucket> <Key>` |
| "上传文件到七牛" | `qshell fput` / `qshell rput`（大文件） |
| "下载这个文件" | `qshell get <Bucket> <Key>` |
| "批量下载" | `qshell qdownload2` |
| "复制文件到另一个 bucket" | `qshell copy` |
| "删除这个文件" | `qshell delete` |
| "改一下存储类型" | `qshell chtype` |
| "刷新 CDN" | `qshell cdnrefresh` |
| "预热 CDN" | `qshell cdnprefetch` |
| "生成私有链接" | `qshell privateurl` |
| "解冻归档文件" | `qshell restorear` |
| "解码 reqid" | `qshell reqid` |
| "算一下 qetag" | `qshell qetag` |
| "有哪些 bucket" | `qshell buckets` |
| "创建 bucket" | `qshell mkbucket` |
| "这个文件的域名是什么" | `qshell domains <Bucket>` |
| "批量删除" | `qshell batchdelete`（需确认后加 `-y`） |
| "转码这个视频" | `qshell pfop` |
| "查一下处理进度" | `qshell prefop` |

---

## 安全规则

### 危险操作（必须与用户确认后再执行）

- `qshell delete` / `qshell batchdelete` — 删除文件不可恢复
- `qshell move` / `qshell batchmove` — 源文件会被删除
- `qshell forbidden` — 禁用文件访问
- `qshell batchexpire` — 批量设置过期可能导致数据丢失
- 任何带 `--overwrite` 的上传/复制操作

### 安全操作（可直接执行）

- 所有查询类：`stat`、`listbucket2`、`buckets`、`bucket`、`domains`、`prefop`
- 工具类：`reqid`、`qetag`、`match`、`b64encode`、`b64decode`、`ip`
- `privateurl`、`saveas`、`batchsign` — 只是生成 URL

---

## 输出格式

### 文件信息 (stat)

```
## 文件信息: <Bucket>/<Key>

| 属性 | 值 |
|------|-----|
| Key | path/to/file.jpg |
| 大小 | 1.5 MB |
| Hash | Fh8... |
| MIME | image/jpeg |
| 存储类型 | 标准存储 |
| 上传时间 | 2024-03-01 10:30:00 |
```

### 文件列表 (listbucket2)

```
## <Bucket> 文件列表 (前缀: <Prefix>)

| Key | 大小 | MIME | 存储类型 | 上传时间 |
|-----|------|------|----------|----------|
| file1.jpg | 1.5 MB | image/jpeg | 标准 | 2024-03-01 |
| file2.pdf | 3.2 MB | application/pdf | 低频 | 2024-02-15 |

共 N 个文件，总大小 X MB
```

---

## 错误处理

| 错误 | 处理方式 |
|------|----------|
| `no such file or directory` | 检查本地文件路径 |
| `no such bucket` | 检查 bucket 名称，用 `qshell buckets` 列出可用 bucket |
| `no such file or key` | 检查文件 key 是否正确 |
| `bad token` / `unauthorized` | 账号配置问题，提示 `qshell account` 重新配置 |
| `bucket not match` | 操作的 bucket 与 key 不匹配 |
| `file exists` | 文件已存在，需要加 `--overwrite` 选项 |
| `614` | 文件已存在（同上） |
| `612` | 文件不存在 |
| `631` | Bucket 不存在 |
| `401` | 鉴权失败，检查 AK/SK |
| `403` | 权限不足 |
