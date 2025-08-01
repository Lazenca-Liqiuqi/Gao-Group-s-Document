# 如何使用命令行AI Agent

Lazenca

## 概况

Gemini CLI和Claude Code是Google和Anthropic提供了命令行AI Agent工具，Agent与Chatbot的不同之处在于，Agent在Terminal（bash、cmd、Powershell）中运行，我们可以授权Agent一定的系统权限，直接修改文件、代码甚至可以自动进行调试与debug。其他相关工具也有Cursor，但Cursor是一个单独的IDE，而前两者不需要更换IDE，而与Vscode的其他AI插件（比如Cline、Github Copilot）相比，这类插件主要提供类似于Chatbot的功能并且有代码补全、生成测试的功能，而命令行Agent可以直接扫描目录，而非当前打开的代码文件，提供更项目化的上下文，并且可以利用系统权限直接操作文件，可以进行文件管理、撰写README等工作。

目前为止（2025.7.11），Gemini CLI是免费的，默认使用Gemini 2.5 Pro模型，Tokens用量接近于无限（很难想象每天1000次调用能用得完），只需要使用Google登陆即可，而Claude Code据称比Gemini CLI、Cursor都要强很多，并且有许多模型服务商都提供了Claude Code的接口，价格可以参阅《LLM价格一览》。两个工具的主要技术路线差异在于，Gemini CLI使用长上下文单Agent路线，而Claude Code使用主Agent领导多个子Agent分配任务。

2025.7.15：Windows原生Claude Code，原有WSL安装方法可以作用于Linux。
2025.7.17：Github Copilot
2025.7.21：Trae。
2025.7.23：Qwen3-Coder与Qwen Code发布。

我接下来提供Win下的使用方法，本人对Claude Code的用法进行了更详细的介绍。

## 科学上网

在使用命令行工具的全程都需要使用魔法，除非使用国内中转站，我使用了Claude Code的中转站，但是Gemini CLI仍然需要全程使用魔法。

首先需要一个靠谱的魔法软件，最好需要有以下功能：修改端口，TUN模式，系统代理，自定义代理规则。如果有其他的梯子的订阅没用完，至少得有全局代理和修改端口的功能，但不保证成功，这是因为，使用CLI工具需要让Terminal和WSL也能够通过代理。

以Clash为例，我使用了iKuuu机场提供的定制Clash客户端和配置方案，根据网页教程配置完成之后，关闭端口随机，选择一个自己想要的端口，然后打开TUN模式，启动系统代理，最好用美国节点。

<img src="./assets/image-20250711175750810.png" alt="image-20250711175750810" style="zoom:50%;" />

然后修改系统代理，在Win11的设置-网络和Internet-代理中，使用手动设置代理，打开并编辑端口与Clash一致，系统会根据配置方案生成白名单。

<img src="./assets/image-20250711180037458.png" alt="image-20250711180037458" style="zoom: 50%;" />

<img src="./assets/image-20250711180103770.png" alt="image-20250711180103770" style="zoom: 67%;" />

之后，修改系统的环境变量（直接在设置中搜索修改系统环境变量），也可以在命令行中修改。

<img src="./assets/image-20250711180326090.png" alt="image-20250711180326090" style="zoom: 67%;" />

新建两个新的系统变量，7890改为你需要的端口。同时也可以再新建两个全部为小写的变量，一般而言两者会相互替换，保留全大写的就可以了，只有全小写的系统变量似乎不能够成功代理。WSL一般而言会直接继承外部的系统代理。到此为止网络设置完成。可以测试一下在Terminal中是否能够访问外网。

```cmd
curl -vvk google.com
```

## Gemini CLI

### 安装并运行

Gemini CLI支持直接在Windows下运行，我们直接全局安装，也可以使用npx一次性使用。

先根据网络教程安装node.js，此处略过，确保下列命令可以输出。

```powershell
node -v
npm -v
```

然后在命令行中运行（也可以直接在Powershell中运行，这样可以跳过3.2）。

```cmd
npm install -g @google/gemini-cli
```

这样就安装完成了，在登陆之前，我们先确保我们的Google账户有相关权限。进入https://console.cloud.google.com/，进入API和服务。点击“启用API和服务”。

<img src="./assets/image-20250711181640217.png" alt="image-20250711181640217" style="zoom: 67%;" />

搜索Gemini，然后将下列三个全部启用。

<img src="./assets/image-20250711181803897.png" alt="image-20250711181803897" style="zoom:50%;" />

然后在系统环境变量中加入Google Cloud的项目ID。

![image-20250711182640504](./assets/image-20250711182640504.png)

<img src="./assets/image-20250711182722500.png" alt="image-20250711182722500" style="zoom:67%;" />

