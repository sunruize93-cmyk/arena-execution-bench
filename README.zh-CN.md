# Arena Execution Bench

[English](README.md)

一个离线、可复现的 Agent 支付执行市场：评测 Agent 在费用、失败风险、异步确认和资金占用之间如何选择。

v0.1 已实现模拟环境、基线、机制实验和开放接口。所有资产、供应商与结算记录均为模拟数据；运行不需要钱包、链节点、GPU 或模型 API key。付费模型 pilot 尚未运行。

## 快速运行

```bash
git clone https://github.com/sunruize93-cmyk/arena-execution-bench.git
cd arena-execution-bench
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -e '.[dev]'
aeb run --suite mechanism-v1 --policy expected-cost --seed 7 --out runs/first
python examples/no_key_demo.py
```

需要 Python 3.10 或更新版本。Windows 使用 `.venv\Scripts\Activate.ps1` 激活环境；可选子进程模型适配器目前要求 POSIX 系统。

## 已实现

- 整数双重记账，区分 available、reserved、spent 和 receivable；退款到账才变成可用余额。
- 真实状态与可见凭证分开：`submission_unknown` 不会被当作失败，也不会因本地超时释放资金。
- Guarded 默认阻止危险的新授权；Diagnostic 只在离线模拟中允许重付生效。两条轨道均保留预算保护。
- 风险阈值、迟到确认、资金占用、综合采购四类场景；每个 train/dev/eval split 含 8 个条件。
- 多种规则／统计基线；另有适用范围明确的单任务精确 DP，以及故意错误重付的诊断策略。
- 公共轨迹、评估器轨迹、模型决策记录、逐 epoch checkpoint、指标重算、确定性重放、按 episode 配对区间。
- 可选模型接口与调用限额；无 key 测试适配器和自选服务的 HTTPS Chat Completions 示例。
- 标记为 synthetic 的 Arena replay 文件导出。

## 复现实验

```bash
python scripts/reproduce.py --out runs/reproduction
aeb verify --episode runs/first/episodes/late-unknown-dev--seed-7
aeb replay --trace runs/first/episodes/late-unknown-dev--seed-7/events.jsonl
```

首轮为 3 个基线 × 6 个 seeds × 8 个条件，共 144 个 episode；另有 24 个错误重付诊断 episode。它们验证机制构造，不构成模型排行榜或论文结论。[结果与边界](artifacts/mechanism-v1/README.md)。

配套 Lab 尚无已接入的发布版 schema，因此本版使用明确标记的临时契约。Arena 端需要后端导入与投影；本仓库没有修改或部署 Arena402 产品前端。[接口边界](docs/INTEGRATION.md)。

本项目采用 [Apache-2.0](LICENSE)；英文文档提供完整的[执行语义](docs/SEMANTICS.md)、[实验协议](docs/EXPERIMENTS.md)和[模型接入方法](docs/AGENTS.md)。
