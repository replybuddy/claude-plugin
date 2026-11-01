---
runs: 3
max_turns: 15
allowed_tools: [Read, Write, Glob, Grep, Skill]
---
Эндпоинт вебхука ReplyBuddy на событие deal.moved у нас уже создан, секрет лежит в переменной окружения REPLYBUDDY_WEBHOOK_SECRET. Напиши приёмник на Python и FastAPI в файле webhook_receiver.py. Он должен проверять, что запрос правда от ReplyBuddy, и на каждое событие deal.moved вызывать crm.process_deal_moved(deal_id, from_stage_id, moved_by). Эта функция уже есть в нашем модуле crm и работает около минуты. В ReplyBuddy ничего не меняй.
