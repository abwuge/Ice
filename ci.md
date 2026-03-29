# BUILD CI 说明

1. 编译
2. 签名
3. 上传构建产物

如果你不想在本地编译，可以直接使用 CI 生成的 Artifact（zip）。

## 当前工作流

工作流文件：`.github/workflows/build.yml`

触发条件：

1. push 到 `main`，且改动命中 `Ice/**`、`Ice.xcodeproj/**` 或 workflow 本身
2. pull_request，且改动命中同样路径
3. 网页触发

执行步骤：

1. checkout 代码
2. 恢复 SwiftPM 缓存
3. `xcodebuild` 编译（`CODE_SIGNING_ALLOWED=NO`）
4. 如果配置了签名 secrets，则执行签名与验签
5. 打包 `Ice.app` 为 zip
6. 上传 Artifact

## Secrets 配置

如果你只想自动编译，不需要配置任何 secrets。

如果你希望 CI 同时完成签名，需要配置以下 3 个仓库 secrets：

1. `BUILD_CERTIFICATE_BASE64`
2. `P12_PASSWORD`
3. `SIGNING_IDENTITY`

### 1) 生成 SIGNING_IDENTITY

在 macOS 终端执行：

```bash
security find-identity -v -p codesigning
```

复制目标身份的完整文本（例如 `Apple Development: ... (...)` 或 `Developer ID Application: ... (...)`）。

### 2) 生成 p12 并得到 BUILD_CERTIFICATE_BASE64

建议用 Keychain Access 导出证书（必须包含私钥）为 `.p12`，设置导出密码。

然后在终端执行：

```bash
base64 -i signing_certificate.p12 | tr -d '\n'
```

输出的一整行就是 `BUILD_CERTIFICATE_BASE64`。

导出 p12 时设置的密码就是 `P12_PASSWORD`。

### 3) 在 GitHub 中添加 secrets

仓库页面：

`Settings -> Secrets and variables -> Actions -> New repository secret`

按名称逐项创建：

1. `BUILD_CERTIFICATE_BASE64`
2. `P12_PASSWORD`
3. `SIGNING_IDENTITY`

## 生成证书

如果需要自己签名：

1. Xcode
	- 安装 Xcode，登录 Apple ID
	- 让 Xcode 自动生成/下载证书
	- 优点是最省事，缺点是 Xcode 体积较大

2. 手动生成私钥与证书
	- 本地生成私钥与 CSR
	- 到 Apple 开发者后台申请证书并下载
	- 导入钥匙串后再导出 p12 给 CI 使用

3. Xcode Command Line Tools
	- 可安装体积更小的命令行工具用于基础开发命令
	- 但“证书自动管理能力”不如完整 Xcode 方便
