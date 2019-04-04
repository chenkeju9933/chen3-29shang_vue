<template>
  <div class="todo-container">
    <div class="todo-wrap">
      <!-- <TodoHeader @addTodo="addTodo"/> 给todoHeader 标签对象绑定addTodo 事件监听 -->
      <TodoHeader ref="header"/>
      <TodoList :todos="todos"/>
      <!-- <Todo-footer
        :todos="todos"
        :deleteCompleteTodos="deleteCompleteTodos"
        :selectAllTodos="selectAllTodos"
      /> -->
      <todo-footer>
        <input type="checkbox" v-model="isAllCheck" slot="checkAll">
        <span slot="count">已完成{{completeSize}}/全部{{todos.length}}</span>
      
        <button slot="delete" class="btn btn-danger" v-show="completeSize>0"
      @click="deleteCompleteTodos()">清除已完成任务</button>
      </todo-footer>
    </div>
  </div>
</template>
<!--
绑定事件监听
触发事件

订阅消息
发布消息
 -->

<script>
import PubSub from 'pubsub-js'
import TodoHeader from "./components/TodoHeader.vue";
import TodoList from "./components/TodoList.vue";
import TodoFooter from "./components/TodoFooter.vue";
import storageUtil from './util/storageUtil.js'
export default {
  data() {
    return {
      todos: //JSON.parse(window.localStorage.getItem('todos_key') || '[]')
      storageUtil.readTodos()
    };
  },
  computed: {
    completeSize() {
      return this.todos.reduce(
        (preTotal, todo) => preTotal + (todo.complete ? 1 : 0),0
      );
    },
    isAllCheck: {
      get() {
        return this.completeSize === this.todos.length && this.completeSize>0
      },
      set(value) {
        // value 是当前checkbox的最新的值
        this.selectAllTodos(value);
      }
    }
  },
  mounted () { //执行异步代码
    // 给<TodoHeader/> 绑定addTodo事件监听
    // this.$on('addTodo',this.addTodo) 给APP绑定监听不对
    this.$refs.header.$on('addTodo',this.addTodo)
    // 订阅消息
    PubSub.subscribe('deleteTodo',(msg,index) => {
      this.deleteTodo(index)
    })
  },
  methods: {
    addTodo(todo) {
      this.todos.unshift(todo);
    },
    deleteTodo(index) {
      this.todos.splice(index, 1);
    },
    // 删除所有选中的todo
    deleteCompleteTodos() {
      this.todos = this.todos.filter(todo => !todo.complete);
    },
    // 全选 全不选
    selectAllTodos(check) {
      this.todos.forEach(todo => (todo.complete = check));
    }
  },
  watch: {
    todos: {
      deep: true, // 深度监视
      /*handler: function(value) {
        // 将todos 最新的值的json数据 保存到localStorage
        //window.localStorage.setItem('todos_key',JSON.stringify(todos))
        storageUtil.saveTodos(value)
      }*/
      handler: storageUtil.saveTodos
      /* handler: function(todos) {
        window.localStorage.setItem(TODOS_KEY,JSON.stringify(todos))
      }  看懂上面的写法 至少14k */
    }
  },
  components: {
    TodoHeader,
    TodoList,
    TodoFooter
  }
};
</script>

<style>
.todo-container {
  width: 600px;
  margin: 0 auto;
}
.todo-container .todo-wrap {
  padding: 10px;
  border: solid 1px #ddd;
  border-radius: 5px;
}
</style>
