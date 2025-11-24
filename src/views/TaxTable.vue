<script setup lang="ts">
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  PointElement,
  LineElement,
  Title,
  Tooltip,
  Legend,
} from 'chart.js'
import { Line } from 'vue-chartjs'
// 실제 프로젝트 경로에 맞게 수정하세요.
import { tax_table } from '@/constants/tax_table.json'
import { onMounted, ref } from 'vue'

ChartJS.register(CategoryScale, LinearScale, PointElement, LineElement, Title, Tooltip, Legend)

// ✅ 수정된 부분: 숫자(1-11)를 영어 단어로 매핑하는 객체 추가
const ntos = {
  1: 'one',
  2: 'two',
  3: 'three',
  4: 'four',
  5: 'five',
  6: 'six',
  7: 'seven',
  8: 'eight',
  9: 'nine',
  10: 'ten',
  11: 'eleven',
}

/**
 * 차트의 옵션
 */
const options = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'top',
    },
    title: {
      display: true,
      text: '가구 수별 세금 기준 변화',
    },
  },
  scales: {
    y: {
      title: {
        display: true,
        text: '세액 (단위: 만 원 또는 금액)', // tax_table의 데이터 단위에 맞게 수정
      },
    },
    x: {
      title: {
        display: true,
        text: '소득 구간 (단위: 천 원)',
      },
    },
  },
}

// 📊 핵심: 차트 데이터는 변경 시 전체 객체 교체가 필요합니다.
const chartData = ref({
  labels: [] as string[],
  datasets: [] as any[], // Chart.js Dataset 타입
})

// tax_table에서 추출된 원본 데이터 저장
const labels = ref<string[]>([])
// { 'family_one': [...], 'family_two': [...], ... } 형태로 저장
const allFamilyData = ref<Record<string, number[]>>({})

// 현재 선택된 가구 수 (UI 활성화 표시용)
const selectedFamily = ref<number>(1)

/**
 * 선택된 가구 수에 따라 차트 데이터를 업데이트하고 재렌더링을 유도합니다.
 * @param familySize 표시할 가구 수 (1부터 11까지)
 */
const updateChartData = (familySize: number) => {
  // ✅ 수정된 부분: familyKey도 'family_one' 형식으로 생성
  const familyName = ntos[familySize as keyof typeof ntos]
  const familyKey = `family_${familyName}`
  const dataArray = allFamilyData.value[familyKey]

  // 가구 수에 따른 대표 색상 지정 (11개 가구에 대해 다양한 색상을 순환합니다)
  const colors = [
    '#f87979',
    '#4bc0c0',
    '#ff9f40',
    '#9966ff',
    '#ffcd56',
    '#c9cbcf',
    '#36a2eb',
    '#e6005c',
    '#00b359',
    '#7a0099',
    '#ff6384',
  ]
  const color = colors[(familySize - 1) % colors.length]

  if (dataArray) {
    // 1. 완전히 새로운 datasets 배열 생성
    const newDatasets = [
      {
        label: `${familySize}인 가구`,
        backgroundColor: color,
        borderColor: color,
        data: dataArray,
        tension: 0.4, // 라인을 부드럽게
        pointRadius: 5, // 점 크기
      },
    ]

    // 2. **labels와 새로운 datasets를 포함하는 완전히 새로운 객체 할당**
    // 이 작업이 Vue-Chartjs의 업데이트를 트리거합니다.
    chartData.value = {
      labels: labels.value,
      datasets: newDatasets,
    }
  }
  selectedFamily.value = familySize
}

onMounted(() => {
  // 1. tax_table 데이터 구조 분석 및 labels/allFamilyData 추출
  tax_table.forEach((each) => {
    // X축 라벨 생성: "more_than 이상 more_less 미만"
    labels.value.push(`${each.more_than} 이상 ${each.more_less} 미만`)

    // 1인 가구부터 11인 가구까지의 데이터 추출
    for (let i = 1; i <= 11; i++) {
      // ✅ 수정된 부분: i를 영어 이름으로 변환
      const familyName = ntos[i as keyof typeof ntos]

      // ✅ 수정된 부분: allFamilyData의 키도 'family_one' 형식으로 사용
      const key = `family_${familyName}`
      if (!allFamilyData.value[key]) {
        allFamilyData.value[key] = []
      }

      // ✅ 수정된 부분: JSON 객체의 해당 속성 이름으로 데이터 접근
      const propKey = `family_${familyName}`
      allFamilyData.value[key].push(each[propKey as keyof typeof each] as number)
    }
  })

  // 2. 초기 차트 데이터 설정 (1인 가구)
  updateChartData(1)
})
</script>

<template>
  <div class="chart-container">
    <div class="family-buttons">
      <button
        v-for="i in 11"
        :key="i"
        @click="updateChartData(i)"
        :class="{ active: selectedFamily === i }"
      >
        {{ i }}인 가구
      </button>
    </div>

    <div style="height: 500px; width: 100%">
      <Line :data="chartData" :options="options" />
    </div>
  </div>
</template>

<style scoped>
/* 버튼 스타일링 예시 */
.family-buttons {
  margin-bottom: 20px;
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.family-buttons button {
  padding: 8px 12px;
  border: 1px solid #ccc;
  background-color: #f0f0f0;
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.2s;
}
.family-buttons button.active {
  background-color: #4bc0c0; /* 선택된 버튼 색상 */
  color: white;
  border-color: #4bc0c0;
  font-weight: bold;
}
.family-buttons button:hover:not(.active) {
  background-color: #e0e0e0;
}
</style>
