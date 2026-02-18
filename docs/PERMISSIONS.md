### Permissions Matrix

####  Platform & Tenant Management

| Action | Super Admin | Tenant Admin | User |
|--------|:---:|:---:|:---:|
| Create tenant | Yes | No | No |
| View all tenants | Yes | No | No |
| Update tenant details (name, slug) | Yes | No | No |
| Activate/Deactivate tenant | Yes | No | No |
| Soft-delete tenant | Yes | No | No |
| View organization metrics (counts) | Yes | Yes | No |
| Access actual list data (cross-tenant) | No | No | No |

####  User Management

| Action | Super Admin | Tenant Admin | User |
|--------|:---:|:---:|:---:|
| Create admin for any tenant | Yes | No | No |
| Create user in their tenant | No | Yes | No |
| List all users in tenant | Yes | Yes | No |
| View any user in tenant | Yes | Yes | No |
| View own profile | Yes | Yes | Yes |
| Update any admin in any tenant | Yes | No | No |
| Update any user in tenant | No | Yes | No |
| Update own profile | Yes | Yes | Yes |
| Delete any tenant admin | Yes | No | No |
| Delete any user in tenant | No | Yes | No |
| Delete own account | No | No | Yes |
| Suspend/Unsuspend user | No | Yes | No |
| Restore soft-deleted user | No | Yes | No |

**Key Rules:**
- **Super Admin**: Manages the tenant ecosystem and tenant admins; restricted from private organization data.
- **Tenant Admin**: Full oversight of users and content within their organization. Cannot modify other administrators.
- **User**: Collaborative role; restricted to their own shared resources.

####  Shopping List Management

| Action | Super Admin | Tenant Admin | List Owner | List Member |
|--------|:---:|:---:|:---:|:---:|
| Create shopping list | No | Yes | Yes | — |
| View organization lists | No | Yes | Yes | Yes (if member) |
| Update list name/settings | No | Yes | Yes | No |
| Delete shopping list | No | Yes | Yes | No |
| Invite members to list | No | Yes | Yes | No |
| Remove member from list | No | Yes | Yes | No (self-leave) |
| Update member permissions | No | Yes | Yes | No |
| Restore deleted list | No | Yes | Yes | No |

**Key Rules:**
- **Tenant Admin**: Has elevated (override) access to all lists within their specific organization.
- **List Owner**: The creator of the list; possesses full control over list membership and settings.
- **Members**: Can only manage their own participation (leave) unless granted specific flags.

####  Item Management

| Action | Super Admin | Tenant Admin | List Owner | Member (w/ Flag) | Member (Default) |
|--------|:---:|:---:|:---:|:---:|:---:|
| Add item to list | No | Yes | Yes | Yes (`can_add`) | No |
| Update item details | No | Yes | Yes | Yes (`can_update`) | No |
| Toggle item status (Purchased) | No | Yes | Yes | Yes (`can_update`) | No |
| Delete item | No | Yes | Yes | Yes (`can_delete`) | No |
| Restore deleted item | No | Yes | Yes | No | No |

**Key Permission Flags:**
- **Can Add Item**: Allows the member to contribute new items to the list.
- **Can Update Item**: Allows changing quantities, names, or marking items as purchased.
- **Can Delete Item**: Allows removing items from the collaborative list.

####  Invitations & Access

| Action | Super Admin | Tenant Admin | List Owner | User |
|--------|:---:|:---:|:---:|:---:|
| Generate invitation token | No | Yes | Yes | No |
| Resend invitation email | No | Yes | Yes | No |
| Cancel pending invitation | No | Yes | Yes | No |
| Accept/Reject invitation | No | No | No | Yes (Invitee) |

####  Real-Time Communication (Chat & Notifications)

| Action | Super Admin | Tenant Admin | List Owner | List Member |
|--------|:---:|:---:|:---:|:---:|
| Join list chat room | No | Yes | Yes | Yes |
| Send chat message | No | Yes | Yes | Yes |
| View chat history | No | Yes | Yes | Yes |
| Delete own chat message | No | Yes | Yes | Yes |
| Delete any chat message | No | Yes | Yes | No |
| Receive real-time notifications | No | Yes | Yes | Yes |
| Mark notifications as read | No | Yes | Yes | Yes |
