<template>
  <div class="board bg-indigo-100 h-full overflow-auto">
    <div class="flex flex-row items-start">
      <BoardColumn 
        v-for="(column, $columnIndex) in board.columns" 
        :key="$columnIndex"
        :column="column"
        :columnIndex="$columnIndex"
        :board="board"
      />

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
import BoardColumn from '@/components/BoardColumn'

export default {
  components: {
    BoardColumn
  },
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
    close(){
      this.$router.push({ name: 'board' })
    },
    createColumn(){
      this.$store.commit('CREATE_COLUMN', {
        name: this.newColumnName
      })

      this.newColumnName = ''
    }
  }
}
</script>

<style>
.task-bg{
  @apply absolute;
  width: 100%;
  height: 100%;
  top: 0;
  background: rgba(0,0,0,0.5);
}
</style>