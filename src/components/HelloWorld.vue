<template>
  <div id="app">
    <ul class="todo-list">
      <li
        @dragover.prevent
        @drop="dragFinish(-1, $event)"
        v-if="dragging > -1"
        class="trash-drop todo-item"
        v-bind:class="{drag: isDragging}"
      >Delete</li>

      <li v-else>
        <input
          placeholder="Type new task and press enter"
          type="text"
          class="new-todo todo-item"
          v-model="newItem"
          @keyup.enter="addItem"
        >
      </li>

      <li
        class="todo-item"
        v-for="(item, i) in todos"
        v-bind:key="i"
        draggable="true"
        @dragstart="dragStart(i, $event)"
        @dragover.prevent
        @dragenter="dragEnter"
        @dragleave="dragLeave"
        @dragend="dragEnd"
        @drop="dragFinish(i, $event)"
        v-bind:class="{ active: item['dragging'] }"
      >
        <input type="checkbox" v-model="item.done">
        <span :class="{done: item.done}">{{ item.title }}</span>
        <span class="remove-item" @click="removeItem(item)">x</span>
      </li>
    </ul>
  </div>
</template>

<script>
const TODO_STORAGE_KEY = "todostorage";

let todoStorage = {
  fetch: () => JSON.parse(localStorage.getItem(TODO_STORAGE_KEY) || "[]"),
  save: todos => localStorage.setItem(TODO_STORAGE_KEY, JSON.stringify(todos))
};
export default {
  name: "HelloWorld",
  props: {
    msg: String
  },
  data: () => {
    return {
      todos: todoStorage.fetch(),
      newItem: "",
      dragging: -1,
      elementGettingDragged: null
    };
  },
  created: function() {
    // `this` points to the vm instance
    this.todos = [
      {
        title: `It is a long established fact that a reader will be distracted by the readable 
        content of a page when looking at its layout. The point of using Lorem Ipsum is that it 
        has a more-or-less normal distribution of letters, as opposed to using 'Content here, 
        content here', making it look like readable English.`,
        done: false
      },
      {
        title: `Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.`,
        done: false
      }
    ];
  },
  methods: {
    addItem() {
      if (!this.newItem) {
        return;
      }
      this.todos.push({
        title: this.newItem,
        done: false
      });
      this.newItem = "";
    },
    removeItem(item) {
      this.todos.splice(this.todos.indexOf(item), 1);
    },
    removeItemAt(index) {
      this.todos.splice(index, 1);
    },
    dragStart(which, ev) {
      ev.dataTransfer.setData("Text", this.id);
      ev.dataTransfer.dropEffect = "move";
      this.dragging = which;
      this.todos[which]["dragging"] = true;
      
      var draggedElement = ev.srcElement;
      var psuedoEle = document.createElement('div');
      psuedoEle.innerHTML = draggedElement.innerHTML;
      psuedoEle.id = 'draggedPsuedoEle';    

      const computedStyle = window.getComputedStyle(draggedElement)
      Array.from(computedStyle).forEach(key => psuedoEle.style.setProperty(key, computedStyle.getPropertyValue(key), computedStyle.getPropertyPriority(key)));

      document.body.appendChild(psuedoEle);

      // hiding default ghost image
      var dragGhost = draggedElement.cloneNode(true);
      dragGhost.classList.add('hidden-drag-ghost');
      dragGhost.id = 'dragGhost';
      document.body.appendChild(dragGhost);
      ev.dataTransfer.setDragImage(dragGhost, 0, 0);

      // this.elementGettingDragged = JSON.parse(
      //   JSON.stringify(this.todos[which])
      // );
      // this.todos.splice(which, 0, this.elementGettingDragged);
    },
    dragEnter(ev) {
      var draggEnterElement = ev.srcElement;
      draggEnterElement.classList.add('border-bottom');
      
      /* 
      if (ev.clientY > ev.target.height / 2) {
        ev.target.style.marginBottom = '10px'
      } else {
        ev.target.style.marginTop = '10px'
      }
      */
    },
    dragLeave(ev) {
      var draggEnterElementTarget = ev.target;
      draggEnterElementTarget.classList.remove('border-bottom');
      var draggEnterElement = ev.srcElement;
      draggEnterElement.classList.remove('border-bottom');
      /* 
      ev.target.style.marginTop = '2px'
      ev.target.style.marginBottom = '2px'
      */
    },
    dragEnd(ev) {
      //this.todos.splice(this.dragging, 1);
      this.dragging = -1;
      
      var draggEnterElementTarget = ev.target;
      draggEnterElementTarget.classList.remove('border-bottom');
      var draggEnterElement = ev.srcElement;
      draggEnterElement.classList.remove('border-bottom');
      

      // remove element
      var psuedoEle = document.getElementById('draggedPsuedoEle');
      var dragghost = document.getElementById('dragGhost');
    
      if(psuedoEle) {
        psuedoEle.parentNode.removeChild(psuedoEle);
      }
      if(dragghost) {
        dragghost.parentNode.removeChild(dragghost);
      }
    },
    dragFinish(to, ev) {
      this.moveItem(this.dragging, to);
      this.todos[to]["dragging"] = false;
      ev.target.style.marginTop = "2px";
      ev.target.style.marginBottom = "2px";

      // remove element
      var psuedoEle = document.getElementById('draggedPsuedoEle');
      if(psuedoEle) {
        psuedoEle.parentNode.removeChild(psuedoEle);
      }

      var dragghost = document.getElementById('dragGhost');
      if(dragghost) {
        dragghost.parentNode.removeChild(dragghost);
      }
    },
    moveItem(from, to) {
      if (to === -1) {
        this.removeItemAt(from);
      } else {
        this.todos.splice(to, 0, this.todos.splice(from, 1)[0]);
      }
    },
    onDrag(event) {
      var psuedoEle = document.getElementById('draggedPsuedoEle');

      if(psuedoEle) {
        var xPos = event.clientX;
        var yPos = event.clientY;

        psuedoEle.style.position = 'absolute';
        psuedoEle.style.left = (xPos - psuedoEle.clientWidth / 2) + 'px';
        psuedoEle.style.top = (yPos + 10 ) + 'px';
        psuedoEle.style.opacity = '1';			      
      }
    }
  },
  computed: {
    isDragging() {
      return this.dragging > -1;
    }
  },
  mounted() {
    window.addEventListener('drag', this.onDrag);
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
body {
  font-family: "Source Sans Pro", "Arial", sans-serif;
}

* {
  box-sizing: border-box;
}

.todo-list {
  list-style-type: none;
  padding: 10px;
}

.done {
  text-decoration: line-through;
  color: #888;
}

.new-todo {
  width: 100%;
}

.trash-drop {

  text-align: center;
  color: #e33;
}

.trash-drop:-moz-drag-over {
  border: 2px solid red;
}

.todo-item {
  border: 1px solid #ccc;
  border-radius: 2px;
  padding: 14px 8px;
  margin-bottom: 3px;
  background-color: #fff;
  box-shadow: 1px 2px 2px #ccc;
  font-size: 22px;
}

.remove-item {
  float: right;
  color: #a45;
  opacity: 0.5;
}

.todo-item:hover .remove-item {
  opacity: 1;
  font-size: 28px;
}

.active {
  opacity: 0.3;
}
.hidden-drag-ghost {
  position: absolute;
  top: 0;
  left: 0;
  opacity: 0.01;
  visibility: hidden;
  overflow: hidden;
  border: none;
  box-shadow: none;
  outline: none;
}
li.border-bottom {
  border: 4px solid red;
}
</style>
