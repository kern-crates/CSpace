CSpace模块为ReL4(seL4 的Rust重写) 提供了capability的抽象以及cspace的管理。

## What's a capability
- A capability is a unique, unforgeable token that gives the possessor permission to access an entity or object in system.

- capability 是访问系统中一切实体或对象的令牌，这个令牌包含了某些访问权力，只有持有这个令牌，才能按照特定的方式访问系统中的某个对象。

- 可以将 capability 视为一个拥有访问权限的指针。

## capability 类型
- Untyped: 空闲物理内存块。
- CapEndpointCap：用于IPC的内核对象。
- CapNotificationCap：用于异步通知的内核对象。
- CapReplyCap：用于响应的内核对象。
- CapCNodeCap：CSpace内核对象。
- CapThreadCap：线程控制块。
- CapIrqControlCap：中断控制cap。
- CapIrqHandlerCap：中断处理cap。
- CapZombieCap：用于资源回收的中间态cap。
- CapDomainCap：调度域cap。
- CapFrameCap：物理页cap。
- CapPageTableCap：页表/地址空间的内核对象。

## Example

```rust
use cspace::{cte_t, cap_t, mdb_node_t};
use common::structures::exception_t;

// 构建一个cnode cap和对应的slot
let cnode_cap = cap_t::new_cnode_cap(capCNodeRadix, capCNodeGuardSize, capCNodeGuard, capCNodePtr);
assert_eq!(cnode_cap.get_cap_type(), CapTag::CapCNodeCap);
let mut src_slot = cte_t { cap: cnode_cap, cteMDBNode: mdb_node_t::default() };

// 构建一个空的dest，并将第一个slot的cap复制派生到dest中
let mut dest_slot = cte_t { cap: cap_t::new_null_cap(), cteMDBNode: mdb_node_t::default() };
let dc_ret = src_slot.derive_cap(cnode_cap);
cte_insert(&dc_ret.cap, src_slot, dest_slot);
assert!(src_slot.is_mdb_parent_of(dest_slot));

// src slot通过revoke回收之前派生出去的cap
let _ = src_slot.revoke();
assert_eq!(src_slot.ensure_no_children(), exception_t::EXCEPTION_NONE);
```

## 相关project
- [ReL4](https://github.com/rel4team/rel4_kernel)