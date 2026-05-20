# Time Range Validation in Tests (C#)

## Problem

Verifying that API responses contain timestamped entries within an
expected time range, using System.Text.Json.

## Fix Pattern

```csharp
foreach (var item in items.EnumerateArray())
{
    var time = DateTimeOffset.Parse(item.GetProperty("time").GetString()!);
    Assert.True(time >= from && time <= to);
}
```

## Notes

- Uses `DateTimeOffset` for timezone-aware comparisons
- Iterates `JsonElement.EnumerateArray()` for streaming JSON arrays
- Assert per-item to get granular failure messages
