<template>
<section>
    <section class="todoapp" >
			<header class="header">
				<h1>todos</h1>
				<input class="new-todo" autofocus autocomplete="off" 
            placeholder="What needs to be done?" 
            v-model="newTodo"
            @keyup.enter="addTodo"
        >
			</header>
      <section class="main">
        <input id="toggle-all" class="toggle-all" type="checkbox" v-model="checkAll">
        <label for="toggle-all">Mark all as complete</label>
        <ul class="todo-list">
          <li class="todo" 
              :class="{ completed: item.completed, editing: currentId === item.id }"
              v-for="item in recordShow" :key="item.id"
          >
            <div class="view">
							<input class="toggle" type="checkbox" v-model="item.completed">
							<label @dblclick="editTodo(item)">{{item.record}}</label>
							<button class="destroy" @click="deleteRec(item.id)"></button>
            </div>    
            <input  class="edit" type="text" 
                    v-model="currentRecord" 
                    @blur="doneEdit(item)" 
                    @keydown.enter="doneEdit(item)" 
                    @keydown.esc="cancelEdit()"
              > 
          </li>
        </ul>
      </section>
      <footer class="footer" v-if="list.length > 0">
				<span class="todo-count">
					{{itemLeft}} left
				</span>
				<ul class="filters" @click.capture="showRecord($event)">
					<li><a :class="{selected: visibility === 'all'}">All</a></li>
					<li><a :class="{selected: visibility === 'Active'}">Active</a></li>
					<li><a :class="{selected: visibility === 'Completed'}">Completed</a></li>
				</ul>
				<button class="clear-completed"
            v-show="completedItems > 0"
            @click="clearCompleted"
        >
					Clear completed
				</button>
			</footer>
      
		</section>
    <footer class="info">
			<p>Double-click to edit a todo</p>
			<p>Learn more <a href="http://todomvc.com">TodoMVC</a></p>
		</footer>
    </section>

</template>

<script>
import { reactive, toRefs, computed, watch } from 'vue'
export default {
  setup () {
    let state = reactive({
      // 新增的todo
      newTodo: '',
      // 正在修改的todo
      currentId: null,
      currentRecord: '',
      // 展示的列表类型
      visibility: 'All',
      // 任务列表
      list: JSON.parse(localStorage.getItem('todo-vuejs'))|| []
    })

    const completedData = reactive({
      recordShow: computed(() => {
        if (state.visibility === 'All') {
          return state.list
        } else if (state.visibility === 'Active') {
          return state.list.filter((item) => !item.completed)
        } else {
          return state.list.filter((item) => item.completed)
        }
      }),
      itemLeft: computed(() => {
      /*let num = 0
      for (let item of state.list) {
        if (item.completed === false) num++
      }
      return num
      */
     return state.list.filter((item) => !item.completed).length
    }),
    completedItems: computed(() => {
      return state.list.filter((item) => item.completed).length
    }),
    // 通过控制checkAll 实现全选和反选
    checkAll: computed({
      get: () => {
        return state.list.every(item => item.completed)
      },
      set: (value) => {
        state.list.forEach(item => { item.completed = value })
      }
    })

    })

    const showRecord = (e) => {
      // console.log(e.explicitOriginalTarget.innerText)
      const value = e.explicitOriginalTarget.innerText
      state.visibility = value
    }

    const addTodo = () => {
      if (state.newTodo.length <= 0) return
      // console.log(state.newTodo)
      const newItem = {
        id: Date.now(),
        record: state.newTodo,
        completed: false
      }
      state.list.unshift(newItem)
      state.newTodo = ''
    }

    const deleteRec = (id) => {
      /* 也可以这样实现 state.list = state.list.filter(item => item.id !== id)
         但是直接修改引用，所以导致watch无法监控
      */
      for (let i = 0; i < state.list.length; i++) {
        if (state.list[i].id === id) { 
          state.list.splice(i, 1)
          break
        }
      }
    }

    const editTodo = (item) => {
      state.currentId = item.id
      state.currentRecord = item.record
    }

    const doneEdit = (item) => {
      if (state.currentRecord.length > 0) {
        item.record = state.currentRecord
      }
      cancelEdit()
    }

    const cancelEdit = () => {
      state.currentId = null
      state.currentRecord = ''
    }

    const clearCompleted = () => {
      /* 直接更改了state.list 的引用，所以watch拿不到，不能在更改之后存储
      state.list = state.list.filter((item) => !item.completed)
      */
     for (let i = 0; i < state.list.length; i++) {
       if (state.list[i].completed === true) state.list.splice(i, 1)
     }
    }

    const saveToLocal = () => {
      localStorage.setItem('todo-vuejs', JSON.stringify(state.list))
    }

    watch(state.list, () => {
      console.log('change')
      saveToLocal()
    },{
      // lazy 不在初始化时执行watch中的函数
      lazy: true,
      // deep 深度监听对象变化, 但是对于
      deep: true,
    })

    return {
      ...toRefs(state),
      ...toRefs(completedData),
      showRecord,
      deleteRec,
      addTodo,
      editTodo,
      doneEdit,
      cancelEdit,
      clearCompleted
    }
  }

}
</script>
