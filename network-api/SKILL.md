# 听云 Network API 接口技能

听云 Network 产品（network.tingyun.com）全部后端 API 接口文档与调用指南。
涵盖任务管理（创建、复制、修改、状态更新、列表查询）的完整接口契约。

【触发词】"network 接口""network API""听云 network""Network 平台""network.tingyun.com"
"复制任务""创建任务""修改任务""任务列表""任务管理""批量任务""network-report-data"
"authkey""copy-json-authkey""edit-json-authkey""add-json-authkey""list-json-authkey"
"任务有效期""监测频率""自定义计划""自定义执行计划""节点组""套餐ID"
"update-json-authkey""detail_id""probgrp_id""Interval"
以及用户提到需要在听云 Network 平台管理监控任务、批量配置任务、操作任务 API 的场景

## 全局约定

### 鉴权
- 所有接口都需要 `authkey` 参数（String，必填）
- authkey 是平台授权的凭证，需在 Network 平台获取
- authkey 错误时返回 `status: 403`，错误信息 `PermissionDeniedError`

### 请求格式
- 所有接口统一使用 **POST** 方法
- Content-Type: **application/x-www-form-urlencoded**
- 参数以 form data 方式提交

### 响应格式
所有接口返回统一的 JSON 结构：
```json
{
    "status": 200,        // 状态码: 200=成功, 403=授权码错误
    "data": [],           // 返回数据（任务ID或任务列表）
    "message": "",        // 错误信息
    "error_code": 0,      // 错误码: 0=成功
    "request": "",        // 请求追踪
    "time": "2021-04-15 08:56:31"  // 响应时间
}
```

---

## 接口清单

### 1. 任务列表

| 项目 | 内容 |
|------|------|
| **接口地址** | `https://network.tingyun.com/network-report-data/task-config/list-json-authkey` |
| **方法** | POST |
| **用途** | 查询已有任务列表，获取任务 ID 用于后续操作 |

**请求参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `authkey` | String | ✅ | 授权码 |
| `detail_id` | Int | 否 | 套餐 ID（与 task_id 二选一） |
| `task_id` | Int | 否 | 任务 ID（与 detail_id 二选一） |
| `status` | Int | 否 | 任务状态：0=禁用, 1=启用 |

**响应字段（data 数组中每个任务对象）：**

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | Int | 任务 ID |
| `name` | String | 任务名称 |
| `url` | String | 任务 URL |
| `ctime` | Date | 创建时间 |
| `expire` | Date | 任务有效期（yyyy-MM-dd） |
| `type` | Int | 任务类型 |
| `task_option` | String | 流媒体类型（V/H/M/A/D/VLC） |
| `status` | Int | 状态：0=禁用, 1=启用 |
| `detail_id` | Int | 套餐 ID |
| `probgrp_id` | Int | 节点组 ID |
| `Interval` | Int | 监测频率（秒） |

**调用流程：**
1. 调用任务列表 API
2. 获取关键参数 `id`（任务ID）

---

### 2. 任务复制

| 项目 | 内容 |
|------|------|
| **接口地址** | `https://network.tingyun.com/network-report-data/task-config/copy-json-authkey` |
| **方法** | POST |
| **用途** | 复制已有任务配置，只需指定新任务名称 |

**请求参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `authkey` | String | ✅ | 授权码 |
| `copy_task` | Int | ✅ | 要复制的源任务 ID |
| `name` | String | ✅ | 新任务名称，多个以英文逗号 `,` 分隔，任务名称个数必须与 URL 个数相同 |

**响应示例：**
```json
{
    "status": 200,
    "data": [72886],
    "message": "",
    "error_code": 0,
    "request": "",
    "time": "2021-04-15 08:56:31"
}
```
`data` 数组中的数字是复制后生成的新任务 ID。

**API 依赖：**
- `copy_task` → 需要先调用**任务列表 API** 获取任务 ID

**错误说明：**

| 字段 | 错误信息 |
|------|----------|
| `copy_task` | 复制任务的编号为必填选项 |

> ⚠️ 注意：复制接口**只能改任务名称**，其他字段（URL、有效期、监测频率等）沿用源任务配置。
> 如需修改这些字段，复制后需再调用**任务修改 API**。

---

### 3. 任务修改

