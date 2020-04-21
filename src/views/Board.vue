<template>
  <div class="board bg-indigo-100 h-full overflow-auto">
    <div class="flex flex-row items-start">
      <div 
        class="column" 
        v-for="(column, $columnIndex) in board.columns" 
        :key="$columnIndex"
        draggable
        @drop="moveTaskOrColumn($event, column.tasks, $columnIndex)"
        @dragover.prevent
        @dragenter.prevent
        @dragstart.self="pickupColumn($event, $columnIndex)">
        <div class="flex items-center mb-2 font-bold">
          {{ column.name }}
        </div>
        <div class="list-reset">
          <div 
            class="task" 
            v-for="(task, $taskIndex) in column.tasks" :key="$taskIndex"
            draggable
            @dragstart="pickupTask($event, $taskIndex, $columnIndex)"
            @click="goToTask(task)"
            @dragover.prevent
            @dragenter.prevent
            @drop.stop="moveTaskOrColumn($event, column.tasks, $columnIndex, $taskIndex)">
            <span class="w-full text-left font-bold">{{ task.name }}</span>
            <p v-if="task.description" class="text-sm mt-1">{{ task.description }}</p>
          </div>
          <input 
            type="text"
            class="block p-2 w-full bg-transparant"
            placeholder="+ Enter new task"
            @keyup.enter="createTask($event, column.tasks)">
        </div>
      </div>
      <div class="column">
        <input 
          type="text" 
          class="p-2 mr-2 flex-grow"
          placeholder="New column name"
          v-model="newColumnName"
          @keyup.enter="createColumn()">
      </div>
    </div>
    <div 
      class="task-bg" 
      v-if="isTaskOpen"
      @click.self="close">
      <router-view/>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex'

export default {
  data(){
    return {
      newColumnName: ''
    }
  },
  computed: {
    ...mapState(['board']),
    isTaskOpen(){
      return this.$route.name === 'task'
    }
  },
  methods: {
    goToTask(task){
      this.$router.push({ name: 'task', params: { id: task.id } })
    },
    close(){
      this.$router.push({ name: 'board' })
    },
    createTask(e, tasks){
      this.$store.commit('CREATE_TASK', {
        tasks,
        name: e.target.value
      })
      e.target.value = ''
    },
    createColumn(){
      this.$store.commit('CREATE_COLUMN', {
        name: this.newColumnName
      })

      this.newColumnName = ''
    },
    pickupTask(e, taskIndex, fromColumnIndex){
      //drag and drop API
      e.dataTransfer.effectAllowed = 'move'
      e.dataTransfer.dropEffect = 'move'

      e.dataTransfer.setData('from-task-index', taskIndex)
      e.dataTransfer.setData('from-column-index', fromColumnIndex)
      e.dataTransfer.setData('type', 'task')
    },
    pickupColumn(e, fromColumnIndex){
      //drag and drop API
      e.dataTransfer.effectAllowed = 'move'
      e.dataTransfer.dropEffect = 'move'

      e.dataTransfer.setData('from-column-index', fromColumnIndex)
      e.dataTransfer.setData('type', 'column')
    },
    moveTaskOrColumn(e, toTasks, toColumnIndex, toTaskIndex){
      const type = e.dataTransfer.getData('type')
      if(type === 'task'){
        this.moveTask(e, toTasks, toTaskIndex !== undefined ? toTaskIndex : toTasks.length)
      } else {
        this.moveColumn(e, toColumnIndex)
      }
    },
    moveTask(e, toTasks, toTaskIndex){
      const fromColumnIndex = e.dataTransfer.getData('from-column-index')
      const fromTasks = this.board.columns[fromColumnIndex].tasks
      const fromTaskIndex = e.dataTransfer.getData('from-task-index')

      this.$store.commit('MOVE_TASK', {
        fromTasks,
        fromTaskIndex,
        toTasks,
        toTaskIndex
      })
    },
    moveColumn(e, toColumnIndex){
      const fromColumnIndex = e.dataTransfer.getData('from-column-index')

      this.$store.commit('MOVE_COLUMN', {
        fromColumnIndex,
        toColumnIndex
      })
    }
  }
}
</script>

<style>
.column{
  @apply bg-gray-100 p-2 mr-4;
  min-width: 350px;
}
.task {
  @apply flex items-center flex-wrap shadow mb-2 py-2 px-2 rounded bg-white no-underline;
}
.task-bg{
  @apply absolute;
  width: 100%;
  height: 100%;
  top: 0;
  background: rgba(0,0,0,0.5);
}
</style>