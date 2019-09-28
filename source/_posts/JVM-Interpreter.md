---
title: JVM-Interpreter
date: 2018-10-09 14:57:11
tags: JVM
---


### 牛逼的文章
https://blog.csdn.net/new_abc/article/details/53639050

### klassVtable
``` Conditional Breakpoint
this->_klass._value->_name->equals("OopAndKlass", 11)
```

```
klassVtable::initialize_vtable(bool, Thread*) klassVtable.cpp:163
InstanceKlass::link_class_impl(instanceKlassHandle, bool, Thread*) instanceKlass.cpp:631
InstanceKlass::link_class(Thread*) instanceKlass.cpp:506
get_class_declared_methods_helper(JNIEnv_*, _jclass*, unsigned char, bool, Klass*, Thread*) jvm.cpp:1818
::JVM_GetClassDeclaredMethods(JNIEnv *, jclass, jboolean) jvm.cpp:1871
<unknown> 0x000000010e4acbb0
<unknown> 0x000000010e48a220
<unknown> 0x000000010e48a220
<unknown> 0x000000010e48a220
<unknown> 0x000000010e48a220
<unknown> 0x000000010e48a220
<unknown> 0x000000010e48a4a3
<unknown> 0x000000010e47f9f1
JavaCalls::call_helper(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:410
os::os_exception_wrapper(void (*)(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*), JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) os_bsd.cpp:3682
JavaCalls::call(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:306
jni_invoke_static(JNIEnv_*, JavaValue*, _jobject*, JNICallType, _jmethodID*, JNI_ArgumentPusher*, Thread*) jni.cpp:1119
::jni_CallStaticObjectMethod(JNIEnv *, jclass, jmethodID, ...) jni.cpp:1846
LoadMainClass 0x000000010b351ab5
JavaMain 0x000000010b35080d
_pthread_body 0x00007fffc136e93b
_pthread_start 0x00007fffc136e887
thread_start 0x00007fffc136e08d
```


```
// Initialize the vtable and interface table after
// methods have been rewritten since rewrite may
// fabricate new Method*s.
// also does loader constraint checking
//
// initialize_vtable and initialize_itable need to be rerun for
// a shared class if the class is not loaded by the NULL classloader.
ClassLoaderData * loader_data = this_k->class_loader_data();
if (!(this_k->is_shared() &&
    loader_data->is_the_null_class_loader_data())) {
ResourceMark rm(THREAD);
this_k->vtable()->initialize_vtable(true, CHECK_false);
this_k->itable()->initialize_itable(true, CHECK_false);
}
```