---
title: 'python 内核分析（五）：Python字符串类型对象'
date: 2018-10-05 16:14:17
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/default.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# 关于python3

在看任何语言的字符串之前，我们首先应该看其编码问题。

python3中默认使用unicode编码，我们找到它的定义：

```c
/* Py_UCS4 and Py_UCS2 are typedefs for the respective
   unicode representations. */
typedef uint32_t Py_UCS4;
typedef uint16_t Py_UCS2;
typedef uint8_t Py_UCS1;

typedef struct {
    PyCompactUnicodeObject _base;
    union {
        void *any;
        Py_UCS1 *latin1;
        Py_UCS2 *ucs2;
        Py_UCS4 *ucs4;
    } data;                     /* Canonical, smallest-form Unicode buffer */
} PyUnicodeObject;

```

在`[Objects/unicodeoject.c]`中，提供了许多创建unicode对象的方法。我们考虑字符串，所以寻找`PyUnicode_FromString()`

```c
[Objects/unicodeoject.c]

PyObject *
PyUnicode_FromString(const char *u)
{
    size_t size = strlen(u);
    if (size > PY_SSIZE_T_MAX) {
        PyErr_SetString(PyExc_OverflowError, "input too long");
        return NULL;
    }
    return PyUnicode_DecodeUTF8Stateful(u, (Py_ssize_t)size, NULL, NULL);
}
```

在这里，其设置了一个数据的上限`PY_SSIZE_T_MAX`，其值为`((Py_ssize_t)(((size_t)-1)>>1))`，考虑我们64位系统，`size_t`为`__int64`。

明显的，当其没有溢出时，就转入下一步。

```c
[Objects/unicodeoject.c]

PyObject *
PyUnicode_DecodeUTF8Stateful(const char *s,
                             Py_ssize_t size,
                             const char *errors,
                             Py_ssize_t *consumed)
{
    _PyUnicodeWriter writer;
    const char *starts = s;
    const char *end = s + size;

    Py_ssize_t startinpos;
    Py_ssize_t endinpos;
    const char *errmsg = "";
    PyObject *error_handler_obj = NULL;
    PyObject *exc = NULL;
    _Py_error_handler error_handler = _Py_ERROR_UNKNOWN;

    if (size == 0) {
        if (consumed)
            *consumed = 0;
        _Py_RETURN_UNICODE_EMPTY();
    }

/* ASCII is equivalent to the first 128 ordinals in Unicode. */
    if (size == 1 && (unsigned char)s[0] < 128) {
        if (consumed)
            *consumed = 1;
        return get_latin1_char((unsigned char)s[0]);
    }

    _PyUnicodeWriter_Init(&writer);
    writer.min_length = size;
    if (_PyUnicodeWriter_Prepare(&writer, writer.min_length, 127) == -1)
        goto onError;

    writer.pos = ascii_decode(s, end, writer.data);
    s += writer.pos;
    while (s < end) {
        Py_UCS4 ch;
        int kind = writer.kind;

        if (kind == PyUnicode_1BYTE_KIND) {
            if (PyUnicode_IS_ASCII(writer.buffer))
                ch = asciilib_utf8_decode(&s, end, writer.data, &writer.pos);
            else
                ch = ucs1lib_utf8_decode(&s, end, writer.data, &writer.pos);
        } else if (kind == PyUnicode_2BYTE_KIND) {
            ch = ucs2lib_utf8_decode(&s, end, writer.data, &writer.pos);
        } else {
            assert(kind == PyUnicode_4BYTE_KIND);
            ch = ucs4lib_utf8_decode(&s, end, writer.data, &writer.pos);
        }

        switch (ch) {
        case 0:
            if (s == end || consumed)
                goto End;
            errmsg = "unexpected end of data";
            startinpos = s - starts;
            endinpos = end - starts;
            break;
        case 1:
            errmsg = "invalid start byte";
            startinpos = s - starts;
            endinpos = startinpos + 1;
            break;
        case 2:
        case 3:
        case 4:
            errmsg = "invalid continuation byte";
            startinpos = s - starts;
            endinpos = startinpos + ch - 1;
            break;
        default:
            if (_PyUnicodeWriter_WriteCharInline(&writer, ch) < 0)
                goto onError;
            continue;
        }

        if (error_handler == _Py_ERROR_UNKNOWN)
            error_handler = get_error_handler(errors);

        switch (error_handler) {
        case _Py_ERROR_IGNORE:
            s += (endinpos - startinpos);
            break;

        case _Py_ERROR_REPLACE:
            if (_PyUnicodeWriter_WriteCharInline(&writer, 0xfffd) < 0)
                goto onError;
            s += (endinpos - startinpos);
            break;

        case _Py_ERROR_SURROGATEESCAPE:
        {
            Py_ssize_t i;

            if (_PyUnicodeWriter_PrepareKind(&writer, PyUnicode_2BYTE_KIND) < 0)
                goto onError;
            for (i=startinpos; i<endinpos; i++) {
                ch = (Py_UCS4)(unsigned char)(starts[i]);
                PyUnicode_WRITE(writer.kind, writer.data, writer.pos,
                                ch + 0xdc00);
                writer.pos++;
            }
            s += (endinpos - startinpos);
            break;
        }

        default:
            if (unicode_decode_call_errorhandler_writer(
                    errors, &error_handler_obj,
                    "utf-8", errmsg,
                    &starts, &end, &startinpos, &endinpos, &exc, &s,
                    &writer))
                goto onError;
        }
    }

End:
    if (consumed)
        *consumed = s - starts;

    Py_XDECREF(error_handler_obj);
    Py_XDECREF(exc);
    return _PyUnicodeWriter_Finish(&writer);

onError:
    Py_XDECREF(error_handler_obj);
    Py_XDECREF(exc);
    _PyUnicodeWriter_Dealloc(&writer);
    return NULL;
}

```

