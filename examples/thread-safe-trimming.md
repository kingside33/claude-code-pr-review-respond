# Thread-Safe Concurrent Queue Trimming (C#)

## Problem

A thread-safe bounded queue needs to trim excess entries atomically
without blocking concurrent readers.

## Fix Pattern

```csharp
private readonly object _trimLock = new();

public void Append(T item)
{
    _entries.Enqueue(item);
    var newCount = Interlocked.Increment(ref _count);

    if (newCount > MaxCapacity)
    {
        lock (_trimLock)
        {
            while (_count > MaxCapacity)
            {
                if (_entries.TryDequeue(out _))
                    Interlocked.Decrement(ref _count);
            }
        }
    }
}
```

## Notes

- `Interlocked` for lock-free counter operations
- Lock only for the dequeue path (hot read path stays lock-free)
- Double-check pattern: if-check outside lock, while-loop inside lock
