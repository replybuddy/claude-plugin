# Analytics: clients and deals over a period

Both analytics tools compare a period with another one.

- `get_user_analytics` counts new and active clients and their incoming and outgoing messages, by day, source and tag.
- `get_deal_analytics` covers deals: conversion by stage, sources, managers, loss reasons, revenue, tasks and first response.

For one stage's current load (conversations, open deals, queued reminders) use `get_workspace_overview`; for tracking link clicks, `get_tracking_link_stats` (see [marketing.md](marketing.md)).
