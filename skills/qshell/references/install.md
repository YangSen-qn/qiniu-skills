# qshell 下载与安装指南

qshell 二进制保存在本 skill 的 `bin/` 目录下（即 `<skill_base_dir>/bin/qshell`）。

## 安装步骤

### 1. 获取最新版本号

从七牛官方文档页面 https://developer.qiniu.com/kodo/1302/qshell 抓取最新版本号：
```bash
VERSION=$(curl -sL -e https://developer.qiniu.com "https://developer.qiniu.com/kodo/1302/qshell" | grep -oE 'qshell-v[0-9]+\.[0-9]+\.[0-9]+' | head -1 | sed 's/^qshell-v//')
echo "最新版本: v${VERSION}"
```

### 2. 检测平台和架构，下载二进制

下载地址：`https://kodo-toolbox-new.qiniu.com/qshell-v${VERSION}-${SUFFIX}.tar.gz`（必须带 Referer `-e https://developer.qiniu.com`）

```bash
SKILL_BIN_DIR="<skill_base_dir>/bin"
mkdir -p "$SKILL_BIN_DIR"

OS=$(uname -s | tr '[:upper:]' '[:lower:]')
ARCH=$(uname -m)

if [ "$OS" = "darwin" ]; then
  if [ "$ARCH" = "arm64" ]; then
    SUFFIX="darwin-arm64"
  else
    SUFFIX="darwin-amd64"
  fi
elif [ "$OS" = "linux" ]; then
  if [ "$ARCH" = "x86_64" ]; then
    SUFFIX="linux-amd64"
  elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
    SUFFIX="linux-arm64"
  elif [ "$ARCH" = "i386" ] || [ "$ARCH" = "i686" ]; then
    SUFFIX="linux-386"
  fi
fi

URL="https://kodo-toolbox-new.qiniu.com/qshell-v${VERSION}-${SUFFIX}.tar.gz"
curl -sL -e https://developer.qiniu.com -o /tmp/qshell.tar.gz "$URL"
```

### 3. 解压并安装到 skill 的 bin/ 目录

```bash
tar -xzf /tmp/qshell.tar.gz -C /tmp/
chmod +x /tmp/qshell
mv /tmp/qshell "$SKILL_BIN_DIR/qshell"
rm -f /tmp/qshell.tar.gz
```

### 4. 验证安装

```bash
"$SKILL_BIN_DIR/qshell" version
```

## 配置账号

安装完成后配置七牛账号：
```bash
"$SKILL_BIN_DIR/qshell" account <AccessKey> <SecretKey> <Name>
```

- `AccessKey` 和 `SecretKey`：从七牛控制台获取（https://portal.qiniu.com/user/key）
- `Name`：自定义名称，用于本地区分多个账号

### 账号管理

```bash
# 查看当前账号
"$SKILL_BIN_DIR/qshell" account

# 列出所有已配置账号
"$SKILL_BIN_DIR/qshell" user ls

# 切换账号
"$SKILL_BIN_DIR/qshell" user cu <Name>
```

## 更新 qshell

重新执行上述安装步骤即可覆盖更新，会自动从官方文档获取最新版本。
