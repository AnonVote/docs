# API Surface

The REST API exposed by `AnonVote/core`. Base URL: `/api`.  
Auth: JWT via HTTP-only cookie set on login.

---

## Organizations

| Method | Endpoint                  | Auth    | Description                |
| ------ | ------------------------- | ------- | -------------------------- |
| POST   | `/organizations`          | —       | Register organization      |
| POST   | `/organizations/login`    | —       | Login, sets session cookie |
| POST   | `/organizations/logout`   | Session | Clear session              |
| GET    | `/organizations/me`       | Session | Current org profile        |
| PATCH  | `/organizations/me`       | Session | Update name or email       |
| PATCH  | `/organizations/password` | Session | Change password            |

---

## Ballots

| Method | Endpoint       | Auth    | Description         |
| ------ | -------------- | ------- | ------------------- |
| GET    | `/ballots`     | Session | List org's ballots  |
| POST   | `/ballots`     | Session | Create ballot       |
| GET    | `/ballots/:id` | —       | Get ballot (public) |
| PATCH  | `/ballots/:id` | Session | Edit ballot         |
| DELETE | `/ballots/:id` | Session | Delete ballot       |

**Create ballot body:**

```json
{
  "topic": "string",
  "options": ["string"],
  "deadline": "ISO8601",
  "eligibilityListId": "uuid",
  "allowWeightedVoting": false,
  "allowRankedChoice": false,
  "maxRankings": null
}
```

---

## Eligibility

| Method | Endpoint       | Auth    | Description                           |
| ------ | -------------- | ------- | ------------------------------------- |
| POST   | `/eligibility` | Session | Upload voter list (CSV or plain text) |

Identifiers are SHA-256 hashed server-side. Originals never stored.

---

## Tokens

| Method | Endpoint                  | Auth    | Description                     |
| ------ | ------------------------- | ------- | ------------------------------- |
| POST   | `/tokens`                 | —       | Request voter token             |
| POST   | `/tokens/reissue`         | —       | Reissue lost token              |
| POST   | `/tokens/reset/:ballotId` | Session | Reset tokenIssued flags (admin) |

**Request body:** `{ "ballotId": "uuid", "voterIdentifier": "string" }`  
**Response:** `{ "data": { "token": "64-char-hex", "weight": 1 } }`

---

## Votes

| Method | Endpoint | Auth | Description           |
| ------ | -------- | ---- | --------------------- |
| POST   | `/votes` | —    | Submit anonymous vote |

**Body:** `{ "ballotId": "uuid", "voterToken": "64-char-hex", "optionId": "uuid", "weight": 1, "rank": null }`

---

## Results

| Method | Endpoint                   | Auth    | Description            |
| ------ | -------------------------- | ------- | ---------------------- |
| GET    | `/results/:ballotId`       | —       | Get published result   |
| POST   | `/results/:ballotId/tally` | Session | Close and tally ballot |

---

## Audit

| Method | Endpoint           | Auth | Description                   |
| ------ | ------------------ | ---- | ----------------------------- |
| GET    | `/audit/:ballotId` | —    | Event counts + Stellar tx IDs |

---

## Delegations

| Method | Endpoint       | Auth | Description                    |
| ------ | -------------- | ---- | ------------------------------ |
| POST   | `/delegations` | —    | Delegate vote to another token |

---

## Verification

| Method | Endpoint                 | Auth | Description                           |
| ------ | ------------------------ | ---- | ------------------------------------- |
| POST   | `/verification/generate` | —    | Generate verification hash for a vote |
| POST   | `/verification/verify`   | —    | Verify a vote by hash                 |

---

## Admin

| Method | Endpoint               | Auth    | Description              |
| ------ | ---------------------- | ------- | ------------------------ |
| GET    | `/admin/rate-limit`    | Session | Get rate limit config    |
| PATCH  | `/admin/rate-limit`    | Session | Update rate limit preset |
| GET    | `/admin/tokens-issued` | Session | Total tokens issued      |

---

## Errors

```json
{ "error": "BadRequest", "message": "Human-readable description" }
```

| Status | Key                  | When                                |
| ------ | -------------------- | ----------------------------------- |
| 400    | `BadRequest`         | Invalid input                       |
| 401    | `Unauthorized`       | No session                          |
| 403    | `Forbidden`          | Session but not permitted           |
| 404    | `NotFound`           | Resource missing                    |
| 409    | `AlreadyVoted`       | Token already used                  |
| 409    | `TokenAlreadyIssued` | Token already issued for identifier |
