# WebSocket Protocol Specification

MiniMart uses WebSockets to provide real-time synchronization for shopping lists and instant communication via chat.

---

## Connection Endpoints

### 1. Global Sync Scope
**URL:** `ws://<base_url>/ws?access_token=<jwt_token>`
- **Purpose:** Used for real-time item updates, membership changes, and general notifications across all lists a user is a member of.
- **Requirement:** User must be authenticated.

### 2. Chat Scope
**URL:** `ws://<base_url>/ws/shopping-lists/{list_id}/chat?access_token=<jwt_token>`
- **Purpose:** Dedicated bi-directional communication for a specific shopping list's chat room.
- **Requirement:** User must be an accepted member of the shopping list.

---

## Establishing a Connection

1. **Authentication:** The `access_token` must be provided as a query parameter during the initial handshake.
2. **Identification:** The system automatically extracts the `User-ID` and `Tenant-ID` from the token.
3. **Implicit Subscriptions:** Upon connecting to the global scope, the user is automatically subscribed to all shopping lists they currently belong to.

---

## Client-to-Server Messages

Clients communicate with the server using a consistent JSON format:
```json
{
  "type": "<message_type>",
  "payload": { ... }
}
```

### 1. Subscription Management (Global Scope Only)
| Message Type | Payload | Description |
|--------------|---------|-------------|
| `subscribe` | `{"list_id": "UUID"}` | Manually subscribe to real-time events for a specific list. |
| `unsubscribe` | `{"list_id": "UUID"}` | Stop receiving real-time events for a specific list. |

### 2. Connection Health
| Message Type | Payload | Description |
|--------------|---------|-------------|
| `ping` | `{}` | Sent by the client to keep the connection alive. |

---

## Server-to-Client Events

The server broadcasts events when resources are modified.

### 1. Item Lifecycle Events
| Event Type (`type`) | Payload Overview | Trigger |
|--------------------|------------------|---------|
| `item.added` | `id, name, quantity, status, added_by` | A new item is added to the list. |
| `item.updated` | `id, name, quantity, status` | An item's name, qty, or purchase status is changed. |
| `item.deleted` | `id` | An item is removed from the list. |

### 2. Membership & List Events
| Event Type (`type`) | Payload Overview | Trigger |
|--------------------|------------------|---------|
| `member.removed` | `id` | A member is removed from the list by an admin/owner. |
| `member.left` | `id` | A member voluntarily leaves the list. |
| `list_updated` | `id, name` | The shopping list's title is modified. |
| `list_deleted` | `id` | The entire list is soft-deleted. |

### 3. Chat Events
**Note:** These events are scoped specifically to the Chat WebSocket connection.
| Event Type (`type`) | Payload Overview | Trigger |
|--------------------|------------------|---------|
| `chat.message` | `id, list_id, sender_id, sender_name, message, created_at` | A new chat message is sent. |

### 4. Control & Error Responses
| Message Type | Payload | Description |
|--------------|---------|-------------|
| `pong` | `{}` | Server response to a `ping` message. |
| `subscribed` | `{"list_id": "UUID"}` | Confirmation of a successful subscription. |
| `unsubscribed` | `{"list_id": "UUID"}` | Confirmation of unsubscription. |
| `error` | `{"message": "..."}` | Error alert (e.g., "Not a member" or "Invalid format"). |

---

## Implementation Notes

- **Real-Time Sync Scoping:** Item and list events are only broadcast to users currently connected to the Global Scope and who are accepted members of the target list.
- **Heartbeats:** It is recommended to send a `ping` every 30-60 seconds to prevent connection timeouts in some network environments.
- **Disconnection:** The server will immediately disconnect a user if their account is deactivated or their role is changed to an insufficient level.
