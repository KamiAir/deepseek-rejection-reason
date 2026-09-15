# deepseek-rejection-reason
Adds an optional rejection reason to DeepSeek Harness tool approvals: users can explain why they reject a request, and the reason is logged and fed back to the model.
# 审批「拒绝原因」补丁包

给 DeepSeek Harness 的工具审批弹窗增加「拒绝原因」输入：用户在「等待审批」面板点「拒绝」后，可（可选）填写拒绝原因再确认；原因会写入会话日志，并作为一条用户消息反馈给 AI，让模型知道为何被拒、便于调整做法。


## 适用版本

基于 DeepSeek Harness `0.1.5-rc.2`（根 `package.json` 的 `version`）。应用到其它版本前请先确认 `packages/interaction/user-approval`、`packages/client/ui-approval`、`packages/session/session-format-v0-to-v1` 的基线一致，否则 `git apply` 可能冲突。

## 安装

```powershell
# 1. 进入你的 DeepSeek Harness checkout
cd <path-to>/deepseek-harness

# 2. 应用补丁
git apply <path-to>/rejection-reason-plugin/rejection-reason.patch

# 3. 重建（Host + Client 产物）
pnpm run build
```

> 客户端 UI 变更在 `ui-approval` 的 `lib/client.js` 中；Host 侧变更在 `user-approval` 与 `session-format-v0-to-v1` 的产物中。重建后重启 `dsh web` 即生效。

## 改动文件清单（21 个）

### 核心实现
- `packages/interaction/user-approval/src/types.ts`
- `packages/interaction/user-approval/src/index.ts`
- `packages/session/session-format-v0-to-v1/src/dispositions.ts`
- `packages/session/session-format-v0-to-v1/src/payload-validation.ts`
- `packages/client/ui-approval/src/client/contract/slots.ts`
- `packages/client/ui-approval/src/client/ApprovalPanel.tsx`
- `packages/client/ui-approval/src/client/locales.ts`
- `packages/client/ui-approval/src/client/ApprovalPanel.module.css`
- `packages/client/ui-approval/src/client/index.ts`

### 测试
- `packages/interaction/user-approval/tests/approval.spec.ts`
- `packages/client/ui-approval/tests/ui-approval.client.spec.tsx`
- `packages/session/session-format-v0-to-v1/tests/validation.spec.ts`

### 生成器与重新生成的目录
- `scripts/gen-cordis-catalog.ts`
- `packages/extensions/tool-cordis/src/api-catalog.ts`
- `docs/subsystems/approval.md` / `approval.zh.md` / `approval.i18n.yaml`
- `docs/persistence-catalog.md`
- `docs/event-producer-consumer.md` / `event-producer-consumer.zh.md` / `event-producer-consumer.i18n.yaml`

## 架构说明（为什么是补丁而非独立 `dsh plugin`）

DSH 有分层红线：feature 插件禁止运行时 import 另一个 feature 插件的值，UI 只能通过 slot 跨包。本功能的拒绝原因必须沿 `approval/request` 瀑布从浏览器传回 Host、并写进审批 seam 自己的 `approval/decided` 事件，因此天然落在核心审批链路内（`user-approval` + `ui-approval`），无法干净地封装为独立插件。若强行做成独立插件，需要重写整个审批面板并与内置 `ui-approval` 抢占 waterfall 顺序，脆弱且会与内置审批面板冲突。
