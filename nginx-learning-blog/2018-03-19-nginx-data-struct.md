


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