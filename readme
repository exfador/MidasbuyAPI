<h1 align="center">Midasbuy Redeem API</h1>

<p align="center">
  Lightweight HTTP API for redeeming Midasbuy codes and looking up player info.
  <br/>
  <sub>Per-key quotas · per-IP guard · encrypted cookies at rest · Telegram bot admin.</sub>
</p>

<p align="center">
  <a href="https://t.me/exfador"><img src="https://img.shields.io/badge/Buy_API_key-@exfador-2CA5E0?logo=telegram&logoColor=white" alt="Buy"></a>
  <a href="https://t.me/midasbuyapi"><img src="https://img.shields.io/badge/News-@midasbuyapi-229ED9?logo=telegram&logoColor=white" alt="Channel"></a>
  <img src="https://img.shields.io/badge/node-%E2%89%A518-3C873A?logo=node.js&logoColor=white" alt="Node">
  <img src="https://img.shields.io/badge/python-%E2%89%A53.10-3776AB?logo=python&logoColor=white" alt="Python">
</p>

---

## Quick start

```bash
curl -X POST http://<your-server>:8000/redeem \
  -H "x-key: <api-key>" \
  -H "Content-Type: application/json" \
  -d '{"player_id":"51569444543","code":"Grmndgic2f2272H8be"}'
```

```json
{ "success": true, "message": "Code redeemed successfully", "nickname": "GNCOD" }
```

