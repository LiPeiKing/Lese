# SpringBoot之ApplicationListener

> applicationListener总是和contextrefreshedevent相对出现

### 1、原理

在spring的ioc容器初始化完成后，ioc容器会发布一个事件的动作，代码如下：

```java
	protected void finishRefresh() {
		// Clear context-level resource caches (such as ASM metadata from scanning).
		clearResourceCaches();

		// Initialize lifecycle processor for this context.
		initLifecycleProcessor();

		// Propagate refresh to lifecycle processor first.
		getLifecycleProcessor().onRefresh();

		// Publish the final event.
		publishEvent(new ContextRefreshedEvent(this));

		// Participate in LiveBeansView MBean, if active.
		if (!NativeDetector.inNativeImage()) {
			LiveBeansView.registerApplicationContext(this);
		}
	}
```

`publishEvent`方法发布事件，实现`ApplicationListener`重写`onApplicationEvent`方法即可获得该事件

### 2、实现

```java
@Component
public class SiteInfoInitListener implements ApplicationListener<ContextRefreshedEvent>{

    private static boolean isStarted = false;
    @Override
    public void onApplicationEvent(ContextRefreshedEvent contextRefreshedEvent) {
        if(!isStarted) { 
            // contextRefreshedEvent.getApplicationContext().getParent() == null;限制只执行一次
            // do ...
        }
    }
}
```

### 3、注意事项

在web项目中，application context有两个容器，一个是root application  context，一个是serlet context子容器，所以这两个容器初始化完成后都会发布这个事件，所以onApplicationEvent方法将会调用两个，为了防止onApplicationEvent方法执行两次，我们可以if判断符合：ContextRefreshedEvent.getApplication().getParent()限制到父容器才去执行我们需要的逻辑代码。因为root application context本身就是最上级的容器，所以他没有父容器，但是servletcontext的父容器是root application context
