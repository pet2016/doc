### Mac mini 安装OpenClaw

#### 1. 先安装node.js

##### 下载并安装 nvm：

> curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash

##### 配置环境

>  vi ~/.zshrc
>
> #添加以下内容
>
> export NVM_DIR="$HOME/.nvm"
>
> [ -s "/Users/bai/.nvm/nvm.sh" ] && . "/Users/bai/.nvm/nvm.sh"
>

##### 下载并安装 Node.js
 
> nvm install 24

##### 验证 Node.js 版本：

> node -v  # 打印版本号

##### 验证 npm 版本：

> npm -v # Should print "11.8.0".


#### 2. 安装 pnpm (包管理器)

>npm install -g pnpm
>
>pnpm config set registry https://registry.npmmirror.com/ 

#### 3. 下载openclaw

>git clone https://gitee.com/OpenClaw-CN/openclaw-cn.git
>
>cd openclaw-cn
>
>git checkout v2026.2.2-cn

#### 4. 安装与构建

OpenClaw 是一个现代化的全栈应用，首次运行需要编译前端 UI 和后端核心

##### 安装依赖

>pnpm install

##### 构建前端界面
    >pnpm ui:build

##### 构建核心服务
>pnpm build

##### 启动初始化向导(可选择DeepSeek)：

> pnpm openclaw onboard --install-daemon

#### 5. 启动服务 (初始化完成后)

初始化完成后，你可以通过以下命令再次启动网关服务（前提是网关已经关闭）：

##### 启动网关 (Gateway)
    
>node openclaw.mjs gateway --port 18789 --verbose

如果你关闭了管理页面，可以通过以下命令再次打开：

##### 打开管理面板 (Dashboard)
> node openclaw.mjs dashboard






####


4.5 版本起不来
npm install @buape/carbon @larksuiteoapi/node-sdk @slack/web-api grammy