| 项目 | 内容 |
|------|------|
| **接口地址** | `https://network.tingyun.com/network-report-data/task-config/edit-json-authkey` |
| **方法** | POST |
| **用途** | 修改已有任务的配置（名称、URL、有效期、监测频率等全部参数） |

**请求参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `authkey` | String | ✅ | 授权码 |
| `id` | Int | ✅ | 任务 ID |
| `name` | String | 否 | 任务名称，多个以逗号分隔，个数须与 URL 个数相同 |
| `url` | String | 否 | 任务 URL，多个以逗号分隔，个数须与任务名称个数相同 |
| `expire` | Date | 否 | 任务有效期，格式 yyyy-MM-dd |
| `probgrp_id` | Int | 否 | 节点组 ID |
| `Interval` | Int | 否 | 监测频率（秒）：60/300/600/900/1800/3600/7200/10800/14400/21600/28800/43200/86400 |
| `bs_type` | String | 否 | 浏览器内核：I=IE, C=Chrome, F=Firefox |
| `timeout` | Int | 否 | 超时时间，范围 1~300，默认 120 |
| `ext_wait` | Int | 否 | 额外等待时间，范围 0~5，默认 0 |
| `client_redirects` | Int | 否 | 客户端重定向次数，默认 0 |
| `do_ping` | Int | 否 | 执行 Ping 监测：0=不开启, 1=页面错误时, 3=总是, 默认 0 |
| `ping_size` | Int | 否 | Ping 包大小，范围 0~65500 |
| `ping_count` | Int | 否 | Ping 次数，范围 0~50 |
| `if_tcp_ping` | Int | 否 | 是否 TCP Ping：0=否, 1=是, 默认 0 |
| `rst_ok` | Int | 否 | TCP 连接重置时视为 Ping 到达：0=否, 1=是 |
| `do_nslookup` | Int | 否 | 执行 NSLookup：0=不开启, 1=错误时, 3=总是, 默认 0 |
| `do_tracert` | Int | 否 | 执行 Traceroute：0=不开启, 1=错误时, 3=总是, 默认 0 |
| `do_scrshot` | Int | 否 | 执行页面截屏：0=不开启, 1=页面错误时, 2=错误或内容错误时, 3=总是, 默认 0 |
| `do_src` | Int | 否 | 上传 HTML 源代码：0/1/2/3，同上 |
| `do_head` | Int | 否 | 保留 HTTP 头：0=不保留, 1=仅主元素, 2=所有元素响应头, 3=请求+响应头, 默认 0 |
| `do_pcap` | Int | 否 | 是否抓包：0/1/2/3，同上 |
| `check_dns` | Int | 否 | 检查 DNS 服务器设置：0=否, 1=是, 默认 1 |
| `support_ipv6` | Int | 否 | 指定 IPv6 节点：0=混合, 1=仅 IPv6, 2=仅 IPv4, 默认 2 |
| `dns_match` | Int | 否 | DNS 协议：0=自动, 1=与节点一致, 默认 0 |
| `lan_task` | Int | 否 | 内网监测：0=否, 1=是, 默认 0 |
| `if_bandwidth` | Int | 否 | 带宽选项：0=不启用, 1=启用, 默认 0 |
| `bandwidth_min` | Int | 否 | 带宽最小值：0(不限制)/2/5/10/20/30/50/100 |
| `bandwidth_max` | Int | 否 | 带宽最大值：0(不限制)/2/5/10/20/30/50/100 |
| `ignoreHostIdStr` | String | 否 | 排除监测节点（按 ID），多个以换行符 `\n` 分隔 |
| `ignoreProbeStr` | String | 否 | 排除监测节点（按 IP），多个以换行符 `\n` 分隔 |
| `containsHostIdStr` | String | 否 | 包含监测节点（按 ID），多个以换行符 `\n` 分隔 |
| `containsProbeIpStr` | String | 否 | 包含监测节点（按 IP），多个以换行符 `\n` 分隔 |
| `os_bs_filter_mode` | Int | 否 | OS/浏览器筛选模式：0=排除, 1=包含, 默认 1 |
| `osBsFilterStr` | String | 否 | OS/浏览器筛选配置字符串 |
| `req_plugin_opt` | String | 否 | 播放器类型：V=Flash, D=Flash Audio, M=Windows |

