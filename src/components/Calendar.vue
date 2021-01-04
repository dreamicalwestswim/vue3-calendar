<template>
    <teleport to="body">
        <div class="calendar" :class="{'is-show':visible}">
            <Header @back="handlerClose"/>

            <Body
                    :type="type"
                    :minDate="minDate"
                    :maxDate="maxDate"
                    :selectedDate="selectedDate"
                    :maxRange="maxRange"
            />

            <div class="calendar__footer">
                <button :disabled="isDisable" @click="handlerConfirm">{{isDisable?confirmDisabledText:confirmText}}</button>
            </div>
        </div>
    </teleport>
</template>

<script>

import {provide, reactive, toRefs, computed} from 'vue'
import {copyDates} from './useCalendar'
import Header from './Header'
import Body from './Body'
export const CALENDAR_KEY = 'base-Calendar';
export default {
    name: 'Calendar',
    props: {
        // 是否可见
        visible: Boolean,
        // 选择类型(single单个日期,multiple多个日期,range日期区间)
        type: {
            type: String,
            default: 'single'
        },
        // 可选择的最小日期(今天)
        minDate: {
            type: Date,
            default () {
                const now = new Date();
                return new Date(now.getFullYear(), now.getMonth(), now.getDate());
            }
        },
        // 可选择的最大日期(今天往后延6个月)
        maxDate: {
            type: Date,
            default () {
                const now = new Date();
                return new Date(now.getFullYear(), now.getMonth() + 6, now.getDate());
            }
        },
        // 默认选中日期(不需要传时间，内部用毫秒对比，每天的0时0分0秒0毫秒开始)
        defaultDate: {
            type: [Date,Array],
            default () {
                const now = new Date();
                return new Date(now.getFullYear(), now.getMonth(), now.getDate());
            }
        },
        // 底部文字配置
        bottomFormatter: Function,
        // 是否允许日期范围的起止时间为同一天
        allowSameDay: Boolean,
        // 日期区间或多选最多可选天数
        maxRange: [Number, String],
        // 区间选中超过最大值触发,默认用alert提示外部可以结合其他ui库提示
        rangePrompt: {
            type: Function,
            default (n) {
                alert(`选择天数不能超过${n}天`)
            }
        },
        // 底部按钮文字
        confirmText: {
            type: String,
            default: '确定'
        },
        // 底部按钮禁用文字
        confirmDisabledText: {
            type: String,
            default: '确定'
        }
    },
    emits: ['confirm', 'update:visible'],
    components: {
        Header,
        Body,
    },
    setup(props, { emit }) {
        // 获取初始日期
        const getInitialDate = () => {
            // 不是单选又不是数组就用数组包装起来
            return (props.type !== 'single' && !Array.isArray(props.defaultDate)) ? (props.defaultDate?[props.defaultDate]:[]) : props.defaultDate
        };

        const state = reactive({
            selectedDate: getInitialDate()
        });

        // 计算按钮禁用状态
        const isDisable = computed(() => {
            if(!state.selectedDate || (props.type === 'multiple' && !state.selectedDate.length) || (props.type === 'range' && state.selectedDate.length < 2)){
                return true
            }
        });

        // 处理日期选择
        const handlerSelect = (date) => {
            // 单选
            if(props.type === 'single'){
                state.selectedDate = date
            } else if(props.type === 'multiple'){
                // 多选
                if(state.selectedDate.some(item => item.getTime() === date.getTime())) {
                    // 存在就过滤掉
                    state.selectedDate = state.selectedDate.filter(item => item.getTime() !== date.getTime())
                } else {
                    // 不存在就加入进去
                    state.selectedDate.push(date)
                }
            } else if(props.type === 'range'){
                // 区间选择
                let range = state.selectedDate.slice()

                if(range.length === 1){
                    // 第二个小于第一个直接删掉第一个, 不允许同一天也会删除第一个
                    if(date.getTime() < range[0].getTime() || (!props.allowSameDay && date.getTime() === range[0].getTime())){
                        range.shift()
                    } else {
                        let timeStampRange = date.getTime() - range[0].getTime()
                        let timeStampDay = 1000 * 60 * 60 * 24
                        if(timeStampRange / timeStampDay > props.maxRange){
                            // 由外部去提示自由度会高很多
                            props.rangePrompt(props.maxRange)
                            // 结束日期设置成最大允许日期
                            date = new Date(range[0].getTime() + timeStampDay * props.maxRange)
                        }
                    }
                } else if(range.length > 1){
                    // 已经有两个值直接清空数组
                    range.length = 0
                }

                range.push(date)
                state.selectedDate = range
            }
        };

        // 回到默认值(外部获取实例调用)
        const reset = () => {
            state.selectedDate = getInitialDate();
        };

        // 关闭日历
        const handlerClose = () => {
            emit('update:visible',false)
        };

        // 确定提交数据
        const handlerConfirm = () => {
            // 拷贝一份普通数据给外部使用防止外部修改
            emit('confirm',copyDates(state.selectedDate))
            handlerClose()
        };

        // 提供给子组件使用
        provide(CALENDAR_KEY, {
            handlerSelect,
            bottomFormatter: props.bottomFormatter
        });

        return {
            ...toRefs(state),
            isDisable,
            handlerConfirm,
            reset,
            handlerClose
        }
    }
}
</script>

