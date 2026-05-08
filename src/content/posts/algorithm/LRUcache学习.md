---
title: LRUCache 编写中容易遇到的问题
published: 2026-05-08
tags: [CPP,算法]
category: algorithm
image: 'https://t.alcy.cc/moez'
draft: false
---

# LRUCache 编写中容易遇到的问题

本笔记记录我在实现 LRUCache 过程中常见的坑，结合之前几个版本中出现过的错误整理而成。

## 1. 节点与哈希表的对应关系
- **节点没有保存 key**：淘汰时无法从 `map_` 删除正确的条目，导致 `map_` 留下悬空指针。
- **节点 key 未赋值**：即使有 `key_` 字段，如果构造时忘记赋值，`map_.erase(last->key_)` 会删除错误条目或删除失败。

## 2. put() 逻辑错误
- **已存在 key 直接 return**：没有更新值，也没有更新最近使用顺序。
- **已存在 key 又创建新节点**：旧节点留在链表里，新节点覆盖 `map_[key]`，会造成链表中“孤儿节点”与内存泄漏。
- **新节点没插入链表**：创建了节点但插入的却是错误指针，导致链表结构被破坏。

## 3. 淘汰逻辑错误
- **删除错误的哈希表项**：在淘汰时用 `map_.erase(target)`，但 `target` 不是被淘汰节点对应的迭代器。
- **容量为 0 的处理**：若不禁止 `capacity_ == 0`，可能会删除哨兵节点，破坏链表结构。

## 4. LRU 顺序维护不一致
- **get 与 put 采用不同方向**：`get()` 把节点移到头部，但 `put()` 插到尾部，造成“最近使用”的语义混乱，淘汰策略错误。

## 5. 析构与内存安全问题
- **析构函数中使用已释放的指针**：删除 `head_` 后仍用 `head_->next_` 继续遍历，属于未定义行为。
- **类可拷贝导致双重释放**：有裸指针但未禁用拷贝/赋值，会在拷贝对象销毁时双重释放。

## 6. 计数与容量维护
- **nums_ 未同步更新**：淘汰时如果不减少计数，`nums_` 会一直等于 `capacity_`，影响后续逻辑或可读性。

## 7. 其他可维护性问题
- **命名歧义**：`get(K value)` 这样的参数名容易误导，应明确叫 `key`。
- **拼写问题**：`capcity_` 建议改为 `capacity_`。

## 建议的基本规范
- `map_` 只存真实节点，绝不存 `head_`/`tail_` 哨兵。
- `put()`：存在则更新值并移动到头部；不存在则插入并可能淘汰尾部。
- `get()`：命中则移动到头部并返回值；未命中返回默认值或抛异常。
- 禁用拷贝，必要时实现移动语义或使用智能指针。

## 结合当前代码的对照说明
- 节点保存 `key_` 和 `val_`，保证淘汰时能从 `map_` 删除正确项：
```cpp
template<typename K,typename T>
struct Node
{
	Node* pre_ = nullptr;
	Node* next_ = nullptr;
	T val_;
	K key_;
	Node(K key,T val):key_(key),val_(val){}
	Node(){
		val_ = T{};
		key_ = K{};
	}
};
```
- `put()` 对已存在 key 会更新 `val_` 并移动到头部：
```cpp
auto target = map_.find(key);
if(target != map_.end()){
	remove_(target->second,false);
	target->second->val_ = value;
	insert_(head_,target->second);
	return;
}
```
- 新插入节点进入头部并在容量满时淘汰尾部：
```cpp
auto tmp = new Node<K,V>(key,value);
map_[key] = tmp;
if(nums_ >= capacity_){
	Node<K,V>* last = tail_->pre_;
	map_.erase(last->key_);
	remove_(last);
}else{
	nums_++;
}
insert_(head_,tmp);
```
- 构造函数禁止 `capacity_ == 0`：
```cpp
LRUCache(int capacity):capacity_(capacity){
	if(capacity == 0) throw runtime_error("capacity can not be zero");
	head_ = new Node<K,V>();
	tail_ = new Node<K,V>();
	head_->next_ = tail_;
	tail_->pre_ = head_;
}
```
- 析构函数按链表顺序释放：
```cpp
~LRUCache(){
	auto iter = head_;
	while(iter){
		auto tmp = iter->next_;
		delete iter;
		iter = tmp;
	}
}
```

## 源码
```cpp
#include <iostream>
#include <unordered_map>
#include <stdexcept>

using namespace std;

template<typename K,typename T>
struct Node
{
	Node* pre_ = nullptr;
	Node* next_ = nullptr;
	T val_;
	K key_;
	Node(K key,T val):key_(key),val_(val){}
	Node(){
		val_ = T{};
		key_ = K{};
	}
};



template<typename K,typename V>
class LRUCache{

private:
	Node<K,V>* head_;
	Node<K,V>* tail_;
	int capacity_ = 0;
	int nums_ = 0;
	unordered_map<K,Node<K,V>*> map_;

    
	void insert_(Node<K,V>* pre,Node<K,V>* target){
			auto next = pre->next_;
			pre->next_ = target;
			target->pre_ = pre;
			target->next_ = next;
			if(next){
				next->pre_ = target;
			}
	}

	void remove_(Node<K,V>* target,bool is_delete = true){
		auto pre = target->pre_;
		auto next = target->next_;
		if(pre) pre->next_ = next;
		if(next) next->pre_ = pre;
		if(is_delete)  delete target;
	}

	LRUCache(const LRUCache& cache) = delete;

	LRUCache& operator=(const LRUCache& cache)= delete;

public:
	LRUCache(int capacity):capacity_(capacity){
		if(capacity == 0) throw runtime_error("capacity can not be zero");
		head_ = new Node<K,V>();
		tail_ = new Node<K,V>();
		head_->next_ = tail_;
		tail_->pre_ = head_;
	}
   
	void put(K key,V value){
		auto target = map_.find(key);
		if(target != map_.end()){
			remove_(target->second,false);
			target->second->val_ = value;
			insert_(head_,target->second);
			return;
		}
		auto tmp = new Node<K,V>(key,value);
		map_[key] = tmp;
		if(nums_ >= capacity_){
			Node<K,V>* last = tail_->pre_;
			map_.erase(last->key_);
			remove_(last);
		}else{
			nums_++;
		}
		insert_(head_,tmp);
	}

	V get(K key){
		auto target = map_.find(key);
		if(target == map_.end()){
			return V{};
		}

		remove_(target->second,false);
		insert_(head_,target->second);
		return target->second->val_;

	}

	~LRUCache(){
		auto iter = head_;
		while(iter){
			auto tmp = iter->next_;
			delete iter;
			iter = tmp;
		}
	}

};


int main()
{
	try
	{
		LRUCache<int,int> cache(1);
		cache.put(1,1);
		cache.put(2,2);
		cout<<cache.get(2);
	}
	catch(const std::exception& e)
	{
		std::cerr << e.what() << '\n';
	}
}
```