`PyUnicode_DecodeUTF8Stateful()`这个函数就会自动判断格式。

# 字符串对象共享机制

我们在之前讨论过了小整数的机制，其预先实现了小整数的dict，再对其进行引用。

字符串也同样有这种类似的引用机制itern

```c
[Objects/unicodeoject.c]

void
PyUnicode_InternInPlace(PyObject **p)
{
    PyObject *s = *p;
    PyObject *t;
#ifdef Py_DEBUG
    assert(s != NULL);
    assert(_PyUnicode_CHECK(s));
#else
    if (s == NULL || !PyUnicode_Check(s))
        return;
#endif
    /* If it's a subclass, we don't really know what putting
       it in the interned dict might do. */
    if (!PyUnicode_CheckExact(s))
        return;
    if (PyUnicode_CHECK_INTERNED(s))
        return;
    if (interned == NULL) {
        interned = PyDict_New();
        if (interned == NULL) {
            PyErr_Clear(); /* Don't leave an exception */
            return;
        }
    }
    Py_ALLOW_RECURSION
    t = PyDict_SetDefault(interned, s, s);
    Py_END_ALLOW_RECURSION
    if (t == NULL) {
        PyErr_Clear();
        return;
    }
    if (t != s) {
        Py_INCREF(t);
        Py_SETREF(*p, t);
        return;
    }
    /* The two references in interned are not counted by refcnt.
       The deallocator will take care of this */
    Py_REFCNT(s) -= 2;
    _PyUnicode_STATE(s).interned = SSTATE_INTERNED_MORTAL;
}

```

传入的字符串首先经过类型和状态检查`PyUnicode_Check()`、`PyUnicode_CheckExact()`、`PyUnicode_CHECK_INTERNED()`。如果都通过，且没有被`interned`，就新建dict。

特别的`Py_REFCNT(s) -= 2`，由于对象添加进入dict时会对对其引用进行两次`+1`操作，所以需要对引用`-2`平衡。

注意其仍旧是生成了一个临时字符串对象，在引用交接后将其销毁了。

以c的字符串创建一样需要创建对象：

```c
[Objects/unicodeoject.c]

PyObject *
PyUnicode_InternFromString(const char *cp)
{
    PyObject *s = PyUnicode_FromString(cp);
    if (s == NULL)
        return NULL;
    PyUnicode_InternInPlace(&s);
    return s;
}

```

我们看见字符串对象有属性`interned`，在经历了`PyUnicode_InternInPlace()`后会改变，其值有两种选择`SSTATE_INTERNED_MORTAL`和`SSTATE_INTERNED_IMMORTAL`，分别对应了其是否会被销毁。

我们可以在`PyUnicode_InternImmortal()`中更改状态。

```c
[Objects/unicodeoject.c]

void
PyUnicode_InternImmortal(PyObject **p)
{
    PyUnicode_InternInPlace(p);
    if (PyUnicode_CHECK_INTERNED(*p) != SSTATE_INTERNED_IMMORTAL) {
        _PyUnicode_STATE(*p).interned = SSTATE_INTERNED_IMMORTAL;
        Py_INCREF(*p);
    }
}
```

# 字符缓冲池

单个unicode字符就直接以一个缓冲池保存

```c
[Objects/unicodeoject.c]

/* Single character Unicode strings in the Latin-1 range are being
   shared as well. */
static PyObject *unicode_latin1[256] = {NULL};
```

