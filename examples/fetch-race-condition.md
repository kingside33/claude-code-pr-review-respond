# Fetch Race Condition Fix (React)

## Problem

Concurrent fetch requests in React components can cause race conditions
where a later request's response overwrites an earlier one, leading to
stale state updates.

## Fix Pattern

```tsx
const fetchIdRef = useRef(0);

function fetchData() {
  const fetchId = ++fetchIdRef.current;
  fetch('/api/data')
    .then(res => res.json())
    .then(data => {
      if (fetchId !== fetchIdRef.current) return;
      setState(data);
    });
}
```

## When to use

When a component fires multiple async requests and state updates
must reflect only the most recent request.
