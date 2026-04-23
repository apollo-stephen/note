# 链表知识体系总结

## 1. 链表基础概念

### 1.1 链表与数组的对比

| 特性 | 数组 | 链表 |
|------|------|------|
| 内存分配 | 连续内存空间 | 非连续内存空间 |
| 访问方式 | 随机访问(通过索引) | 顺序访问(必须从头遍历) |
| 插入/删除效率 | O(n)需要数据搬移 | O(1)只需修改指针 |
| 内存利用率 | 可能浪费空间 | 按需分配，利用率高 |

### 1.2 链表节点定义

**力扣简化版（单链表）：**
```cpp
struct ListNode {
    int val;
    ListNode *next;
    ListNode(int x) : val(x), next(nullptr) {}
};
```

**实际开发版（双链表+模板）：**
```cpp
template<typename T>
struct Node {
    T val;
    Node<T> *next;
    Node<T> *prev;
    Node(T value) : val(value), next(nullptr), prev(nullptr) {}
};
```

## 2. 链表基本操作详解

### 2.1 单链表操作

#### 遍历与查找
```cpp
// 遍历单链表
ListNode* p = head;
while (p != nullptr) {
    cout << p->val << endl;
    p = p->next;
}

// 按索引查找
ListNode* getNode(ListNode* head, int index) {
    ListNode* p = head;
    for (int i = 0; i < index; i++) {
        p = p->next;
    }
    return p;
}
```

#### 插入操作
```cpp
// 头部插入
ListNode* newNode = new ListNode(0);
newNode->next = head;
head = newNode;

// 尾部插入
ListNode* p = head;
while (p->next != nullptr) p = p->next;
p->next = new ListNode(6);

// 中间插入（在索引2后插入）
ListNode* p = head;
for (int i = 0; i < 2; i++) p = p->next;
ListNode* newNode = new ListNode(66);
newNode->next = p->next;
p->next = newNode;
```

#### 删除操作
```cpp
// 头部删除
ListNode* temp = head;
head = head->next;
delete temp;

// 尾部删除
ListNode* p = head;
while (p->next->next != nullptr) p = p->next;
delete p->next;
p->next = nullptr;

// 中间删除（删除索引3的节点）
ListNode* p = head;
for (int i = 0; i < 2; i++) p = p->next;
ListNode* toDelete = p->next;
p->next = p->next->next;
delete toDelete;
```

### 2.2 双链表操作

双链表相比单链表的优势：
- 支持双向遍历
- 尾部操作更高效
- 删除操作更简单

#### 双链表插入
```cpp
// 头部插入
Node<int>* newHead = new Node<int>(0);
newHead->next = head;
head->prev = newHead;
head = newHead;

// 中间插入（需要调整四个指针）
Node<int>* newNode = new Node<int>(66);
newNode->next = p->next;
newNode->prev = p;
p->next->prev = newNode;
p->next = newNode;
```

## 3. 链表实现技巧

### 3.1 虚拟头节点技巧

**解决的问题：**
- 统一头尾操作逻辑
- 避免空指针异常
- 简化边界条件处理

**实现原理：**
```
空链表：dummyHead <-> dummyTail
有元素：dummyHead <-> 1 <-> 2 <-> 3 <-> dummyTail
```

**代码示例：**
```cpp
template<typename T>
class MyLinkedList {
private:
    struct Node {
        T val;
        Node* next;
        Node* prev;
        Node(T value) : val(value), next(nullptr), prev(nullptr) {}
    };
    
    Node* head; // 虚拟头节点
    Node* tail; // 虚拟尾节点
    int size;
    
public:
    MyLinkedList() {
        head = new Node(T());
        tail = new Node(T());
        head->next = tail;
        tail->prev = head;
        size = 0;
    }
    
    ~MyLinkedList() {
        // 析构函数需要释放所有节点
        while (size > 0) {
            removeFirst();
        }
        delete head;
        delete tail;
    }
};
```

