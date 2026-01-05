# Events Tips

## Observers

- Don't overuse observers/events for simple model logic — they make debugging harder
- Put simple logic in mutators or model methods instead
- Save events for when multiple things respond to an action

## Skipping Events

- Use `->touchQuietly()` to update timestamps without triggering events
- Use `->withoutEvents()` when you need to skip observers (imports, seeding)
