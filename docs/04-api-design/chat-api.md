# PetZonic — Chat API

> **Base**: `/api/v1/chat`  
> **WebSocket**: `wss://api.petzonic.com/chat`

---

## REST Endpoints

### POST /chat/rooms

Create or get existing chat room for a pet listing.

**Auth**: Required

**Request**:
```json
{
  "petListingId": "uuid",
  "sellerId": "uuid"
}
```

**Response (200/201)**:
```json
{
  "success": true,
  "data": {
    "id": "room-uuid",
    "petListing": {
      "id": "uuid",
      "title": "Golden Retriever Puppy",
      "primaryImage": "...",
      "price": 25000
    },
    "otherUser": {
      "id": "uuid",
      "name": "Rahul S.",
      "avatarUrl": "...",
      "isOnline": true,
      "lastSeen": "2026-05-28T10:00:00Z"
    },
    "createdAt": "2026-05-28T10:00:00Z"
  }
}
```

### GET /chat/rooms

List all chat rooms for current user.

**Auth**: Required

**Query**: `?page=1&limit=20`

**Response**: Sorted by last message time. Includes unread count per room, last message preview.

```json
{
  "data": [
    {
      "id": "room-uuid",
      "petListing": { "id": "uuid", "title": "...", "image": "..." },
      "otherUser": { "id": "uuid", "name": "...", "avatarUrl": "...", "isOnline": false },
      "lastMessage": {
        "content": "Is the puppy still available?",
        "sentAt": "2026-05-28T09:30:00Z",
        "isFromMe": false
      },
      "unreadCount": 2
    }
  ]
}
```

### GET /chat/rooms/:id/messages

Get messages in a chat room (cursor paginated, newest first).

**Auth**: Required (participant only)

**Query**: `?cursor=<message-uuid>&limit=50&direction=before`

**Response**:
```json
{
  "data": [
    {
      "id": "msg-uuid",
      "senderId": "uuid",
      "content": "Is the puppy still available?",
      "messageType": "TEXT",
      "mediaUrl": null,
      "isRead": true,
      "sentAt": "2026-05-28T09:30:00Z"
    },
    {
      "id": "msg-uuid-2",
      "senderId": "uuid",
      "content": null,
      "messageType": "IMAGE",
      "mediaUrl": "https://media.petzonic.com/chat/...",
      "isRead": true,
      "sentAt": "2026-05-28T09:31:00Z"
    }
  ],
  "pagination": {
    "hasMore": true,
    "nextCursor": "msg-uuid-50"
  }
}
```

### POST /chat/rooms/:id/block

Block user in chat.

**Auth**: Required (participant only)

**Request**: `{ "reason": "Abusive messages" }`

---

## WebSocket Protocol

### Connection
```
wss://api.petzonic.com/chat?token=<jwt_access_token>
```

On connect: server sends `connected` event with user's online status updated.

### Client Events (emit)

#### `join_room`
```json
{ "roomId": "room-uuid" }
```

#### `leave_room`
```json
{ "roomId": "room-uuid" }
```

#### `send_message`
```json
{
  "roomId": "room-uuid",
  "content": "Hello, is this pet still available?",
  "messageType": "TEXT"
}
```

For images:
```json
{
  "roomId": "room-uuid",
  "content": null,
  "messageType": "IMAGE",
  "mediaUrl": "https://media.petzonic.com/chat/uploaded-image.jpg"
}
```

For location:
```json
{
  "roomId": "room-uuid",
  "content": "Meeting location",
  "messageType": "LOCATION",
  "mediaUrl": null,
  "metadata": { "latitude": 12.97, "longitude": 77.59 }
}
```

#### `typing`
```json
{ "roomId": "room-uuid" }
```

#### `mark_read`
```json
{ "roomId": "room-uuid", "messageId": "last-read-msg-uuid" }
```

### Server Events (listen)

#### `new_message`
```json
{
  "id": "msg-uuid",
  "roomId": "room-uuid",
  "senderId": "uuid",
  "content": "Yes, still available!",
  "messageType": "TEXT",
  "mediaUrl": null,
  "sentAt": "2026-05-28T10:05:00Z"
}
```

#### `user_typing`
```json
{ "roomId": "room-uuid", "userId": "uuid" }
```

#### `message_read`
```json
{ "roomId": "room-uuid", "readBy": "uuid", "lastReadMessageId": "msg-uuid" }
```

#### `user_online` / `user_offline`
```json
{ "userId": "uuid" }
```

---

## Chat Moderation

- Messages containing phone numbers or external links are flagged automatically
- System message injected: "⚠️ Sharing contact details outside PetZonic is against our policy"
- Flagged messages still delivered but reported to admin moderation queue
- Blocked users cannot send messages or create new chat rooms
