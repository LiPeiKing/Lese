# Vue生命周期

> 每个 Vue 应用都是通过用 `Vue` 函数创建一个新的 **Vue 实例**开始的

## 生命周期图示

借用官网一句话：你不需要立马弄明白所有的东西，不过随着你的不断学习和使用，它的参考价值会越来越高。

![Vue 实例生命周期](https://cn.vuejs.org/images/lifecycle.png)

## 关键钩子

> vue关键钩子created、mounted、beforeMount间的生命周期

```html
<div id="app">
   <h1>{{message}} 我是在默认HTML中</h1>
</div>
<script>
var vm = new Vue({
    data() {
        return {
            message: 'hello vue'
        };
    },
    beforeCreate() {
        console.group('beforeCreate');
        console.log('el: ', this.$el);
    },
    created() {
        console.group('created');
        console.log('el: ', this.$el);
    },
    beforeMount() {
        console.group('beforeMount');
        console.log('el: ', this.$el);
    },
    mounted() {
        console.group('mounted');
        console.log('el: ', this.$el);
    }
});
</script>
```

### el参数对生命周期的影响

+ 先判断是否有el项，如果没有执行到created后停止编译，
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606153000423.png)

+ 直到在该vue实例上调用vm.$mount(el)才会继续编译![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606153133769.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpaGVmZWlfY29kZXI=,size_16,color_FFFFFF,t_70)

### template参数对生命周期的影响

+ 如果没有template选项，el则将默认HTML作为模板编译   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606154244409.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpaGVmZWlfY29kZXI=,size_16,color_FFFFFF,t_70)

+ 如果有template参数选项，则将其作为模板编译成render函数。   ![在这里插入图片描述](https://img-blog.csdnimg.cn/2019060615481252.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpaGVmZWlfY29kZXI=,size_16,color_FFFFFF,t_70)

+ 如果同时存在template和render参数选项，则优先使用render渲染。   ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190606160359161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2xpaGVmZWlfY29kZXI=,size_16,color_FFFFFF,t_70)
  **综合排名优先级：render函数选项 > template选项 > outer HTML.**



























