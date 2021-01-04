<template>
    <div ref="bodyRef"
         class="calendar__body"
         @scroll="onScroll"
         @touchstart="onTouchstart"
         @touchmove="onTouchmove"
    >
        <div
                v-for="(item, index) in calendarData"
                :key="index"
                class="calendar__month"
        >
            <div class="calendar__month-title">
                {{`${item.year}年${item.month+1}月`}}
            </div>
            <div class="calendar__days">
                <div class="calendar__month-mark">{{item.month+1}}</div>
                <Day
                        v-for="day in item.days"
                        :key="day"
                        :item="item"
                        :day="day"
                        :type="getDayType(item, day)"
                />
            </div>
        </div>
    </div>

    <div class="calendar__month-title is-fixed" :class="{'is-show':dateVisible}">
        {{`${calendarData[dateIndex].year}年${calendarData[dateIndex].month+1}月`}}
    </div>
</template>

<script>

import {computed, reactive, ref, toRefs, onMounted} from "vue";
import {getCalendarData} from "./useCalendar";
import Day from './Day';

export default {
    name: 'CalendarBody',
    components: {
        Day
    },
    props :{
        // 选择类型
        type: String,
        // 生成的最小日历
        minDate: Date,
        // 生成的最大日历
        maxDate: Date,
        // 当前选中的日期
        selectedDate: [Date,Array],
        // 日期区间最多可选天数
        maxRange: [Number, String],
    },
    setup(props) {
        const state = reactive({
            // 日期标题索引
            dateIndex: 0,
            dateVisible: false,
        });

        // 日历展示的数据
        const calendarData = computed(() => {
            return getCalendarData(props.minDate, props.maxDate)
        });

        // body节点
        const bodyRef = ref()

        // 判断滚动位置
        const onScroll = (e) => {
            const scrollTop = e.target.scrollTop
            const bodyNode = bodyRef.value
            for(let i = 0,l = bodyNode.children.length; i < l; i++){
                const childNode = bodyNode.children[i]
                if(scrollTop > childNode.offsetTop  && scrollTop < childNode.offsetTop  + childNode.offsetHeight){
                    changeDateIndex(i)
                    return
                }
            }
        };

        // 修改日期标题索引
        let timerId;
        const changeDateIndex = (i) => {
            if(state.dateIndex === i)return
            state.dateIndex = i
            state.dateVisible = true
            clearTimeout(timerId)
            timerId = setTimeout(()=>{
                state.dateVisible = false
            },1000)
        };

        // 获取每一天的类型
        const getDayType = (item, day) => {
            const timeStamp = +new Date(item.year, item.month, day)
            // 超过可选日期范围禁用
            if(timeStamp < props.minDate.getTime() || timeStamp > props.maxDate.getTime()) {
                return 'disabled'
            }
            // 选择数据时的类型
            if(!props.selectedDate)return
            if(props.type === 'single'){
                return props.selectedDate.getTime() === timeStamp ? 'selected' : ''
            } else if(props.type === 'multiple'){
                for(let i = 0;i<props.selectedDate.length;i++){
                    if(props.selectedDate[i].getTime() === timeStamp){
                        return 'selected'
                    }
                }
                // 选中数量达到最大数其他的都禁用
                if(props.selectedDate.length >= props.maxRange)return 'disabled'
            } else if(props.type === 'range'){
                let res;
                for(let i = 0;i<props.selectedDate.length;i++){
                    if(props.selectedDate[i].getTime() === timeStamp){
                        res = i === 0 ? 'start' : 'end'
                    }
                }
                if(props.selectedDate.length === 2){
                    if(timeStamp > props.selectedDate[0].getTime() && timeStamp < props.selectedDate[1].getTime()){
                        return 'middle'
                    }
                }
                return res;
            }
        };

        // 解决ios-webkit-overflow-scrolling: touch回弹引起的bug
        let lastY = 0
        const onTouchstart = (e) => {
            lastY = e.touches[0].clientY;
        };
        const onTouchmove = (e) => {
            let direction = (lastY - e.touches[0].clientY) < 0 ? "down" : "up";
            const bodyNode = bodyRef.value
            // 设置回弹后滑到顶部或者底部再滑一下即可卡死不动，判断滚到顶或低阻止默认行为可以解决卡死bug
            if(bodyNode.scrollTop === 0 && direction === 'down' || (bodyNode.scrollTop === (bodyNode.scrollHeight - bodyNode.clientHeight) && direction === 'up')){
                e.preventDefault();
            }
        };

        // 挂载时设置初始位置(初始位置定在选中最小日期月份位置上)
        onMounted(() => {
            if(!props.selectedDate)return
            const miniSort = (selectedDate) => {
                // 获得选中的最小日期
                return [].concat(selectedDate).sort(function (a, b) {
                    return a.getTime() - b.getTime();
                })[0]
            };
            let minDate = Array.isArray(props.selectedDate) ? miniSort(props.selectedDate) : props.selectedDate
            // 选中的最小日期年减去可选的最小日期年得到他们的间隔
            let yearInterval = (minDate.getFullYear() - props.minDate.getFullYear())
            // 月间隔也一样
            let monthInterval = minDate.getMonth() - props.minDate.getMonth()
            // 年间隔乘12加上月间隔得到月间隔的总数(这个总数刚好和ui的月模块索引值对应上)
            let totalMonths = yearInterval * 12 + monthInterval
            // 初始位置定义在对应的月模块上
            const bodyNode = bodyRef.value
            bodyNode.scrollTop = bodyNode.children[totalMonths].offsetTop
        });

        return {
            ...toRefs(state),
            bodyRef,
            onScroll,
            onTouchstart,
            onTouchmove,
            calendarData,
            getDayType
        }
    }
}
</script>

<style>

</style>