> The `x-key` is issued via the Telegram bot. Buy one from **[@exfador](https://t.me/exfador)**.
> Updates and announcements: **[@midasbuyapi](https://t.me/midasbuyapi)**.

## Endpoints

| Method | Path        | What it does                          |
| :----- | :---------- | :------------------------------------ |
| POST   | `/redeem`   | Redeem a code on a player account     |
| POST   | `/info`     | Look up player info by `player_id`    |
| POST   | `/stats`    | Usage counters and remaining quota    |
| POST   | `/history`  | Recent redeem attempts                |

All endpoints are **POST**, body is JSON (≤ 16 KB). Every request needs the
`x-key: <api-key>` header. Anything else returns `404 {"detail":"error"}`.

## Rate limits

| Scope         | Limit                       | Reset            |
| :------------ | :-------------------------- | :--------------- |
| per API key   | 5 requests / minute         | rolling 60 s     |
| per API key   | 1000 redemptions / day      | 00:00 MSK        |
| per IP        | 30 requests / minute        | rolling 60 s     |
| server-wide   | 10 concurrent `/redeem`     | —                |

Order of checks: per-IP → key auth → 5/min → 1000/day → concurrency.
The first violation wins.

## Status codes

| Code | Meaning                                                                   |
| :--: | :------------------------------------------------------------------------ |
| 200  | Processed. See `success` in the body for the actual outcome.              |
| 400  | Validation error. Body has `success: false` and a localized `message`.    |
| 401  | Auth failed. Body: `{"detail":"error"}`. No details on purpose.           |
| 404  | Unknown endpoint / wrong method. Body: `{"detail":"error"}`.              |
| 429  | Quota exceeded. Localized `message` for key limits.                       |

After 10 consecutive `401` from the same IP the IP is banned for 1 hour.

---

## `POST /redeem`

Redeem a Midasbuy code on the given player account.

**Body**

```json
{ "player_id": "51569444543", "code": "Grmndgic2f2272H8be" }
```

**Success** (`HTTP 200`)

```json
{
  "success":  true,
  "message":  "Код успешно активирован — товар зачислен на аккаунт",
  "nickname": "GNCOD"
}
```

**Failure** (`HTTP 200`, `success: false`)

```json
{ "success": false, "message": "<localized error>" }
```

Possible reasons: invalid code · already used · expired · player not found ·
authorization outdated · generic failure. See **[Error messages](#error-messages)**
for exact RU/EN texts.

---

## `POST /info`

Look up player info by `player_id`.

**Body**

```json
{ "player_id": "51569444543" }
```

**Success**

```json
{
  "success":   true,
  "player_id": "51569444543",
  "nickname":  "GNCOD",
  "zone_id":   "1",
  "is_ban":    false
}
```

**Failure** (`success: false`) — player not found · authorization outdated ·
generic failure. See **[Error messages](#error-messages)**.

---

## `POST /stats`

Returns usage counters and remaining quota for the current API key.
Payload is language-independent.

**Body**: `{}`

**Response**

```json
{
  "success": true,
  "key": {
    "expires_at":     1782384336,
    "expires_at_iso": "2026-06-25 10:45:36",
    "ttl_seconds":    2590409
  },
  "rate_limit_minute": {
    "max": 5, "used": 1, "remaining": 4, "window_seconds": 60
  },
  "daily_redeem": {
    "limit": 1000, "used": 2, "remaining": 998,
    "reset_at":         1779829200,
    "reset_at_msk":     "2026-05-27 00:00:00 MSK",
    "reset_in_seconds": 35273,
    "timezone":         "MSK (UTC+3)"
  },
  "totals": { "success": 1, "fail": 1, "all_time": 2 }
}
```

| Field                          | Meaning                                    |
| :----------------------------- | :----------------------------------------- |
| `key.ttl_seconds`              | seconds until the subscription expires     |
| `rate_limit_minute.remaining`  | requests still allowed this minute         |
| `daily_redeem.remaining`       | `/redeem` calls left today                 |
| `daily_redeem.reset_in_seconds`| seconds until daily counter resets         |
| `totals`                       | lifetime success / fail / total for the key |

---

## `POST /history`

Recent `/redeem` attempts for the current API key, newest first.

**Body**

```json
{ "limit": 50 }
```

`limit` — `1..500`, default `50`.

**Response**

```json
{
  "success": true,
  "count":   2,
  "items": [
    {
      "created_at":     1779792974,
      "created_at_iso": "2026-05-26 10:56:14",
      "player_id":      "51889127192",
      "code":           "Grmndgic2f2272H8be",
      "success":        true
    }
  ]
}
```

---

## Error messages

The `message` field follows the language the key owner picked in the Telegram
bot (Russian or English). Switching the language in the bot takes effect
on the very next API call.

<details>
<summary><b><code>/redeem</code> — outcomes</b></summary>

| Reason                 | RU |
| :--------------------- | :-- |
| success                | Код успешно активирован — товар зачислен на аккаунт |
| invalid code           | Код недействителен. Проверьте правильность ввода |
| already used           | Этот код уже был активирован ранее |
| expired                | Срок действия кода истёк, активация больше недоступна |
| player not found       | Игрок с таким ID не найден. Проверьте Player ID и попробуйте снова |
| authorization outdated | Авторизация Midasbuy устарела. Обратитесь к администратору для обновления |
| generic failure        | Не удалось активировать код. Попробуйте позже или обратитесь к администратору |

| Reason                 | EN |
| :--------------------- | :-- |
| success                | Code redeemed successfully — the item has been delivered to the account |
| invalid code           | Code is invalid. Please check the value and try again |
| already used           | This code has already been redeemed |
| expired                | Code has expired and can no longer be redeemed |
| player not found       | Player with this ID was not found. Check the Player ID and try again |
| authorization outdated | Midasbuy authorization is outdated. Please contact the administrator |
| generic failure        | Failed to redeem the code. Try again later or contact the administrator |

</details>

<details>
<summary><b><code>/info</code> — outcomes</b></summary>

| Reason                 | RU |
| :--------------------- | :-- |
| player not found       | Игрок с таким ID не найден. Проверьте Player ID и попробуйте снова |
| authorization outdated | Авторизация Midasbuy устарела. Обратитесь к администратору для обновления |
| generic failure        | Не удалось получить информацию об игроке. Попробуйте позже |

| Reason                 | EN |
| :--------------------- | :-- |
| player not found       | Player with this ID was not found. Check the Player ID and try again |
| authorization outdated | Midasbuy authorization is outdated. Please contact the administrator |
| generic failure        | Failed to retrieve player info. Please try again later |

</details>

<details>
<summary><b>Quota errors — <code>HTTP 429</code></b></summary>

| Reason            | RU |
| :---------------- | :-- |
| 5/min per key     | Слишком много запросов. Лимит: 5 в минуту. Подождите немного |
| 1000/day per key  | Достигнут суточный лимит активаций (1000 в сутки). Счётчик обнулится в 00:00 по МСК |
| server overloaded | Сервер сейчас перегружен. Повторите запрос через несколько секунд |

| Reason            | EN |
| :---------------- | :-- |
| 5/min per key     | Too many requests. Limit: 5 per minute. Please slow down |
| 1000/day per key  | Daily redeem limit reached (1000 per day). Counter resets at 00:00 MSK |
| server overloaded | Server is currently overloaded. Please retry in a few seconds |

</details>

---

## Contacts

- 🔑 **Buy an API key** — [@exfador](https://t.me/exfador)
- 📣 **News & updates** — [@midasbuyapi](https://t.me/midasbuyapi)
