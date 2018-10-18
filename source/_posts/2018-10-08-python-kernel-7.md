---
title: 'python 内核分析（七）：Dict类型对象'
date: 2018-10-08 20:56:34
tags: PYTHON
categories: PYTHON # 分类
thumbnail: /assets/img/posts/PYTHON-KERNEL/default.jpg
---

>本系列作为[本人@takooctopus](https://takooctopus.github.io/ "TAKONOHEYA")深入学习python机制的记录，这个博客遵照着[栖迟于一丘](https://www.hongweipeng.com/ "栖迟于一丘")的博客上面的流程进行的，也包含我在实际查看源码时的感想，特此列出，表示感谢。

# 关于Dict对象

python中的`dict`对象也即`PyDictObject`对象，因为对搜索的效率要求很高，所以选择了散列表（hash table），因为在最优情况下，散列表能够提供O(1)的搜索效率。

散列表的基本思想是：通过一定的函数将需要搜索的键值映射为一个整数，根据这个整数作为索引去访问某片连续的内存区域。用于映射的函数称为映射函数，映射所产生的值称为散列值（`hash value`）。散列函数对搜索效率有直接的决定性作用。在使用散列函数将不同的值可能映射到相同的散列值，这个时候就需要冲突解决（装载率大于2/3时，冲突的概率就会大大增加）

冲突解决在python中使用的是开放定址法「闭散列法」，就是通过一个二次探测函数f，计算下一个位置，一直到找到下一个可用的位置为止，在这个过程中会到达多个位置，这些位置就形成了一个“冲突探测链”，这个冲突探测链在查找某个元素的时候起到重要作用，所以在删除某个位置上的元素，不能直接将这个位置的内容删除，如果删除的话，则导致后续依赖于这个位置的其他值就都无法寻找到了，所以只能进行“伪删除”（通过给元素设置状态，dummy态，表示没有存储具体的值但是还会用到的废弃态）

先列出`PyDictObject`的定义：

```c
[Include/dictobject.h]

/* The ma_values pointer is NULL for a combined table
 * or points to an array of PyObject* for a split table
 */
typedef struct {
    PyObject_HEAD

    /* Number of items in the dictionary */
    Py_ssize_t ma_used;

    /* Dictionary version: globally unique, value change each time
       the dictionary is modified */
    uint64_t ma_version_tag;

    PyDictKeysObject *ma_keys;

    /* If ma_values is NULL, the table is "combined": keys and values
       are stored in ma_keys.

       If ma_values is not NULL, the table is splitted:
       keys are stored in ma_keys and values are stored in ma_values */
    PyObject **ma_values;
} PyDictObject;
```

我们看到其中属性中除了对象类型、`Active`元素个数，全局唯一id`ma_version_tag`，还有一个类`PyDictKeys`，以及一个`PyObject`的指针。我们去看一看它的定义：

```c
[Objects/dict-common.h]

/* See dictobject.c for actual layout of DictKeysObject */
struct _dictkeysobject {
    Py_ssize_t dk_refcnt;

    /* Size of the hash table (dk_indices). It must be a power of 2. */
    Py_ssize_t dk_size;

    /* Function to lookup in the hash table (dk_indices):

       - lookdict(): general-purpose, and may return DKIX_ERROR if (and
         only if) a comparison raises an exception.

       - lookdict_unicode(): specialized to Unicode string keys, comparison of
         which can never raise an exception; that function can never return
         DKIX_ERROR.

       - lookdict_unicode_nodummy(): similar to lookdict_unicode() but further
         specialized for Unicode string keys that cannot be the <dummy> value.

       - lookdict_split(): Version of lookdict() for split tables. */
    dict_lookup_func dk_lookup;

    /* Number of usable entries in dk_entries. */
    Py_ssize_t dk_usable;

    /* Number of used entries in dk_entries. */
    Py_ssize_t dk_nentries;

    /* Actual hash table of dk_size entries. It holds indices in dk_entries,
       or DKIX_EMPTY(-1) or DKIX_DUMMY(-2).

       Indices must be: 0 <= indice < USABLE_FRACTION(dk_size).

       The size in bytes of an indice depends on dk_size:

       - 1 byte if dk_size <= 0xff (char*)
       - 2 bytes if dk_size <= 0xffff (int16_t*)
       - 4 bytes if dk_size <= 0xffffffff (int32_t*)
       - 8 bytes otherwise (int64_t*)

       Dynamically sized, SIZEOF_VOID_P is minimum. */
    char dk_indices[];  /* char is required to avoid strict aliasing. */

    /* "PyDictKeyEntry dk_entries[dk_usable];" array follows:
       see the DK_ENTRIES() macro */
};

```

我们看见`_dictkeysobject`的属性中包含了引用数`dk_refcnt`，键数`dk_size = 2^n`,以及哈希表寻找的函数`dk_lookup`「包含了4种函数」，`Active + Dummy`的元素个数`dk_usable`，`Active`的元素个数`dk_nentries`，以及哈希表的索引`dk_indices[]`「其中每个单元的大小与`dk_size`的大小有关」

```c
[Objects/dict-common.h]

#define DKIX_EMPTY (-1)
#define DKIX_DUMMY (-2)  /* Used internally */
#define DKIX_ERROR (-3)
```

我们可以看见在这里设立了三个宏，表示了`entry`键值对的对象状态。

那么什么是`entry`键值对呢，其被称为关联容器，即`PyDictKeyEntry`

```c
[Objects/dict-common.h]

typedef struct {
    /* Cached hash code of me_key. */
    Py_hash_t me_hash;
    PyObject *me_key;
    PyObject *me_value; /* This field is only meaningful for combined tables */
} PyDictKeyEntry;
```

在`dict_lookup_func()`中会返回键值对的`index`「主键」，我们可以像`DK_ENTRIES(dk)[index]`的方式使用主键，其三个值`-1`代表没有找到这个关联容器，`-3`代表当其比较时返回错误,`-2`的`dummy`代表了一种伪删除状态。

当然上面那个是在寻找键值对关联容器时的返回值，但关联容器本身也是有三种状态的。

- Unused(未使用):`index == DKIX_EMPTY`
- Active(活动): `index >= 0`, `me_key != NULL` and `me_value != NULL`
- Dummy(删除状态): `index == DKIX_DUMMY` 之所以不能直接转成Unused状态是因为这要会导致冲突探测链中断

# 新的PyPyDict

在其文档中提到现在其使用了[新的PyPy字典结构](https://morepypy.blogspot.com/2015/01/faster-more-memory-efficient-and-more.html "PyPy Status Blog")

在此简单的提一下其两者的差别：

原来的字典结构：

```c
struct dict {
   long num_items;
   dict_entry* items;   /* pointer to array */
}

struct dict_entry {
   long hash;
   PyObject* key;
   PyObject* value;
}
```
其`dict`由一大块内存区组成，并伴有指向数组的指针。这些关联容器数组实际上是一个稀疏数组「其中大约1/3到1/2的空间是`NULL`」，形成大量的空间浪费。

相比之下新的字典结构：

```c
struct dict {
    long num_items;
    variable_int *sparse_array;
    dict_entry* compact_array;
}

struct dict_entry {
    long hash;
    PyObject *key;
    PyObject *value;
}
```

其`dict`的实现使用了两个数组，一个用于存储索引的`sparse_array[]`，其中只存储有符号的整数「`+`」用作hash的索引以及`KIX_EMPTY「-1」`或`DKIX_DUMMY「-2」`。另一个`compact_array []`按照插入顺序储存所有的关联容器。

因为在`sparse_array[]`中存储的其实是在`compact_array[]`中`item`所在的插入序号，当改变`sparse_array[]`中的排列时，只是索引变化，不需要改动我们实际上存储的关联容器的储存顺序。

这个设计不仅减少了操作量，更重要的是其节省内存的功能。因为`sparse_array[]`中存储的都是`int`，其内存占用量极小。举例来说，在64位系统下，我们最多，虽然几乎用不到，能储存大约40亿个元素。对于一个100个元素的字典，原来的字典设计会占用大约`100 * 12/7 * WORD * 3 =~ 4100 bytes`，而新的字典设计仅需要`100 * 12/7 + 3 * WORD * 100 =~ 2600 bytes`，内存使用大大减少。

# PyDictObject的创建操作

简单的，其通过`PyDict_New()`来新建一个`dict`对象

```c
[Objects/dictobject.c]

PyObject *
PyDict_New(void)
{
    PyDictKeysObject *keys = new_keys_object(PyDict_MINSIZE);
    if (keys == NULL)
        return NULL;
    return new_dict(keys, NULL);
}

```

其默认`#define PyDict_MINSIZE 8`，即会附带创建8个关联容器`PyDictKeyEntry`

```c
[Objects/dictobject.c]

static PyDictKeysObject *new_keys_object(Py_ssize_t size)
{
    PyDictKeysObject *dk;
    Py_ssize_t es, usable;

    assert(size >= PyDict_MINSIZE);
    assert(IS_POWER_OF_2(size));

    usable = USABLE_FRACTION(size);
    if (size <= 0xff) {
        es = 1;
    }
    else if (size <= 0xffff) {
        es = 2;
    }
#if SIZEOF_VOID_P > 4
    else if (size <= 0xffffffff) {
        es = 4;
    }
#endif
    else {
        es = sizeof(Py_ssize_t);
    }

    if (size == PyDict_MINSIZE && numfreekeys > 0) {
        dk = keys_free_list[--numfreekeys];
    }
    else {
        dk = PyObject_MALLOC(sizeof(PyDictKeysObject)
                             + es * size
                             + sizeof(PyDictKeyEntry) * usable);
        if (dk == NULL) {
            PyErr_NoMemory();
            return NULL;
        }
    }
    DK_DEBUG_INCREF dk->dk_refcnt = 1;
    dk->dk_size = size;
    dk->dk_usable = usable;
    dk->dk_lookup = lookdict_unicode_nodummy;
    dk->dk_nentries = 0;
    memset(&dk->dk_indices[0], 0xff, es * size);
    memset(DK_ENTRIES(dk), 0, sizeof(PyDictKeyEntry) * usable);
    return dk;
}
```

其分配的大小随`size`的大小分成了几个区间。

我们看其最后分配的内存空间，当没有使用缓冲池时，其分配的内存为

```c
[Objects/dictobject.c/new_keys_object()]

dk = PyObject_MALLOC(sizeof(PyDictKeysObject)
                             + es * size
                             + sizeof(PyDictKeyEntry) * usable);

```

包含了三个部分，第一个`sizeof(PyDictKeysObject)`是这个空对象的内存大小，第二个是hash索引表的大小，第三个`sizeof(PyDictKeyEntry) * usable`是申请的关联容器数组的大小，这里`usable`一般取在`1/2`到`2/3`就比较合适。

```c
[Objects/dictobject.c]
/* USABLE_FRACTION is the maximum dictionary load.
 * Increasing this ratio makes dictionaries more dense resulting in more
 * collisions.  Decreasing it improves sparseness at the expense of spreading
 * indices over more cache lines and at the cost of total memory consumed.
 *
 * USABLE_FRACTION must obey the following:
 *     (0 < USABLE_FRACTION(n) < n) for all n >= 2
 *
 * USABLE_FRACTION should be quick to calculate.
 * Fractions around 1/2 to 2/3 seem to work well in practice.
 */
#define USABLE_FRACTION(n) (((n) << 1)/3)


[Objects/dictobject.c/new_keys_object()]

usable = USABLE_FRACTION(size);
```

在创建完主键对象后，再继续创建关联容器对象组，即`dict`的对象。


# PyDictObject的获取操作

获取Dict对象时我们调用的是`PyDict_GetItem()`

```c
[Objects/dictobject.c]

/* Note that, for historical reasons, PyDict_GetItem() suppresses all errors
 * that may occur (originally dicts supported only string keys, and exceptions
 * weren't possible).  So, while the original intent was that a NULL return
 * meant the key wasn't present, in reality it can mean that, or that an error
 * (suppressed) occurred while computing the key's hash, or that some error
 * (suppressed) occurred when comparing keys in the dict's internal probe
 * sequence.  A nasty example of the latter is when a Python-coded comparison
 * function hits a stack-depth error, which can cause this to return NULL
 * even if the key is present.
 */
PyObject *
PyDict_GetItem(PyObject *op, PyObject *key)
{
    Py_hash_t hash;
    Py_ssize_t ix;
    PyDictObject *mp = (PyDictObject *)op;
    PyThreadState *tstate;
    PyObject *value;

    if (!PyDict_Check(op))
        return NULL;
    if (!PyUnicode_CheckExact(key) ||
        (hash = ((PyASCIIObject *) key)->hash) == -1)
    {
        hash = PyObject_Hash(key);
        if (hash == -1) {
            PyErr_Clear();
            return NULL;
        }
    }

    /* We can arrive here with a NULL tstate during initialization: try
       running "python -Wi" for an example related to string interning.
       Let's just hope that no exception occurs then...  This must be
       _PyThreadState_Current and not PyThreadState_GET() because in debug
       mode, the latter complains if tstate is NULL. */
    tstate = PyThreadState_GET();
    if (tstate != NULL && tstate->curexc_type != NULL) {
        /* preserve the existing exception */
        PyObject *err_type, *err_value, *err_tb;
        PyErr_Fetch(&err_type, &err_value, &err_tb);
        ix = (mp->ma_keys->dk_lookup)(mp, key, hash, &value);
        /* ignore errors */
        PyErr_Restore(err_type, err_value, err_tb);
        if (ix < 0)
            return NULL;
    }
    else {
        ix = (mp->ma_keys->dk_lookup)(mp, key, hash, &value);
        if (ix < 0) {
            PyErr_Clear();
            return NULL;
        }
    }
    return value;
}
```

我们在进行了类型检查和字符检查后，在确定其状态不为`empty「-1」`后，我们调用其属性中的`dk_lookup`，这个是查找方法，在类的初始化时被定义，有四种可能性：

- lookdict(): general-purpose, and may return DKIX_ERROR if (and only if) a comparison raises an exception.

- lookdict_unicode(): specialized to Unicode string keys, comparison of which can never raise an exception; that function can never return DKIX_ERROR.

- lookdict_unicode_nodummy(): similar to lookdict_unicode() but further specialized for Unicode string keys that cannot be the <dummy> value.

- lookdict_split(): Version of lookdict() for split tables.


一般来说，`lookdict()`是通用的方法，我们去看一下其具体内容：

```c
[Objects/dictobject.c]

/*
The basic lookup function used by all operations.
This is based on Algorithm D from Knuth Vol. 3, Sec. 6.4.
Open addressing is preferred over chaining since the link overhead for
chaining would be substantial (100% with typical malloc overhead).

The initial probe index is computed as hash mod the table size. Subsequent
probe indices are computed as explained earlier.

All arithmetic on hash should ignore overflow.

The details in this version are due to Tim Peters, building on many past
contributions by Reimer Behrends, Jyrki Alakuijala, Vladimir Marangozov and
Christian Tismer.
*/

static Py_ssize_t _Py_HOT_FUNCTION
lookdict(PyDictObject *mp, PyObject *key,
         Py_hash_t hash, PyObject **value_addr)
{
    size_t i, mask, perturb;
    PyDictKeysObject *dk;
    PyDictKeyEntry *ep0;

top:
    dk = mp->ma_keys;
    ep0 = DK_ENTRIES(dk);
    mask = DK_MASK(dk);
    perturb = hash;
    i = (size_t)hash & mask;

    for (;;) {
        Py_ssize_t ix = dk_get_index(dk, i);
        if (ix == DKIX_EMPTY) {
            *value_addr = NULL;
            return ix;
        }
        if (ix >= 0) {
            PyDictKeyEntry *ep = &ep0[ix];
            assert(ep->me_key != NULL);
            if (ep->me_key == key) {
                *value_addr = ep->me_value;
                return ix;
            }
            if (ep->me_hash == hash) {
                PyObject *startkey = ep->me_key;
                Py_INCREF(startkey);
                int cmp = PyObject_RichCompareBool(startkey, key, Py_EQ);
                Py_DECREF(startkey);
                if (cmp < 0) {
                    *value_addr = NULL;
                    return DKIX_ERROR;
                }
                if (dk == mp->ma_keys && ep->me_key == startkey) {
                    if (cmp > 0) {
                        *value_addr = ep->me_value;
                        return ix;
                    }
                }
                else {
                    /* The dict was mutated, restart */
                    goto top;
                }
            }
        }
        perturb >>= PERTURB_SHIFT;
        i = (i*5 + perturb + 1) & mask;
    }
    Py_UNREACHABLE();
}
```

首先`mask`值是`2^(dk->dk_size)-1`，`i`相当于对其取模。

在循环中，在取出第dict表中的稀疏数组`parse_array[]`即`dk_indices[]`，直接取出对应位置i的关联容器储存位置「即我们在前面新PyPyDict中提到的`compact_array[]`」的id:`ix`。

再依次判断其是否非空，是否来自同一个引用。

经过了上两步后，还需考虑因为hash导致的重复冲突，我们对两个对象进行相等性的比较「其中涉及了简单比较和完整比较(完整比较中返回`1`为相等，`0`为不等，还有一个`-1`视为出错)」，根据比较的结果，如果相等就返回；如果不相等，我们对hash值做位平移，并修改我们的hash探测函数值`i`，然后重新循环，继续下一次的探测。

>注意：我们这里简单比较中采用了值比较`==`而不是引用相同`is`，因为要考虑非小整数对象的影响。

其中关联容器`entry`两种状态`Unuse`以及`Dummy`态，函数会分别以冲突弹窗结束搜索，或者设置为`freeslot`继续探测。

关于冲突函数`i`，当遇到hash冲突时，`i = ((i << 2) + i + perturb + 1) & mask`，很神奇，其化简为`i = (i * 5 + i) % dk_size` 刚好能遍历所有元素。


#PyDictObject的插入操作

插入操作在`PyDict_SetItem()`方法中：

```c
[Objects/dictobject.c]

int
PyDict_SetItem(PyObject *op, PyObject *key, PyObject *value)
{
    PyDictObject *mp;
    Py_hash_t hash;
    if (!PyDict_Check(op)) {
        PyErr_BadInternalCall();
        return -1;
    }
    assert(key);
    assert(value);
    mp = (PyDictObject *)op;
    if (!PyUnicode_CheckExact(key) ||
        (hash = ((PyASCIIObject *) key)->hash) == -1)
    {
        hash = PyObject_Hash(key);
        if (hash == -1)
            return -1;
    }

    /* insertdict() handles any resizing that might be necessary */
    return insertdict(mp, key, hash, value);
}
```

最后调用了`insertdict()`方法：


```c
[Objects/dictobject.c]


/*
Internal routine to insert a new item into the table.
Used both by the internal resize routine and by the public insert routine.
Returns -1 if an error occurred, or 0 on success.
*/
static int
insertdict(PyDictObject *mp, PyObject *key, Py_hash_t hash, PyObject *value)
{
    PyObject *old_value;
    PyDictKeyEntry *ep;

    Py_INCREF(key);
    Py_INCREF(value);
    if (mp->ma_values != NULL && !PyUnicode_CheckExact(key)) {
        if (insertion_resize(mp) < 0)
            goto Fail;
    }

    Py_ssize_t ix = mp->ma_keys->dk_lookup(mp, key, hash, &old_value);
    if (ix == DKIX_ERROR)
        goto Fail;

    assert(PyUnicode_CheckExact(key) || mp->ma_keys->dk_lookup == lookdict);
    MAINTAIN_TRACKING(mp, key, value);

    /* When insertion order is different from shared key, we can't share
     * the key anymore.  Convert this instance to combine table.
     */
    if (_PyDict_HasSplitTable(mp) &&
        ((ix >= 0 && old_value == NULL && mp->ma_used != ix) ||
         (ix == DKIX_EMPTY && mp->ma_used != mp->ma_keys->dk_nentries))) {
        if (insertion_resize(mp) < 0)
            goto Fail;
        ix = DKIX_EMPTY;
    }

    if (ix == DKIX_EMPTY) {
        /* Insert into new slot. */
        assert(old_value == NULL);
        if (mp->ma_keys->dk_usable <= 0) {
            /* Need to resize. */
            if (insertion_resize(mp) < 0)
                goto Fail;
        }
        Py_ssize_t hashpos = find_empty_slot(mp->ma_keys, hash);
        ep = &DK_ENTRIES(mp->ma_keys)[mp->ma_keys->dk_nentries];
        dk_set_index(mp->ma_keys, hashpos, mp->ma_keys->dk_nentries);
        ep->me_key = key;
        ep->me_hash = hash;
        if (mp->ma_values) {
            assert (mp->ma_values[mp->ma_keys->dk_nentries] == NULL);
            mp->ma_values[mp->ma_keys->dk_nentries] = value;
        }
        else {
            ep->me_value = value;
        }
        mp->ma_used++;
        mp->ma_version_tag = DICT_NEXT_VERSION();
        mp->ma_keys->dk_usable--;
        mp->ma_keys->dk_nentries++;
        assert(mp->ma_keys->dk_usable >= 0);
        assert(_PyDict_CheckConsistency(mp));
        return 0;
    }

    if (_PyDict_HasSplitTable(mp)) {
        mp->ma_values[ix] = value;
        if (old_value == NULL) {
            /* pending state */
            assert(ix == mp->ma_used);
            mp->ma_used++;
        }
    }
    else {
        assert(old_value != NULL);
        DK_ENTRIES(mp->ma_keys)[ix].me_value = value;
    }

    mp->ma_version_tag = DICT_NEXT_VERSION();
    Py_XDECREF(old_value); /* which **CAN** re-enter (see issue #22653) */
    assert(_PyDict_CheckConsistency(mp));
    Py_DECREF(key);
    return 0;

Fail:
    Py_DECREF(value);
    Py_DECREF(key);
    return -1;
}
```

其最重要的还是基于搜索出来返回的`ix`值，如果其返回的是拥有`Active`的关联容器，可以直接对`value_addr`赋值`value`。而如果是`Unuse`或者`Dummy`,就需要完整的设置`dict_entry`结构体了。

当空间达到一个阈值时，会调用`insertion_resize()`方法，即`dictresize()`方法。将空间扩大2倍并迁移数据。

```c
[Objects/dictobject.c]

/*
Restructure the table by allocating a new table and reinserting all
items again.  When entries have been deleted, the new table may
actually be smaller than the old one.
If a table is split (its keys and hashes are shared, its values are not),
then the values are temporarily copied into the table, it is resized as
a combined table, then the me_value slots in the old table are NULLed out.
After resizing a table is always combined,
but can be resplit by make_keys_shared().
*/
static int
dictresize(PyDictObject *mp, Py_ssize_t minsize)
{
    Py_ssize_t newsize, numentries;
    PyDictKeysObject *oldkeys;
    PyObject **oldvalues;
    PyDictKeyEntry *oldentries, *newentries;

    /* Find the smallest table size > minused. */
    for (newsize = PyDict_MINSIZE;
         newsize < minsize && newsize > 0;
         newsize <<= 1)
        ;
    if (newsize <= 0) {
        PyErr_NoMemory();
        return -1;
    }

    oldkeys = mp->ma_keys;

    /* NOTE: Current odict checks mp->ma_keys to detect resize happen.
     * So we can't reuse oldkeys even if oldkeys->dk_size == newsize.
     * TODO: Try reusing oldkeys when reimplement odict.
     */

    /* Allocate a new table. */
    mp->ma_keys = new_keys_object(newsize);
    if (mp->ma_keys == NULL) {
        mp->ma_keys = oldkeys;
        return -1;
    }
    // New table must be large enough.
    assert(mp->ma_keys->dk_usable >= mp->ma_used);
    if (oldkeys->dk_lookup == lookdict)
        mp->ma_keys->dk_lookup = lookdict;

    numentries = mp->ma_used;
    oldentries = DK_ENTRIES(oldkeys);
    newentries = DK_ENTRIES(mp->ma_keys);
    oldvalues = mp->ma_values;
    if (oldvalues != NULL) {
        /* Convert split table into new combined table.
         * We must incref keys; we can transfer values.
         * Note that values of split table is always dense.
         */
        for (Py_ssize_t i = 0; i < numentries; i++) {
            assert(oldvalues[i] != NULL);
            PyDictKeyEntry *ep = &oldentries[i];
            PyObject *key = ep->me_key;
            Py_INCREF(key);
            newentries[i].me_key = key;
            newentries[i].me_hash = ep->me_hash;
            newentries[i].me_value = oldvalues[i];
        }

        DK_DECREF(oldkeys);
        mp->ma_values = NULL;
        if (oldvalues != empty_values) {
            free_values(oldvalues);
        }
    }
    else {  // combined table.
        if (oldkeys->dk_nentries == numentries) {
            memcpy(newentries, oldentries, numentries * sizeof(PyDictKeyEntry));
        }
        else {
            PyDictKeyEntry *ep = oldentries;
            for (Py_ssize_t i = 0; i < numentries; i++) {
                while (ep->me_value == NULL)
                    ep++;
                newentries[i] = *ep++;
            }
        }

        assert(oldkeys->dk_lookup != lookdict_split);
        assert(oldkeys->dk_refcnt == 1);
        if (oldkeys->dk_size == PyDict_MINSIZE &&
            numfreekeys < PyDict_MAXFREELIST) {
            DK_DEBUG_DECREF keys_free_list[numfreekeys++] = oldkeys;
        }
        else {
            DK_DEBUG_DECREF PyObject_FREE(oldkeys);
        }
    }

    build_indices(mp->ma_keys, newentries, numentries);
    mp->ma_keys->dk_usable -= numentries;
    mp->ma_keys->dk_nentries = numentries;
    return 0;
}
```