然后在cmd中输入gemini，跟随他一步一步登陆就好啦！

<img src="./assets/v2-cbc7b0d9feb5b2f12926c78e8152ffac_r.jpg" alt="img" style="zoom: 50%;" />

在这三项中选第一个，如果觉得用的多了模型变笨了，可以用API Key。

实测，不翻墙也可以正常使用，但是他会经常让你重新登陆，此时需要保证网络环境完整配置。

![image-20250711231356507](./assets/image-20250711231356507.png)

### 在Vscode中使用Gemini CLI

Vscode中可以直接调出Terminal，但是默认的Terminal是Powershell，可以输入cmd进入命令行再输入gemini启动，也可以新建一个cmd的Terminal使用，接下来我们让Powershell也可以直接启动gemini。

在开始菜单中以管理员身份运行Powershell，然后输入`Get-ExecutionPolicy`查看当前执行策略。然后输入

```
Set-ExecutionPolicy RemoteSigned
```

允许外部脚本运行，此时`Get-ExecutionPolicy`的结果应该是

<img src="./assets/image-20250711183748031.png" alt="image-20250711183748031" style="zoom: 50%;" />

此时可以直接在Powershell中输入gemini启动，在Vscode中也可以直接输入gemini启动。

<img src="./assets/image-20250711231455232.png" alt="image-20250711231455232" style="zoom:80%;" />

Gemini CLI会将当前目录的GEMINI.md作为提示词。

## Claude Code

> [!NOTE]
>
> 2025.7.15，Anthropic发布了Windows原生的Claude Code，可以直接跳过4.1 安装WSL，直接在命令行中安装Claude Code了。请自行搜索Windows安装node.js并且运行npm install -g @anthropic-ai/claude-code

### 安装WSL

在控制面板-程序和功能-启用或关闭Windows功能中，开启Virtual Machine Platform（Windows虚拟机监控程序平台）与Windows Subsystem for Linux Support（适用于Linux的Windows子系统）。Win11再开启Windows Hypervisor Platform（可能没有）和Hyper-V。

<img src="./assets/image-20250711225400154.png" alt="image-20250711225400154" style="zoom:60%;" /><img src="./assets/image-20250711225425888.png" alt="image-20250711225425888" style="zoom:60%;" />

可以选择将WSL版本设定为WSL2。
```cmd
wsl --set-default-version 2
```

安装Ubuntu，版本任选。

```cmd
wsl --install -d Ubuntu - 20.04
```

在开始面板搜索Ubuntu，启动WSL，或者在命令行启动WSL。

```cmd
wsl -d Ubuntu
```

### 安装ClaudeCode

建议先看到4.2.1的结尾。

#### 官方版本

在WSL里安装node.js，

```bash
curl -fsSL https://deb.nodesource.com/setup_lts.x | sudo bash -
sudo apt-get install -y nodejs
node --version
```

