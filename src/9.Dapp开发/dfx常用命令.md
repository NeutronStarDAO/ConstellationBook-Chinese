## dfx 常用命令

DFINITY 命令行执行环境（dfx）是用于创建、部署和管理 IC 上 DApp 的主要工具。

在[官方文档](https://internetcomputer.org/docs/current/references/cli-reference)里可以看到 dfx 的所有命令，这里只列出实践中最常用的一部分命令。

<br>

```bash
# 查询dfx版本
dfx --version
# 更新dfx
dfx upgrade

# 查看某个命令的帮助
dfx help
dfx help new # 查看new命令的帮助信息

dfx new hello_ic # 创建一个名为hello_ic的新项目
```

<br>

### identity 相关：

```bash
# 列出所有identity
dfx identity list
# 显示当前的principal-id
dfx identity get-principal
# 查询开发者身份的名字
dfx identity whoami
# 显示接收转账的account-id
dfx ledger account-id
# 从一个账户转账到另一个账户
dfx --identity xxxx ledger --network ic transfer --memo 0 --amount 0.5
# 创建新的identity身份，命名为neutronstarpro
dfx identity new neutronstarpro
# 使用某个identity身份
dfx identity use neutronstarpro
# 当前账户还有多少ICP
dfx ledger --network ic balance
# 名为nuetronstarpro的身份的账户还有多少ICP
dfx --identity neutronstarpro ledger --network ic balance
```

<br>

### wallet 相关：

```bash
# 显示当前cycles钱包的id
dfx identity --network ic get-wallet
# 当前钱包的cycles余额
dfx wallet --network ic balance
# 给某个canister充值cycles
dfx canister --network ic deposit-cycles 1000000 your-canister-principal
# 将dfx里的1个icp转换为cycles并充值给canister
dfx ledger top-up $(dfx canister id your-canister-name) --amount 1
===================================================================================================
# 创建一个canister并把dfx账户里的10个icp转换成cycles为canister充值；--amount意思是将指定的ICP转换为cycles
dfx ledger --network ic create-canister $(dfx identity get-principal) --amount 10
# 在canister里安装cycles钱包，安装之后这个canister就变成钱包专属canister了
dfx identity --network ic deploy-wallet <canister-id>
# 给默认身份下的cycles钱包充值（后面那个数量根据情况调整）
dfx wallet --network ic send $(dfx identity --network ic get-wallet) 80000590000
```

<br>

### deploy 部署相关：

```bash
# 启动本地环境
dfx start
# 清除缓存并启动本地环境
dfx start --clean
# 在后台启动本地环境，看不见它的信息
dfx start --background
# 启动模拟器模式本地环境
dfx start --emulator
# 启动无延迟响应模式的本地环境（默认的本地环境会模拟IC的网络，人为制造一些延迟，加入了达成共识的时间）
dfx start -no-artificial-delay
# 部署到本地
dfx deploy
# 关闭本地计算机运行的本地容器执行环境进程
dfx stop
# ====================================================================================
# 检查IC网络的当前状态和是否能连接
dfx ping ic
# 部署到IC网络
dfx deploy --network ic
# 部署到IC网络，指定了每个canister里充 1T cycles
dfx deploy --network ic --with-cycles=1000000000000
# 部署单个canister
dfx deploy --network ic <dapp_name>
```

<br>

### canister 相关：

```bash
# 查询自己身份下名为hello_assets的canister-id
dfx canister --network ic id hello_assets
# =============================================================
# 获取所有canister状态，--all可以换成canister的id或者canister的名字
dfx canister --network ic status --all
# 停止canister运行
dfx canister --network ic stop --all
# 删除canister里的代码
dfx canister --network ic uninstall-code --all
# 删除canister并回收cycles
dfx canister --network ic delete --all
# 重新部署canister，会清除所有canister里数据
dfx deploy --network ic <canister_name> --mode reinstall
# 或者
dfx canister install <canister_name> --mode reinstall
```

<br>

```bash
# 去Dfinity的Github仓库里的SDK下载DFX，解压后设置环境变量可以直接用
# 如果电脑里装了不同版本的DFX  根据环境变量决定用哪个版本的DFX
# 我在.profile文件里设置了DFX的环境
export PATH=/home/neutronstarpro/.dfx:$PATH
```

<br>

```shell
DFX_CONFIG_ROOT=~/ic-root
```

使用 DFX_CONFIG_ROOT 环境变量指定不同的位置来存储 .cache 和 的 .config 子目录 dfx 。

默认情况下， .cache 和 .config 目录位于开发环境的主目录中。例如，在 macOS 上，默认位置在 `/Users/<YOUR-USER-NAME>` 目录中。使用DFX_CONFIG_ROOT 环境变量为这些目录指定不同的位置。

<br>

```shell
DFX_INSTALLATION_ROOT
```

如果您不使用操作系统的默认位置，请使用 `DFX_INSTALLATION_ROOT` 环境变量为 dfx 二进制文件指定不同的位置。

该 `.cache/dfinity/uninstall.sh` 脚本使用此环境变量来标识 DFINITY Canister SDK 安装的根目录。

<br>

`DFX_TELEMETRY_DISABLED` 是选择是否集有关 dfx 使用情况的数据。

默认情况下，dfx 会收集匿名信息——即没有 IP 地址或用户信息等识别信息——有关 dfx 命令活动和错误的数据。默认情况下会启用收集匿名数据，根据使用模式和行为来改善开发人员体验。

如果要关掉收集匿名数据，通过将 `DFX_TELEMETRY_DISABLED` 环境变量设置为 1 来明确选择关闭。

```bash
DFX_TELEMETRY_DISABLED=1
```

<br>
