<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { getFirestore, collection, query, orderBy, getDocs } from 'firebase/firestore'
import { useUserStore } from '@/stores/user'
import type { IInterview } from '@/interfaces'

const userStore = useUserStore()
const db = getFirestore()

const interviews = ref<IInterview[]>([])
const chartData = ref()
const chartOptions = ref(null)

const getAllInterviews = async <T extends IInterview>(): Promise<T[]> => {
  let getData
  getData = query(
    collection(db, `users/${userStore.userId}/interviews`),
    orderBy('createdAt', 'desc')
  )
  const listDocs = await getDocs(getData)
  return listDocs.docs.map((doc) => doc.data() as T)
}

onMounted(async () => {
  interviews.value = await getAllInterviews()
  chartData.value = setChartData()
  chartOptions.value = setChartOptions()
})

const setChartData = () => {
  const data: number[] = [0, 0, 0]
  interviews.value.forEach((interview: IInterview) => {
    if (interview.result === 'Offer') {
      data[0]++
    } else if (interview.result === 'Refusal') {
      data[1]++
    } else {
      data[2]++
    }
  })

  return {
    labels: ['Оффер', 'Отказ', 'В процессе'],
    datasets: [
      {
        data,
        backgroundColor: ['tomato', 'cyan', 'gray'],
        hoverBackgroundColor: ['orange', 'blue', 'darkgray']
      }
    ]
  }
}

const setChartOptions = () => {
  const documentStyle = getComputedStyle(document.documentElement)
  const textColor = documentStyle.getPropertyValue('--p-text-color')

  return {
    plugins: {
      legend: {
        labels: {
          cutout: '60%',
          color: textColor
        }
      }
    }
  }
}
</script>

<template>
  <h1>Статистика</h1>
  <div class="card flex justify-center">
    <app-chart
      type="doughnut"
      :data="chartData"
      :options="chartOptions"
      class="donut w-9/12 md:w-[50rem]"
    />
  </div>
</template>
<style>
.donut {
  width: 50%;
}
</style>
