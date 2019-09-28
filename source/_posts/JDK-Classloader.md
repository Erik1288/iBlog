---
title: JVM-Classloader
date: 2018-08-28 17:47:54
tags: JVM
---


### 怎么样设计一个程序，会有class冲突问题，然后用Classloader来解决问题？


### 
断点:class_name._temp->index_of_at(0, "Student", 7) > -1


jvm_define_class_common(JNIEnv_*, char const*, _jobject*, signed char const*, int, _jobject*, char const*, Thread*) jvm.cpp:927
::JVM_DefineClassWithSource(JNIEnv *, const char *, jobject, const jbyte *, jsize, jobject, const char *) jvm.cpp:960
Java_java_lang_ClassLoader_defineClass1 ClassLoader.c:136
<unknown> 0x0000000115fecbb0
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fca220
<unknown> 0x0000000115fbf9f1
JavaCalls::call_helper(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:410
os::os_exception_wrapper(void (*)(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*), JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) os_bsd.cpp:3682
JavaCalls::call(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:306
JavaCalls::call_virtual(JavaValue*, KlassHandle, Symbol*, Symbol*, JavaCallArguments*, Thread*) javaCalls.cpp:193
JavaCalls::call_virtual(JavaValue*, Handle, KlassHandle, Symbol*, Symbol*, Handle, Thread*) javaCalls.cpp:206
SystemDictionary::load_instance_class(Symbol*, Handle, Thread*) systemDictionary.cpp:1586
SystemDictionary::resolve_instance_class_or_null(Symbol*, Handle, Handle, Thread*) systemDictionary.cpp:840
SystemDictionary::resolve_or_null(Symbol*, Handle, Handle, Thread*) systemDictionary.cpp:248
SystemDictionary::resolve_or_fail(Symbol*, Handle, Handle, bool, Thread*) systemDictionary.cpp:185
ConstantPool::klass_at_impl(constantPoolHandle const&, int, bool, Thread*) constantPool.cpp:256
ConstantPool::klass_at(int, Thread*) constantPool.hpp:342
InterpreterRuntime::_new(JavaThread*, ConstantPool*, int) interpreterRuntime.cpp:141
<unknown> 0x000000011600d6a9
<unknown> 0x0000000115fbf9f1
JavaCalls::call_helper(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:410
os::os_exception_wrapper(void (*)(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*), JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) os_bsd.cpp:3682
JavaCalls::call(JavaValue*, methodHandle const&, JavaCallArguments*, Thread*) javaCalls.cpp:306
jni_invoke_static(JNIEnv_*, JavaValue*, _jobject*, JNICallType, _jmethodID*, JNI_ArgumentPusher*, Thread*) jni.cpp:1119
::jni_CallStaticVoidMethod(JNIEnv *, jclass, jmethodID, ...) jni.cpp:1989
JavaMain 0x000000010fa4dc90
_pthread_body 0x00007fffc9ad493b
_pthread_start 0x00007fffc9ad4887
thread_start 0x00007fffc9ad408d