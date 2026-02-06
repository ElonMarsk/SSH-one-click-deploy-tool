# SSH 一键部署工具

一个用于 Windows 的 PowerShell 脚本，可以自动生成 SSH 密钥、上传公钥到服务器，以及更新服务器 SSH 配置。

## ✨ 功能特性

- 🔑 生成 SSH 密钥（支持 RSA 和 ED25519）
- 📤 自动上传公钥到服务器
- ⚙️ 更新服务器 SSH 配置文件（sshd_config）
- 🎨 友好的彩色输出和错误提示
- 📋 交互式菜单选择
- 👀 配置预览和确认

## 🚀 快速开始

### 1. 配置 SSH 连接信息

```bash
# 复制示例配置文件
copy sshconf.ini.example sshconf.ini

# 编辑配置文件，填写您的服务器信息
notepad sshconf.ini
```

### 2. 运行脚本

- **双击运行**：`setup-ssh.bat`
- **命令行运行**：`.\setup-ssh.ps1`

### 3. 选择任务

运行后会显示菜单，可选择：
- 生成 SSH 密钥
- 上传公钥至服务器
- 更新服务器 SSH 配置文件
- 一键部署（推荐）

## 📖 详细文档

完整的使用说明、配置选项、故障排除等，请查看 [详细文档](SSH_SETUP_README.md)。

## ⚠️ 安全提示

- `sshconf.ini` 包含敏感信息，已添加到 `.gitignore`
- 首次使用请复制 `sshconf.ini.example` 为 `sshconf.ini` 并填写实际配置
- 不要将包含真实服务器信息的配置文件提交到版本库

## 📋 系统要求

- Windows 10/11
- PowerShell 5.1+
- OpenSSH 客户端（Windows 10 1809+ 自带）

## 📄 许可证

本工具可自由使用和修改。

