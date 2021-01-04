<template>
  <div class="home">
    <h3>单选</h3>

    <div>
      <button @click="singleShow1=true">默认无值</button>
      {{singleDate1}}
    </div>
    <Calendar v-model:visible="singleShow1" :default-date="null" @confirm="onConfirmSingle1" />

    <div>
      <button @click="singleShow2=true">默认有值(今天)</button>
      {{singleDate2}}
    </div>
    <Calendar v-model:visible="singleShow2" @confirm="onConfirmSingle2" />

    <div>
      <button @click="singleShow3=true">默认有值(自定义)</button>
      {{singleDate3}}
    </div>
    <Calendar v-model:visible="singleShow3" :default-date="singleDefaultDate" @confirm="onConfirmSingle3" />

    <h3>多选</h3>
    <div>
      <button @click="multipleShow1=true">选择多个日期</button>
      {{multipleDate1}}
    </div>
    <Calendar v-model:visible="multipleShow1" type="multiple" @confirm="onConfirmMultiple1" />

    <div>
      <button @click="multipleShow2=true">选择多个日期(限制选择5个日期)</button>
      {{multipleDate2}}
    </div>
    <Calendar v-model:visible="multipleShow2" type="multiple" max-range="5" @confirm="onConfirmMultiple2" />

    <h3>区间选择</h3>
    <div>
      <button @click="rangeShow1=true">选择区间日期</button>
      {{rangeDate1}}
    </div>
    <Calendar v-model:visible="rangeShow1" type="range"  @confirm="onConfirmRange1" />

    <div>
      <button @click="rangeShow2=true">选择区间日期(可以选同一天)</button>
      {{rangeDate2}}
    </div>
    <Calendar v-model:visible="rangeShow2" type="range" allowSameDay  @confirm="onConfirmRange2" />

    <div>
      <button @click="rangeShow3=true">选择区间日期(自定义第三行文字)</button>
      {{rangeDate3}}
    </div>
    <Calendar v-model:visible="rangeShow3" type="range" :bottomFormatter="bottomFormatter" @confirm="onConfirmRange3" />
  </div>
</template>

<script>


import {ref} from 'vue'
export default {
  name: 'App',
  components: {

  },
  setup() {
    const formatDate = (date) => `${date.getMonth() + 1}/${date.getDate()}`;

    const singleDate1 = ref('');
    const singleShow1 = ref(false);
    const onConfirmSingle1 = (value) => {
      singleDate1.value = formatDate(value);
    };

    const singleDate2 = ref('');
    const singleShow2 = ref(false);
    const onConfirmSingle2 = (value) => {
      singleDate2.value = formatDate(value);
    };

    const singleDefaultDate = new Date('2021/5/6');
    const singleDate3 = ref('');
    const singleShow3 = ref(false);
    const onConfirmSingle3 = (value) => {
      singleDate3.value = formatDate(value);
    };

    const multipleDate1 = ref('');
    const multipleShow1 = ref(false);
    const onConfirmMultiple1 = (value) => {
      let res = []
      for(let i = 0;i<value.length;i++){
        res.push(formatDate(value[i]))
      }
      multipleDate1.value = res;
    };

    const multipleDate2 = ref('');
    const multipleShow2 = ref(false);
    const onConfirmMultiple2 = (value) => {
      let res = []
      for(let i = 0;i<value.length;i++){
        res.push(formatDate(value[i]))
      }
      multipleDate2.value = res;
    };

    const rangeDate1 = ref('');
    const rangeShow1 = ref(false);
    const onConfirmRange1 = (value) => {
      rangeDate1.value = `${formatDate(value[0])} - ${formatDate(value[1])}`;
    };

    const rangeDate2 = ref('');
    const rangeShow2 = ref(false);
    const onConfirmRange2 = (value) => {
      rangeDate2.value = `${formatDate(value[0])} - ${formatDate(value[1])}`;
    };

    const rangeDate3 = ref('');
    const rangeShow3 = ref(false);
    const onConfirmRange3 = (value) => {
      rangeDate3.value = `${formatDate(value[0])} - ${formatDate(value[1])}`;
    };
    const bottomFormatter = ({type}) => {
      if(type === 'start'){
        return '开学'
      } else if(type === 'end'){
        return '放假'
      }else if(type === 'middle'){
        return '学习'
      }
    };

    return {
      singleDate1,
      singleShow1,
      onConfirmSingle1,
      singleDate2,
      singleShow2,
      onConfirmSingle2,
      singleDate3,
      singleShow3,
      onConfirmSingle3,
      singleDefaultDate,
      multipleDate1,
      multipleShow1,
      onConfirmMultiple1,
      multipleDate2,
      multipleShow2,
      onConfirmMultiple2,
      rangeDate1,
      rangeShow1,
      onConfirmRange1,
      rangeDate2,
      rangeShow2,
      onConfirmRange2,
      rangeDate3,
      rangeShow3,
      onConfirmRange3,
      bottomFormatter,
    };
  }
}
</script>

<style lang="scss">

</style>
