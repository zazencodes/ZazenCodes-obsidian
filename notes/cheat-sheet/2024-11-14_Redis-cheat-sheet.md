---
date: 2024-11-14
tags:
    - cheat-sheet
hubs:
    - "[[cli]]"
    - "[[database]]"
---

## Basic Commands

```bash
# Connect to the Redis server
>>> redis-cli

# Authenticate if password protected
auth '[PASS]'

# Set a key to a value
SET key value

# Retrieve the value of a key
GET key

# Delete a key
DEL key

# Check if a key exists
EXISTS key

# Increment the integer value of a key by one
INCR key

# Decrement the integer value of a key by one
DECR key

# Get all keys matching a pattern
KEYS *

# Set a key with an expiration time
SETEX key seconds value

# Publish a message to a channel
PUBLISH channel message

# Subscribe to a channel
SUBSCRIBE channel
```
