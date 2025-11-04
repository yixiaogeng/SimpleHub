# AGENGTS

## 概述
SimpleHub 通过一组协同工作的智能组件来完成站点监控、签到和通知等核心流程。本文件记录每个 Agent 的定位、输入输出和协作方式，方便快速上手或扩展自动化能力。

## Agent 清单

### MonitorAgent
- **目标**：周期性抓取各站点的模型、余额和状态，形成标准化的站点快照。
- **触发**：定时任务调度、手动一键检测、分类批量检测。
- **主要依赖**：`server/src/run.js`、站点凭证、模型配置缓存。
- **输出**：最新站点快照、模型差异记录、检测日志。

### CheckinAgent
- **目标**：在支持的平台执行每日签到，领取额度奖励。
- **触发**：定时任务或手动发起的签到动作。
- **主要依赖**：`server/src/checkin.js`、站点 cookie/token。
- **输出**：签到状态、领取额度、失败原因。

### BalanceGuardAgent
- **目标**：对站点余额阈值进行持续比对并生成预警。
- **触发**：MonitorAgent 完成检测后、手动余额刷新。
- **主要依赖**：站点配置中的额度阈值、自定义用量查询配置。
- **输出**：余额预警事件、通知上下文。

### NotifierAgent
- **目标**：把模型变更、余额预警、签到失败等事件转换成邮件或其他渠道通知。
- **触发**：MonitorAgent、BalanceGuardAgent、CheckinAgent 的事件回调。
- **主要依赖**：`server/src/notifier.js`、通知模板、SMTP 配置。
- **输出**：HTML 邮件、聚合日报、即时告警。

### SchedulerAgent
- **目标**：统筹各站点的检测与签到时间窗口，避免资源冲突。
- **触发**：应用启动、自定义调度配置变更。
- **主要依赖**：`server/src/scheduler.js`、Prisma 任务配置。
- **输出**：Cron/定时任务、任务执行链路日志。

### AdminConsoleAgent
- **目标**：提供前端管理界面，支撑站点 CRUD、分类管理与数据导入导出。
- **触发**：用户操作前端界面。
- **主要依赖**：`web/src/pages`、Ant Design 组件、后端 REST API。
- **输出**：用户操作指令、表单校验、可视化面板。

## 协作流程
1. SchedulerAgent 根据全局与分类配置触发 MonitorAgent、CheckinAgent。
2. MonitorAgent 将监测结果同步到数据库，并推送给 BalanceGuardAgent。
3. BalanceGuardAgent 评估余额阈值，必要时向 NotifierAgent 推送告警。
4. CheckinAgent 完成后将结果写入数据库，并告知 NotifierAgent。
5. NotifierAgent 聚合当天事件，通过邮件或其他渠道通知管理员。
6. AdminConsoleAgent 提供管理界面，支持手动触发和配置调整。

## 数据持久化
- 使用 `Prisma` 数据模型存储站点信息、检测记录、事件日志。
- 缓存层用于保存短期凭证与检测结果，避免频繁请求第三方接口。
- 所有 Agent 应以数据库为单一事实来源，确保状态一致性。

## 扩展建议
- 新增 Agent 前，优先检查是否可复用现有逻辑，避免重复实现。
- 遵循 KISS/YAGNI 原则，仅实现当前必须的自动化步骤。
- Agent 之间通过明确的事件结构解耦，新增字段时保持向后兼容。

## 运维提示
- 变更调度配置后重启 SchedulerAgent，以保证新计划生效。
- 监控 SMTP、API 凭证等敏感配置的有效期，避免通知中断。
- 定期审查日志输出，确保异常在第一时间被捕获与处理。
