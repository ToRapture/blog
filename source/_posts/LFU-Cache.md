---
title: LFU Cache
date: 2020-11-02 14:59:56
tags:
- Design
---

[460. LFU Cache](https://leetcode.com/problems/lfu-cache/)

```cpp
class LFUCache {
    struct LFUItem {
        int key;
        int value;
        int freq;
        LFUItem() {}
        LFUItem(int key, int value, int freq) : key(key), value(value), freq(freq) {}
    };

    int capacity;
    int min_freq;
    unordered_map<int, list<LFUItem>> freq_to_list;
    unordered_map<int, list<LFUItem>::iterator> key_to_iter;

   public:
    LFUCache(int capacity) {
        this->capacity = capacity;
        this->min_freq = 1;
    }

    int get(int key) {
        if (capacity <= 0) return -1;

        auto it = key_to_iter.find(key);
        if (it == key_to_iter.end()) return -1;
        int value = it->second->value;
        put(key, value);
        return value;
    }

    void put(int key, int value) {
        if (capacity <= 0) return;

        auto it = key_to_iter.find(key);
        if (it != key_to_iter.end()) {
            auto list_node = it->second;
            freq_to_list[list_node->freq + 1].push_back(LFUItem(key, value, list_node->freq + 1));
            key_to_iter[key] = prev(freq_to_list[list_node->freq + 1].end());
            freq_to_list[list_node->freq].erase(list_node);
            if (freq_to_list[min_freq].empty()) min_freq++;
        } else {
            if (key_to_iter.size() == capacity) {
                key_to_iter.erase(freq_to_list[min_freq].begin()->key);
                freq_to_list[min_freq].pop_front();
            }
            min_freq = 1;
            freq_to_list[min_freq].push_back(LFUItem(key, value, min_freq));
            key_to_iter[key] = prev(freq_to_list[min_freq].end());
        }
    }
};
```
