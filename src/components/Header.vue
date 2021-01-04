<template>
    <div class="calendar__header">
        <div class="calendar__title">
            <div class="calendar__title-left" @click="handlerClick"><i class="iconfont-calendar icon-jiantouxiangshang" /></div>
           日期选择
        </div>
        <div class="calendar__weekdays">
            <div v-for="(item,index) in weekdays" :key="index" class="calendar__weekday">
                {{item}}
            </div>
        </div>
    </div>
</template>

<script>
import { reactive, toRefs } from 'vue'
export default {
    name: 'CalendarHeader',
    setup(props,{emit}) {
        const state = reactive({
            weekdays: ['日', '一', '二', '三', '四', '五', '六',]
        });

        const handlerClick = () => {
            emit('back')
        };

        return {
            ...toRefs(state),
            handlerClick
        }
    }
}
</script>

<style >

</style>
