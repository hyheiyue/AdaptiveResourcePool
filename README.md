# 🌊 AdaptiveResourcePool

**AdaptiveResourcePool** is a **thread-safe, intelligent resource manager** for C++.
It keeps a pool of reusable objects (GPU contexts, inference engines, DB connections, etc.), and can **release** unused ones to save resources or **restore** them when needed — all customizable via user-defined rules.

---

## ✨ Features

- **🔄 Adaptive lifecycle**

  - Release idle resources to save memory.

  - Restore them when demand increases.

- **🧵 Thread-safe**

  - Safe for concurrent use across multiple threads.

- **⚙ Fully customizable**

  - User defines initialization, release, and restoration logic.

- **📊 Resource tracking**

  - Check idle resources instantly with `idleCount()`.

---

## 📦 Installation

Just include the header in your project:

```C++
#include "AdaptiveResourcePool.h"
```


---

## 🚀 Quick Start

### 1️⃣ Define your resource type

```C++
struct MyResource {
    int id;
    void doWork() { /* ... */ }
};
```


### 2️⃣ Create the pool

```C++
AdaptiveResourcePool<MyResource>::Params params;

// Initial resource creation
params.resource_initializer = []() {
    std::vector<std::unique_ptr<MyResource>> res;
    for (int i = 0; i < 4; ++i) {
        res.push_back(std::make_unique<MyResource>(MyResource{i}));
    }
    return res;
};

// Restore released resource
params.restore_func = [](size_t index) {
    return std::make_unique<MyResource>(MyResource{static_cast<int>(index)});
};

// Release resource
params.release_func = [](std::unique_ptr<MyResource>& res) {
    // Optional cleanup logic
};

// When to restore?
params.can_restore = [](size_t active_count) {
    return active_count < 4;
};

// When to release?
params.should_release = [](size_t active_count) {
    return active_count > 2;
};

// Optional logging
params.logger = [](const std::string& msg) {
    std::cout << "[Pool] " << msg << "\n";
};

AdaptiveResourcePool<MyResource> pool(params);
```


### 3️⃣ Acquire and release

```C++
if (auto* res = pool.acquire()) {
    res->doWork();
    pool.release(res);
}
```


### 4️⃣ Check idle resources

```C++
std::cout << "Idle resources: " << pool.idleCount() << "\n";
```


---

## 📖 API Overview

|Method|Description|
|-|-|
|`T* acquire()`|Acquire a free resource, or return `nullptr` if none available.|
|`void release(T* resource)`|Release a resource back into the pool.|
|`size_t idleCount() const`|Return number of idle (available) resources.|

---

## 🔒 Thread Safety

- All public methods are **thread-safe**.

- Fine-grained locking ensures multiple threads can acquire/release simultaneously without data races.

---

## 💡 Example Use Cases

- GPU inference engine pooling

- Database connection pooling

- Network socket pooling

- Buffer/memory block pooling

---

## 📌 Notes

- Resource release/restore policies are **entirely controlled by you** via callbacks.

- The pool automatically calls `release_func` before freeing resources and `restore_func` when restoring them.

---

