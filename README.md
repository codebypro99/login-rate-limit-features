# Login Rate Limit Feature Sample

A small TypeScript class that records failed login attempts and calculates a retry delay using per-key and global sliding time windows.

**Source:** [rate-limit.ts](./features/rate-limit.ts)

## Purpose

This feature shows how I isolate failure tracking and retry-delay calculation from authentication and HTTP handling. The caller chooses the key, reports outcomes, and decides how to apply the returned delay.

## Failure Windows

- Tracks failure timestamps in an in-memory map.
- Applies a short per-key window with a default limit of five failures per minute.
- Applies a longer per-key window with a default limit of twenty failures per hour.
- Applies a shared window with a default limit of thirty failures per minute.
- Returns the maximum delay across the three windows, rounded up to whole seconds.
- Returns zero when no configured window requires waiting.

**What this demonstrates:** combining burst and sustained failure policies behind one retry-delay interface.

## Outcome Recording

- `fail(key)` records a timestamp for both the supplied key and the shared bucket.
- `succeed(key)` clears the supplied key's history while retaining shared failure history.
- History is pruned to the most recent hour whenever a bucket is accessed.
- The clock is injectable, allowing deterministic time control by a caller or test harness.

**What this demonstrates:** keeping policy calculations separate from password checks, request identity, and response formatting.

## Example Integration Flow

1. Derive a key using the application's own identity policy.
2. Use `retryAfter(key)` when applying the retry policy.
3. Call `fail(key)` after a failed authentication attempt.
4. Call `succeed(key)` after successful authentication.

The placement of delay checks relative to password verification belongs to the caller. This excerpt alone does not establish whether correct credentials remain usable while a delay is active.

## Technology

TypeScript, an in-memory Map, timestamp arrays, and an injectable clock. The class has no imports or framework dependency.

## Behavior Boundaries

The shared bucket exists within one class instance, not across processes or server replicas. History is lost when the instance is discarded. Buckets are pruned on access, without a background cleanup mechanism for unused keys.

The literal `*` is reserved for the shared bucket; using it as a caller key would mix per-key and global state. Limit values are supplied through the constructor and are not validated by this excerpt.

Authentication, key selection, request enforcement, response headers, and distributed persistence are outside the class. Original source code is preserved. The description is based on source inspection; the feature has not been executed as part of this collection.

## Portfolio Message

I can express a configurable retry policy as a focused TypeScript utility with explicit failure windows and deterministic time inputs.
