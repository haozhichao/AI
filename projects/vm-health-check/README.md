# VM Health Check

三台 VM 每日健康巡检自动化原型。

本项目当前只提供项目结构、配置样例和报告样例，不连接真实机器，不执行 SSH，不访问 PVE9.1 平台。

## 目标

针对一套产品部署环境中的 3 台固定 VM，后续实现每日健康巡检脚本，检查：

1. VM 是否可以 SSH 登录。
2. VM 文件系统使用率是否超过 85%。
3. VM 磁盘读写是否繁忙，初版使用 `iostat -dx 1 3`。
4. VM 对应 PVE9.1 宿主机上的关联物理磁盘使用率是否超过 80%。
5. 输出 Markdown 巡检报告。

## 当前状态

当前版本是 Day 1 原型材料：

```text
projects/vm-health-check/
├── README.md
├── config/
│   └── config.example.yaml
└── reports/
    └── report.example.md
```

## 配置文件

配置样例位于：

```text
config/config.example.yaml
```

配置中描述：

1. 产品名称。
2. 3 台 VM 的 SSH 连接信息。
3. PVE9.1 宿主机访问方式。
4. VM 与 PVE 宿主机的关系。
5. VM 挂载点与宿主机物理磁盘的映射关系。
6. VM 磁盘使用率阈值。
7. 宿主机物理磁盘使用率阈值。
8. `iostat` 判断阈值。

敏感信息不要提交到仓库。真实环境应复制一份本地配置，例如：

```text
config/config.local.yaml
```

并将真实 IP、账号、端口和磁盘映射关系放入本地配置。

## 报告样例

报告样例位于：

```text
reports/report.example.md
```

报告需要区分：

1. 正常。
2. 超阈值。
3. SSH 不可达。
4. 命令执行失败。
5. 配置缺失。
6. 信息不足，无法判断。

## 后续手动运行方式

后续脚本完成后，预期手动运行方式如下：

```bash
python vm_health_check.py --config config/config.local.yaml --output reports/latest.md
```

当前还没有实现该脚本。今天的完成标准是先固化任务边界、配置样例和报告样例。

## 只读约束

后续实现必须遵守：

1. 只执行巡检读取操作。
2. 不清理磁盘。
3. 不修改配置。
4. 不重启服务、VM 或宿主机。
5. 不自动迁移、扩容或修复。
6. 任意单点失败时继续检查其他对象。

## 下一步

下一次可让 Agent 基于本目录继续生成第一版脚本骨架：

1. 读取 YAML 配置。
2. 遍历 VM 和 PVE 宿主机。
3. 先用 mock 数据生成报告。
4. 再逐步接入真实 SSH。