**响应示例（同复制接口）：**
```json
{
    "status": 200,
    "data": [72886],
    "message": "",
    "error_code": 0,
    "request": "",
    "time": "2021-04-15 08:56:31"
}
```

**API 依赖：**
- `id` → 需要先调用**任务列表 API** 获取任务 ID
- `probgrp_id` → 如需修改节点组，需调用**节点组列表 API** 获取

---

### 4. 任务状态更新

| 项目 | 内容 |
|------|------|
| **接口地址** | `https://network.tingyun.com/network-report-data/task-config/update-json-authkey` |
| **方法** | POST |
| **用途** | 批量启用/禁用任务 |

**请求参数：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `authkey` | String | ✅ | 授权码 |
| `task_ids` | String | ✅ | 任务 ID 列表，逗号分隔 |
| `status` | Int | ✅ | 状态：0=禁用, 1=启用 |

**响应示例：**
```json
{
    "status": 200,
    "message": "",
    "error_code": 0,
    "request": "",
    "time": "2021-04-15 06:13:47"
}
```

---

### 5. 任务创建

| 项目 | 内容 |
|------|------|
| **接口地址** | `https://network.tingyun.com/network-report-data/task-config/add-json-authkey` |
| **方法** | POST |
| **用途** | 创建全新任务 |

**核心请求参数（必填）：**

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `authkey` | String | ✅ | 授权码 |
| `type` | Int | ✅ | 任务类型：1=页面类型 |
| `detail_id` | Int | ✅ | 套餐 ID |
| `name` | String | ✅ | 任务名称，多个逗号分隔 |
| `url` | String | ✅ | 任务 URL，多个逗号分隔 |
| `expire` | Date | ✅ | 任务有效期 yyyy-MM-dd |
| `probgrp_id` | Int | ✅ | 节点组 ID |
| `Interval` | Int | ✅ | 监测频率（秒），见下表 |

**监测频率 (Interval) 取值：**

| 值 | 含义 | 值 | 含义 |
|----|------|----|------|
| 60 | 1 分钟 | 1800 | 30 分钟 |
| 300 | 5 分钟 | 3600 | 1 小时 |
| 600 | 10 分钟 | 7200 | 2 小时 |
| 900 | 15 分钟 | 10800 | 3 小时 |
| 14400 | 4 小时 | 28800 | 8 小时 |
| 21600 | 6 小时 | 43200 | 12 小时 |
| 86400 | 24 小时 | | |

**其余可选参数** 与任务修改接口一致（参见第 3 节）。

**API 依赖：**
- `detail_id` → 需调用**套餐列表 API** 获取
- `probgrp_id` → 需调用**节点组列表 API** 获取

---

## 典型使用场景

### 场景 A：批量复制任务 + 修改配置

```
步骤 1：任务列表 API → 获取源任务 ID
步骤 2：任务复制 API → 传入 copy_task + 新 name → 获得新任务 ID
步骤 3：任务修改 API → 传入新任务 ID + 修改 url/expire/Interval 等
步骤 4（可选）：任务状态更新 API → 启用新任务
```

### 场景 B：基于模板快速创建 72 个监控任务

```
对于每个比赛：
  1. 复制基础任务模板 → 用任务名称区分
  2. 修改 URL 为比赛对应的直播页链接
  3. 修改有效期为比赛当天
  4. 修改自定义计划时间为比赛前 1h ～ 比赛后 2h（共 3h 窗口）
```

---

## 自定义计划（Custom Plan）

「自定义计划」是 Network 平台的高级调度功能，用于设置任务在特定时间段内执行。
相关 API 参数正在收集中，当前可通过以下方式实现时间段控制：

- **方案 1**：在任务修改 API 中调整 `Interval` 为较高频率（如 60s），任务持续运行但只在比赛时段关注数据
- **方案 2**：通过平台 UI 手动配置自定义计划模板，再通过 API 复制已配置计划的任务

> 📌 如果你有自定义计划 API 的具体文档或抓包数据，请提供给我，我会补充到本技能中。

---

## 参考资料

- 官方 API 文档站：https://network.tingyun.com/network-doc-web/
- GitBook 详细文档：http://wukongdoc.tingyun.com/network/api/task/create/
- API 服务总览：http://releasenotes-network.tingyun.com/7629035m0
- Network 平台入口：https://network.tingyun.com/
