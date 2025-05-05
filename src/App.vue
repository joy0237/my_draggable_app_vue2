<template>
  <div class="app">
    <div class="header">Vue2 Free-Placement Drag & Drop</div>
    <div class="container">
      <div
        class="left-pane drop-zone"
        @dragover.prevent
        @drop="onDrop($event)"
        ref="dropZone"
      >
        <div class="pane-title">Drop Zone</div>
        <div
          v-for="item in droppedOptions"
          :key="item.id"
          class="dropped-item resizable-box"
          :style="{
            top: item.y + 'px',
            left: item.x + 'px',
            height: item.height + 'px',
            width: item.width + 'px'
          }"
          @mousedown="startDrag(item, $event)"
        >
          <div class="remove-btn" @click.stop="removeItem(item)">×</div>
          <div class="resize-handle vertical" @mousedown.stop.prevent="startResizing(item, $event, 'height')"></div>
          <div class="resize-handle horizontal" @mousedown.stop.prevent="startResizing(item, $event, 'width')"></div>
          <div class="resize-handle corner" @mousedown.stop.prevent="startResizing(item, $event, 'both')"></div>
          <div class="option-header">{{ item.name }}</div>
        </div>
        <button class="submit-btn" @click="submitLayout">Submit</button>
      </div>
      <div class="right-pane options-pane">
        <div class="pane-title">Options</div>
        <div
          v-for="option in options"
          :key="option.id"
          class="draggable-item"
          :style="{ height: option.height + 'px', width: option.width + 'px' }"
          draggable="true"
          @dragstart="onDragStart(option, $event)"
        >
        <div class="option-header">{{ option.name }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      options: [
        { id: 1, name: 'Option 1', height: 60, width: 120 },
        { id: 2, name: 'Option 2', height: 60, width: 120 },
        { id: 3, name: 'Option 3', height: 60, width: 120 }
      ],
      droppedOptions: [],
      draggingItem: null,
      resizingItem: null,
      resizingType: '',
      startMouseX: 0,
      startMouseY: 0,
      startWidth: 0,
      startHeight: 0,
      startX: 0,
      startY: 0,
      submittedJson: null
    };
  },
  methods: {
    onDragStart(option, event) {
      this.draggingItem = option;
      event.dataTransfer.setData('text/plain', option.id);
    },
    isOverlapping(x, y, width, height, excludeId = null) {
      const submitBtnPadding = 10;
      const submitBtnWidth = 120;
      const submitBtnHeight = 40;

      const dropZoneWidth = this.$refs.dropZone.clientWidth;
      const dropZoneHeight = this.$refs.dropZone.clientHeight;

      const submitX = dropZoneWidth - submitBtnWidth - submitBtnPadding;
      const submitY = dropZoneHeight - submitBtnHeight - submitBtnPadding;

      const overlapsSubmitButton =
        x < submitX + submitBtnWidth &&
        x + width > submitX &&
        y < submitY + submitBtnHeight &&
        y + height > submitY;

      if (overlapsSubmitButton) return true;

      const headerHeight = 40; 
      if (y < headerHeight) return true; 

      return this.droppedOptions.some(item => {
        if (item.id === excludeId) return false;
        return !(
          x + width <= item.x ||
          x >= item.x + item.width ||
          y + height <= item.y ||
          y >= item.y + item.height
        );
      });
    },
    onDrop(event) {
      const id = Number(event.dataTransfer.getData('text/plain'));
      const option = this.options.find(o => o.id === id);
      if (!option) return;

      const rect = this.$refs.dropZone.getBoundingClientRect();
      let x = event.clientX - rect.left - option.width / 2;
      let y = event.clientY - rect.top - option.height / 2;

      x = Math.max(0, Math.min(x, rect.width - option.width));
      y = Math.max(0, Math.min(y, rect.height - option.height));

      if (this.isOverlapping(x, y, option.width, option.height)) return;

      const clone = { ...option, x, y };
      this.droppedOptions.push(clone);
      this.options = this.options.filter(o => o.id !== id);
    },
    removeItem(item) {
      this.droppedOptions = this.droppedOptions.filter(i => i.id !== item.id);
      const original = this.options.find(o => o.id === item.id);
      const defaultSize = { width: 120, height: 60 };
      this.options.push({
        id: item.id,
        name: item.name,
        width: original ? original.width : defaultSize.width,
        height: original ? original.height : defaultSize.height
      });
    },
    startResizing(item, event, type) {
      this.resizingItem = item;
      this.resizingType = type;
      this.startMouseX = event.clientX;
      this.startMouseY = event.clientY;
      this.startWidth = item.width;
      this.startHeight = item.height;
      window.addEventListener('mousemove', this.resize);
      window.addEventListener('mouseup', this.stopResizing);
    },
    resize(event) {
      if (!this.resizingItem) return;
      const dx = event.clientX - this.startMouseX;
      const dy = event.clientY - this.startMouseY;
      const dropZone = this.$refs.dropZone.getBoundingClientRect();

      let newWidth = this.startWidth;
      let newHeight = this.startHeight;

      if (this.resizingType === 'width' || this.resizingType === 'both') {
        newWidth = Math.max(60, this.startWidth + dx);
        if (this.resizingItem.x + newWidth > dropZone.width) {
          newWidth = dropZone.width - this.resizingItem.x;
        }
      }

      if (this.resizingType === 'height' || this.resizingType === 'both') {
        newHeight = Math.max(30, this.startHeight + dy);
        if (this.resizingItem.y + newHeight > dropZone.height) {
          newHeight = dropZone.height - this.resizingItem.y;
        }
      }

      if (!this.isOverlapping(
        this.resizingItem.x,
        this.resizingItem.y,
        newWidth,
        newHeight,
        this.resizingItem.id
      )) {
        this.resizingItem.width = newWidth;
        this.resizingItem.height = newHeight;
      }
    },
    stopResizing() {
      this.resizingItem = null;
      window.removeEventListener('mousemove', this.resize);
      window.removeEventListener('mouseup', this.stopResizing);
    },
    startDrag(item, event) {
      const offsetX = event.offsetX;
      const offsetY = event.offsetY;

      const onMouseMove = e => {
        const rect = this.$refs.dropZone.getBoundingClientRect();
        let newX = e.clientX - rect.left - offsetX;
        let newY = e.clientY - rect.top - offsetY;

        newX = Math.max(0, Math.min(newX, rect.width - item.width));
        newY = Math.max(0, Math.min(newY, rect.height - item.height));

        if (!this.isOverlapping(newX, newY, item.width, item.height, item.id)) {
          item.x = newX;
          item.y = newY;
        }
      };

      const onMouseUp = () => {
        window.removeEventListener('mousemove', onMouseMove);
        window.removeEventListener('mouseup', onMouseUp);
      };

      window.addEventListener('mousemove', onMouseMove);
      window.addEventListener('mouseup', onMouseUp);
    },
    submitLayout() {
      const layout = this.droppedOptions.map(item => ({
        name: item.name,
        x: item.x,
        y: item.y,
        width: item.width,
        height: item.height
      }));
      console.log('Submitted Layout:', layout);
      this.submittedJson = JSON.stringify(layout, null, 2);
    }
  }
};
</script>

