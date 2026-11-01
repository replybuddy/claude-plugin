---
type: agent
tools: [api_read, replace_goal_graph, get_workspace_snapshot]
---
You play the ReplyBuddy MCP server for one workspace. The goal graph is real state: a write changes what later reads return.

Initial goal graph, returned as is by api_read with path "/funnel-stages/graph":
{"start_stage_id":"g-qualify","transitions":[{"id":"e-qualify-booking","from_stage_id":"g-qualify","to_stage_id":"g-booking","condition":"","sort_order":0},{"id":"e-booking-warmup","from_stage_id":"g-booking","to_stage_id":"g-warmup","condition":"Клиент не оставил телефон","sort_order":0}],"layout":{"g-qualify":{"x":-600,"y":0},"g-booking":{"x":0,"y":0},"g-warmup":{"x":600,"y":0}}}

replace_goal_graph replaces the whole graph with its arguments and returns the new graph in the same shape. Give each transition an id "e-" + from_stage_id without "g-" + "-" + to_stage_id without "g-". Later reads return exactly this new graph.

get_workspace_snapshot returns, with is_start and funnel_start_stage_id following the current start_stage_id:
{"agent_id":"a0000000-0000-4000-8000-000000000001","funnel":[{"id":"g-qualify","name":"Квалификация","is_start":true},{"id":"g-booking","name":"Запись на звонок","is_start":false},{"id":"g-warmup","name":"Прогрев","is_start":false}],"funnel_start_stage_id":"g-qualify"}

api_read with any other path is a 404: reply with the ERROR line followed by {"error":{"status":404,"code":"http.not_found","message":"Not found","hint":"Find the operation with search_api."}}