### 3.2 单链表优化技巧

**持有尾节点引用：**
```cpp
template<typename T>
class MyLinkedList2 {
private:
    struct Node {
        T val;
        Node* next;
        Node(T value) : val(value), next(nullptr) {}
    };
    
    Node* head; // 虚拟头节点
    Node* tail; // 实际尾节点引用
    int size;
    
public:
    MyLinkedList2() {
        head = new Node(T());
        tail = head;
        size = 0;
    }
    
    // 尾部插入优化为O(1)
    void addLast(T e) {
        Node* newNode = new Node(e);
        tail->next = newNode;
        tail = newNode;
        size++;
    }
};
```

## 4. 内存管理注意事项

### 4.1 C++内存管理

**正确使用new/delete：**
```cpp
// 创建节点
ListNode* node = new ListNode(1);

// 删除节点
ListNode* temp = node;
node = node->next;
delete temp; // 必须手动释放内存
```

**良好习惯：**
```cpp
// 删除节点时的完整操作
ListNode* toDelete = head;
head = head->next;
toDelete->next = nullptr; // 断开指针
delete toDelete;          // 释放内存
```

### 4.2 指针操作规范

双链表删除时的完整操作：
```cpp
// 1. 调整前后节点的指针
prev->next = next;
next->prev = prev;

// 2. 断开被删除节点的指针
toDelete->prev = nullptr;
toDelete->next = nullptr;

// 3. 释放内存
delete toDelete;
```

## 5. 复杂度分析

| 操作 | 单链表 | 双链表 | 数组 |
|------|--------|--------|------|
| 访问 | O(n) | O(n) | O(1) |
| 头部插入 | O(1) | O(1) | O(n) |
| 尾部插入 | O(n)或O(1)* | O(1) | O(1)或O(n)** |
| 中间插入 | O(n) | O(n) | O(n) |
| 头部删除 | O(1) | O(1) | O(n) |
| 尾部删除 | O(n) | O(1) | O(1) |

*单链表如果有尾节点引用则为O(1)
**数组如果不需要扩容则为O(1)

## 6. 完整链表实现示例

### 6.1 双链表完整实现

```cpp
#include <iostream>
#include <stdexcept>

template<typename T>
class DoublyLinkedList {
private:
    struct Node {
        T data;
        Node* prev;
        Node* next;
        Node(const T& value) : data(value), prev(nullptr), next(nullptr) {}
    };
    
    Node* head;
    Node* tail;
    int size;
    
public:
    DoublyLinkedList() : head(nullptr), tail(nullptr), size(0) {}
    
    ~DoublyLinkedList() {
        clear();
    }
    
    // 在尾部添加元素
    void push_back(const T& value) {
        Node* newNode = new Node(value);
        if (tail == nullptr) {
            // 空链表
            head = tail = newNode;
        } else {
            tail->next = newNode;
            newNode->prev = tail;
            tail = newNode;
        }
        size++;
    }
    
    // 在头部添加元素
    void push_front(const T& value) {
        Node* newNode = new Node(value);
        if (head == nullptr) {
            head = tail = newNode;
        } else {
            newNode->next = head;
            head->prev = newNode;
            head = newNode;
        }
        size++;
    }
    
    // 删除尾部元素
    void pop_back() {
        if (tail == nullptr) return;
        
        Node* temp = tail;
        if (head == tail) {
            // 只有一个元素
            head = tail = nullptr;
        } else {
            tail = tail->prev;
            tail->next = nullptr;
        }
        delete temp;
        size--;
    }
    
    // 删除头部元素
    void pop_front() {
        if (head == nullptr) return;
        
        Node* temp = head;
        if (head == tail) {
            head = tail = nullptr;
        } else {
            head = head->next;
            head->prev = nullptr;
        }
        delete temp;
        size--;
    }
    
    // 清空链表
    void clear() {
        while (head != nullptr) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
        tail = nullptr;
        size = 0;
    }
    
    // 获取大小
    int getSize() const { return size; }
    
    // 判断是否为空
    bool empty() const { return size == 0; }
    
    // 打印链表
    void print() const {
        Node* current = head;
        while (current != nullptr) {
            std::cout << current->data << " ";
            current = current->next;
        }
        std::cout << std::endl;
    }
};
```

