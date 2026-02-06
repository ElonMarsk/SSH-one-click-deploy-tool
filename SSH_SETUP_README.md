# SSH 一键部署工具 - 详细文档

> 本文档提供 SSH 一键部署工具的详细使用说明。快速开始请查看 [README.md](README.md)。

## 目录

- [配置说明](#配置说明)
- [使用方法](#使用方法)
- [工作流程](#工作流程)
- [密钥位置](#密钥位置)
- [连接服务器](#连接服务器)
- [故障排除](#故障排除)
- [安全提示](#安全提示)
- [示例](#示例)

## 配置说明

### 配置文件结构

### 首次配置

1. 复制示例配置文件：`sshconf.ini.example` → `sshconf.ini`
2. 编辑 `sshconf.ini` 文件，设置以下参数：

```ini
# SSH 密钥配置
KeyType = ed25519        # 可选: rsa, ed25519 (推荐)
KeyName = my_server_key  # 密钥文件名（不含路径）

# 任务参数
KeyPath =                # 密钥保存路径（可选，留空则使用脚本运行路径）
PublicKeyPath =          # 公钥文件完整路径（可选，用于上传公钥任务，留空则使用 KeyPath/KeyName.pub）
PrivateKeyPath =         # 私钥文件完整路径（可选，用于上传公钥/更新sshd配置时使用；留空则默认使用 KeyPath/KeyName）
AutoRestartSSH = yes     # 更新配置后自动重启SSH服务（可选: yes, no，默认: yes）

# 服务器配置
Host = your.server.com   # 服务器地址或IP
User = root              # 用户名
Port = 22                # SSH 端口

# SSH 服务器配置 (sshd_config)
PermitRootLogin = yes
PasswordAuthentication = no
PubkeyAuthentication = yes
# ... 更多配置项
```

### 配置项说明

| 配置项 | 说明 | 必填 | 默认值 |
|--------|------|------|--------|
| `KeyType` | 密钥类型 | ✅ | - |
| `KeyName` | 密钥文件名 | ✅ | - |
| `KeyPath` | 密钥保存路径 | ❌ | 脚本运行目录 |
| `PublicKeyPath` | 公钥文件路径 | ❌ | `KeyPath/KeyName.pub` |
| `PrivateKeyPath` | 私钥文件路径 | ❌ | `KeyPath/KeyName` |
| `AutoRestartSSH` | 自动重启SSH服务 | ❌ | yes |
| `Host` | 服务器地址 | ✅ | - |
| `User` | 用户名 | ✅ | - |
| `Port` | SSH 端口 | ❌ | 22 |

**配置项详细说明**：
- `KeyPath`：密钥保存路径（可选）
  - 留空：使用脚本运行目录
  - 相对路径：基于脚本目录，例如 `keys` 或 `../ssh_keys`
  - 绝对路径：完整路径，例如 `C:\Users\YourName\.ssh` 或 `D:\SSHKeys`
- `PublicKeyPath`：公钥文件完整路径（可选，用于上传公钥任务）
  - 留空：自动使用 `KeyPath/KeyName.pub`
  - 指定路径：使用指定的公钥文件路径（支持相对路径和绝对路径）
- `PrivateKeyPath`：私钥文件完整路径（可选，用于上传公钥 / 更新 SSH 配置时通过 `ssh -i` 连接）
  - 留空：自动使用 `KeyPath/KeyName`
  - 指定路径：使用已有的、已在服务器上授权的私钥
- `AutoRestartSSH`：更新配置后是否自动重启SSH服务（可选）
  - `yes`：自动重启（默认），配置更新后会自动尝试重启SSH服务
  - `no`：不自动重启，需要手动执行 `sudo systemctl restart sshd` 或 `sudo service ssh restart`

**注意事项**：
- `sshconf.ini` 文件包含敏感信息，已添加到 `.gitignore`，不会被提交到版本库
- 请填写实际的服务器地址、用户名等信息

## 使用方法

### 1. 运行脚本

**方法 1：双击运行（推荐）**
```bash
setup-ssh.bat
```

**方法 2：PowerShell 命令行**
```powershell
.\setup-ssh.ps1
```

**方法 3：指定配置文件**
```powershell
.\setup-ssh.ps1 -ConfigFile "path\to\your\config.ini"
```

### 2. 选择任务

运行脚本后会显示菜单，您可以选择：

1. **生成 SSH 密钥** - 在本地生成 SSH 密钥对
2. **上传公钥至服务器** - 将公钥上传到服务器的 `~/.ssh/authorized_keys`
3. **更新服务器 SSH 配置文件** - 更新服务器的 `/etc/ssh/sshd_config` 配置
4. **一键部署** - 依次执行上述所有步骤
0. **退出** - 退出程序

> **提示**：更新 SSH 配置前会显示将要更新的配置项列表，需要确认后才会执行。

## 工作流程

### 生成 SSH 密钥
1. **读取配置**：解析 `sshconf.ini` 文件
2. **验证配置**：检查必需参数是否已正确设置
3. **确定保存路径**：
   - 如果配置了 `KeyPath`，使用配置的路径
   - 否则使用脚本运行目录
   - 如果目录不存在，自动创建
4. **生成密钥**：
   - 检查密钥是否已存在
   - 如果不存在或选择覆盖，生成新的 SSH 密钥对
   - 密钥保存在指定或默认路径

### 上传公钥
1. **确定公钥路径**：
   - 如果配置了 `PublicKeyPath`，使用配置的路径
   - 否则使用 `KeyPath/KeyName.pub`（如果 `KeyPath` 未配置，使用脚本目录）
2. **确定私钥路径**：
   - 如果配置了 `PrivateKeyPath`，上传公钥和更新配置时会优先通过 `ssh -i PrivateKeyPath` 连接
   - 否则默认使用 `KeyPath/KeyName` 作为私钥路径
3. **检查公钥文件**：验证公钥文件是否存在
4. **连接服务器**：尝试连接服务器（优先使用私钥；如果服务器允许密码，可退回密码登录）
5. **创建目录**：在服务器上创建 `~/.ssh` 目录（如果不存在）
6. **上传公钥（幂等）**：将公钥追加到 `~/.ssh/authorized_keys`，如已存在则不会重复添加
7. **设置权限**：设置正确的文件权限

### 更新 SSH 配置
1. **显示配置项**：显示将要更新的配置项列表
2. **用户确认**：等待用户确认是否继续
3. **备份配置**：自动备份原配置文件
4. **更新配置**：更新 `/etc/ssh/sshd_config` 文件（每个配置项先清理旧行，再写入一行最新值，避免重复）
5. **验证语法**：验证配置文件语法是否正确
6. **自动重启**（如果启用）：
   - 尝试使用 `systemctl restart sshd` 重启服务
   - 如果失败，尝试使用 `service ssh restart` 或 `service sshd restart`
   - 如果都失败，提示用户手动重启
7. **提示信息**：显示配置更新和重启状态

## 密钥位置

密钥文件的保存位置取决于配置：

### 默认情况（KeyPath 留空）
密钥文件保存在脚本运行的目录中：
- **私钥**：`<脚本目录>\<KeyName>`
- **公钥**：`<脚本目录>\<KeyName>.pub`

例如，如果脚本在 `C:\SSHService\` 目录，`KeyName = my_server_key`，则：
- 私钥：`C:\SSHService\my_server_key`
- 公钥：`C:\SSHService\my_server_key.pub`

### 指定路径（KeyPath 已配置）
如果配置了 `KeyPath`，密钥将保存在指定路径：
- **私钥**：`<KeyPath>\<KeyName>`
- **公钥**：`<KeyPath>\<KeyName>.pub`

例如，`KeyPath = C:\Users\YourName\.ssh`，`KeyName = my_server_key`，则：
- 私钥：`C:\Users\YourName\.ssh\my_server_key`
- 公钥：`C:\Users\YourName\.ssh\my_server_key.pub`

### 自定义公钥路径（PublicKeyPath 已配置）
如果配置了 `PublicKeyPath`，上传公钥任务将使用指定的公钥文件路径，而不是自动生成的路径。

## 连接服务器

上传成功后，可以使用以下命令连接：

```powershell
# 在脚本目录下运行
ssh -i .\<KeyName> -p <Port> <User>@<Host>

# 或使用完整路径
ssh -i <脚本完整路径>\<KeyName> -p <Port> <User>@<Host>
```

或者配置 SSH config 文件（`%USERPROFILE%\.ssh\config`）：

```
Host <Host>
    User <User>
    Port <Port>
    IdentityFile <脚本完整路径>\<KeyName>
```

配置后可以直接使用：
```powershell
ssh <Host>
```

## 故障排除

### 1. 脚本无法执行

如果遇到"无法加载，因为在此系统上禁止运行脚本"错误：

```powershell
# 以管理员身份运行 PowerShell，执行：
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### 2. 找不到 ssh-keygen 或 ssh 命令

确保已安装 OpenSSH 客户端（Windows 10 1809+ 自带）：

1. 打开"设置" → "应用" → "可选功能"
2. 搜索"OpenSSH 客户端"
3. 如果没有安装，点击"安装"

或者通过 PowerShell 安装：
```powershell
Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0
```

### 3. 公钥上传失败

如果自动上传失败，脚本会提供三种手动上传方法：

**方法 1（推荐）**：使用 PowerShell 管道
```powershell
# 在脚本目录下运行
Get-Content .\<KeyName>.pub | ssh -p <Port> <User>@<Host> "cat >> ~/.ssh/authorized_keys"

# 或使用完整路径
Get-Content <脚本完整路径>\<KeyName>.pub | ssh -p <Port> <User>@<Host> "cat >> ~/.ssh/authorized_keys"
```

**方法 2**：使用 CMD
```cmd
# 在脚本目录下运行
type .\<KeyName>.pub | ssh -p <Port> <User>@<Host> "cat >> ~/.ssh/authorized_keys"

# 或使用完整路径
type <脚本完整路径>\<KeyName>.pub | ssh -p <Port> <User>@<Host> "cat >> ~/.ssh/authorized_keys"
```

**方法 3**：手动复制粘贴
1. 打开公钥文件：`<脚本目录>\<KeyName>.pub`
2. 复制全部内容
3. 登录服务器后执行：
   ```bash
   mkdir -p ~/.ssh
   chmod 700 ~/.ssh
   echo '复制的公钥内容' >> ~/.ssh/authorized_keys
   chmod 600 ~/.ssh/authorized_keys
   ```

### 4. 权限问题

确保服务器上的权限设置正确：
- `~/.ssh` 目录权限：`700`
- `~/.ssh/authorized_keys` 文件权限：`600`

## 安全提示

- ⚠️ **私钥安全**：私钥文件包含敏感信息，请妥善保管，不要分享给他人
- ⚠️ **密钥密码**：当前脚本生成的密钥没有密码保护，如需更高安全性，可以手动使用 `ssh-keygen` 生成带密码的密钥
- ⚠️ **服务器安全**：确保服务器已正确配置防火墙和 SSH 安全设置

## 示例

### 示例配置

#### 示例 1：使用默认路径（脚本目录）

```ini
# SSH 密钥配置
KeyType = ed25519
KeyName = production_server
# KeyPath 留空，使用脚本运行目录

# 服务器配置
Host = 192.168.1.100
User = admin
Port = 2222
```

#### 示例 2：指定密钥保存路径

```ini
# SSH 密钥配置
KeyType = ed25519
KeyName = production_server
KeyPath = C:\Users\YourName\.ssh
# 或使用相对路径: KeyPath = keys

# 服务器配置
Host = 192.168.1.100
User = admin
Port = 2222
```

#### 示例 3：使用自定义公钥路径

```ini
# SSH 密钥配置
KeyType = ed25519
KeyName = production_server
KeyPath = C:\Users\YourName\.ssh
PublicKeyPath = C:\Users\YourName\.ssh\custom_public_key.pub
# 如果指定了 PublicKeyPath，上传公钥任务将使用此路径

# 服务器配置
Host = 192.168.1.100
User = admin
Port = 2222
```

### 运行示例

```
=========================================
  SSH 一键部署工具
=========================================

读取配置文件: C:\SSHService\sshconf.ini

配置信息:
  密钥类型: ed25519
  密钥名称: my_server_key
  服务器: root@your.server.com:22
  密钥路径: C:\SSHService

请选择要执行的操作:

  1. 生成 SSH 密钥
  2. 上传公钥至服务器
  3. 更新服务器 SSH 配置文件
  4. 一键部署 (生成密钥 + 上传公钥 + 更新配置)

  0. 退出

请输入选项 (0-4): 1

=== 1. 生成 SSH 密钥 ===

正在生成 SSH 密钥...
  类型: ed25519
  名称: my_server_key
  路径: C:\SSHService
SSH 密钥生成成功!
  私钥: C:\SSHService\my_server_key
  公钥: C:\SSHService\my_server_key.pub
密钥生成任务完成!

按 Enter 键继续
```

## 安全提示

### 配置文件安全
- ⚠️ **不要提交敏感配置**：`sshconf.ini` 文件包含服务器地址、用户名等敏感信息，已添加到 `.gitignore`
- ✅ **使用示例文件**：项目包含 `sshconf.ini.example` 作为模板，可以安全提交到版本库
- ✅ **首次使用**：复制 `sshconf.ini.example` 为 `sshconf.ini` 并填写实际配置

### 密钥安全
- ⚠️ **私钥安全**：私钥文件包含敏感信息，请妥善保管，不要分享给他人
- ⚠️ **密钥密码**：当前脚本生成的密钥没有密码保护，如需更高安全性，可以手动使用 `ssh-keygen` 生成带密码的密钥
- ⚠️ **服务器安全**：确保服务器已正确配置防火墙和 SSH 安全设置

### 最佳实践
- 定期更换 SSH 密钥
- 使用强密码保护私钥（如使用带密码的密钥）
- 限制 SSH 访问权限（使用防火墙规则）
- 定期审查服务器上的授权密钥列表

## 许可证

本脚本为工具脚本，可自由使用和修改。

