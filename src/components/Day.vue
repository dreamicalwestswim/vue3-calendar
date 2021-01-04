<template>
    <div
            class="calendar__day"
            :style="day === 1 ? {marginLeft: firstDayIndex * 14.285 + '%'} : null"
            @click="handlerClick"
    >
        <div class="calendar__box"
             :class="selfClass"
        >
            <div class="calendar__day-solar">{{day}}</div>
            <div class="calendar__day-lunar">{{lunarFestival}}</div>
            <div>{{bottomText}}</div>
        </div>
    </div>
</template>

<script>
import {calendar, getFestivals} from "./useCalendar";
import {computed, inject} from 'vue'
import {CALENDAR_KEY} from "./Calendar";

/**
 * 日历每天组件(负责展示每一天的内容)
 */
export default {
    name: 'CalendarDay',
    props :{
        // 每月日历数据包含了年月天数
        item: Object,
        // 每天日期
        day: [String, Number],
        // 类型(disabled,selected,middle,start,end)
        type: String
    },
    setup(props) {

        // 每一天的日期对象
        const date = new Date(props.item.year, props.item.month, props.day)

        // 获取顶级父组件的方法
        const { handlerSelect, bottomFormatter } = inject(CALENDAR_KEY)

        // 计算农历日和节日
        const festivals = getFestivals()
        const lunarFestival = computed(() => {
            // 优先显示今天
            const now = new Date();
            if(date.getTime() === +new Date(now.getFullYear(), now.getMonth(), now.getDate())){
                return '今天'
            }
            let info = calendar.solar2lunar(props.item.year,props.item.month,props.day)
            // 默认显示农历天，1号显示月
            let res = info.lDay === 1 ? info.IMonthCn : info.IDayCn
            // 阳历节日
            let sFestival = festivals.solar[info.cMonth][info.cDay]
            if(sFestival){
                res = sFestival
            }
            // 农历节日
            let lFestival = festivals.lunar[info.lMonth][info.lDay]
            if(lFestival){
                res = lFestival
            }
            return res
        });

        // 底部默认提示文字
        const bottomText = computed(() => {
            let res = '';
            if(props.type === 'start'){
                res = '开始'
            } else if(props.type === 'end'){
                res = '结束'
            }else if(props.type === 'middle' || props.type === 'selected'){
                res = '✓'
            }else if(props.type === 'disabled'){
                res = '-'
            }
            // 外部配置了就用外部文字
            let newText = bottomFormatter && bottomFormatter({
                date,
                type: props.type,
            });

            return newText || res
        });

        // 计算每月的第一天索引
        const firstDayIndex = computed(() => {
            return new Date(props.item.year, props.item.month, 1).getDay();
        });

        // 根据类型创建自身class名
        const selfClass = computed(() => {
            const day = date.getDay()
            return {
                // 周末样式
                'is-weekend': day === 0 || day === 6,
                // 禁用
                'is-disabled': props.type === 'disabled',
                // 选中
                'is-selected': props.type === 'selected',
                // 中间
                'is-middle': props.type === 'middle',
                // 开始
                'is-selected is-start': props.type === 'start',
                // 结束
                'is-selected is-end': props.type === 'end'
            }
        });

        // 点击日期给最顶层传递过去
        const handlerClick = () => {
            if(props.type !== 'disabled'){
                handlerSelect(date)
            }
        };

        return {
            firstDayIndex,
            selfClass,
            handlerClick,
            lunarFestival,
            bottomText
        }
    }
}
</script>

<style scoped>

</style>
