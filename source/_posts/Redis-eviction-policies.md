---
title: Redis eviction policies
date: 2020-04-16 13:47:00
mathjax: true
tags:
- Redis
---

## Official document
[Using Redis as an LRU cache](https://redis.io/topics/lru-cache)

## Eviction policies
There are total 6 evicition policies so far:

- noeviction
- allkeys-lru
- allkeys-random
- volatile-lru
- volatile-random
- volatile-ttl

`allkeys` for all keys, while `volatile` means among keys that have an expire set.
`lru` means evict keys by trying to remove the less recently used (LRU) keys first, `random` means evict keys randomly, `ttl` means try to evict keys with a shorter time to live (TTL) first.



_ _ _

The range of the keys to be evicted:
Only the keys that have a expire set are in both the `dict` and `expires` two dicts, others are in `dict` only.
```cpp
if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
    server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM)
{
    dict = server.db[j].dict;
} else {
    dict = server.db[j].expires;
}
```

_ _ _

Getting a paricular key by using random:
```cpp
/* volatile-random and allkeys-random policy */
if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_RANDOM ||
    server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_RANDOM)
{
    de = dictGetRandomKey(dict);
    bestkey = dictGetKey(de);
}
```

_ _ _

Getting a particular key by using LRU:
```cpp
/* volatile-lru and allkeys-lru policy */
else if (server.maxmemory_policy == REDIS_MAXMEMORY_ALLKEYS_LRU ||
    server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_LRU)
{
    struct evictionPoolEntry *pool = db->eviction_pool;

    while(bestkey == NULL) {
        evictionPoolPopulate(dict, db->dict, db->eviction_pool);
        /* Go backward from best to worst element to evict. */
        for (k = REDIS_EVICTION_POOL_SIZE-1; k >= 0; k--) {
            if (pool[k].key == NULL) continue;
            de = dictFind(dict,pool[k].key);

            /* Remove the entry from the pool. */
            sdsfree(pool[k].key);
            /* Shift all elements on its right to left. */
            memmove(pool+k,pool+k+1,
                sizeof(pool[0])*(REDIS_EVICTION_POOL_SIZE-k-1));
            /* Clear the element on the right which is empty
             * since we shifted one position to the left.  */
            pool[REDIS_EVICTION_POOL_SIZE-1].key = NULL;
            pool[REDIS_EVICTION_POOL_SIZE-1].idle = 0;

            /* If the key exists, is our pick. Otherwise it is
             * a ghost and we need to try the next element. */
            if (de) {
                bestkey = dictGetKey(de);
                break;
            } else {
                /* Ghost... */
                continue;
            }
        }
    }
}
```

_ _ _

Getting a particular key by using TTL:
```cpp
/* volatile-ttl */
else if (server.maxmemory_policy == REDIS_MAXMEMORY_VOLATILE_TTL) {
    for (k = 0; k < server.maxmemory_samples; k++) {
        sds thiskey;
        long thisval;
        de = dictGetRandomKey(dict);
        thiskey = dictGetKey(de);
        thisval = (long) dictGetVal(de);
        /* Expire sooner (minor expire unix timestamp) is better
         * candidate for deletion */
        if (bestkey == NULL || thisval < bestval) {
            bestkey = thiskey;
            bestval = thisval;
        }
    }
}
```