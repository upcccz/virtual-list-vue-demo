<template>
  <div
    class="list-view"
    @scroll="handleScroll"
  >
    <div
      class="list-view-phantom"
      :style="{
         height: contentHeight + 'px'
      }">
    </div>
    <div
      ref="content"
      class="list-view-content">
      <div
        class="list-view-item"
        v-for="(item, index) in visibleData"
        :key="index"
        :style="{
          height: item.height + 'px'
        }"
      >
        {{ item.value }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ListView',
  props: {
    data: {
      type: Array,
      required: true
    },
    itemSizeGetter: {
      type: Function
    }
  },
  computed: {
    contentHeight() {
      const { data, itemSizeGetter } = this;
      let total = 0;
      for (let i = 0, j = data.length; i < j; i++) {
        total += itemSizeGetter.call(null, data[i], i);
      }
      return total;
    }
  },
  mounted () {
    this.updateVisibleData()
  },

  data () {
    return {
      visibleData: []
    }
  },

  methods: {
    getItemSizeAndOffset(index) {
    	const { data, itemSizeGetter } = this;
      let total = 0;
      for (let i = 0, j = Math.min(index, data.length - 1); i <= j; i++) {
        const size = itemSizeGetter.call(null, data[i], i);
        
        if (i === j) {
          return {
            offset: total,
            size
          };
        }
        total += size;
      }

      return {
        offset: 0,
        size: 0
      };
    },
    findNearestItemIndex(scrollTop) {
      const { data, itemSizeGetter } = this;
      let total = 0;
      for (let i = 0, j = data.length; i < j; i++) {
        const size = itemSizeGetter.call(null, data[i], i);
        total += size;
        if (total >= scrollTop || i === j -1) {
          return i;
        }
      }

      return 0;
    },
    updateVisibleData(scrollTop) {
    	scrollTop = scrollTop || 0;
      const start = this.findNearestItemIndex(scrollTop);
      console.log('start', start);
      const end = this.findNearestItemIndex(scrollTop + this.$el.clientHeight);
      this.visibleData = this.data.slice(start, Math.min(end + 1, this.data.length));
      this.$refs.content.style.webkitTransform = `translate3d(0, ${ this.getItemSizeAndOffset(start).offset }px, 0)`;
    },
    handleScroll () {
      const scrollTop = this.$el.scrollTop
      this.updateVisibleData(scrollTop)
    },
    
  }
}
</script>

<style scoped>
  .list-view {
  height: 400px;
  overflow: auto;
  position: relative;
  border: 1px solid #aaa;
}

.list-view-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  z-index: -1;
}

.list-view-content {
  left: 0;
  right: 0;
  top: 0;
  position: absolute;
}

.list-view-item {
  color: #666;
  padding: 5px;
  box-sizing: border-box;
  border: 1px solid pink;
}

</style>
