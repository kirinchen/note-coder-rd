---
title: Python Tread Async sync
tags: [python]

---

# Python Tread Async sync

## Async From Sync

```python=
async def get_chat_id(name):
    await asyncio.sleep(3)
    return "chat-%s" % name

def main():
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)
    result = loop.run_until_complete(get_chat_id("django"))
```

###### tags: `python`