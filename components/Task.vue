<!-- eslint-disable vue/no-v-html -->
<template>
  <div v-if="task">
    <h3 class="mb-4">Tâche {{ idTask }} : {{ task.title }}</h3>

    <ul>
      <li v-for="(description, i) in task.description" :key="i"><span v-html="description"></span></li>
    </ul>

    <img v-if="task.img" :src="`/exercises/${task.img}.png`" alt="" class="task-img" />

    <div v-if="task.css" class="css-part">
      <h4>Partie CSS</h4>
      <ul>
        <li v-for="(description, i) in task.css.description" :key="i"><span v-html="description"></span></li>
      </ul>

      <img v-if="task.css.img" :src="`/exercises/${task.css.img}.png`" alt="" class="task-img" />
    </div>

    <div class="objective">
      <el-tooltip class="item" placement="bottom">
        <div slot="content">
          <ul v-if="task.objectives.length > 0">
            <li v-for="(objective, j) in task.objectives" :key="j">{{ objective }}</li>
          </ul>

          <ul v-if="task.css && task.css.objectives.length > 0">
            <li v-for="(objective, k) in task.css.objectives" :key="k">[CSS] {{ objective }}</li>
          </ul>
        </div>

        <el-button type="info">Objectif de la tâche <i class="el-icon-info"></i></el-button>
      </el-tooltip>
    </div>
  </div>
</template>

<script>
import jsonExercises from '@/assets/exercises.json'

export default {
  props: {
    idExercise: {
      type: Number,
      required: true
    },
    idTask: {
      type: Number,
      required: true
    }
  },
  data() {
    return {
      exercises: jsonExercises,
      task: null
    }
  },
  created() {
    const exercise = this.exercises.find((exercise) => exercise.id === this.idExercise)
    if (exercise) {
      this.task = exercise.tasks[this.idTask - 1]
    }
  }
}
</script>

<style>
.css-part {
  margin: 1rem 0;
}

.objective {
  display: flex;
  justify-content: center;

  margin-top: 1rem;
}

.task-img {
  margin: 1rem 0;
  max-width: 100%;
}
</style>
