<template>
  <div>
    <header class="page-header">
      <h1>Vue Draggable Layout</h1>
    </header>
    <div class="container">
      <div class="main">
        <div class="top">
          <draggable
            v-for="(list, i) in smallBoxes"
            :key="'small-'+i"
            :list="list"
            group="items"
            class="small-box"
            @remove="onRemove"
          >
            <div class="box-label">Small Box {{ i + 1 }}</div>
            <div v-for="(item, index) in list" :key="index" class="dropped-item">
              {{ item.name }}
              <button @click="removeItem(list, index)">×</button>
            </div>
          </draggable>
        </div>
        <div class="bottom">
          <draggable
            v-for="(list, i) in bigBoxes"
            :key="'big-'+i"
            :list="list"
            group="items"
            class="big-box"
            @remove="onRemove"
          >
            <div class="box-label">Big Box {{ i + 1 }}</div>
            <div v-for="(item, index) in list" :key="index" class="dropped-item">
              {{ item.name }}
              <button @click="removeItem(list, index)">×</button>
            </div>
          </draggable>
        </div>
      </div>

      <div class="sidebar">
        <h3>Options</h3>
        <draggable :list="options" group="items" class="option-list">
          <div v-for="(item, index) in options" :key="index" class="option-item">
            {{ item.name }}
          </div>
        </draggable>
      </div>
    </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable';

export default {
  name: 'DraggableLayout',
  components: {
    draggable,
  },
  data() {
    return {
      options: [
        { name: 'Option 1' },
        { name: 'Option 2' },
        { name: 'Option 3' },
        { name: 'Option 4' },
        { name: 'Option 5' },
        { name: 'Option 6' },
        { name: 'Option 7' },
        { name: 'Option 8' }
      ],
      smallBoxes: [[], [], [], []],
      bigBoxes: [[], []]
    };
  },
  methods: {
    removeItem(list, index) {
      const removedItem = list.splice(index, 1)[0];
      this.options.push(removedItem);
    },
    onRemove() {
      // handled by removeItem
    }
  }
};
</script>

<style scoped>
.page-header {
  text-align: center;
  padding: 15px;
  background-color: #3f51b5;
  color: white;
  font-size: 1.5em;
  font-weight: bold;
}

.container {
  display: flex;
  height: calc(100vh - 70px);
}

.main {
  width: 67%;
  display: flex;
  flex-direction: column;
  padding: 10px;
  gap: 10px;
}

.sidebar {
  width: 33%;
  background: #f5f5f5;
  padding: 10px;
  box-shadow: -2px 0 5px rgba(0,0,0,0.1);
}

.option-list {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.option-item {
  background: #e0e0e0;
  padding: 10px;
  border-radius: 4px;
  cursor: move;
  text-align: center;
}

.top {
  flex: 1;
  display: flex;
  gap: 10px;
}

.small-box {
  flex: 1;
  min-height: 100px;
  background: #d9f1ff;
  padding: 10px;
  border: 1px solid #90caf9;
  border-radius: 4px;
  position: relative;
}

.bottom {
  flex: 1;
  display: flex;
  gap: 10px;
}

.big-box {
  flex: 1;
  min-height: 200px;
  background: #f0e4ff;
  padding: 10px;
  border: 1px solid #ce93d8;
  border-radius: 4px;
  position: relative;
}

.dropped-item {
  background: #ffffff;
  border: 1px solid #ccc;
  padding: 8px;
  margin-bottom: 5px;
  border-radius: 4px;
  position: relative;
}

.dropped-item button {
  position: absolute;
  top: 2px;
  right: 5px;
  background: red;
  border: none;
  color: white;
  font-weight: bold;
  cursor: pointer;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  line-height: 16px;
  text-align: center;
}

.box-label {
  font-weight: bold;
  margin-bottom: 5px;
  text-align: center;
}
</style>
