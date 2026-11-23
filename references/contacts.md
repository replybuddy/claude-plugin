# Contacts: clients, tags, notes and history

A **contact** is one client: from a messenger chat, or added by hand with `create_contact`. `list_contacts` finds them with filters; `get_contact` shows a contact with its conversation and its open deal, or its latest deal when none is open. `update_contact` changes its fields; changing the assignee also hands the contact's latest conversation to that assignee, under the same rules as `assign_conversation_ai` and `assign_conversation_user`. Only a contact added by hand can be deleted (`delete_contact`).

## Tags

Tags belong to the contact. A conversation shows its contact's tags, so `add_conversation_tag` and `add_contact_tag` change the same set, as do their `remove_` and `replace_` counterparts.

| Task | Tools |
|---|---|
| The workspace's tags | `list_tags`, `create_tag`, `update_tag` (name, color), `delete_tag` (removes it from every contact and conversation) |
| Tags of one contact | `list_contact_tags`, `add_contact_tag` (keeps the others), `remove_contact_tag`, `replace_contact_tags` (replaces all) |

## Notes, history and deals

- Notes: `list_contact_notes`, `create_contact_note`, `update_contact_note`, `delete_contact_note`. The last two take `platform` and `platformUserId` from `list_contact_notes`.
- `list_contact_history` lists the change history of a contact.
- `list_contact_deals` lists the deals linked to a contact; `link_deal_contact` links it to a deal, see [crm.md](crm.md).
