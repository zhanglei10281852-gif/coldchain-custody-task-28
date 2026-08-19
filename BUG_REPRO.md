# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

同一 outbox 任务在第一次领取尚未完成时可以再次被领取，attempts 连续增长并触发重复处理。请先不要修改代码，定位领取查询与条件更新为何允许 running 任务再次进入，并给出两次领取证据。 调查全程不要修改目标仓库中的生产代码、测试代码或配置。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-28
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-28.git
- parent SHA：1729ebaccfaad9e84b50d48f4777bebc706dbd45

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-28.git bug-repro
cd bug-repro
git checkout --detach 1729ebaccfaad9e84b50d48f4777bebc706dbd45
go test ./internal/storage/sqlite -run "^TestJobCannotBeClaimedTwice$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run "^TestJobCannotBeClaimedTwice$" -count=1
--- FAIL: TestJobCannotBeClaimedTwice (0.12s)
    annotation_behavior_test.go:62: second claim = [{ID:job_once Kind:shipment_planned AggregateID:ship_once Payload:[123 125] Status:running Attempts:2 MaxAttempts:3 AvailableAt:2026-08-18 08:00:00 +0000 UTC LockedAt:2026-08-18 08:00:00 +0000 UTC LastError: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC}]
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/storage/sqlite	0.130s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/storage/sqlite -run "^TestJobCannotBeClaimedTwice$" -count=1
--- FAIL: TestJobCannotBeClaimedTwice (0.30s)
    annotation_behavior_test.go:62: second claim = [{ID:job_once Kind:shipment_planned AggregateID:ship_once Payload:[123 125] Status:running Attempts:2 MaxAttempts:3 AvailableAt:2026-08-18 08:00:00 +0000 UTC LockedAt:2026-08-18 08:00:00 +0000 UTC LastError: CreatedAt:2026-08-18 08:00:00 +0000 UTC UpdatedAt:2026-08-18 08:00:00 +0000 UTC}]
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/storage/sqlite	0.492s
FAIL

```

stderr：

```text
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/storage/sqlite/store_test.go has type 100755, expected 100644

```

## 通过条件

准确定位根因，指出具体 Go 文件和符号，解释错误行为如何导致题面症状，并给出实际复现、调用链或持久化证据。 完成时目标仓库代码、测试和配置零改动。
