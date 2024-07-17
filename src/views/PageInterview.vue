<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'
import { getFirestore, doc, getDoc, updateDoc, Timestamp } from 'firebase/firestore'
import { useUserStore } from '@/stores/user'
import type { IInterview, IStage } from '@/interfaces'

const db = getFirestore()
const userStore = useUserStore()
const route = useRoute()

const isLoading = ref<boolean>(true)
const interview = ref<IInterview>()

const docRef = doc(db, `users/${userStore.userId}/interviews`, route.params.id as string)

const getData = async (): Promise<void> => {
  isLoading.value = true
  const docSnap = await getDoc(docRef)
  if (docSnap.exists()) {
    const data = docSnap.data() as IInterview
    if (data.stages && data.stages.length) {
      data.stages = data.stages.map((stage: IStage) => {
        if (stage.date && stage.date instanceof Timestamp) {
          return { ...stage, date: stage.date.toDate() }
        } else {
          return stage
        }
      })
    }
    interview.value = data
  }
  console.log('interview => ', interview.value)
  isLoading.value = false
}

onMounted(async () => {
  await getData()
})

const addStage = () => {
  if (interview.value) {
    if (!interview.value.stages) {
      interview.value.stages = []
    }
    interview.value.stages.push({ name: '', date: null, description: '' })
  }
}

const removeStage = (index: number) => {
  if (interview.value) {
    interview.value.stages?.splice(index, 1)
  }
}

const saveInterview = async (): Promise<void> => {
  isLoading.value = true

  await updateDoc(docRef, { ...interview.value })
  await getData()
  isLoading.value = false
}
</script>

<template>
  <app-progress v-if="isLoading" />
  <div v-else-if="interview?.id && !isLoading" class="content-interview">
    <app-card>
      <template #title>Собеседование в компанию {{ interview.company }} </template>\
      <template #content>
        <div class="flex flex-column gap-2">
          <label for="company">Компания</label>
          <app-input-text class="input mb-3" id="company" v-model="interview.company" />
        </div>
        <div class="flex flex-column gap-2">
          <label for="vacancyLink">Описание вакансии (ссылка)</label>
          <app-input-text class="input mb-3" id="vacancyLink" v-model="interview.vacancyLink" />
        </div>
        <div class="flex flex-column gap-2">
          <label for="hrName">Контакт (имя)</label>
          <app-input-text class="input mb-3" id="hrName" v-model="interview.hrName" />
        </div>
        <div class="flex flex-column gap-2">
          <label for="contactTelegram">Telegram HR</label>
          <app-input-text
            class="input mb-3"
            id="contactTelegram"
            v-model="interview.contactTelegram"
          />
        </div>
        <div class="flex flex-column gap-2">
          <label for="contactWhatsApp">WhatsApp HR</label>
          <app-input-text
            class="input mb-3"
            id="contactWhatsApp"
            v-model="interview.contactWhatsApp"
          />
        </div>
        <div class="flex flex-column gap-2">
          <label for="contactPhone">Phone HR</label>
          <app-input-text class="input mb-3" id="contactPhone" v-model="interview.contactPhone" />
        </div>
        <div class="flex flex-wrap gap-3 p-fluid mb-3">
          <div class="flex-auto">
            <app-input-number
              inputId="salaryFrom"
              mode="currency"
              currency="RUB"
              locale="ru-RU"
              placeholder="Зарплата от"
              v-model="interview.salaryFrom"
            ></app-input-number>
          </div>
          <div class="flex-auto">
            <app-input-number
              inputId="salaryTo"
              mode="currency"
              currency="RUB"
              locale="ru-RU"
              placeholder="Зарплата до"
              v-model="interview.salaryTo"
            ></app-input-number>
          </div>
        </div>

        <template v-if="interview.stages">
          <div class="interview-stage" v-for="(stage, index) in interview.stages" :key="index">
            <div class="flex flex-column gap-2">
              <label :for="`stage-name-${index}`">Название этапа</label>
              <app-input-text class="input mb-3" :id="`stage-name-${index}`" v-model="stage.name" />
            </div>
            <div class="flex flex-column gap-2">
              <label :for="`stage-date-${index}`">Дата прохождения этапа</label>
              <app-calendar
                class="input mb-3"
                :inputId="`stage-date-${index}`"
                date-format="dd.mm.yy"
                v-model="stage.date"
              />
            </div>
            <div class="flex flex-column gap-2">
              <label :for="`stage-description-${index}`">Комментарии</label>
              <app-textarea
                :id="`stage-description-${index}`"
                class="input mb-3"
                rows="5"
                autoResize
                v-model="stage.description"
              />
            </div>
            <app-button
              severity="danger"
              label="Удалить этап"
              icon="pi pi-trash"
              class="mb-3"
              @click="removeStage(index)"
            />
          </div>
        </template>

        <app-button
          label="Добавить этап"
          severity="info"
          icon="pi pi-plus"
          class="mb-3"
          @click="addStage"
        />

        <div class="flex flex-wrap gap-3 mb-3">
          <div class="flex align-items-center">
            <app-radio-button
              inputId="interviewResult1"
              name="result"
              value="Refusal"
              v-model="interview.result"
            />
            <label for="interviewResult1" class="ml-2">Отказ</label>
          </div>
          <div class="flex align-items-center">
            <app-radio-button
              inputId="interviewResult2"
              name="result"
              value="Offer"
              v-model="interview.result"
            />
            <label for="interviewResult2" class="ml-2">Оффер</label>
          </div>
        </div>
        <app-button
          label="Сохранить"
          severity="info"
          icon="pi pi-save"
          class="mt-3"
          @click="saveInterview"
        />
      </template>
    </app-card>
  </div>
</template>

<style scoped>
.content-interview {
  max-width: 600px;
  margin: auto;
}
.input {
  width: 100%;
}
.interview-stage {
  border: 1px solid #ccc;
  border-radius: 6px;
  padding: 10px;
  margin-bottom: 10px;
}
</style>
