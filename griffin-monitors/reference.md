# Assertion API Reference

Use with `Assert(accessor).predicate()`. Return an array from the `.assert((state) => [ ... ])` callback.

## Accessors

| Accessor   | Description          | Example                                 |
| ---------- | -------------------- | --------------------------------------- |
| `.body`    | Response body        | `state["node"].body["data"]["id"]`      |
| `.headers` | Response headers     | `state["node"].headers["content-type"]` |
| `.status`  | HTTP status code     | `state["node"].status`                  |
| `.latency` | Request latency (ms) | `state["node"].latency`                 |

**Status and latency** support only **binary** predicates (e.g. `.equals(200)`, `.lessThan(500)`). Do not use unary predicates (`.isNull()`, `.isDefined()`, `.isEmpty()`) on `status` or `latency`—they throw at build time. **Body and headers** support both unary and binary predicates.

For steps with `response_format: NoContent` (204 with no body), assert only on `status` (e.g. `Assert(state["node"].status).equals(204)`); there is no body to assert on.

**Array access:** use `.at(index)` for array elements, e.g. `state["node"].body["items"].at(0)["name"]` or `state["node"].body["matrix"].at(2).at(3)` for nested arrays.

---

## Unary predicates

(Valid for **body** and **headers** only, not status or latency.)

| Predicate      | Description                  | Negation                         |
| -------------- | ---------------------------- | -------------------------------- |
| `.isNull()`    | Value is null                | `.not.isNull()` / `.isDefined()` |
| `.isDefined()` | Value is not null            | `.not.isDefined()` / `.isNull()` |
| `.isTrue()`    | Value is true                | `.not.isTrue()`                  |
| `.isFalse()`   | Value is false               | `.not.isFalse()`                 |
| `.isEmpty()`   | String/array/object is empty | `.not.isEmpty()`                 |

---

## Binary predicates

| Predicate                | Example                               |
| ------------------------ | ------------------------------------- |
| `.equals(x)`             | `Assert(status).equals(200)`          |
| `.not.equals(x)`         | `Assert(code).not.equals(500)`        |
| `.lessThan(x)`           | `Assert(latency).lessThan(500)`       |
| `.lessThanOrEqual(x)`    | `Assert(count).lessThanOrEqual(100)`  |
| `.greaterThan(x)`        | `Assert(age).greaterThan(0)`          |
| `.greaterThanOrEqual(x)` | `Assert(score).greaterThanOrEqual(0)` |
| `.contains(x)`           | `Assert(header).contains("json")`    |
| `.not.contains(x)`       | `Assert(url).not.contains("staging")`|
| `.startsWith(x)`         | `Assert(path).startsWith("/api/")`   |
| `.not.startsWith(x)`     | —                                     |
| `.endsWith(x)`           | `Assert(email).endsWith("@example.com")` |
| `.not.endsWith(x)`       | —                                     |

Negated comparison: e.g. `.not.greaterThan(100)` → value ≤ 100; `.not.lessThan(0)` → value ≥ 0.

---

## Common patterns

| Task              | Code                                                     |
| ----------------- | -------------------------------------------------------- |
| Status is 200     | `Assert(state["n"].status).equals(200)`                  |
| Status in 2xx     | `Assert(state["n"].status).lessThan(400)`                |
| Latency < 500ms   | `Assert(state["n"].latency).lessThan(500)`               |
| Body field exists | `Assert(state["n"].body["x"]).not.isNull()`             |
| String contains   | `Assert(state["n"].body["msg"]).contains("ok")`          |
| Number in range   | `Assert(state["n"].body["age"]).greaterThanOrEqual(18)` |
| Boolean true      | `Assert(state["n"].body["active"]).isTrue()`             |
| Header has value  | `Assert(state["n"].headers["content-type"]).contains("application/json")` |
| Array not empty   | `Assert(state["n"].body["items"]).not.isEmpty()`        |
| First array item  | `Assert(state["n"].body["items"].at(0)["id"]).isDefined()` |
| Nested path       | `Assert(state["n"].body["user"]["profile"]["email"])...` |

### Status checks

```typescript
Assert(state["node"].status).equals(200);
Assert(state["node"].status).lessThan(400);   // any 2xx/3xx
Assert(state["node"].status).not.equals(500);
```

### Latency checks

```typescript
Assert(state["node"].latency).lessThan(500);
Assert(state["node"].latency).lessThanOrEqual(1000);
Assert(state["node"].latency).greaterThan(0);  // sanity
```

### Existence and value

```typescript
Assert(state["node"].body["id"]).not.isNull();
Assert(state["node"].body["data"]).isDefined();
Assert(state["node"].body["items"]).not.isEmpty();
Assert(state["node"].body["email"]).equals("test@example.com");
Assert(state["node"].body["items"].at(0)["name"]).equals("First");
```

Use `.not` before any predicate to negate (e.g. `Assert(x).not.equals(500)`).
