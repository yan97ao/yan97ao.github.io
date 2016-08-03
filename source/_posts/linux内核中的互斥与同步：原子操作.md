---
title: linux内核中的互斥与同步：原子操作
date: 2016-06-19 13:21:17
tags:
	- kernel
	- concurrency
	- atomic
categories: tech
---

https://en.wikipedia.org/wiki/Atomicity_(database_systems)

# atomic类型定义
``` c
typedef struct {
	int counter;
} atomic_t;

#ifdef CONFIG_64BIT
typedef struct {
	long counter;
} atomic64_t;
#endif
```

<!---more-->
# 相关API

|函数名  |描述  | 
|--------|------|
| ATOMIC_INIT(i) | 初始化一个atomic_t |
| atomic_read(const atomic_t *v) | 原子读 |
| atomic_set(atomic_t *v, int i) | 原子设置 |
| atomic_add(int i, atomic_t *v) | 原子加 |
| atomic_sub(int i, atomic_t *v) | 原子减 |
| atomic_sub_and_test(int i, atomic_t *v) | 原子减并判断是否为0 |
| atomic_inc(atomic_t *v) | 原子加一 | 
| atomic_dec(atomic_t *v) | 原子减一 | 
| atomic_dec_and_test(atomic_t *v) | 原子减一并判断是否为0 | 
| atomic_inc_and_test(atomic_t *v) | 原子加一并判断是否为0 |
| atomic_add_negative(int i, atomic_t *v) | 原子加并判断是否为负数 | 
| atomic_add_return(int i, atomic_t *v) | 原子加并返回结果 |
| atomic_sub_return(int i, atomic_t *v) | 原子减并返回结果 |
| atomic_cmpxchg(atomic_t *v, int old, int new) | 原子比较并交换 |
| atomic_xchg(atomic_t *v, int new) | 原子交换 |
*

# 实现
## x86版本实现在arch/x86/include/asm/atomic.h中，其中有两类实现方法
1.直接加上LOCK_PREFIX
``` c
#ifdef CONFIG_SMP
#define LOCK_PREFIX_HERE \
		".pushsection .smp_locks,\"a\"\n"	\
		".balign 4\n"				\
		".long 671f - .\n" /* offset */		\
		".popsection\n"				\
		"671:"

#define LOCK_PREFIX LOCK_PREFIX_HERE "\n\tlock; "

#else /* ! CONFIG_SMP */
#define LOCK_PREFIX_HERE ""
#define LOCK_PREFIX ""
#endif
```
2.使用READ_ONCE和WRITE_ONCE
``` c
#define __READ_ONCE(x, check)						\
({									\
	union { typeof(x) __val; char __c[1]; } __u;			\
	if (check)							\
		__read_once_size(&(x), __u.__c, sizeof(x));		\
	else								\
		__read_once_size_nocheck(&(x), __u.__c, sizeof(x));	\
	__u.__val;							\
})
#define READ_ONCE(x) __READ_ONCE(x, 1)

#define __READ_ONCE_SIZE						\
({									\
	switch (size) {							\
	case 1: *(__u8 *)res = *(volatile __u8 *)p; break;		\
	case 2: *(__u16 *)res = *(volatile __u16 *)p; break;		\
	case 4: *(__u32 *)res = *(volatile __u32 *)p; break;		\
	case 8: *(__u64 *)res = *(volatile __u64 *)p; break;		\
	default:							\
		barrier();						\
		__builtin_memcpy((void *)res, (const void *)p, size);	\
		barrier();						\
	}								\
})

static __always_inline
void __read_once_size(const volatile void *p, void *res, int size)
{
	__READ_ONCE_SIZE;
}
```

``` c
#define WRITE_ONCE(x, val) \
({							\
	union { typeof(x) __val; char __c[1]; } __u =	\
		{ .__val = (__force typeof(x)) (val) }; \
	__write_once_size(&(x), __u.__c, sizeof(x));	\
	__u.__val;					\
})
static __always_inline void __write_once_size(volatile void *p, void *res, int size)
{
	switch (size) {
	case 1: *(volatile __u8 *)p = *(__u8 *)res; break;
	case 2: *(volatile __u16 *)p = *(__u16 *)res; break;
	case 4: *(volatile __u32 *)p = *(__u32 *)res; break;
	case 8: *(volatile __u64 *)p = *(__u64 *)res; break;
	default:
		barrier();
		__builtin_memcpy((void *)p, (const void *)res, size);
		barrier();
	}
}
```
另外还会根据具体的指令集支持情况选取atomic64_32.h和atomic64_64.h中的atomic64_t相关API实现，不再赘述
## ia64版本实现在arch/ia64/include/asm/atomic.h中
这个文件中利用宏展开生成了很多重复性的代码，看起来非常舒服
在read和set时，同样使用了READ_ONCE和WRITE_ONCE
但是对于其他操作，没有使用LOCK_PREFIX，而是根据情况使用了ia64_fetch_and_add或者ia64_atomic64_##op

# 典型应用
维护各种引用计数

# 参考
[1] Documentation/atomic_ops.txt