Linux也是相同操作，也就是可以在服务器上安装，服务器已经预装了node.js，但版本太低，更新一下。

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.5/install.sh | bash
nvm install --lts
```

切换到国内镜像（淘宝镜像，其他镜像源自行搜索）

```bash 
npm config set registry https://registry.npmmirror.com
```

然后可以安装Claude Code。

```bash
npm install -g @anthropic-ai/claude-code
claude --version
```

接下来可以在魔法环境下启动，选择主题，然后回车登陆。

<img src="./assets/a1d4b89b4e6a938ef3dac9a1e70cef5c.png" alt="img" style="zoom: 50%;" />

会自动跳转到Anthropic的网站，登录后我发现，我的Google账户被Anthropic封了（反华企业！！）

<img src="./assets/image-20250711230949432.png" alt="image-20250711230949432" style="zoom: 50%;" />

#### API来源

为了解决Anthropic对于中国包括港澳地区的IP动辄封号的行为，我们只能使用中转站（API聚合平台或镜像），使用了国内的中转站，不需要使用魔法，网络环境相对稳定一些。

复制API，然后配置环境变量，修改Claude Code的API地址为其他网站，将每一行的“sk-…”替换为API令牌。也可以直接修改.bashrc文件。

> [!NOTE]
>
> 对于Windows原生Claude Code，在C:\Users\User_name\\.claude中直接修改setting.json文件即可。

```bash
echo -e '\n export ANTHROPIC_AUTH_TOKEN=sk-...' >> ~/.bash_profile
echo -e '\n export ANTHROPIC_BASE_URL=...' >> ~/.bash_profile
echo -e '\n export ANTHROPIC_AUTH_TOKEN=sk-...' >> ~/.bashrc
echo -e '\n export ANTHROPIC_BASE_URL=...' >> ~/.bashrc
echo -e '\n export ANTHROPIC_AUTH_TOKEN=sk-...' >> ~/.zshrc
echo -e '\n export ANTHROPIC_BASE_URL=...' >> ~/.zshrc
```

运行 `source ~/.bashrc`让环境变量生效。然后启动。

```bash
claude
```

<img src="./assets/image-20250711232257231.png" alt="image-20250711232257231" style="zoom:50%;" />

##### 稳妥AI

https://api.wentuo.ai/ ，存在一定逆向与模型降级行为。
```bash
export ANTHROPIC_API_KEY="sk-"
export ANTHROPIC_BASE_URL="https://api.wentuo.ai"
```

> [!WARNING]
>
> 使用国内中转站可能存在数据风险，需要谨慎授予权限和配置环境，并且，不建议使用中转站专门提供的Claude Code安装包，风险性更高。
>
> 即便是为了省钱，也建议细致地写好.claudeignore文件和CLAUDE.md作为默认提示词，不然他乱扫一通之后就会吞掉巨量的cache tokens。可以使用时先初始化或者命令Agent扫描工作目录，构造提示词文档。

##### Kimi K2

月之暗面推出的Kimi K2模型强调编程与Agent能力的模型，并且推出了Anthropic格式的api，以支持Claude Code的使用，详情参见https://platform.moonshot.cn/docs/guide/Agent-support#%E8%8E%B7%E5%8F%96-api-key

```json
"ANTHROPIC_BASE_URL": "https://api.moonshot.cn/anthropic",
"ANTHROPIC_API_KEY": "sk-"
```

> [!TIP]
>
> 月之暗面针对账户充值金额有速率限制，请查询相关文档与网络使用心得。

##### 阿里Qwen Coder

2025.7.23，阿里云百炼发布了Qwen 3 Coder模型与Qwen Code CLI工具，其称性能与Claude 4 Sonnet媲美，可以通过

``` bash
npm i -g @qwen-code/qwen-code
export OPENAI_API_KEY="your_api_key_here"
export OPENAI_BASE_URL="https://dashscope-intl.aliyuncs.com/compatible-mode/v1"
export OPENAI_MODEL="qwen3-coder-plus"
qwen
```

安装并使用。

同时，Qwen 3也支持直接通过Claude Code调用，将环境变量替换为下列链接即可。

```bash
export ANTHROPIC_BASE_URL=https://dashscope-intl.aliyuncs.com/api/v2/apps/claude-code-proxy
export ANTHROPIC_AUTH_TOKEN=your-dashscope-apikey
```

##### 智谱GLM

智谱清言在2025.7.28发布了GLM-4.5系列模型，可以直接接入Claude Code与Gemini CLI等命令行工具，具体可以在文档中查看。

```bash
ANTHROPIC_BASE_URL=https://open.bigmodel.cn/api/anthropic
ANTHROPIC_AUTH_TOKEN={YOUR_API_KEY} 
```

##### Openrouter

现在还没有开通Claude Code支持。

### 在Vscode中使用Claude Code

当然，Vscode也是可以在Terminal直接启动WSL的，此时直接输入claude就可以启动。（2025.7.15）安装Windows版之后，直接在Powershell即可启动。

<img src="./assets/image-20250711234751091.png" alt="image-20250711234751091" style="zoom: 67%;" />

Anthropic也在Vscode提供了Claude Code插件，会在编辑器左上角快速进入，并且会自动将所在的文件与选中的代码作为上下文输入。

![image-20250716192309529](./assets/image-20250716192309529.png)

并且，右键菜单也会出现Fix with Claude Code的选项。

### Claude Code最佳实践

Claude Code+Kimi K2/GLM-4.5即可在国内流畅使用。

- sub agents与command有助于进一步提升效率

- 细致编写环境CLAUDE.md与项目CLAUDE.md

- 显式要求think、think hard、think harder、ultrathink开启思考模式

- 配置MCP扩展Claude Code的能力
    
    请查询Anthropic文档https://docs.anthropic.com/en/docs/claude-code/mcp
    
- 

## Github Copilot（等提供Agent的IDE/Editor）

Vscode内置的Github Copilot也具有不错的Agent能力，并且售价相对便宜（Pro 10美元/M，Pro+39美元/M），更重要的是，**通过Github学生认证之后，即可免费使用Pro**。可以使用Gemini 2.5 Pro与Claude 4 Sonnet，额度尚可，并且提供无限的代码补全与GPT 4.1，GPT 4o聊天额度，关于如何方便地通过Github学生认证，请参阅《Github指北》篇。

其他的IDE包括：Cursor，Trae IDE，腾讯CodeBuddy IDE等。