### 6.2 单链表完整实现

```cpp
#include <iostream>
#include <stdexcept>

template<typename T>
class SinglyLinkedList {
private:
    struct Node {
        T data;
        Node* next;
        Node(const T& value) : data(value), next(nullptr) {}
    };
    
    Node* head;
    Node* tail;
    int size;
    
public:
    SinglyLinkedList() : head(nullptr), tail(nullptr), size(0) {}
    
    ~SinglyLinkedList() {
        clear();
    }
    
    // 在尾部添加元素
    void push_back(const T& value) {
        Node* newNode = new Node(value);
        if (tail == nullptr) {
            head = tail = newNode;
        } else {
            tail->next = newNode;
            tail = newNode;
        }
        size++;
    }
    
    // 在头部添加元素
    void push_front(const T& value) {
        Node* newNode = new Node(value);
        if (head == nullptr) {
            head = tail = newNode;
        } else {
            newNode->next = head;
            head = newNode;
        }
        size++;
    }
    
    // 删除头部元素
    void pop_front() {
        if (head == nullptr) return;
        
        Node* temp = head;
        head = head->next;
        if (head == nullptr) {
            tail = nullptr;
        }
        delete temp;
        size--;
    }
    
    // 清空链表
    void clear() {
        while (head != nullptr) {
            Node* temp = head;
            head = head->next;
            delete temp;
        }
        tail = nullptr;
        size = 0;
    }
    
    // 获取大小
    int getSize() const { return size; }
    
    // 判断是否为空
    bool empty() const { return size == 0; }
    
    // 打印链表
    void print() const {
        Node* current = head;
        while (current != nullptr) {
            std::cout << current->data << " ";
            current = current->next;
        }
        std::cout << std::endl;
    }
};
```

## 7. 边界条件处理

### 7.1 索引检查方法

```cpp
template<typename T>
class SafeLinkedList {
private:
    // ... 链表实现
    
    bool isElementIndex(int index) const {
        return index >= 0 && index < size;
    }
    
    bool isPositionIndex(int index) const {
        return index >= 0 && index <= size;
    }
    
    void checkElementIndex(int index) const {
        if (!isElementIndex(index)) {
            throw std::out_of_range("Index: " + std::to_string(index) + 
                                   ", Size: " + std::to_string(size));
        }
    }
    
    void checkPositionIndex(int index) const {
        if (!isPositionIndex(index)) {
            throw std::out_of_range("Index: " + std::to_string(index) + 
                                   ", Size: " + std::to_string(size));
        }
    }
};
```

## 8. 学习建议

### 8.1 掌握顺序
1. 先理解单链表的基本操作
2. 掌握双链表的指针调整逻辑
3. 学习虚拟头节点技巧简化代码
4. 实践完整链表实现

### 8.2 调试技巧
- 画图理解指针变化
- 使用小规模测试用例
- 重点关注边界情况（空链表、头尾操作）

### 8.3 常见错误
- 忘记更新size计数器
- 指针操作顺序错误
- 边界条件处理不全
- 内存管理不当（忘记delete）

### 8.4 实践练习
```cpp
// 测试代码示例
int main() {
    DoublyLinkedList<int> list;
    
    // 测试基本操作
    list.push_back(1);
    list.push_back(2);
    list.push_front(0);
    list.print(); // 输出: 0 1 2
    
    list.pop_front();
    list.print(); // 输出: 1 2
    
    return 0;
}
```

通过系统学习以上内容，应该能够全面掌握链表数据结构的原理、实现和应用。建议结合力扣707题进行实践练习，加深理解。