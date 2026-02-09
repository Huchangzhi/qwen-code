# Qwen Code 打包指南

本文档介绍了将 Qwen Code 项目打包成可执行文件的不同方法。

## 方法一：使用 nexe（推荐）

nexe 是一个现代的 Node.js 应用打包工具，支持最新的 Node.js 版本。

### 本地构建

```bash
# 安装 nexe
npm install -g nexe

# 构建项目
npm run build

# 打包成可执行文件（Linux，需要编译）
nexe ./dist/cli.js --target linux-x64-20.20.0 --output ./dist/qwen-linux --resource "./dist/**/*" --resource "./packages/core/dist/**/*" --resource "./packages/cli/dist/**/*" --build

# 打包成可执行文件（Windows，需要编译）
nexe ./dist/cli.js --target windows-x64-20.20.0 --output ./dist/qwen.exe --resource "./dist/**/*" --resource "./packages/core/dist/**/*" --resource "./packages/cli/dist/**/*" --build
```

## 方法二：使用 @yao-pkg/pkg

@yao-pkg/pkg 是 pkg 的一个分支，支持 Node.js 20+。

### 本地构建

```bash
# 安装 @yao-pkg/pkg
npm install -g @yao-pkg/pkg

# 构建项目
npm run build

# 创建简单的入口点
echo '#!/usr/bin/env node' > standalone-entry.cjs
echo 'require("./dist/cli.js").run().catch(console.error);' >> standalone-entry.cjs

# 打包成可执行文件
pkg standalone-entry.cjs --target node20-linux-x64 --output ./dist/qwen-pkg-linux
```

## 方法三：CI/CD 构建

我们提供了两个 GitHub Actions 工作流文件：

1. `.github/workflows/build-executable.yml` - 使用 nexe 构建
2. `.github/workflows/build-executable-pkg.yml` - 使用 pkg 构建

这些工作流会在每次推送时自动构建可执行文件。

## 方法四：Docker 容器化

我们还提供了 Dockerfile 用于容器化部署：

```bash
# 构建 Docker 镜像
docker build -f Dockerfile.standalone -t qwen-code .

# 运行容器
docker run -it qwen-code --help
```

## 注意事项

1. **nexe 构建时间**：如果使用 `--build` 标志，nexe 需要从源代码编译 Node.js，这可能需要 15-45 分钟。建议在 CI 环境中使用此标志以确保支持最新的 Node.js 版本。

2. **pkg 兼容性**：对于复杂的 ES 模块项目，pkg 可能会出现模块解析问题。在这种情况下，建议使用 nexe 或 Docker 方案。

3. **文件大小**：生成的可执行文件通常比较大（50-100MB+），因为它们包含了整个 Node.js 运行时。

4. **平台兼容性**：在 Linux 上构建的可执行文件只能在 Linux 上运行，在 Windows 上构建的只能在 Windows 上运行。

## 推荐方案

对于生产环境，我们推荐以下顺序：

1. **nexe** - 如果需要单文件分发
2. **Docker** - 如果可以使用容器化部署
3. **pkg** - 作为备选方案
