# Fork 说明

本仓库是 [wangwangit/SubsTracker](https://github.com/wangwangit/SubsTracker) 的 fork，
用于自动同步上游更新并部署到自己的 Cloudflare Workers。

## 相对上游的改动

1. `wrangler.toml`：补上了 KV 命名空间绑定（`SUBSCRIPTIONS_KV`，
   id 沿用 2026-10-14 部署时生成的生产命名空间 `9aee862262da4359b33df8d84fff7bfd`，
   数据不受重新部署影响，不会另建命名空间）。
   注意：上游默认不带 KV id（靠 `npm run setup` 按标题查找自动生成），
   本 fork 把 id 写死，部署脚本 `setup` 运行时会按标题找到同一个命名空间，行为一致。
2. `.github/workflows/sync-upstream.yml`（新增）：每天北京时间 03:00 自动合并上游；
   合并成功后上游自带的 Deploy 工作流会自动跑 lint+测试并部署到 CF。
   若上游改动与本 fork 的改动冲突，同步会显式失败，需手动解决。

## 自动更新链路

`Sync upstream`（每天 03:00 北京时间 / 手动触发）
→ 合并上游到 master
→ 触发上游自带 `Deploy` 工作流（npm ci → lint+test → npm run deploy:safe）
→ 部署到 Cloudflare Worker `subscription-manager`
（https://subscription-manager.ikissu3344.workers.dev）

## 需要的 Secrets（已配置）

- `CLOUDFLARE_API_TOKEN`：CF API Token（Edit Cloudflare Workers 模板 + Workers KV Storage 编辑权限）
- `CLOUDFLARE_ACCOUNT_ID`：CF 账号 ID

## 注意

- 不要直接向本 fork 的 master 推业务代码；本地改动只保留 wrangler.toml 的 KV 绑定和 workflow 文件。
- 定时任务 `0 * * * *`（每小时）写在 wrangler.toml 的 `[triggers]`，每次部署自动生效。
