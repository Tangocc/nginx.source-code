


### 1.字符串
```
typedef struct {
    size_t      len;
    u_char     *data;
} ngx_str_t;

```

nginx字符串是对C字符的简单封装，增加字符串长度字段。从源码中可以看出，nginx编码风格有两个特点：

- 数据类型命名方式以ngx_xxxx_t形式命名
- 关于字符串处理函数多以宏定义方式定义，例如：  

```
#define ngx_string(str)     { sizeof(str) - 1, (u_char *) str }
#define ngx_null_string     { 0, NULL }
#define ngx_str_set(str, text)                                               \
    (str)->len = sizeof(text) - 1; (str)->data = (u_char *) text
#define ngx_str_null(str)   (str)->len = 0; (str)->data = NULL

#define ngx_tolower(c)      (u_char) ((c >= 'A' && c <= 'Z') ? (c | 0x20) : c)
#define ngx_toupper(c)      (u_char) ((c >= 'a' && c <= 'z') ? (c & ~0x20) : c)

```


### 2.哈希表



```
typedef struct {
    void             *value;
    u_short           len;
    u_char            name[1];
} ngx_hash_elt_t;


typedef struct {
    ngx_hash_elt_t  **buckets;
    ngx_uint_t        size;
} ngx_hash_t;


typedef struct {
    ngx_hash_t       *hash;
    ngx_hash_key_pt   key;

    ngx_uint_t        max_size;
    ngx_uint_t        bucket_size;

    char             *name;
    ngx_pool_t       *pool;
    ngx_pool_t       *temp_pool;
} ngx_hash_init_t;

```

### 3.内存池
Nginx作为高性能到web服务器，自然需要满足高效的内存使用率和分配效率。
内存池满足上述两个要求，主要优点：

- 统一内存管理，避免内存碎片化，提高系统到使用率（Nginx做内存对齐处理，牺牲一定到使用率换来寻址速率）
- 避免多次向系统申请内存(涉及内核态和用户态转换)，提高内存到分配效率；
- 内存统一分配和销毁，避免内存泄露。

其数据结构如下所示：

```
typedef struct ngx_pool_large_s  ngx_pool_large_t;
typedef struct ngx_pool_s        ngx_pool_t;  

struct ngx_pool_cleanup_s {
    ngx_pool_cleanup_pt   handler;
    void                 *data;
    ngx_pool_cleanup_t   *next;
};


struct ngx_pool_large_s {
    ngx_pool_large_t     *next;//下一数据块地址
    void                 *alloc;//当前数据块地址
};

//内存池内存块
typedef struct {
    u_char               *last;
    u_char               *end;
    ngx_pool_t           *next;
    ngx_uint_t            failed;
} ngx_pool_data_t;

//内存池结构
struct ngx_pool_s {
    ngx_pool_data_t       d;//内存池到数据区，即是存储内存块
    size_t                max;//每次可分配最大内存，用于判定是走大内存分配还是小内存分配逻辑
    ngx_pool_t           *current;//当前内存快到
    ngx_chain_t          *chain;//缓冲区链表
    ngx_pool_large_t     *large;//分配大小大于max内存块的链表
    ngx_pool_cleanup_t   *cleanup;//销毁内存池回调函数
    ngx_log_t            *log;
};


```

```

ngx_pool_t *
ngx_create_pool(size_t size, ngx_log_t *log)
{
    ngx_pool_t  *p;

    p = ngx_memalign(NGX_POOL_ALIGNMENT, size, log);
    if (p == NULL) {
        return NULL;
    }

    p->d.last = (u_char *) p + sizeof(ngx_pool_t);
    p->d.end = (u_char *) p + size;
    p->d.next = NULL;
    p->d.failed = 0;

    size = size - sizeof(ngx_pool_t);
    p->max = (size < NGX_MAX_ALLOC_FROM_POOL) ? size : NGX_MAX_ALLOC_FROM_POOL;

    p->current = p;
    p->chain = NULL;
    p->large = NULL;
    p->cleanup = NULL;
    p->log = log;

    return p;
}
```









