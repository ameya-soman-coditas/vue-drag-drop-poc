<template>
  <div id="app">
    <ul class="todo-list">
      <!-- <li
        @dragover.prevent
        @drop="dragFinish(-1, $event)"
        v-if="dragging > -1"
        class="trash-drop todo-item"
        v-bind:class="{drag: isDragging}"
      >Delete</li>-->

      <li>
        <input
          placeholder="Type new task and press enter"
          type="text"
          class="new-todo todo-item"
          v-model="newItem"
          @keyup.enter="addItem"
        >
      </li>

      <li
        class="todo-wrapper"
        v-for="(item, i) in todos"
        v-bind:key="i"
        @dragstart="dragStart(i, $event)"
        @dragover.prevent
        @dragenter="dragEnter"
        @dragleave="dragLeave"
        @dragend="dragEnd"
        @drop="dragFinish(i, $event)"
      >
        <div class="handles" v-bind:id="'handle'+i" @mousedown="addEventListener($event)">
          <i class="fas fa-ellipsis-v"></i>
          <i class="fas fa-ellipsis-v"></i>
        </div>
        <div class="todo-item">
          <input type="checkbox" v-model="item.done">
          <span :class="{done: item.done}">{{ item.title }}</span>
        </div>
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
        content of a page when looking at its layout. `,
        done: false
      },
      {
        title: `Lorem Ipsum is simply dummy text of the printing and typesetting industry. Lorem Ipsum has been the industry's standard dummy text ever since the 1500s, when an unknown printer took a galley of type and scrambled it to make a type specimen book.`,
        done: false
      }
    ];
  },
  methods: {
    addEventListener(event) {
      var todoItems = document.getElementsByClassName("todo-wrapper");
      for (let i = 0; i < todoItems.length; i++) {
        todoItems[i].setAttribute("draggable", true);
      }
    },
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
      draggedElement.style.opacity = "0.4";
      var psuedoEle = document.createElement("div");
      psuedoEle.innerHTML = draggedElement.innerHTML;
      psuedoEle.id = "draggedPsuedoEle";

      const computedStyle = window.getComputedStyle(draggedElement);
      Array.from(computedStyle).forEach(key =>
        psuedoEle.style.setProperty(
          key,
          computedStyle.getPropertyValue(key),
          computedStyle.getPropertyPriority(key)
        )
      );
      console.log(ev);
      psuedoEle.style.position = "absolute";
      psuedoEle.style.top = "0px";
      psuedoEle.style.left = "-100%";

      document.body.appendChild(psuedoEle);

      // hiding default ghost image
      var dragGhost = draggedElement.cloneNode(true);
      dragGhost.classList.add("hidden-drag-ghost");
      dragGhost.id = "dragGhost";
      document.body.appendChild(dragGhost);
      ev.dataTransfer.setDragImage(dragGhost, 0, 0);
    },
    dragEnter(ev) {},
    dragLeave(ev) {},
    dragEnd(ev) {
      this.dragging = -1;
      var todoItems = document.getElementsByClassName("todo-wrapper");
      for (let i = 0; i < todoItems.length; i++) {
        todoItems[i].setAttribute("draggable", false);
      }

      ev.srcElement.style.opacity = "1";

      // remove element
      var psuedoEle = document.getElementById("draggedPsuedoEle");
      var dragghost = document.getElementById("dragGhost");

      if (psuedoEle) {
        psuedoEle.parentNode.removeChild(psuedoEle);
      }
      if (dragghost) {
        dragghost.parentNode.removeChild(dragghost);
      }
    },
    dragFinish(to, ev) {
      this.moveItem(this.dragging, to);
      this.todos[to]["dragging"] = false;
      ev.target.style.marginTop = "2px";
      ev.target.style.marginBottom = "2px";

      ev.srcElement.style.opacity = "1";

      // remove element
      var psuedoEle = document.getElementById("draggedPsuedoEle");
      if (psuedoEle) {
        psuedoEle.parentNode.removeChild(psuedoEle);
      }

      var dragghost = document.getElementById("dragGhost");
      if (dragghost) {
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
      var psuedoEle = document.getElementById("draggedPsuedoEle");

      if (psuedoEle) {
        var xPos = event.clientX;
        var yPos = event.clientY;

        psuedoEle.style.position = "absolute";
        psuedoEle.style.left = xPos - 10 + "px";
        psuedoEle.style.top = yPos + 2 + "px";
        psuedoEle.style.opacity = "1";
      }
    }
  },
  computed: {
    isDragging() {
      return this.dragging > -1;
    }
  },
  mounted() {
    window.addEventListener("drag", this.onDrag);
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

.todo-wrapper {
  display: flex;
  padding: 8px 0;
}

.handles {
  display: flex;
  padding: 4px;
}

.todo-list {
  list-style-type: none;
  padding: 10px 10px 40px 10px;
  border: 1px solid rgba(2, 2, 2, 0.1);
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
  width: 100%;
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
  opacity: 0.3 !important;
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
</style>
