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
    },
    estimatedItemSize: {
      type: Number,
      default: 30
    }
  },
  computed: {
    contentHeight() {
      const { data, lastMeasuredIndex, estimatedItemSize } = this;
      let itemCount = data.length;
      if (lastMeasuredIndex >= 0) {
        const lastMeasuredSizeAndOffset = this.getLastMeasuredSizeAndOffset();
        return lastMeasuredSizeAndOffset.offset + lastMeasuredSizeAndOffset.size + (itemCount - 1 - lastMeasuredIndex) * estimatedItemSize;
      } else {
        return itemCount * estimatedItemSize;
      }
    }
  },
  mounted () {
    this.updateVisibleData()
  },

  data () {
    return {
      visibleData: [],
      lastMeasuredIndex: -1,
      startIndex: 0,
      sizeAndOffsetCahce: {},
    }
  },

  methods: {
    getLastMeasuredSizeAndOffset() {
      return this.lastMeasuredIndex >= 0 ? this.sizeAndOffsetCahce[this.lastMeasuredIndex] : { offset: 0, size: 0 };
    },
    getItemSizeAndOffset(index) {
      const { lastMeasuredIndex, sizeAndOffsetCahce, data, itemSizeGetter } = this;
      if (lastMeasuredIndex >= index) {
        return sizeAndOffsetCahce[index];
      }
      let offset = 0;
      if (lastMeasuredIndex >= 0) {
        const lastMeasured = sizeAndOffsetCahce[lastMeasuredIndex];
        if (lastMeasured) {
          offset = lastMeasured.offset + lastMeasured.size;
        }
      }
      for (let i = lastMeasuredIndex + 1; i <= index; i++) {
        const item = data[i];
        const size = itemSizeGetter.call(null, item, i);
        sizeAndOffsetCahce[i] = {
          size,
          offset
        };
        offset += size;
      }
      if (index > lastMeasuredIndex) {
        this.lastMeasuredIndex = index;
      }
      return sizeAndOffsetCahce[index];
    },

    findNearestItemIndex(scrollTop) {
      const lastMeasuredOffset = this.getLastMeasuredSizeAndOffset().offset;
      if (lastMeasuredOffset > scrollTop) {
        return this.binarySearch(0, this.lastMeasuredIndex, scrollTop);
      } else {
        return this.exponentialSearch(scrollTop);
      }
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
    binarySearch(low, high, offset) {
      let index;
      while (low <= high) {
        const middle = Math.floor((low + high) / 2);
        const middleOffset = this.getItemSizeAndOffset(middle).offset;
        if (middleOffset === offset) {
          index = middle;
          break;
        } else if (middleOffset > offset) {
          high = middle - 1;
        } else {
          low = middle + 1;
        }
      }

      if (low > 0) {
        index = low - 1;
      }

      if (typeof index === 'undefined') {
        index = 0;
      }

      return index;
    },
    exponentialSearch(scrollTop) {
      let bound = 1;
      const data = this.data;
      const start = this.lastMeasuredIndex >= 0 ? this.lastMeasuredIndex : 0;
      while (start + bound < data.length && this.getItemSizeAndOffset(start + bound).offset < scrollTop) {
        bound = bound * 2;
      }
      return this.binarySearch(start + Math.floor(bound / 2), Math.min(start + bound, data.length-1), scrollTop);
    }
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
