# Broadcasts: one message to a chosen audience

A broadcast sends one message to the clients an audience filter selects. Nothing is sent until `schedule_broadcast` or `send_broadcast`.

1. `create_broadcast` makes a draft with audience filters and the message; `update_broadcast` edits it. `generate_broadcast_message` writes a text with AI: poll `get_broadcast_message_job` for it.
2. `count_broadcast_audience` counts the clients the filter selects and `preview_broadcast_audience` shows a sample of them; neither sends anything.
3. For an AI broadcast, `start_broadcast_preview` personalizes the message for one conversation of the audience (`conversation_id`): poll `get_broadcast_preview_job`, then `confirm_broadcast_preview` accepts it so the broadcast can be scheduled or sent. A static broadcast sends its text as written and has no preview.
4. `schedule_broadcast` or `send_broadcast` starts it.

While it runs: `list_broadcasts` shows status and schedule, `pause_broadcast` and `resume_broadcast` stop and continue sending, `cancel_broadcast` cancels it. `duplicate_broadcast` copies one into a new draft; `delete_broadcast` deletes one that is not sending.
