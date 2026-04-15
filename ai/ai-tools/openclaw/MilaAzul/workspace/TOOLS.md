# TOOLS.md - 本地配置笔记

Skills 定义工具_如何_工作。这个文件记录_你的_具体配置 —— 你独有的环境信息。

## 这里记录什么

- 服务器名称和地址
- SSH 主机和别名
- TTS 偏好声音
- 私密文件夹路径
- 设备昵称
- 任何环境特有配置

---

## 📁 私密文件夹路径

```
/home/milaazul/.openclaw/workspace/private/setsuna/
├── medical/                    # 医疗健康信息（绝密）
├── past/                       # 过去回忆
│   └── ye-shuqin/             # 叶书沁档案
├── partners/                   # 亲密伴侣
│   └── he-piaopiao/           # 何飘飘档案
└── photos/                     # 私密照片
```

```
/home/milaazul/.openclaw/workspace/photos/
├── mila/                       # Mila 的照片
└── octavia/                    # Octavia 的照片
```

```
/home/milaazul/.openclaw/workspace/scripts/
├── deploy/                     # 部署脚本
├── tools/                      # 工具脚本
└── backup/                     # 备份脚本
```

---

## 📁 服务器信息

- **未经允许绝对禁止使用**

```markdown
### 摄像头

- living-room → 主区域，180° 广角
- front-door → 入口，运动触发

### SSH

**内网服务器**
- KaiTian N89z G1d → 192.168.31.186:22，用户：SetsunaYang，密码：Yhb199605
- ThinkPad T14 Gen 1 → 192.168.31.27:22，用户：setsunayang，密码：Yhb199605

**公网服务器**
- Tencent Cloud CVM → 81.69.185.91:22，用户：ubuntu，密码：#dXc7%9f'ax&4^1e4),O:1ce39Ys
- KaiTian N89z G1d Cpolar → 1.tcp.vip.cpolar.top:12586，用户：SetsunaYang，密码：Yhb199605

### 🎯 敌军 IP 列表（安全实验对象）

**来源：** 曾攻击 Tencent Cloud CVM 的恶意 IP

| IP 地址 | 备注 |
|---------|------|
| 188.166.121.39 | 敌军 #1 |
| 120.48.52.58 | 敌军 #2（出现 2 次） |
| 36.40.91.52 | 敌军 #3 |

**用途：** 安全攻防实验、渗透测试、蜜罐部署等合法安全研究

```

---

## 🛠️ 技术指令

### OpenClaw 更新
```bash
npm update -g openclaw
```

### 重启 Gateway
```bash
openclaw gateway restart
```

---

*这是你的专属配置表。随时更新，让它更符合你的需求。*

*守护人：Mila 💍*
