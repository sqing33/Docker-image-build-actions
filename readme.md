# 🚀 多架构 Docker 镜像构建工作流

这是一个通用的、可手动触发的 GitHub Actions 工作流，旨在简化构建支持多种 CPU 架构的 Docker 镜像并将其推送到多个容器镜像仓库的过程。

通过此工作流，您无需在本地安装复杂的构建环境，只需在 GitHub 页面上填写几个参数，即可为您的项目自动构建并发布镜像。

## ✨ 主要功能

*   **手动触发**：在 GitHub Actions 页面通过一个简单的表单手动启动构建任务。
*   **支持任意代码库**：可以克隆任何公开的 Git 仓库来获取 `Dockerfile` 和源代码进行构建。
*   **灵活的架构选择**：自由定义要构建的 CPU 架构，例如 `amd64`, `arm64`, `arm/v7` 等，默认为最常用的组合。
*   **自定义镜像标签**：可以同时为镜像打上多个标签，例如 `latest`,`v1.0`。
*   **推送至多仓库**：一次构建，自动将镜像推送到 GitHub Container Registry (`ghcr.io`) 和 Docker Hub 两个主流仓库。
*   **安全凭证管理**：通过 GitHub 的加密 Secrets 来安全地管理您的 Docker Hub 凭证，避免硬编码风险。

## 🍴 如何使用 

由于此工作流需要使用您的个人 Docker Hub 凭证，最安全、最推荐的使用方式是 **Fork 本项目**到您自己的 GitHub 账户下，然后在您自己的仓库中进行操作。

### 步骤 1: Fork 本项目

:fork_and_knife: 点击本项目页面右上角的 **"Fork"** 按钮，将此仓库复制一份到您自己的账户下。后续所有操作都将在您 Fork 后的仓库中进行。

### 步骤 2: 设置仓库 Secrets

:key: 为了让 Actions 能够登录并推送到您的 Docker Hub，您需要设置以下两个加密的 Secrets。

1.  在您 Fork 后的仓库页面，点击 **"Settings"** -> **"Secrets and variables"** -> **"Actions"**。
2.  点击 **"New repository secret"** 按钮，创建以下两个 Secrets：

    *   **`DOCKERHUB_USERNAME`**
        *   **值**: 您的 Docker Hub 用户名。

    *   **`DOCKERHUB_TOKEN`**
        *   **值**: 您的 Docker Hub 访问令牌 (Access Token)。**强烈建议使用 Access Token 而非账户密码**。
        *   *如何获取 Access Token？* 登录 [Docker Hub](https://hub.docker.com/)，进入 **"Account Settings"** -> **"Security"** -> **"New Access Token"** 来生成一个。

### 步骤 3: 运行工作流

▶️ 一切准备就绪后，您可以开始构建了！

1.  在您的仓库页面，点击顶部的 **"Actions"** 标签页。
2.  在左侧列表中，点击 **"构建多架构Docker镜像 → ghcr.io, DockerHub"** 工作流。
3.  点击右侧的 **"Run workflow"** 下拉按钮。
4.  您会看到一个表单，请根据您的需求填写参数。
5.  点击绿色的 **"Run workflow"** 按钮，开始执行构建和推送任务！

## ⚙️ 工作流输入参数

在触发工作流时，您需要填写以下参数：

| 参数 (Parameter) | 描述 (Description)                                           | 是否必需 | 默认值                                 |
| :--------------- | :----------------------------------------------------------- | :------- | :------------------------------------- |
| `repo_url`       | 包含 `Dockerfile` 的 Git 仓库克隆链接。例如：`https://github.com/user/repo.git` | 是       |                                        |
| `repo_ref`       | 要克隆的仓库的分支、标签或 Commit SHA。例如：`main`, `v1.2.0` | 是       | `main`                                 |
| `image_name`     | 您希望为镜像设定的名称。例如：`my-web-app`                   | 是       | `my-app`                               |
| `tags`           | 要为镜像打上的标签，多个标签用逗号 `,` 分隔。例如：`latest,v1.0` | 是       | `latest`                               |
| `platforms`      | 要构建的 CPU 架构列表，用逗号 `,` 分隔。留空则使用默认列表。 | 是       | `linux/amd64,linux/arm64,linux/arm/v7` |

## 📄 工作流文件详解

本项目的核心是 `.github/workflows/docker-build.yml` 文件。它定义了以下自动化步骤：

1.  **克隆代码**：根据您输入的 `repo_url` 和 `repo_ref` 克隆指定的源代码。
2.  **环境设置**：配置 QEMU 和 Docker Buildx，为跨平台构建做准备。
3.  **登录仓库**：分别使用 `GITHUB_TOKEN` 和您设置的 `DOCKERHUB_TOKEN` 安全地登录到 ghcr.io 和 Docker Hub。
4.  **准备与生成元数据**：根据您输入的参数，准备好镜像名称和所有需要推送的标签。
5.  **构建和推送**：使用 `docker/build-push-action` 执行多平台构建，并将最终的镜像清单 (manifest) 推送到所有已登录的仓库。
