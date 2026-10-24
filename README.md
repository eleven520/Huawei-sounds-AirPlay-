---
title: 华为SoundX-AirPlay-skill包说明
---

# 华为 Sound X 支持 AirPlay · Skill 包说明

**文件**：`huawei-soundx-airplay.zip`（17 KB）

## 这是什么

一套让**华为 Sound X 支持 AirPlay 投歌**的完整方案：在斐讯 N1 / iStoreOS / OpenWrt 旁路由上跑「AirPlay 2 接收端 → DLNA 转发桥」，让 iPhone / Mac 的隔空投送能直接选到音箱。

上游项目：https://github.com/shinn-y/airplay2-dlna-bridge

## 怎么安装

**方式一：直接放进技能目录（推荐）**

1. 解压 zip
2. 把 `huawei-soundx-airplay` 整个文件夹放进 `~/.workbuddy/skills/`（用户级）或项目的 `.workbuddy/skills/`（项目级）
3. 重启 WorkBuddy，技能即生效

**方式二：直接看内容**  
解压后目录结构：

```
huawei-soundx-airplay/
├── SKILL.md                          # 入口：架构、执行流程、6条设计约束、延迟调优
├── scripts/
│   ├── deploy_soundx_airplay.sh      # 一键部署（幂等，可重复执行）
│   └── diagnose_soundx.py            # 端到端二分诊断，输出断点与修复命令
└── references/
    └── troubleshooting.md             # 踩坑手册：症状索引表、延迟调优、指示器速查
```

## 快速上手

```bash
# 1. 部署（在旁路由上执行）
sh deploy_soundx_airplay.sh <SoundX_IP> <旁路由IP>
#   例: sh deploy_soundx_airplay.sh 192.168.31.134 192.168.31.7

# 2. 验证（iPhone 隔空投送应出现 "Sound X (AirPlay)"）

# 3. 出问题时诊断（在任意能访问局域网的机器上跑）
python3 diagnose_soundx.py <SoundX_IP> [旁路由IP]
```

## 前置条件

- 旁路由：iStoreOS / OpenWrt 24.10，aarch64，已装 `python3`、`ffmpeg`、`shairport-sync`  
  （`opkg install shairport-sync-mbedtls python3 ffmpeg`）
- **必须**在华为「智慧生活」App 里打开 Sound X 的**音频投放 / DLNA 开关**
- Sound X 与旁路由、手机在同一局域网

## 这套方案踩过的坑（都写进包里了）

1. 配置放**私有路径**，不用 `/etc/shairport-sync.conf`（opkg 包会按 UCI 重写它）
2. FIFO 管道**只建不删**（两端按路径重开，删重建会导致 inode 失联 → 无声且音量无反馈）
3. 不用 `pkill`（会静默失效），改用 `ps|grep|awk` 取 PID 后 `kill -9`
4. 不用 `rc.common` 的 shebang（自定义 `start()` 不会被调用）
5. 渲染器端口**必须可配置**（server.py 写死 49152，Sound X 实际是 49494）
6. AirPlay 端口在 N1 上是 **7000 不是 5000**（别把"没有 5000"当故障）

包内 `diagnose_soundx.py` 会用**二分探测**逐环验证并直接指出断点，是这个方案最有价值的部分。

---

*仅供研究参考。*
