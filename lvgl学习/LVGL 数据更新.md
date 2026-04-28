
---

# 📄 global_data 架构设计（MCU + RT-Thread + LVGL）

## 一、设计目标

针对嵌入式 UI（LVGL）数据更新，目标：

1. **统一数据源**
    
    - 全局状态仅存在一份：`g_state`
        
2. **高频数据优化**
    
    - 不直接触发 UI
        
    - 仅标记 dirty，由 UI 定时刷新
        
3. **低频数据即时响应**
    
    - 更新后立即 `lv_msg_send`
        
4. **线程安全**
    
    - RT-Thread mutex 保护
        
5. **UI 解耦**
    
    - UI 不直接访问全局变量
        
    - 使用 snapshot（快照）
        

---

## 二、整体架构

```
           ┌──────────────┐
           │ 数据来源层    │
           │ GPS / OBD    │
           │ Lap / Sensor │
           └──────┬───────┘
                  │
                  ▼
        ┌────────────────────┐
        │ global_state_edit   │
        │ (批量写入)          │
        └────────┬───────────┘
                 │
                 ▼
        ┌────────────────────┐
        │     g_state         │
        │   + dirty flags     │
        └────────┬───────────┘
                 │
      ┌──────────┴──────────┐
      │                     │
      ▼                     ▼
高频路径（timer）     低频路径（事件）
UI定时刷新           lv_msg_send
```

---

## 三、数据更新策略

### 1️⃣ 高频数据（例如：速度、时间、OBD）

- 更新方式：
    

```c
global_state_edit_begin(&state);
state->g_speed = speed;
dirty |= GLOBAL_DIRTY_SPEED;
global_state_edit_commit(dirty);
```

- 特点：
    
    - 不触发 UI
        
    - 只标记 dirty
        
    - UI 自己控制刷新频率
        

---

### 2️⃣ 低频数据（例如：单位、音量、模式）

```c
global_set_units(UNITS_KMH);
```

内部逻辑：

```c
g_state.units = units;
lv_msg_send(UI_MSG_UNIT_CHANGE, &units);
```

---

## 四、UI 更新机制（核心）

### ✔ 推荐方式：Snapshot

```c
uint32_t dirty = global_state_take_snapshot(&snap);
```

### ✔ UI Timer 模型

```c
static void ui_timer_cb(lv_timer_t *t)
{
    global_state_t snap;
    uint32_t dirty;

    dirty = global_state_take_snapshot(&snap);
    if(dirty == 0) return;

    if(dirty & GLOBAL_DIRTY_SPEED) {
        lv_label_set_text_fmt(speed_label, "%d", snap.g_speed);
    }
}
```

---

## 五、线程安全策略

### ✔ 写入端（生产者）

```
lock
  修改 g_state
  设置 dirty
unlock
```

### ✔ UI端（消费者）

```
lock
  memcpy snapshot
  获取 dirty
  清 dirty
unlock

UI刷新（无锁）
```

---

## 六、关键设计原则

### 🚫 禁止

```c
global_lock();
lv_label_set_text(...);   // ❌ 严禁
global_unlock();
```

---

### ✅ 正确

```c
global_lock();
copy data
global_unlock();

lv_label_set_text(...);   // ✔
```

---

## 七、优点总结

|优点|说明|
|---|---|
|高性能|高频数据不触发 UI|
|解耦|UI 与数据完全分离|
|安全|mutex 控制|
|可扩展|新字段无需改核心逻辑|
|易维护|统一入口|


---

# ✔ 最后一句总结（很关键）

你现在这套架构已经是：

> **嵌入式 UI 最优解之一（dirty + snapshot + 分层更新）**

再往上就是：

- 数据总线（你已经在用 lv_msg）
    
- 或 MVVM / 数据驱动 UI（LVGL 不太适合）
    

---

如果你下一步想优化，我可以帮你做这几个进阶（很适合你项目）：

1. **自动 dirty 标记宏（减少手写错误）**
    
2. **UI 自动绑定系统（类似 Vue）**
    
3. **数据录制 + 回放（你做赛道非常有用）**
    
4. **双缓冲 g_state（极限性能版）**
    

你现在这一步，已经从“能用”进化到“工程级设计”了 👍