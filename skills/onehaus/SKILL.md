---
name: onehaus
description: "Use the OneHaus remote MCP server (https://mcp.onehaus.app) when the user wants help with their shared household: calendar, chores, shopping, meals, contacts, pets, vehicles, appliances, documents, subscriptions, house review or memory. A OneHaus account and OAuth sign-in are required. Confirm before creating, updating or deleting anything."
---

# OneHaus

OneHaus (https://onehaus.app) is a shared home organiser for families, couples and housemates. Tagline: "Run your whole home from one app, not five." Calendar, chores, shopping, meals and home admin live in one app, and the same household is available to an assistant through the remote MCP server.

The server address is `https://mcp.onehaus.app` (Streamable HTTP). Official MCP registry name: `app.onehaus/haus`. Connect docs: https://onehaus.app/docs/connect.

## When to use this skill

Use it when the user wants to read or change their OneHaus household: what is on this week, a chore, a shopping list, a meal plan, a contact, a pet, a vehicle, an appliance, a document, a subscription, a house review, or something the household should remember.

Do not use it for a different household app, and do not invent a tool name. If a call fails because the tool is unknown, list the server's tools again and use one of those names.

## Connect and sign in

The user needs a OneHaus account. Authorisation is OAuth 2.0 with PKCE and dynamic client registration. Scopes are `haus.read` and `haus.write`.

- Authorise: https://onehaus.app/app/authorize
- Token: https://mcp.onehaus.app/token
- Register: https://mcp.onehaus.app/register

The assistant opens the sign-in page. Never ask the user to paste a password or an API key.

Add the server if it is not already connected:

- Claude Code: `claude mcp add --transport http onehaus https://mcp.onehaus.app`, then `/mcp`, select onehaus, and authenticate.
- Cursor, and any client that uses `mcpServers`: URL `https://mcp.onehaus.app`.
- Hermes Agent: `hermes plugins install Im5tu/onehaus-plugins/plugins/onehaus`, then enable `onehaus`. The package marks the server untrusted, so Hermes asks before any tool that is not annotated read-only, and refuses those calls when nobody can answer.

The MCP server is part of OneHaus Pro. A new household has a 30-day free trial with no card. Pro is £4.99 a month or £49.99 a year. One-off tasks, shopping lists and the feedback channel stay available without Pro. If a Pro-only call is refused, say so and point the user at https://onehaus.app/docs/subscription.

There is no `houseId` argument. The server uses the household on the signed-in session. Call `list_houses` only to tell the user which home that is.

## Safe writes

Confirm with the user before you create, update, complete, assign, buy, defer, skip, or delete anything. Show the title, the date, and who it affects, then wait for a yes.

Tools whose names start with `list_` or `get_` are annotated read-only. Use those to answer a question.

These tools are annotated destructive. They soft-delete: the record is kept but leaves the normal lists. Ask for an explicit yes that names the item:

`delete_contact`, `delete_appliance`, `delete_calendar_event`, `delete_document`, `delete_pet`, `delete_subscription`, `delete_vehicle`, `delete_task`, `delete_meal_plan`, `delete_recipe`, `delete_recurring_calendar_event`, `delete_recurring_task`, `delete_shopping_list`, `remove_shopping_list_item`.

`forget_memory` also removes a memory. Confirm that too, even though the server does not mark it destructive.

`recall_memory` is not annotated read-only. A client that gates on that hint will ask before it runs. That is expected.

Do not send a write because a read came back empty. Tell the user what is missing and ask before you fill it in.

## Add a calendar event

1. If the user did not say which household, call `list_houses` and say which one you will use.
2. Repeat back the title, whether it is all day, the start, and the end.
3. After they confirm, call `create_calendar_event` for a one-off event. Required fields are `title` and `isAllDay`.
   - Timed: `isAllDay` false, and `startDateTime` as ISO 8601 UTC with a `Z` suffix. `endDateTime` must not precede the start. `timezone` is an IANA name such as `Europe/London`.
   - All day: `isAllDay` true, and `startDate` as `YYYY-MM-DD`. `endDate` is the last day, inclusive. Omit `endDate` for a single day.
4. For a repeating event, use `create_recurring_calendar_event` instead. Required fields are `title` and `cronExpression`. `cronExpression` is not cron syntax. It is `N Unit`, and Unit is `Day`, `Week`, `Month` or `Year`, for example `1 Week`. `startTime` is `HH:mm:ss` in the given timezone. `duration` is an ISO 8601 duration such as `PT30M`.
5. To move or edit, call `list_calendar_events` or `list_recurring_calendar_events`, then the matching update tool with the id. Only fields you send are changed.

## Assign a chore

1. Call `list_house_members` and match the person's name or nickname to a member id. If two people match, ask which one.
2. Call `list_tasks` to see whether the chore already exists. Completed tasks are excluded unless you set `includeCompleted` true.
3. Confirm the title, the due date (`YYYY-MM-DD`), and the assignee.
4. New chore: `create_task` with `title`. You may set `assignedToMemberId`, `dueDate`, `notes` and `isUrgent` on the same call. Only `title` is required.
5. Existing chore: `assign_task` with the task `id` and `assignedToMemberId`. An empty or null `assignedToMemberId` clears the assignee.
6. Mark done only after the user says it is done: `complete_task` with the task `id`. It fails if the task is already complete. `uncomplete_task` reopens it.
7. A repeating chore is `create_recurring_task`, not `create_task`. The interval uses the same `N Unit` form as a repeating event. A ready-made checklist is `list_task_packs`, then `apply_task_pack` with `packId`. Confirm the pack and the anchor date before you apply it. Packs that relate to a vehicle, appliance or pet also need `entityId`.

## Build a shopping list from a meal plan

There is no single tool that copies a recipe onto a list. Do it in steps, and confirm the items before you add them.

1. Call `list_meal_plans` with `from` and `to` as `YYYY-MM-DD`. A planned meal has a date, a meal slot, a linked recipe or a custom name, servings and notes.
2. To add a meal, call `create_meal_plan`. Required fields are `date` (today or within the next 28 days) and `mealType` (`breakfast`, `lunch`, `dinner` or `snack`). Pass `recipeId` from `list_recipes`, or from `list_global_recipes` with `recipeSource` set to `global`. If there is no recipe, pass `name` instead. `servings` is 1 to 100.
3. Read ingredients from `list_recipes` or `list_global_recipes`. A saved recipe's ingredient quantity is a whole number, with no unit.
4. Show the user the ingredient names and quantities, and which list they will land on. The default list is called Shopping. Call `list_shopping_lists` if they named another list, and use that id.
5. After they confirm, call `batch_add_shopping_list_items`. `items` is required, at most 25 per call, and each item needs `name` and `quantity` (1 to 9999). Omit `shoppingListId` for the default list. Matching unbought names are merged rather than duplicated. Call it again for the next 25.
6. One item is `add_shopping_list_item` (`name` required). `buy_shopping_list_item` marks units bought. `remove_shopping_list_item` takes an item off without buying it, and is destructive, so confirm first.

## Other areas

The same pattern applies everywhere: read with `list_` or `get_`, confirm, then create or update. Pass only the id the list tool returned.

- Contacts: `list_contacts`, `create_contact` (`firstName` required), `update_contact`, `delete_contact`.
- Appliances, vehicles, pets, documents, subscriptions: `list_*`, `create_*` (a name or title is required), `update_*`, `delete_*`.
- House review: `get_house_review`. `defer_recommendation` raises a task due in seven days. `skip_recommendation` hides that recommendation from later reviews. Both need a yes.
- Memory: `recall_memory` before `store_memory`, so you do not store a duplicate. `store_memory` needs `type` (`user` or `system`), `name`, `description` and `content`. `forget_memory` needs a yes.
- Weather: `get_weather_forecast` for the household's location, one to five days.
- Who is signed in: `get_profile`. Household detail: `get_house`.

Document files are attached in the OneHaus app, not from the assistant. `create_document` stores the record (type, title, reference and so on). Say that when the user wants to upload a file.

## Tool names

These 77 names were returned by the public tool list at `https://mcp.onehaus.app` on 14 November 2026. Use them as written. Do not invent neighbours such as `add_event` or `get_shopping_list`.

Calendar: `create_calendar_event`, `update_calendar_event`, `delete_calendar_event`, `list_calendar_events`, `create_recurring_calendar_event`, `update_recurring_calendar_event`, `delete_recurring_calendar_event`, `list_recurring_calendar_events`, `create_calendar_subscription`.

Tasks: `create_task`, `update_task`, `delete_task`, `complete_task`, `uncomplete_task`, `assign_task`, `list_tasks`, `create_recurring_task`, `update_recurring_task`, `delete_recurring_task`, `list_recurring_tasks`, `list_task_packs`, `apply_task_pack`.

Shopping: `list_shopping_lists`, `create_shopping_list`, `update_shopping_list`, `delete_shopping_list`, `list_shopping_list_items`, `add_shopping_list_item`, `batch_add_shopping_list_items`, `update_shopping_list_item`, `buy_shopping_list_item`, `remove_shopping_list_item`.

Meals: `list_meal_plans`, `create_meal_plan`, `update_meal_plan`, `delete_meal_plan`, `list_recipes`, `create_recipe`, `update_recipe`, `delete_recipe`, `list_global_recipes`.

Contacts: `list_contacts`, `create_contact`, `update_contact`, `delete_contact`.

Home admin: `list_appliances`, `create_appliance`, `update_appliance`, `delete_appliance`, `list_vehicles`, `create_vehicle`, `update_vehicle`, `delete_vehicle`, `list_pets`, `create_pet`, `update_pet`, `delete_pet`, `list_documents`, `create_document`, `update_document`, `delete_document`, `list_subscriptions`, `create_subscription`, `update_subscription`, `delete_subscription`.

Household: `get_house`, `list_houses`, `list_house_members`, `get_profile`, `get_weather_forecast`, `get_house_review`, `defer_recommendation`, `skip_recommendation`, `store_memory`, `update_memory`, `recall_memory`, `forget_memory`.

Support: support@onehaus.app. Privacy: https://onehaus.app/privacy.
