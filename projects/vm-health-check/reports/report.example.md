# VM Health Check Report

巡检时间：2026-10-30 06:30:00

产品名称：example-product

巡检模式：示例报告，未连接真实机器

## 总览

| 项目 | 结果 |
| --- | --- |
| VM 数量 | 3 |
| VM SSH 正常 | 2 |
| VM SSH 失败 | 1 |
| VM 磁盘超 85% | 1 |
| VM iostat 繁忙 | 1 |
| PVE 宿主机数量 | 2 |
| 宿主机磁盘超 80% | 1 |
| 无法判断项 | 2 |

## VM SSH 连通性

| VM | 地址 | 结果 | 说明 |
| --- | --- | --- | --- |
| vm-01 | 10.0.0.1 | 正常 | SSH 登录成功 |
| vm-02 | 10.0.0.2 | 正常 | SSH 登录成功 |
| vm-03 | 10.0.0.3 | SSH 不可达 | 连接超时，已跳过该 VM 的 df 和 iostat 检查 |

## VM 磁盘使用率

### vm-01

| 文件系统 | 容量 | 已用 | 可用 | 使用率 | 挂载点 | 判断 |
| --- | --- | --- | --- | --- | --- | --- |
| /dev/mapper/openeuler-root | 2.0T | 164G | 1.7T | 9% | / | 正常 |
| /dev/sdb1 | 2.0T | 846G | 1.2T | 42% | /opt/local-path-provisioner | 正常 |
| /dev/sde1 | 4.0T | 3.6T | 400G | 90% | /opt/local-path-provisioner/data/data-clickhouse | 超阈值 |

### vm-02

| 文件系统 | 容量 | 已用 | 可用 | 使用率 | 挂载点 | 判断 |
| --- | --- | --- | --- | --- | --- | --- |
| /dev/mapper/openeuler-root | 2.0T | 400G | 1.6T | 20% | / | 正常 |
| /dev/sdb1 | 2.0T | 1.1T | 900G | 55% | /opt/local-path-provisioner | 正常 |

### vm-03

未采集。原因：SSH 不可达。

## VM iostat

初版判断规则：

1. `%util >= 80` 标记为磁盘繁忙。
2. `await >= 50ms` 标记为 I/O 等待偏高。

| VM | 设备 | %util | await(ms) | 判断 |
| --- | --- | --- | --- | --- |
| vm-01 | sde | 91.20 | 62.50 | 磁盘繁忙，I/O 等待偏高 |
| vm-02 | sdb | 18.30 | 6.20 | 正常 |
| vm-03 | - | - | - | SSH 不可达，无法判断 |

## PVE 宿主机检查

| 宿主机 | 访问模式 | 地址 | 结果 | 说明 |
| --- | --- | --- | --- | --- |
| pve-node-01 | ssh | 10.0.1.1 | 正常 | SSH 采集成功 |
| pve-node-02 | pve_web | https://10.0.1.2:8006 | 无法判断 | Web 模式自动采集策略待实现 |

## 宿主机物理磁盘使用率

| 宿主机 | 物理磁盘 | 使用率 | 阈值 | 判断 | 关联 VM |
| --- | --- | --- | --- | --- | --- |
| pve-node-01 | /dev/sdb | 65% | 80% | 正常 | vm-01 |
| pve-node-01 | /dev/sdc | 83% | 80% | 超阈值 | vm-01 |
| pve-node-02 | /dev/sdb | - | 80% | 无法判断 | vm-03 |

## 异常汇总

1. vm-01 的 `/opt/local-path-provisioner/data/data-clickhouse` 使用率为 90%，超过 85% 阈值。
2. vm-01 的 `sde` 设备 `%util` 为 91.20，`await` 为 62.50ms，磁盘读写繁忙。
3. vm-03 SSH 不可达，无法采集 VM 磁盘和 iostat。
4. pve-node-01 的 `/dev/sdc` 使用率为 83%，超过 80% 阈值。
5. pve-node-02 使用 `pve_web` 模式，自动采集策略待实现。

## 建议人工检查动作

1. 登录 vm-01，确认 ClickHouse 数据目录增长原因。
2. 检查 vm-01 上 `sde` 对应业务组件是否存在批量写入或查询压力。
3. 登录 pve-node-01，确认 `/dev/sdc` 与 VM 磁盘映射是否准确。
4. 排查 vm-03 SSH 不可达原因。
5. 确认 pve-node-02 的 Web 模式后续是接入 PVE API，还是仅保留人工查看入口。

## 无法判断项

1. vm-03 因 SSH 不可达，无法判断磁盘和 iostat 状态。
2. pve-node-02 因当前为 `pve_web` 模式，尚未实现自动采集。
