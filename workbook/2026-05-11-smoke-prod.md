---
date: 2026-05-11
test: smoke PROD (after pilot→new-architecture merge)
purpose: end-to-end через aist_me_bot @aist_me_bot
---

# Smoke PROD: контур работает на проде

Этот файл триггерит:
1. push → gateway-mcp (mcp.aisystant.com)
2. gateway форвардит на aist_me_bot (prod)
3. prod bot уже имеет новый код (merge 822a21b)
4. app_installation_id=121117693 mapping есть → App-path вместо legacy
