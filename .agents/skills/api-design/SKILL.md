---
name: api-design
description: Use when designing or reviewing REST endpoints in a Spring Boot app — naming resources, choosing status codes, shaping error responses (ProblemDetail / RFC 9457), pagination, versioning, or idempotency. Also when a controller returns 200 for errors, 500s leak stack traces to clients, list endpoints have no paging, or the same mistake appears as "should this be POST or PUT?".
metadata:
  author: custom (this starter kit)
  version: "1.0"
---

# REST API design (Spring-flavored)

An API is a contract other people build against. Inconsistency is the real cost:
every oddity becomes client-side `if` statements that live forever. These conventions
are boring on purpose — spend creativity on the domain, not the plumbing.

## Golden rules

1. **Resources are plural nouns; verbs come from HTTP.** `GET /orders/42`, not
   `/getOrder?id=42`. Genuine actions that fit no verb get a sub-resource:
   `POST /orders/42/cancellation`.
2. **Nest at most one level.** `/customers/7/orders` is fine;
   `/customers/7/orders/42/items/3/discounts` is not — after one level, switch to the
   flat resource (`/orders/42/items/3`).
3. **DTOs at the boundary, never entities.** Exposing JPA entities couples your
   schema to the contract and drags lazy-loading and recursion problems into
   serialization. Map explicitly (records are ideal).
4. **Errors are part of the contract.** One error shape everywhere — Spring's
   `ProblemDetail` (RFC 9457) — produced by one `@RestControllerAdvice`. No stack
   traces, no Hibernate messages, no `{"error": "something went wrong"}` ad-libs.
5. **Every list endpoint pages** from day one. Retrofitting paging onto a client that
   expects a bare array is a breaking change; capping an unbounded query later is an
   outage waiting to happen.

## Status codes that carry meaning

| Situation | Code |
|---|---|
| Read OK / update OK with body | 200 |
| Created — include `Location: /orders/43` | 201 |
| Done, nothing to say (delete, async accept) | 204 / 202 |
| Request malformed or fails validation | 400 |
| Not authenticated / authenticated but not allowed | 401 / 403 |
| No such resource (also for "not yours" when hiding existence) | 404 |
| Conflict with current state (duplicate, stale version) | 409 |
| Anything the *server* broke | 500 — and only the server's fault |

Returning 200 with `{"success": false}` defeats monitoring, retries, and HTTP caching
in one move. Don't.

## Errors: ProblemDetail + one advice

```java
@RestControllerAdvice
class ApiExceptionHandler {
    @ExceptionHandler(OrderNotFoundException.class)
    ProblemDetail notFound(OrderNotFoundException ex) {
        var pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        pd.setTitle("Order not found");
        return pd;
    }
    @ExceptionHandler(MethodArgumentNotValidException.class)
    ProblemDetail invalid(MethodArgumentNotValidException ex) {
        var pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST, "Validation failed");
        pd.setProperty("errors", ex.getFieldErrors().stream()
            .map(f -> Map.of("field", f.getField(), "message", f.getDefaultMessage()))
            .toList());
        return pd;
    }
}
```

Validate input with `jakarta.validation` (`@Valid` on the DTO) and let the advice
translate — controllers never try/catch their way to error responses.

## Pagination, sorting, filtering

Spring Data's `Pageable` binds `?page=0&size=20&sort=createdAt,desc` for free —
accept it, cap it, and return a stable envelope:

```java
@GetMapping
Page<OrderDto> list(@PageableDefault(size = 20) Pageable pageable) {
    return orders.findAll(clamp(pageable, 100)).map(OrderDto::from);  // hard cap size
}
```

- Always order by a unique tiebreaker (`sort=createdAt,desc&sort=id,desc`) — without
  it, rows shift between pages.
- Filters are query parameters with the field's name: `?status=OPEN&customerId=7`.
- Huge/fast-moving datasets: prefer cursor (keyset) paging — `?after=<id>` — offset
  paging both drifts and slows down at depth.

## Versioning and evolution

- Additive changes (new optional fields, new endpoints) need no version bump —
  clients must tolerate unknown fields.
- Breaking changes get a new major version in the path: `/api/v2/orders`. It's
  crude, visible, and routable — which is exactly what you want. Don't mix
  header/parameter versioning into the same API.
- Never reuse or repurpose a field name with different semantics; add a new field
  and deprecate the old one in the docs.

## Idempotency and method choice

- `PUT` = full replace at a known URI, idempotent. `PATCH` = partial update.
  `POST` = create or non-idempotent action.
- Retried `POST`s that must not duplicate (payments, orders): accept an
  `Idempotency-Key` header, store it with the result, and replay the stored response
  on repeats — return 409 if the same key arrives with a different body.
- `DELETE` of something already gone: 204. The client's goal state is achieved;
  punishing retries with 404 makes clients write workaround code.

## Consistency details that save clients

- Timestamps: ISO-8601 UTC (`2026-06-11T14:30:00Z`), suffix `At` — `createdAt`.
  Money: integer minor units (`amountCents`) or a string decimal — never float.
- JSON fields camelCase, consistently — set one Jackson naming strategy and stop.
- Absent vs null: omit fields you'll never populate; `null` means "cleared".

## Testing the contract

`@WebMvcTest` + MockMvc asserts the contract, not the implementation:

```java
@Test void missingOrderIs404WithProblemBody() throws Exception {
    mvc.perform(get("/api/orders/999"))
       .andExpect(status().isNotFound())
       .andExpect(content().contentType(MediaType.APPLICATION_PROBLEM_JSON))
       .andExpect(jsonPath("$.title").value("Order not found"));
}
```

Test the error paths and the envelope shape — those are what clients actually
depend on; happy-path JSON rarely regresses.
