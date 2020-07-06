<template>
  <v-container class="container mt-10">
    <v-text-field
      v-model="task"
      label="What are you working on?"
      solo
      @keypress.enter="create"
    >
      <template v-slot:append>
        <v-icon class="ml-2" color="grey" @click="task = ''">mdi-close</v-icon>
      </template>
    </v-text-field>

    <h2 class="display-1 success--text pl-4">
      Tasks:
      <v-fade-transition leave-absolute>
        <span :key="`tasks-${tasks.length}`">
          {{ tasks.length }}
        </span>
      </v-fade-transition>
    </h2>

    <v-divider class="mt-4"></v-divider>

    <v-row class="my-1" align="center">
      <strong class="mx-4 info--text text--darken-2">
        Remaining: {{ remainingTasks }}
      </strong>
      <v-spacer></v-spacer>

      <v-divider vertical></v-divider>

      <v-spacer></v-spacer>
      <strong class="mx-4 success--text text--darken-2">
        Completed: {{ completedTasks }}
      </strong>
      <v-progress-circular
        :rotate="270"
        :value="progress"
        class="mx-4"
      ></v-progress-circular>
      <v-spacer></v-spacer>

      <v-divider vertical></v-divider>

      <v-spacer></v-spacer>
      <v-btn v-if="!isSwap" icon class="mx-4" @click="startSwapTasks">
        <v-icon>mdi-swap-vertical</v-icon>
      </v-btn>
      <v-btn v-else icon class="mx-4" @click="endSwapTasks">
        <v-icon color="green">mdi-check-bold</v-icon>
      </v-btn>
      <v-spacer></v-spacer>
    </v-row>

    <v-divider class="mb-4"></v-divider>

    <v-row v-if="tasks.length > 0">
      <draggable v-if="isSwap" v-model="editTasks" class="fill-width">
        <v-col
          v-for="(task, i) in editTasks"
          :key="`${i}-${task.text}`"
          cols="12"
          class="py-0"
        >
          <v-card outlined class="d-flex px-4 task-list">
            <v-icon class="swap-icon">mdi-drag-vertical</v-icon>
            <div
              :class="(task.done && 'grey--text') || 'primary--text'"
              class="ml-4"
              v-text="task.text"
            ></div>
          </v-card>
        </v-col>
      </draggable>
      <v-col
        v-for="(task, i) in tasks"
        v-else
        :key="`${i}-${task.text}`"
        cols="12"
        class="py-0"
      >
        <v-card outlined class="d-flex px-4 task-list">
          <v-checkbox
            v-model="task.done"
            hide-details
            :color="(task.done && 'grey') || 'primary'"
            class="mt-0 pt-0"
          >
            <template v-if="i !== editTargetIndex" v-slot:label>
              <div
                :class="(task.done && 'grey--text') || 'primary--text'"
                class="ml-4"
                v-text="task.text"
              ></div>
            </template>
          </v-checkbox>

          <v-text-field
            v-if="i === editTargetIndex"
            v-model="editText"
            outlined
            dense
            hide-details
            class="ml-2 mr-1"
          >
            <template v-slot:append>
              <v-icon color="green" @click="saveText(task)"
                >mdi-check-bold</v-icon
              >
              <v-icon color="grey" class="ml-2" @click="endEditText"
                >mdi-close</v-icon
              >
            </template>
          </v-text-field>

          <template v-else>
            <v-spacer></v-spacer>
            <v-btn icon @click="startEditText(i, task.text)">
              <v-icon>mdi-pencil</v-icon>
            </v-btn>
          </template>

          <v-btn icon @click="deleteTask(i)">
            <v-icon>mdi-trash-can-outline</v-icon>
          </v-btn>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'
import draggable from 'vuedraggable'

export type ITask = {
  id: number
  done: boolean
  text: string
  order: number
}

@Component({
  components: {
    draggable,
  },
})
export default class TodoList extends Vue {
  tasks: ITask[] = [
    {
      id: 1,
      done: false,
      text: 'Foobar',
      order: 1,
    },
    {
      id: 2,
      done: false,
      text: 'Fizzbuzz',
      order: 2,
    },
    {
      id: 3,
      done: true,
      text: 'bubble sort',
      order: 3,
    },
  ]

  editTasks: object[] = []
  task: string = ''
  isSwap: boolean = false
  isEdit: boolean = false
  editText: string = ''
  editTargetIndex: number | null = null

  get completedTasks(): number {
    return this.tasks.filter((task) => task.done).length
  }

  get progress(): number {
    return (this.completedTasks / this.tasks.length) * 100
  }

  get remainingTasks(): number {
    return this.tasks.length - this.completedTasks
  }

  create() {
    this.tasks.push({
      id: this.tasks.length + 1,
      done: false,
      text: this.task,
      order: this.tasks.length + 1,
    })
    this.task = ''
  }

  startSwapTasks() {
    this.editTasks = JSON.parse(JSON.stringify(this.tasks))
    this.isSwap = true
    this.endEditText()
  }

  endSwapTasks() {
    this.tasks = this.editTasks.map((obj, index) => {
      return {
        ...obj,
        order: index + 1,
      }
    })
    this.isSwap = false
    this.editTasks = []
  }

  startEditText(index: number, text: string) {
    this.isEdit = true
    this.editTargetIndex = index
    this.editText = text
  }

  endEditText() {
    this.isEdit = false
    this.editTargetIndex = null
    this.editText = ''
  }

  saveText(task: ITask) {
    task.text = this.editText
    // this.save()
    this.endEditText()
  }

  deleteTask(index: number) {
    this.tasks.splice(index, 1)
    this.endEditText()
  }
}
</script>

<style lang="scss" scoped>
.container {
  max-width: 500px;
}
.fill-width {
  width: 100%;
}
.task-list {
  min-height: 48px;
  align-items: center;
}
.swap-icon {
  height: 32px;
  width: 32px;
}
</style>
