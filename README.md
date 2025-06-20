# Free V2Ray 节点收集与测试工具

这是一个用Python编写的工具，用于自动收集、解析、测试和筛选可用的免费V2Ray节点。该工具支持多种代理协议和格式，可以帮助您快速获取可用的代理节点资源。

## 主要功能

- 从多个在线资源自动获取免费节点信息
- 支持多种格式的解析（Base64、YAML、JSON、URI等）
- 支持多种代理协议（VMess、Trojan、VLESS、Shadowsocks、Shadowsocks-R等）
- 使用V2Ray/XRay核心程序进行实际连接测试
- 自动测量节点延迟并筛选可用节点
- 去除重复节点
- 以标准格式导出可用节点信息

## 支持的协议

- VMess
- Trojan
- VLESS
- Shadowsocks (SS)
- Shadowsocks-R (SSR)
- HTTP/HTTPS
- SOCKS
- Hysteria
- WireGuard
## 安装步骤

1. 克隆仓库到本地：
    ```bash
    git clone https://github.com/bruceblink/free-v2ray
    cd free-v2ray
    ```

2. 安装依赖：
    ```bash
    pip install -r requirements.txt
    ```

## 使用方法

### Github Actions定时更新

如果不想安装，或者下载任何代码及软件，可以每天从当前的[Github仓库](https://github.com/bruceblink/free-v2ray)的`v2ray/v2ray.txt`中复制最新的节点，`v2ray.tx`中的节点每3小时(这个更新频率可以自己在[.github/workflows/update_v2ray_node.yml]([update_v2ray_node.yml](https://github.com/bruceblink/free-v2ray/blob/main/.github/workflows/update_v2ray_node.yml))中调整)更新一次新的，具体更新记录可以查看代码的提交记录。

### 本地运行

- 例如使用Pycharm或者vscode打开下载到本地的代码目录

   运行main.py
   ```bash
   python main.py
   ```
   等待v2ray.txt文件的生成和更新

- 或者docker-compose运行

    在代码仓库的本地目录终端中执行如下命令

    构建以及启动

    ```bash
    docker-compose -f docker-compose.yml -p free-v2ray up -d
    ```

    停止及卸载

    ```bash
    docker-compose -f docker-compose.yml -p free-v2ray down
    ```

- 使用docker运行
   如果你没有安装docker-compose，可以直接使用以下命令运行：
   将终端运行的目录(`pwd`目录)下的`v2ray`、`conf`和`logs`文件夹挂载到容器中，需要将项目中的`conf/conf.yml`文件复制到当前目录`conf`目录下。

    ```powershell
     docker run -d --name free-v2ray -v ${PWD}/v2ray:/app/v2ray -v ${PWD}/conf:/app/conf -v ${PWD}/logs:/app/logs likanug515/free-v2ray:latest
    ```

     如果想使用默认的conf.yml，可以不挂载`conf`目录，使用如下命令：        

    ```powershell
    docker run -d --name free-v2ray -v ${PWD}/v2ray:/app/v2ray -v ${PWD}/logs:/app/logs likanug515/free-v2ray:latest
    ```


- 工具会自动：
   - 从预定义的链接获取节点信息
   - 解析不同格式的节点配置
   - 进行去重处理
   - 测试节点的连接质量和延迟
   - 生成最终的节点列表
- 测试完成后，结果会保存到：
   - `v2ray/v2ray.txt`：Base64编码的节点信息（可直接导入到V2Ray客户端）
   - `v2ray_raw.txt`：原始文本格式的节点信息（方便查看）

## 代码结构

- `main.py`：主程序入口，包含所有功能实现
- 主要功能模块：
  - 订阅链接获取：`fetch_content`
  - 节点解析：`extract_nodes`、`parse_clash_yaml`、`parse_v2ray_base64`、`parse_v2ray_uri`、`parse_json_nodes`
  - 节点测试：`test_node_latency`、`test_latency`
  - 节点处理：`process_node`、`remove_duplicates`
  - 核心程序管理：`find_core_program`、`download_xray_core`
- `config/settings.py`:  加载yaml配置文件，以及初始化相关配置

## 订阅链接配置

编辑conf/conf.yaml配置文件可以自定义订阅链接：

```yaml
#单个订阅链接
subscriptions:
  - https://example.com/subscription1
  - https://example.com/subscription2
  - ....
```

支持多种链接格式，包含日期变量和GitHub仓库链接。

## 特性

### 自动下载核心程序

如果系统中未找到XRay核心程序，工具会自动从GitHub下载最新版本。

### 多种格式解析

工具支持按照以下顺序尝试解析节点信息：
1. Base64编码内容
2. YAML/Clash配置
3. 使用正则表达式直接提取
4. JSON格式数据

### 智能链接处理

- 支持日期变量替换（如`{Y}`、`{m}`、`{d}`、`{Ymd}`等）
- 支持GitHub仓库文件自动获取（`{x}`占位符）
- 特殊站点的特殊处理逻辑

### 测试方法

使用XRay核心程序建立真实连接，通过访问Google测试端点（`http://www.gstatic.com/generate_204`）来测量延迟。

## 注意事项

1. 该工具仅供学习和研究使用，请遵守当地法律法规。
2. 部分节点可能由于网络环境不同而表现不同，测试结果仅供参考。
3. 可能需要管理员权限来下载和运行核心程序。
4. 在Windows系统上，可能会有防火墙提示，需要允许程序访问网络。

## 故障排除

- 如果无法获取节点信息，请检查订阅链接是否有效。
- 如果测试节点时出现错误，请确保XRay核心程序正确安装。
- 对于特殊网站的解析问题，可能需要安装额外的依赖（如BeautifulSoup4）。
- 如果遇到编码问题，可能需要调整`fetch_content`函数中的编码处理逻辑。

## 联系方式

如有任何问题或建议，欢迎通过以下方式联系：

- 提交 [Issue](https://github.com/bruceblink/free-v2ray/issues)
- [发送邮件](mailto:likanug.g@qq.com)

## 许可

本项目采用 MIT 许可证 - 详情请查看 [LICENSE](LICENSE) 文件

## 免责声明

本工具仅用于学习和研究网络技术，请勿用于非法用途。使用本工具所产生的任何后果由使用者自行承担。