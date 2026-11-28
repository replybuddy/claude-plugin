# Marketing: automations, lead forms and tracking links

Automations answer on their own and are not broadcasts (see [broadcasts.md](broadcasts.md)). An enabled rule messages clients without a confirmation per message.

- A **comment rule** answers a comment that has its keyword with a public reply and a direct message.
- A **story rule** answers story replies and reactions.
- An **AI reply assignment** lets the agent itself answer comments or story replies under chosen posts.
- A rule's direct message can ask for a subscription first (`subscription_gate`). A file, link or promo code the client should get goes into the message itself: its text, or an attachment in a link's or form's messages.
- A comment goes to a keyword rule first, then to auto-moderation, then to the AI reply. `preview_comment_reply` shows which one takes a given text; nothing is sent, hidden or counted, but with AI moderation on each call makes a billed model call.
- **Lead forms** collect contacts and can message the client after a submission; **tracking links** open a chat with the agent and record where clients came from.

| Task | Tools |
|---|---|
| Comment rules | `list_comment_rules`, `create_comment_rules`, `update_comment_rule`, `toggle_comment_rule`, `delete_comment_rule`, `preview_comment_reply` |
| Story rules | `list_story_rules`, `create_story_rule`, `update_story_rule`, `toggle_story_rule`, `delete_story_rule` |
| AI replies under posts | `list_ai_reply_assignments`, `create_ai_reply_assignments`, `toggle_ai_reply_surface`, `delete_ai_reply_assignment`, `get_ai_reply_settings`, `update_ai_reply_settings` |
| Comments and moderation | `list_comment_posts`, `list_comments`, `get_comment_stats`, `check_comment`, `set_comment_hidden`, `get_comment_moderation_settings`, `update_comment_moderation_settings`, `set_comment_post_auto_moderation` |
| Lead forms | `list_lead_forms`, `create_lead_form`, `update_lead_form`, `toggle_lead_form`, `archive_lead_form`, `restore_lead_form`, `list_lead_form_submissions` |
| Tracking links | `list_tracking_links`, `create_tracking_link`, `update_tracking_link`, `toggle_tracking_link`, `archive_tracking_link`, `restore_tracking_link`, `get_tracking_link_stats` (clicks recorded from 2026-11-02) |
| Unmatched traffic labels | `list_raw_labels`, `dismiss_raw_label` |
