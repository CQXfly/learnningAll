# 从runtime源码探究iOS load 与initialize方法
![看看load执行了啥](http://upload-images.jianshu.io/upload_images/128529-a262c8016de686ae.jpg?imageMogr2/auto-orient/strip%7CimageView2/2)
> 打开 runtime 的源码，我们看到 load_images 的具体实现如下
```
/***********************************************************************
* load_images
* Process +load in the given images which are being mapped in by dyld.
* Calls ABI-agnostic code after taking ABI-specific locks.
*
* Locking: write-locks runtimeLock and loadMethodLock
**********************************************************************/
__private_extern__ const char *
load_images(enum dyld_image_states state, uint32_t infoCount,
            const struct dyld_image_info infoList[])
{
    BOOL found;
		 recursive_mutex_lock(&loadMethodLock);
     // Discover load methods
    rwlock_write(&runtimeLock);
    found = load_images_nolock(state, infoCount, infoList);
    rwlock_unlock_write(&runtimeLock);
 // Call +load methods (without runtimeLock - re-entrant)
    if (found) {
        call_load_methods();
    }
    recursive_mutex_unlock(&loadMethodLock);
    return NULL;
    }
```

* 这里我们发现两个重要的方法 load_images_nolock 和 call_load_methods，接下来我们再一起查看这两个方法的具体实现。

```
/***********************************************************************
* load_images_nolock
* Prepares +load in the given images which are being mapped in by dyld.
* Returns YES if there are now +load methods to be called by call_load_methods.
*
* Locking: loadMethodLock(both) and runtimeLock(new) acquired by load_images
**********************************************************************/
__private_extern__ BOOL 
load_images_nolock(enum dyld_image_states state,uint32_t infoCount,
                   const struct dyld_image_info infoList[])
{
    BOOL found = NO;
    uint32_t i;

    i = infoCount;
    while (i--) {
        header_info *hi;
        for (hi = FirstHeader; hi != NULL; hi = hi->next) {
            const headerType *mhdr = (headerType*)infoList[i].imageLoadAddress;
            if (hi->mhdr == mhdr) {
                prepare_load_methods(hi);
                found = YES;
            }
        }
    }

    return found;
}
```
>prepare_load_methods

```
__private_extern__ void prepare_load_methods(header_info *hi)
{
    size_t count, i;

    rwlock_assert_writing(&runtimeLock);

    class_t **classlist = 
        _getObjc2NonlazyClassList(hi, &count);
    for (i = 0; i < count; i++) {
        class_t *cls = remapClass(classlist[i]);
        schedule_class_load(cls);
    }

    category_t **categorylist = _getObjc2NonlazyCategoryList(hi, &count);
    for (i = 0; i < count; i++) {
        category_t *cat = categorylist[i];
        // Do NOT use cat->cls! It may have been remapped.
        class_t *cls = remapClass(cat->cls);
        realizeClass(cls);
        assert(isRealized(cls->isa));
        add_category_to_loadable_list((Category)cat);
    }
}
```

>这里我们发现，class 和 category 被分开处理，先通过 schedule_class_load 将需要执行 load 的 class 添加到一个全局列表里，之后再通过 add_category_to_loadable_list 将需要执行 load 的 category 添加到另一个全局列表里。这两个列表的定义如下

```
// List of classes that need +load called (pending superclass +load)
// This list always has superclasses first because of the way it is constructed
static struct loadable_class *loadable_classes NOBSS = NULL;

// List of categories that need +load called (pending parent class +load)
static struct loadable_category *loadable_categories NOBSS = NULL;
```
* schedule_class_load

```
/***********************************************************************
* prepare_load_methods
* Schedule +load for classes in this image, any un-+load-ed 
* superclasses in other images, and any categories in this image.
**********************************************************************/
// Recursively schedule +load for cls and any un-+load-ed superclasses.
// cls must already be connected.
static void schedule_class_load(class_t *cls)
{
    assert(isRealized(cls));  // _read_images should realize

    if (cls->data->flags & RW_LOADED) return;

    class_t *supercls = getSuperclass(cls);
    if (supercls) schedule_class_load(supercls);

    add_class_to_loadable_list((Class)cls);
    changeInfo(cls, RW_LOADED, 0); 
}
```

*这里我们可以看出，class 的处理是递归处理父类，确保父类先被添加到 loadable_classes 中。至此，两个列表里已经存好了需要执行 load 方法的类和 category。下面再回到 call_load_methods*
*call_load_methods*
```
__private_extern__ void call_load_methods(void)
{
    static BOOL loading = NO;
    BOOL more_categories;

    recursive_mutex_assert_locked(&loadMethodLock);

    // Re-entrant calls do nothing; the outermost call will finish the job.
    if (loading) return;
    loading = YES;

    do {
        // 1. Repeatedly call class +loads until there aren't any more
        while (loadable_classes_used > 0) {
            call_class_loads();
        }

        // 2. Call category +loads ONCE
        more_categories = call_category_loads();

        // 3. Run more +loads if there are classes OR more untried categories
    } while (loadable_classes_used > 0  ||  more_categories);

    loading = NO;
}
```

`
首先从 loadable_classes 中遍历取出类执行 call_class_loads 方法，该方法的具体实现如下
`
```
/***********************************************************************
* call_class_loads
* Call all pending class +load methods.
* If new classes become loadable, +load is NOT called for them.
*
* Called only by call_load_methods().
**********************************************************************/
static void call_class_loads(void)
{
    int i;

    // Detach current loadable list.
    struct loadable_class *classes = loadable_classes;
    int used = loadable_classes_used;
    loadable_classes = NULL;
    loadable_classes_allocated = 0;
    loadable_classes_used = 0;

    // Call all +loads for the detached list.
    for (i = 0; i < used; i++) {
        Class cls = classes[i].cls;
        IMP load_method = classes[i].method;
        if (!cls) continue; 

        if (PrintLoading) {
            _objc_inform("LOAD: +[%s load]\n", _class_getName(cls));
        }
        (*load_method) ((id) cls, SEL_load);
    }

    // Destroy the detached list.
    if (classes) _free_internal(classes);
}
```

`这里我们发现了 load 方法的本质，是直接执行函数指针，因此 load 方法不会执行 objc_msgSend 的那一整套流程，objc_msgSend 的完整流程可以看`[《深入理解 Objective-C 的方法调用流程》](http://www.jianshu.com/p/114782a909f9)

```
call_category_loads 的最终实现也和 call_class_loads 一样都是直接获取函数指针来执行，这里就不贴源码了。

load 方法调用总结
通过上述源码的分析，我们知道了 load 是在被添加到 runtime 时开始执行，父类最先执行，然后是子类，最后是 Category。又因为是直接获取函数指针来执行，不会像 objc_msgSend 一样会有方法查找的过程

```

[具体学习资料](http://www.jianshu.com/p/5127ce0628be)