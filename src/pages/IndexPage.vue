<template>
  <q-page class="q-pa-lg">

    <!-- Список вопросов -->
    <div v-for="(question, i) in questions" :key="i" class="q-mb-lg q-pa-sm" style="border: 1px solid #010101;">
      <p><b>Вопрос №{{  i + 1 }}</b></p>
      <p>{{ question.text }}</p>

      <!-- варианты ответов -->
      <div v-if="['multipleChoice', 'booleanQuestion'].includes(question.typeQuestions)">
        <div v-for="(answer, i) in question.answerOptions" :key="i" class="q-mb-lg">
          <q-radio 
            v-if="question.typeQuestions === 'booleanQuestion'" 
            v-model="question.userAnswer" 
            :val="answer" 
            :label="answer.text" 
          />
          <q-checkbox v-else v-model="question.userAnswer" :val="answer" :label="answer.text" />
        </div>
      </div>
      <div v-else>
        <q-input placeholder="Введите ответ" label="Ваш ответ" outlined class="q-mb-lg" />
      </div>
    </div>

    <q-btn  
      color="primary"
      label="+ Добавить вопрос" 
      class="text-lowercase"
      @click="openFormForAddQuestion = true"
    />



    <q-dialog v-model="openFormForAddQuestion" persistent>
      <q-card style="width: 700px; max-width: 80vw;">
        <q-card-section class="column">
          <q-item-label class="q-mb-sm">Текст вопроса:</q-item-label>
          <q-input v-model="question.text" placeholder="Введите текст" outlined class="q-mb-lg" />

          <q-item-label>Тип ответа:</q-item-label>
          <q-option-group
            :options="typeQuestions"
            type="radio"
            v-model="question.typeQuestions"
            label="Тип вопроса"
            class="q-mb-lg"
          />

          <!-- добавление вариантов ответов -->
          <div v-if="['multipleChoice', 'booleanQuestion'].includes(question.typeQuestions)">
            <div v-for="(answer, i) in question.answerOptions" :key="i" class="q-mb-lg">
              <div class="flex row items-center justify-between">
                <p class="">Ответ {{ i+1 }}</p>
                <q-icon 
                  v-if="question.typeQuestions === 'multipleChoice'"
                  name="img:https://img.icons8.com/?size=100&id=11705&format=png&color=000000" 
                  @click="removeAnswer(i)"
                />
              </div>
              <div class="flex row justify-between">
                <q-input 
                  v-model="answer.text" 
                  label="Текст ответа" 
                  outlined 
                  :readonly="question.typeQuestions === 'booleanQuestion'"
                  style="width: 450px; max-width: 43vw;"
                />
                <q-input type="number" v-model="answer.score" label="Оценка ответа" outlined />
              </div>
            </div>
            <q-btn 
              v-if="question.typeQuestions === 'multipleChoice'"
              flat 
              label="+ Добавить ответ" 
              class="float-right q-pa-none q-mt-none text-lowercase" 
              :disable="disableForAddAnswerBtn"
              @click="addAnswer"
            />
          </div>
        </q-card-section>

        <q-card-actions align="right">
          <q-btn flat label="Отмена" color="primary" @click="closeForm" />
          <q-btn label="Сохранить" color="primary" @click="saveQuestion" />
        </q-card-actions>
      </q-card>
    </q-dialog>

  </q-page>
</template>

<script setup>
import { ref, computed, watch } from 'vue'

defineOptions({
  name: 'IndexPage'
});

class Answer {
  constructor(text = null, score = null) {
    this.text = text;
    this.score = score;
  }
}

class Question {
  constructor(text = null, type = null, answerOptions, userAnswer = []) {
    this.text = text;
    this.typeQuestions = type;
    this.answerOptions = answerOptions || [];
    this.userAnswer = userAnswer
  }
}

const openFormForAddQuestion = ref(false)

const typeQuestions = [
  { label: 'открытый', value: 'openQuestion' },
  { label: 'множественный выбор', value: 'multipleChoice' },
  { label: 'верно/неверно', value: 'booleanQuestion' }
]

const questions = ref([])

const question = ref(new Question())

function saveQuestion() {
  questions.value.push(question.value)
  closeForm()
}

function closeForm() {
  openFormForAddQuestion.value = false
  question.value = new Question()
}

const disableForAddAnswerBtn = computed(() => {
  return question.value.answerOptions.length && hasEmptyFields(question.value.answerOptions)
})

function addAnswer() {
  question.value.answerOptions.push(new Answer())
}

function removeAnswer(index) {
  removeObjectByIndex(question.value.answerOptions, index)
}

function hasEmptyFields(array) {
  return array.some(obj => Object.values(obj).some(value => value === null || value === ""))
}

function removeObjectByIndex(array, index) {
  if (index >= 0 && index < array.length) {
    array.splice(index, 1);
  } 
}

watch(() => question.value.typeQuestions, (newValue, oldValue) => {
  question.value.answerOptions = []
  if(newValue === 'multipleChoice') {
    question.value.answerOptions.push(new Answer())
  }
  if(newValue === 'booleanQuestion') {
    question.value.answerOptions.push(new Answer('Верно'))
    question.value.answerOptions.push(new Answer('Неверно'))
  }
})
</script>
