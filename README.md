# 选课脚本

一个用于课程注册、查询和会话管理的异步自动化工具。本引擎专为处理状态复杂的遗留系统而设计，通过模拟必要的导航路径并维护持久会话状态来实现自动化操作。

## 技术背景
系统需要特定的会话激活序列以防止服务端错误。关于底层访问逻辑的详细技术分析，请参阅：[about_access_permissions.md](about_access_permissions.md)。

## 安装

本项目需要 Python 3.12+ 并使用 `uv` 进行依赖管理。

1. 安装环境和依赖：
```bash
uv sync
```

## 配置指南

配置通过 `config.toml` 文件管理。从模板开始：
```bash
cp config.toml.example config.toml
```

### 1. 账号 Cookie
所有操作的必要前提。
- **获取方式**：通过[SHIEP-Pre-Selection](https://github.com/TEraby-E/SHIEP-PreSelection-Script)获取，要保证该项目与这个项目处于同一个目录下，需要将个人信息完整填入Pre-Selection的配置项

### 2. 获取 Profile ID
`profileId` 是查询和选课的必要参数。
- **获取方式**：手动登录选课系统并进入选课模块，查看浏览器地址栏中的 URL，`profileId` 即为 `profileId=` 参数后面的数字。
- **使用方法**：将此 ID 填入 `INQUIRY_USER_DATA` 以启用搜索功能，以及填入 `USER_CONFIGS` 用于注册。

### 3. 获取课程信息
在 Cookie 和 `profileId` 有效的情况下，使用查询工具获取具体课程信息。
- **命令**：`uv run main.py --inquire`
- **功能**：该命令可获取课程名称、教师信息、当前选课人数以及唯一的**课程 ID**。
- **使用方法**：将查询结果中的**课程 ID** 复制到 `USER_CONFIGS` 下的 `course_ids` 列表中用于注册。

### 4. 代理与网络环境
`USE_PROXY` 的配置取决于您的连接方式：
- **官方 VPN（EasyConnect）或校园网**：这些环境通常可直连服务器，需将 `USE_PROXY` 设为 `False`。
- **第三方 VPN（如 EasierConnect）**：这些环境通常需要 SOCKS5 代理来转发流量。将 `USE_PROXY` 设为 `True`，并在 `config.toml` 的 `proxies` 字典中指定代理服务器地址和端口。


## 使用方法

命令通过 `uv run main.py <命令>` 执行。

### 可用命令
- `--start`：对 `USER_CONFIGS` 中定义的所有用户执行选课流程。
  - `--endless`：可选参数，无限重试直到成功。用于抢课。
- `--inquire`：交互模式，用于搜索课程和查询选课状态。需要有效的 `profileId`。
- `--validate`：批量验证所有账号的 Cookie 有效性。
- `--check`：实时验证配置中课程的容量和当前选课状态。
- `--help`：显示命令帮助菜单。

## 特性

- **异步并发**：基于 `asyncio` 和 `aiohttp` 构建，高效管理多账号。
- **会话激活**：自动化预访问流程，满足服务端状态要求。
- **频率限制保护**：通过顺序激活和任务交错，避免触发基于 IP 或会话的频率限制。
- **数据修复**：内置针对遗留接口非标准 JSON 响应的恢复机制。

## 维护与安全
- **SSL**：默认禁用证书验证，以兼容内网证书问题。
- **终止**：使用 `Ctrl+C` 安全停止进程。
