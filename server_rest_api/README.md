# Server REST API Documentation

> Server version: v1

---

### GET `/api/info`

Return the server info, exemple:

```
{
    "server_name": "UMES Core Server"
}
```

---

### GET `/api/message?public_id={public_id}`

Return the encrypted message from a public_id

```
{
    "content": "JyY/Cw53L1YeFVBMQg=="
}
```

---

### POST `/api/message`

Create a new message in the database and return a new public_id

Request data:
```
{
    "content": "JyY/Cw53L1YeFVBMQg=="
}
```

Response data:
```
{
    "public_id": "65b0452240704a3daefe99517d05d76a"
}
```

---