先对所创建的字符串（只有一个字符）对象进行intern操作，再将inter的结果缓存到字符缓冲池`unicode_latin1`中,两者都是指向同一个字符对象。

# PyUnicodeObject拼接

我们上面讲了每个PyUnicodeObject对象都是一个不可变对象。我们使用`+`来拼接字符串会导致`n-1`次的内存搬运工作。

```c
[Objects/unicodeoject.c]

/* Concat to string or Unicode object giving a new Unicode object. */

PyObject *
PyUnicode_Concat(PyObject *left, PyObject *right)
{
    PyObject *result;
    Py_UCS4 maxchar, maxchar2;
    Py_ssize_t left_len, right_len, new_len;

    if (ensure_unicode(left) < 0)
        return NULL;

    if (!PyUnicode_Check(right)) {
        PyErr_Format(PyExc_TypeError,
                     "can only concatenate str (not \"%.200s\") to str",
                     right->ob_type->tp_name);
        return NULL;
    }
    if (PyUnicode_READY(right) < 0)
        return NULL;

    /* Shortcuts */
    if (left == unicode_empty)
        return PyUnicode_FromObject(right);
    if (right == unicode_empty)
        return PyUnicode_FromObject(left);

    left_len = PyUnicode_GET_LENGTH(left);
    right_len = PyUnicode_GET_LENGTH(right);
    if (left_len > PY_SSIZE_T_MAX - right_len) {
        PyErr_SetString(PyExc_OverflowError,
                        "strings are too large to concat");
        return NULL;
    }
    new_len = left_len + right_len;

    maxchar = PyUnicode_MAX_CHAR_VALUE(left);
    maxchar2 = PyUnicode_MAX_CHAR_VALUE(right);
    maxchar = Py_MAX(maxchar, maxchar2);

    /* Concat the two Unicode strings */
    result = PyUnicode_New(new_len, maxchar);
    if (result == NULL)
        return NULL;
    _PyUnicode_FastCopyCharacters(result, 0, left, 0, left_len);
    _PyUnicode_FastCopyCharacters(result, left_len, right, 0, right_len);
    assert(_PyUnicode_CheckConsistency(result, 1));
    return result;
}
```

官方推荐使用`join`操作。

```c
[Objects/unicodeoject.c]


/*[clinic input]
str.join as unicode_join

    iterable: object
    /

Concatenate any number of strings.

The string whose method is called is inserted in between each given string.
The result is returned as a new string.

Example: '.'.join(['ab', 'pq', 'rs']) -> 'ab.pq.rs'
[clinic start generated code]*/

static PyObject *
unicode_join(PyObject *self, PyObject *iterable)
/*[clinic end generated code: output=6857e7cecfe7bf98 input=2f70422bfb8fa189]*/
{
    return PyUnicode_Join(self, iterable);
}
```

再看`PyUnicode_Join()`：

```c
[Objects/unicodeoject.c]

PyObject *
PyUnicode_Join(PyObject *separator, PyObject *seq)
{
    PyObject *res;
    PyObject *fseq;
    Py_ssize_t seqlen;
    PyObject **items;

    fseq = PySequence_Fast(seq, "can only join an iterable");
    if (fseq == NULL) {
        return NULL;
    }

    /* NOTE: the following code can't call back into Python code,
     * so we are sure that fseq won't be mutated.
     */

    items = PySequence_Fast_ITEMS(fseq);
    seqlen = PySequence_Fast_GET_SIZE(fseq);
    res = _PyUnicode_JoinArray(separator, items, seqlen);
    Py_DECREF(fseq);
    return res;
}
```

再我们看`_PyUnicode_JoinArray()`的实现：

