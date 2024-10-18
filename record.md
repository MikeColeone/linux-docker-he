常用命令;

- 清屏
  
  > clear

- 快速删除
  
  > ctrl+U

- 列出目录内容
  
  > ls

- 显示当前目录文件和子目录
  
  > ls -l

- 罗列出详细信息 权限大小 修改时间
  
  > ls -a

- 进入目录
  
  > cd + 目录名

- 进入指定目录
  
  > cd /etc/docker

- 退出目录
  
  > cd ..

- 返回上一级
  
  > cd -

- 查看当前目录
  
  > pwd

- 创建目录
  
  > mkdir + 目录名

- 删除目录
  
  > rmdir

- 删除空目录
  
  > rm -r 目录名

- 重命名
  
  > mv 就目录 新目录

- 查看操作系统版本
  
  > cat /etc/os-release

- 查看内核版本
  
  > uname -r

- 查看CPU
  
  > lscpu

- 内存使用情况
  
  > free -h

- 磁盘占用
  
  > df -h

- 删除整行
  
  > dd





实现推送 GitHub 代码时自动部署，可以通过 GitHub Actions 实现自动化工作流。以下是实现自动部署的思路和步骤，使用 GitHub Actions 进行代码构建、测试、打包，并在推送到 GitHub 仓库后，自动将更新部署到服务器。

### 1. **配置 GitHub Actions**

GitHub Actions 是一个 CI/CD 工具，可以用来在特定事件发生时自动执行指定的任务。为了实现自动部署，通常会在推送代码时触发工作流，包括以下步骤：

- 构建项目
- 运行测试
- 打包应用
- 部署到服务器

#### 创建 `.github/workflows/deploy.yml`

在 GitHub 仓库的根目录下创建 `.github/workflows/deploy.yml` 文件，这是 GitHub Actions 的配置文件，用于定义工作流。

```yaml
name: Deploy on Push

on:
  push:
    branches:
      - main  # 当推送到 main 分支时触发

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # 检出代码
      - name: Checkout repository
        uses: actions/checkout@v2

      # 设置 Node.js 环境
      - name: Set up Node.js
        uses: actions/setup-node@v2
        with:
          node-version: '16'  # 根据你的项目需求选择版本

      # 安装依赖
      - name: Install dependencies
        run: npm install

      # 运行测试 (可选)
      - name: Run tests
        run: npm test

      # 构建项目
      - name: Build project
        run: npm run build

      # 部署到服务器
      - name: Deploy to server
        run: |
          scp -r ./dist username@server_ip:/path/to/your/project
          ssh username@server_ip "cd /path/to/your/project && docker-compose down && docker-compose up -d"

      # 清理旧镜像（如果使用 Docker 部署）
      - name: Remove old images
        run: ssh username@server_ip "docker image prune -f"
```

### 2. **Secrets 配置**

为了让 GitHub Actions 能够访问远程服务器进行部署，你需要配置 `GitHub Secrets` 来存储敏感信息，比如服务器的 SSH 密钥。具体步骤如下：

1. 进入 GitHub 仓库的 "Settings" > "Secrets and variables" > "Actions"。
2. 点击 `New repository secret`，添加以下 Secrets：
   - **SSH_PRIVATE_KEY**：SSH 私钥，用于连接服务器。
   - **SERVER_IP**：服务器的 IP 地址。
   - **USERNAME**：用于登录服务器的用户名。

然后在 `deploy.yml` 文件中引用这些 Secrets：

```yaml
- name: Deploy to server
  env:
    SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
    SERVER_IP: ${{ secrets.SERVER_IP }}
    USERNAME: ${{ secrets.USERNAME }}
  run: |
    mkdir -p ~/.ssh
    echo "$SSH_PRIVATE_KEY" > ~/.ssh/id_rsa
    chmod 600 ~/.ssh/id_rsa
    ssh-keyscan $SERVER_IP >> ~/.ssh/known_hosts
    scp -r ./dist $USERNAME@$SERVER_IP:/path/to/your/project
    ssh $USERNAME@$SERVER_IP "cd /path/to/your/project && docker-compose down && docker-compose up -d"
```

### 3. **服务器端部署配置**

在服务器端，你可以使用 `docker-compose` 来简化部署流程。你可以在服务器上编写一个 `docker-compose.yml` 文件，以便在 GitHub Actions 推送代码后重新拉取最新的镜像并启动服务。

#### 服务器上的 `docker-compose.yml` 示例

```yaml
version: '3'
services:
  app:
    image: username/app:latest  # 使用你构建好的镜像
    ports:
      - "80:80"
    volumes:
      - ./dist:/usr/share/nginx/html
    restart: always
```

### 4. **持续集成和部署流程**

1. 开发者将代码推送到 GitHub 仓库的 `main` 分支。
2. GitHub Actions 自动触发 `deploy.yml` 工作流：
   - 检出代码、安装依赖、构建项目。
   - 使用 SSH 将构建好的项目文件传输到远程服务器。
   - 使用 `docker-compose` 在服务器上重新启动服务，部署新的应用版本。
3. 自动清理旧的 Docker 镜像，确保服务器不被过多的镜像占用。

### 5. **总结**

通过 GitHub Actions，可以轻松实现推送代码时自动化部署。流程的关键在于：

- **GitHub Actions 配置**：定义自动执行的步骤，包括代码检出、构建、测试、部署等。
- **Secrets 管理**：安全存储敏感信息，如 SSH 密钥、服务器 IP。
- **服务器配置**：使用 Docker 或其他工具简化部署流程，确保代码能自动更新。