<style scoped>
.app {
  font-family: Arial, sans-serif;
}
.header {
  background: #42b983;
  color: white;
  text-align: center;
  padding: 10px;
  font-size: 24px;
}
.container {
  display: flex;
  height: 90vh;
}
.left-pane.drop-zone {
  flex: 1;
  position: relative;
  background-color: #ffffff;
  padding: 20px;
  margin: 20px;
  border-radius: 12px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  display: flex;
  flex-direction: column;
}
.right-pane.options-pane {
  width: 33%;
  background-color: #f0f0f0;
  padding: 10px;
}
.pane-title {
  margin: 0 0 10px;
  font-size: 20px;
  color: white;
  text-align: center;
  background-color: #79d8ad; /* green highlight */
  padding: 8px;
  border-radius: 4px;
  font-weight: bold;
}
.option-header {
  text-align: center;
  font-weight: bold;
  background-color: #bbdefb;
  padding: 5px;
  border-bottom: 1px solid #90caf9;
}
.draggable-item {
  background-color: #e3f2fd;
  border: 1px solid #90caf9;
  margin: 10px 0;
  padding: 10px;
  cursor: grab;
  box-sizing: border-box;
  user-select: none;
}
.dropped-item {
  position: absolute;
  background-color: #e3f2fd;
  border: 1px solid #90caf9;
  padding: 10px;
  box-sizing: border-box;
  cursor: move;
}
 .remove-btn {
  position: absolute;
  top: 5px;
  right: 5px;
  cursor: pointer;
  font-weight: bold;
  font-size: 16px;
  color: red; 
} 
.remove-btn:hover {
  color: darkred;
}

.resize-handle.vertical {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 15px;
  height: 5px;
  cursor: ns-resize;
  background-color: #2196f3;
}
.resize-handle.horizontal {
  position: absolute;
  top: 0;
  bottom: 15px;
  right: 0;
  width: 5px;
  cursor: ew-resize;
  background-color: #2196f3;
}
.resize-handle.corner {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 12px;
  height: 12px;
  background: #2196f3;
  cursor: nwse-resize;
}
.submit-btn {
  position: absolute;
  bottom: 10px;
  right: 10px;
  padding: 12px 20px;
  font-size: 16px;
  background: #27ae60;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
.json-box {
  position: absolute;
  bottom: 50px;
  right: 10px;
  width: 250px;
  max-height: 200px;
  overflow: auto;
  background: #f9f9f9;
  border: 1px solid #ccc;
  padding: 8px;
  font-size: 12px;
  white-space: pre-wrap;
}
</style>