```c
[Objects/unicodeoject.c]


PyObject *
_PyUnicode_JoinArray(PyObject *separator, PyObject *const *items, Py_ssize_t seqlen)
{
    PyObject *res = NULL; /* the result */
    PyObject *sep = NULL;
    Py_ssize_t seplen;
    PyObject *item;
    Py_ssize_t sz, i, res_offset;
    Py_UCS4 maxchar;
    Py_UCS4 item_maxchar;
    int use_memcpy;
    unsigned char *res_data = NULL, *sep_data = NULL;
    PyObject *last_obj;
    unsigned int kind = 0;

    /* If empty sequence, return u"". */
    if (seqlen == 0) {
        _Py_RETURN_UNICODE_EMPTY();
    }

    /* If singleton sequence with an exact Unicode, return that. */
    last_obj = NULL;
    if (seqlen == 1) {
        if (PyUnicode_CheckExact(items[0])) {
            res = items[0];
            Py_INCREF(res);
            return res;
        }
        seplen = 0;
        maxchar = 0;
    }
    else {
        /* Set up sep and seplen */
        if (separator == NULL) {
            /* fall back to a blank space separator */
            sep = PyUnicode_FromOrdinal(' ');
            if (!sep)
                goto onError;
            seplen = 1;
            maxchar = 32;
        }
        else {
            if (!PyUnicode_Check(separator)) {
                PyErr_Format(PyExc_TypeError,
                             "separator: expected str instance,"
                             " %.80s found",
                             Py_TYPE(separator)->tp_name);
                goto onError;
            }
            if (PyUnicode_READY(separator))
                goto onError;
            sep = separator;
            seplen = PyUnicode_GET_LENGTH(separator);
            maxchar = PyUnicode_MAX_CHAR_VALUE(separator);
            /* inc refcount to keep this code path symmetric with the
               above case of a blank separator */
            Py_INCREF(sep);
        }
        last_obj = sep;
    }

    /* There are at least two things to join, or else we have a subclass
     * of str in the sequence.
     * Do a pre-pass to figure out the total amount of space we'll
     * need (sz), and see whether all argument are strings.
     */
    sz = 0;
#ifdef Py_DEBUG
    use_memcpy = 0;
#else
    use_memcpy = 1;
#endif
    for (i = 0; i < seqlen; i++) {
        size_t add_sz;
        item = items[i];
        if (!PyUnicode_Check(item)) {
            PyErr_Format(PyExc_TypeError,
                         "sequence item %zd: expected str instance,"
                         " %.80s found",
                         i, Py_TYPE(item)->tp_name);
            goto onError;
        }
        if (PyUnicode_READY(item) == -1)
            goto onError;
        add_sz = PyUnicode_GET_LENGTH(item);
        item_maxchar = PyUnicode_MAX_CHAR_VALUE(item);
        maxchar = Py_MAX(maxchar, item_maxchar);
        if (i != 0) {
            add_sz += seplen;
        }
        if (add_sz > (size_t)(PY_SSIZE_T_MAX - sz)) {
            PyErr_SetString(PyExc_OverflowError,
                            "join() result is too long for a Python string");
            goto onError;
        }
        sz += add_sz;
        if (use_memcpy && last_obj != NULL) {
            if (PyUnicode_KIND(last_obj) != PyUnicode_KIND(item))
                use_memcpy = 0;
        }
        last_obj = item;
    }

    res = PyUnicode_New(sz, maxchar);
    if (res == NULL)
        goto onError;

    /* Catenate everything. */
#ifdef Py_DEBUG
    use_memcpy = 0;
#else
    if (use_memcpy) {
        res_data = PyUnicode_1BYTE_DATA(res);
        kind = PyUnicode_KIND(res);
        if (seplen != 0)
            sep_data = PyUnicode_1BYTE_DATA(sep);
    }
#endif
    if (use_memcpy) {
        for (i = 0; i < seqlen; ++i) {
            Py_ssize_t itemlen;
            item = items[i];

            /* Copy item, and maybe the separator. */
            if (i && seplen != 0) {
                memcpy(res_data,
                          sep_data,
                          kind * seplen);
                res_data += kind * seplen;
            }

            itemlen = PyUnicode_GET_LENGTH(item);
            if (itemlen != 0) {
                memcpy(res_data,
                          PyUnicode_DATA(item),
                          kind * itemlen);
                res_data += kind * itemlen;
            }
        }
        assert(res_data == PyUnicode_1BYTE_DATA(res)
                           + kind * PyUnicode_GET_LENGTH(res));
    }
    else {
        for (i = 0, res_offset = 0; i < seqlen; ++i) {
            Py_ssize_t itemlen;
            item = items[i];

            /* Copy item, and maybe the separator. */
            if (i && seplen != 0) {
                _PyUnicode_FastCopyCharacters(res, res_offset, sep, 0, seplen);
                res_offset += seplen;
            }

            itemlen = PyUnicode_GET_LENGTH(item);
            if (itemlen != 0) {
                _PyUnicode_FastCopyCharacters(res, res_offset, item, 0, itemlen);
                res_offset += itemlen;
            }
        }
        assert(res_offset == PyUnicode_GET_LENGTH(res));
    }

    Py_XDECREF(sep);
    assert(_PyUnicode_CheckConsistency(res, 1));
    return res;

  onError:
    Py_XDECREF(sep);
    Py_XDECREF(res);
    return NULL;
}
```

其在统计了list中的对象及长度后，统一进行申请空间并逐一进行拷贝。