<style lang="scss">
    @import "iconfont.css";
    .calendar{
        -webkit-tap-highlight-color:rgba(0,0,0,0);
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
        "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei",
        SimSun, sans-serif;
        display: flex;
        flex-direction: column;
        width: 100%;
        height: 100%;
        color: #0c0c0c;
        background-color: #fff;
        user-select: none;
        position: fixed;
        top: 0;
        left: 100%;
        opacity: 0;
        visibility: hidden;
        transition: .3s all ease-in-out;
        &.is-show{
            left: 0;
            opacity: 1;
            visibility: visible;
        }


        &__header{
            flex-shrink: 0;
            box-shadow: 0 2px 10px rgba(125, 126, 128, 0.16);
            position: relative;
            z-index: 2;
        }

        &__title{
            font-size: 18px;
            height: 44px;
            line-height: 44px;
            text-align: center;
            padding: 0 15px;
            position: relative;
        }

        &__title-left{
            position: absolute;
            .icon-jiantouxiangshang{
                transform:rotate(-90deg);
            }
        }

        &__weekdays {
            display: flex;
        }

        &__weekday {
            flex: 1;
            font-size: 12px;
            line-height: 30px;
            text-align: center;
        }

        &__body{
            flex: 1;
            overflow: auto;
            -webkit-overflow-scrolling: touch;
            position: relative;
            scrollbar-width: none; /* firefox */
            -ms-overflow-style: none; /* IE 10+ */
            &::-webkit-scrollbar {
                display: none; /* Chrome Safari */
            }
        }

        &__month-title{
            text-align: center;
            width: 100%;
            height: 40px;
            line-height: 40px;
            font-weight: bold;
            font-size: 16px;
            background: #fff;
            &.is-fixed{
                position: absolute;
                top: 74px;
                opacity: 0;
                visibility: hidden;
                transition: all .3s ease-in-out;
            }
            &.is-show{
                opacity: 1;
                visibility: visible;
            }
        }

        &__days {
            position: relative;
            display: flex;
            flex-wrap: wrap;
        }

        &__month-mark{
            position: absolute;
            top: 50%;
            left: 50%;
            z-index: 0;
            color: rgba(242, 243, 245, 0.6);
            font-size: 160px;
            transform: translate(-50%, -50%);
        }

        &__day {
            position: relative;
            width: 14.285%;
            height: 64px;
            box-sizing: border-box;
            font-size: 10px;
            cursor: pointer;
            text-align: center;
            padding-top: 5px;

            &-solar{
                padding-top: 2px;
                font-size: 16px;
            }
            &-lunar{
                color: #8d8d8d;
            }
        }

        &__box{
            width: 100%;
            max-width: 54px;
            height: 54px;
            display: inline-block;
            border-radius: 4px;
            &.is-weekend {
                color: rgba(255, 10, 37, 0.82);
                .calendar__day-lunar{
                    color: rgba(255, 10, 37, 0.82);
                }
            }
            &.is-disabled {
                color: #c8c9cc;
                cursor: default;
                .calendar__day-lunar{
                    color: #c8c9cc;
                }
            }
            &.is-selected {
                background-color: #ee0a24;
                color: #fff;
                .calendar__day-lunar{
                    color: #fff;
                }
            }
            &.is-middle{
                max-width: 100%;
                border-radius: 0;
                color: #ee0a24;
                background-color: #fde7ea;
                .calendar__day-lunar{
                    color: #ee0a24;
                }
            }
            &.is-start{
                max-width: 100%;
                border-radius: 4px 0 0 4px;
            }
            &.is-end{
                max-width: 100%;
                border-radius: 0 4px 4px 0px;
            }
        }

        &__footer {
            flex-shrink: 0;
            padding: 0 16px;
            box-shadow: 0 -2px 10px rgba(125, 126, 128, 0.16);
            padding-bottom: constant(safe-area-inset-bottom);
            padding-bottom: env(safe-area-inset-bottom);
            button{
                outline: none;
                color: #fff;
                background-color: #ee0a24;
                border: 1px solid #ee0a24;
                width: 100%;
                display: block;
                font-size: 14px;
                height: 36px;
                margin: 7px 0;
                border-radius: 36px;
                &:disabled{
                    opacity: .5;
                    cursor: not-allowed;
                }
            }
        }
    }
</style>
