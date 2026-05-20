# Stale File Writer Cleanup (C#)

## Problem

Long-running applications accumulate stale file writers keyed by date,
causing resource leaks as daily rollover creates new writers.

## Fix Pattern

```csharp
private static void CleanupStaleWriters()
{
    var today = DateTime.Now.ToString("yyyy-MM-dd", CultureInfo.InvariantCulture);
    foreach (var key in s_logWriters.Keys)
    {
        if (!key.Contains(today, StringComparison.OrdinalIgnoreCase))
            if (s_logWriters.TryRemove(key, out var writer))
                try { writer.Dispose(); } catch { }
    }
}
```

## Notes

- Uses `ConcurrentDictionary.TryRemove` for atomic key removal
- `CultureInfo.InvariantCulture` ensures date format consistency across locales
- Swallows dispose exceptions to avoid disrupting the cleanup loop
