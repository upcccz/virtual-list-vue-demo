<template>
  <div id="app">
    <div class="tab">
      <div :class="[activeIndex === 1 ? 'active' : '' ]" @click="activeIndex = 1">定高 virtual list</div>
      <div :class="[activeIndex === 2 ? 'active' : '' ]" @click="activeIndex = 2">不定高 virtual list</div>
    </div>
    <virtualList1 v-if="activeIndex === 1" :data="arr"/>
    <virtualList2
      v-if="activeIndex === 2" 
      :data="arr2" 
      :item-size-getter="itemSizeGetter"
      :estimated-item-size="30"
    />
  </div>
</template>

<script>
import virtualList1 from './components/virtual-list1.vue'
import virtualList2 from './components/virtual-list2.vue'

export default {
  name: 'App',
  components: {
    virtualList1,
    virtualList2
  },
  data () {
    return {
      arr: [],
      arr2: [],
      activeIndex: 1,
    }
  },
  created () {
    const a = []
    for (let index = 0; index < 100; index++) {
      a.push({ value: index})
    }
    this.arr = a;
    const a2 = [];
    for (let index2 = 0; index2 < 100; index2++) {
      a2.push({ value: index2, height: 30 + index2})
    }
    this.arr2 = a2;
  },
  methods: {
    itemSizeGetter(item) {
    	return 30 + item.value;
    }
  },
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.tab {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-bottom: 10px;
}

.tab div {
  flex: 1;
}

.tab div.active {
  color: red;
}
</style>
