---
title: LRU Cache
date: 2020-11-01 22:33:54
tags:
- Design
---

[146. LRU Cache](https://leetcode.com/problems/lru-cache/)

Using linked list:
```cpp
class LinkedNode {
   public:
    LinkedNode *prev, *next;
    int val;
    LinkedNode(int val) {
        this->val = val;
        this->prev = nullptr;
        this->next = nullptr;
    }
};

class LinkedList {
   private:
    LinkedNode *head, *tail;

   public:
    LinkedList() {
        this->head = nullptr;
        this->tail = nullptr;
    }
    void erase(LinkedNode *p) {
        if (p == tail) {
            tail = tail->prev;
        } else if (p == head) {
            head = head->next;
        }
        if (p->prev) {
            p->prev->next = p->next;
        }
        if (p->next) {
            p->next->prev = p->prev;
        }
    }
    LinkedNode *begin() { return head; }
    LinkedNode *rbegin() { return tail; }
    void push_front(int val) {
        LinkedNode *node = new LinkedNode(val);
        node->prev = nullptr;
        node->next = head;
        if (head) {
            head->prev = node;
        }
        head = node;
        if (tail == nullptr) {
            tail = head;
        }
    }
    void pop_back() { erase(tail); }
};

class LRUCache {
    LinkedList lst;
    unordered_map<int, LinkedNode *> key_to_node;
    unordered_map<int, int> key_to_value;
    int capacity;

   public:
    LRUCache(int capacity) { this->capacity = capacity; }

    int get(int key) {
        auto it = key_to_node.find(key);
        if (it == key_to_node.end()) {
            return -1;
        }
        int value = key_to_value[key];
        put(key, value);
        return value;
    }

    void put(int key, int value) {
        auto it = key_to_node.find(key);
        if (it != key_to_node.end()) {
            lst.erase(it->second);
            key_to_node.erase(key);
            key_to_value.erase(key);
        }
        lst.push_front(key);
        key_to_node[key] = lst.begin();
        key_to_value[key] = value;

        if (key_to_node.size() > capacity) {
            LinkedNode *tail = lst.rbegin();
            key_to_node.erase(tail->val);
            key_to_value.erase(tail->val);
            lst.pop_back();
        }
    }
};
```

------

Using STL:
```cpp
class LRUCache {
    int capacity;
    list<pair<int, int>> lst;
    unordered_map<int, list<pair<int, int>>::iterator> mp;
public:
    LRUCache(int capacity) {
        this->capacity = capacity;
        lst.clear();
        mp.clear();
    }
    
    int get(int key) {
        auto it = mp.find(key);
        if (it == mp.end()) {
            return -1;
        }
        int value = it->second->second;
        put(key, value);
        return value;
    }
    
    void put(int key, int value) {
        auto keyToIter = mp.find(key);
        if (keyToIter != mp.end()) {
            lst.erase(keyToIter->second);
            mp.erase(keyToIter);
        }
        lst.push_front({key, value});
        mp[key] = lst.begin();
        
        if (lst.size() > capacity) {
            mp.erase(lst.rbegin()->first);
            lst.pop_back();
        }
    }
};
```