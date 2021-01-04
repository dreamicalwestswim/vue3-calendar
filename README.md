# vue3-calendar
基于vue3.0开发的移动版日历组件,用于选择日期或日期区间  
支持阳历、农历 节日、第三行状态文字

## 说明
在写这个组件之前，有朋友问我，为什么ui框架里面有日历组件，你还要去写？究竟有什么意义？

其实意义非常大，首先不同的项目用的ui框架也会不一样，功能也会有些许差异，在功能不能满足你需求的情况下，是另外装一个ui框架？
还是去改框架的源码？(ui框架装多了不仅代码容量大，也容易造成一定的冲突和混乱，改框架源码就意味着你不能再更新版本,在这个浮躁的社会！也不是每个人都能改的) 这些都不是优雅的解决方案。

这个时候这种独立且不基于任何ui框架的组件，就展现出了它的用武之地。
1. 独立 灵活 可以集成到任何项目里面；
2. 轻量 易修改 如果需求不能满足，可以很轻易的二次开发。

### Demo
[地址](https://dreamicalwestswim.github.io/vue3-calendar/demo)

## 安装
```
npm install vue3-calendar -S
or
yarn add vue3-calendar
```

## 引入
```
import { createApp } from 'vue'
import Calendar from "vue3-calendar";
import 'vue3-calendar/lib/vue3-calendar.css';

const app = createApp(App)
app.use(Calendar)

```

#### 选择单个日期
```
<div>
    <button @click="show=true">选择单个日期</button>
    {{date}}
</div>
<Calendar v-model:visible="show"  @confirm="onConfirm" />
```
```
import { ref } from 'vue';

export default {
  setup() {
    const date = ref('');
    const show = ref(false);

    const formatDate = (date) => `${date.getMonth() + 1}/${date.getDate()}`;
    const onConfirm = (value) => {
      date.value = formatDate(value);
    };

    return {
      date,
      show,
      onConfirm,
    };
  },
};
```

#### 选择多个日期
```
<div>
    <button @click="show=true">选择多个日期</button>
    {{date}}
</div>
<Calendar v-model:visible="show" type="multiple" max-range="3" @confirm="onConfirm" />
```
```
import { ref } from 'vue';

export default {
  setup() {
    const date = ref('');
    const show = ref(false);

    const formatDate = (date) => `${date.getMonth() + 1}/${date.getDate()}`;
    const onConfirm = (value) => {
      let res = []
      for(let i = 0;i<value.length;i++){
        res.push(formatDate(value[i]))
      }
      date.value = res;
    };

    return {
      date,
      show,
      onConfirm,
    };
  },
};
```

#### 选择日期区间
```
<div>
    <button @click="show=true">选择日期区间</button>
    {{date}}
</div>
<Calendar v-model:visible="show" type="range"  @confirm="onConfirm" />
```
```
import { ref } from 'vue';

export default {
  setup() {
    const date = ref('');
    const show = ref(false);

    const formatDate = (date) => `${date.getMonth() + 1}/${date.getDate()}`;
    const onConfirm = (value) => {
      date.value = `${formatDate(value[0])} - ${formatDate(value[1])}`;
    };

    return {
      date,
      show,
      onConfirm,
    };
  },
};
```

#### Props
 tips:   
 所有的日期对象只需要年月日(new Date(1990/10/22))，不需要时间，内部用时间戳对比大小  
 如果传了时间,日期就不是从0点开始了，这样就比较怪。

  参数  | 说明 | 类型 | 默认值
 ---- | ----- | ------ | ------  
 visible  | 是否显示日历弹窗 | boolean | false 
 type  | 选择类型(single单个日期,multiple多个日期,range日期区间) | string | single   
 minDate  | 可选择的最小日期 | Date | 当天日期   
 maxDate  | 可选择的最大日期 | Date | 当天日期往后延6个月   
 defaultDate  | 默认选中日期(type 为 multiple 或 range 时为数组，传入 null 表示默认不选择) | Date | 今天日期
 bottomFormatter  | 底部第三行文字配置(可以根据这个函数里面的date,type重新设置第三行文字) | ({date,type})=>{} | 
 allowSameDay  | 是否允许日期范围的起止时间为同一天 | boolean | false
 maxRange  | 日期区间或多选最多可选天数 | Number, String |
 rangePrompt  | 区间选择超过最大值触发,默认用alert提示外部可以结合其他ui库提示 | (n)=>{} |
 confirmText  | 底部按钮文字 | String | 确定
 confirmDisabledText  | 底部按钮禁用文字 | String | 确定

