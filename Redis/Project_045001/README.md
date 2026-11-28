# Redis: Inventory — Baseline Setup for School Lab (Project 045001)

## Project overview

Shared classroom equipment provides the setting for this inventory exercise. The embedded code is a reference implementation based on the student guide and uses fictional data.

## Problem statement

The school lab needs a repeatable inventory baseline that another student can verify.

## Project objectives

- Record the school lab inventory baseline.
- Run the inventory calculation inside Redis and delete the temporary hash.
- Reproduce the sample result and inspect how the project handles its documented edge case.

## Technologies used

Redis Lua scripting and hashes.

## System architecture

`CLI values → atomic Lua script → temporary Redis hash → reorder result`

## Features

- Record the school lab inventory baseline.
- Run the inventory calculation inside Redis and delete the temporary hash.

## Prerequisites

Use a local Redis instance and redis-cli. Use a text editor and a local practice directory.

## Complete Redis source code

```lua
if #KEYS ~= 1 or #ARGV ~= 3 then
    return redis.error_reply("Pass one scratch key, an asset, stock, and a threshold")
end

local key, asset = KEYS[1], ARGV[1]
local stock, threshold = tonumber(ARGV[2]), tonumber(ARGV[3])
if string.sub(key, 1, 24) ~= "portfolio:inventory:lab:" or asset == "" then
    return redis.error_reply("Use a portfolio scratch key and a non-empty asset")
end
if not stock or not threshold or stock < 0 or threshold < 1
   or stock > 1000000000 or threshold > 1000000000
   or stock ~= math.floor(stock) or threshold ~= math.floor(threshold) then
    return redis.error_reply("Stock and threshold must be valid integers")
end
if redis.call("EXISTS", key) == 1 then
    return redis.error_reply("Scratch key already exists")
end

redis.call("HSET", key, "asset", asset, "stock", stock, "reorder_at", threshold)
redis.call("EXPIRE", key, 60)
local row = redis.call("HMGET", key, "asset", "stock", "reorder_at")
redis.call("DEL", key)
return cjson.encode({
    asset = row[1],
    stock = tonumber(row[2]),
    reorder_at = tonumber(row[3]),
    needs_reorder = tonumber(row[2]) < tonumber(row[3])
})
```

## Implementation explanation

Run the inventory calculation inside Redis and delete the temporary hash.

## Sample input data

Use this example input.

```json
{
  "asset": "LAB-17",
  "stock": 2,
  "reorder_at": 4
}
```

## Execution instructions

Save the block as `inventory.lua`. Use a fresh scratch key on your local practice instance.

```sh
redis-cli --eval inventory.lua "portfolio:inventory:lab:sample-17" , LAB-17 2 4
```

## Example output

Illustrative excerpt for the synthetic sample.

```json
{
  "asset": "LAB-17",
  "stock": 2,
  "reorder_at": 4,
  "needs_reorder": true
}
```

## Testing and validation

Executed the embedded script in Redis 7. Two runs matched and cleaned up the scratch key. Negative stock returned an error.

```json
{
  "asset": "LAB-17",
  "stock": 2,
  "reorder_at": 4,
  "needs_reorder": true
}
```

## Error handling

The script rejects malformed values and refuses to replace an occupied key. A short expiry bounds the scratch hash lifetime.

## Troubleshooting

- Confirm the documented runtime and copy the complete source block.
- Start with the documented sample and use the filename or request path shown above.
- Compare the observed value with the example before changing an input.

## Limitations

The example processes one inventory row and deletes its scratch hash after reading it.

## Learning outcomes

Practice inventory using Redis and check a repeatable baseline setup workflow in the school lab scenario.

## Conclusion

The project records an observable inventory result for training case `LAB-17`.

[Back to the Redis project index](../README.md)
