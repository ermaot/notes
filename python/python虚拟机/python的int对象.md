## 一、对象池

python应用程序中，整数使用非常广泛，创建和销毁非常频繁，为了保证性能，使用了对象池机制，即整数对象池。几乎所有的内建对象，都会有自己特有的对象池机制。

## 二、int对象的定义

[intobject.h]

```
typedef struct {
    PyObject_HEAD
    long ob_ival;
} PyIntObject;
```

intobject.h内容非常简单，主要是这个结构体的定义

```
PyTypeObject PyInt_Type = {
	PyObject_HEAD_INIT(&PyType_Type)
	0,
	"int",
	sizeof(PyIntObject),
	0,
	(destructor)int_dealloc,		/* tp_dealloc */
	(printfunc)int_print,			/* tp_print */
	0,					/* tp_getattr */
	0,					/* tp_setattr */
	(cmpfunc)int_compare,			/* tp_compare */
	(reprfunc)int_repr,			/* tp_repr */
	&int_as_number,				/* tp_as_number */
	0,					/* tp_as_sequence */
	0,					/* tp_as_mapping */
	(hashfunc)int_hash,			/* tp_hash */
        0,					/* tp_call */
        (reprfunc)int_repr,			/* tp_str */
	PyObject_GenericGetAttr,		/* tp_getattro */
	0,					/* tp_setattro */
	0,					/* tp_as_buffer */
	Py_TPFLAGS_DEFAULT | Py_TPFLAGS_CHECKTYPES |
		Py_TPFLAGS_BASETYPE,		/* tp_flags */
	int_doc,				/* tp_doc */
	0,					/* tp_traverse */
	0,					/* tp_clear */
	0,					/* tp_richcompare */
	0,					/* tp_weaklistoffset */
	0,					/* tp_iter */
	0,					/* tp_iternext */
	int_methods,				/* tp_methods */
	0,					/* tp_members */
	0,					/* tp_getset */
	0,					/* tp_base */
	0,					/* tp_dict */
	0,					/* tp_descr_get */
	0,					/* tp_descr_set */
	0,					/* tp_dictoffset */
	0,					/* tp_init */
	0,					/* tp_alloc */
	int_new,				/* tp_new */
	(freefunc)int_free,           		/* tp_free */
};
```

其中int_methods是int的方法，定义如下：

```
static PyMethodDef int_methods[] = {
	{"__getnewargs__",	(PyCFunction)int_getnewargs,	METH_NOARGS},
	{NULL,		NULL}		/* sentinel */
};
```

int_free 是释放方法

```
static void
int_free(PyIntObject *v)
{
	v->ob_type = (struct _typeobject *)free_list;
	free_list = v;
}
```

int_dealloc是析构方法

```
static void
int_dealloc(PyIntObject *v)
{
	if (PyInt_CheckExact(v)) {
		v->ob_type = (struct _typeobject *)free_list;
		free_list = v;
	}
	else
		v->ob_type->tp_free((PyObject *)v);
}
```

int_compare是比较大小的方法

```
static int
int_compare(PyIntObject *v, PyIntObject *w)
{
	register long i = v->ob_ival;
	register long j = w->ob_ival;
	return (i < j) ? -1 : (i > j) ? 1 : 0;
}
```

## 三、对象的创建

int对象的创建方法有3种(来自[intobject.h])

- PyInt_FromLong()
- PyInt_FromString()
- PyInt_FromUnicode()

```
PyObject *
PyInt_FromLong(long ival)
{
	register PyIntObject *v;
#if NSMALLNEGINTS + NSMALLPOSINTS > 0
	if (-NSMALLNEGINTS <= ival && ival < NSMALLPOSINTS) {
		v = small_ints[ival + NSMALLNEGINTS];
		Py_INCREF(v);
#ifdef COUNT_ALLOCS
		if (ival >= 0)
			quick_int_allocs++;
		else
			quick_neg_int_allocs++;
#endif
		return (PyObject *) v;
	}
#endif
	if (free_list == NULL) {
		if ((free_list = fill_free_list()) == NULL)
			return NULL;
	}
	/* Inline PyObject_New */
	v = free_list;
	free_list = (PyIntObject *)v->ob_type;
	PyObject_INIT(v, &PyInt_Type);
	v->ob_ival = ival;
	return (PyObject *) v;
}
```

## 四、小整数对象

```
static PyIntObject *small_ints[NSMALLNEGINTS + NSMALLPOSINTS];
```

其中NSMALLNEGINTS = 5，NSMALLPOSINTS = 257

将小整数的指针存放在内存里

```
In [1]: a1 = 10

In [2]: a2 = 10

In [3]: a3 = 256

In [4]: a4 = 256

In [5]: a5 = 257

In [6]: a6 = 257

In [7]: id(a1)
Out[7]: 140509061477888

In [8]: id(a2)
Out[8]: 140509061477888

In [9]: id(a3)
Out[9]: 140509061485760

In [10]: id(a4)
Out[10]: 140509061485760

In [11]: id(a5)
Out[11]: 140508929670064

In [12]: id(a6)
Out[12]: 140508929670288

In [13]: a7 = 257

In [14]: id(a7)
Out[14]: 140508929670224

In [15]: a8 = 256

In [16]: id(a8)
Out[16]: 140509061485760

```

可以看到相同小整数的内存地址是相同的（a1和a2，a3和a4和a8）。一旦超出了256，就需要重新分配内存了，每一次的地址都不同。