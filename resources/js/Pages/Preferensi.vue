<script setup>
import AuthenticatedLayout from '@/Layouts/AuthenticatedLayout.vue';
import axios from 'axios';
import { Chart, registerables } from 'chart.js';
import { onMounted, ref } from 'vue';

Chart.register(...registerables);

// Reactive table data
const tableData = ref(null);

// Refs for Charts
const rankingChartRef = ref(null);
let rankingChart = null;

// Fetch Data Function (API call)
const fetchTableData = async () => {
    try {
        // Make API request to get the data
        const response = await axios.get('/saw/calculate'); // Replace with your actual API endpoint

        if (response.data.success) {
            tableData.value = response.data.data;
            // createRadarChart();
            createRankingChart();
        } else {
            console.error('Failed to fetch data');
        }
    } catch (error) {
        console.error('Error fetching data:', error);
    }
};

// Initialize Ranking Bar Chart (Limit to top 3 rankings)
const createRankingChart = () => {
    if (!tableData.value?.ranking) return; // Prevent errors if ranking is undefined

    if (rankingChart) rankingChart.destroy(); // Destroy previous chart if exists

    // Limit to top 3 rankings
    const top3Ranking = tableData.value.ranking.slice(0, 3);

    const labels = top3Ranking.map((item) => item.name);
    const data = top3Ranking.map((item) => item.score);

    rankingChart = new Chart(rankingChartRef.value, {
        type: 'bar',
        data: {
            labels,
            datasets: [
                {
                    label: 'Final Score',
                    data,
                    backgroundColor: 'rgba(255, 165, 0, 0.7)',
                    borderColor: 'rgba(75, 192, 192, 1)',
                    borderWidth: 1,
                },
            ],
        },
        options: {
            responsive: true,
            maintainAspectRatio: false,
            scales: {
                y: {
                    beginAtZero: true,
                },
            },
        },
    });
};

// Fetch data on mount
onMounted(() => {
    fetchTableData();
});
</script>

<template>
    <AuthenticatedLayout>
        <div class="space-y-8">
            <!-- <div class="card">
                <h2 class="text-xl font-bold">
                    Matrix Visualization (Radar Chart)
                </h2>
                <div class="relative h-80">
                    <canvas ref="radarChartRef"></canvas>
                </div>
            </div> -->
            <!-- Ranking Bar Chart -->
            <div class="card mx-40">
                <h2 class="text-xl font-bold">
                    Final Ranking Scores (Bar Chart)
                </h2>
                <div class="relative h-80">
                    <canvas ref="rankingChartRef"></canvas>
                </div>
            </div>
            <!-- Original Tables -->
            <div
                class="flex overflow-auto"
                v-if="tableData && tableData['matrix']"
            >
                <table class="table-pin-cols table">
                    <thead class="bg-base-300 text-center text-lg font-bold">
                        <tr>
                            <th style="width: 25%">Alternatif</th>
                            <th
                                v-for="(value, key) in tableData['matrix'][1]"
                                :key="key"
                                class="text-center"
                            >
                                C{{ key }}
                            </th>
                        </tr>
                    </thead>
                    <tbody class="text-center">
                        <tr
                            v-for="(item, index) in tableData['matrix']"
                            :key="index"
                        >
                            <td>A{{ index }}</td>
                            <td v-for="(child, key) in item" :key="key">
                                {{ child }}
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="divider my-10"></div>
            <div
                class="flex overflow-auto"
                v-if="tableData && tableData['weighted']"
            >
                <table class="table-pin-cols table">
                    <thead class="bg-base-300 text-center text-lg font-bold">
                        <tr>
                            <th style="width: 25%">Alternatif</th>
                            <th
                                v-for="(value, key) in tableData['matrix'][1]"
                                :key="key"
                                class="text-center"
                            >
                                C{{ key }}
                            </th>
                        </tr>
                    </thead>
                    <tbody class="text-center">
                        <tr
                            v-for="(item, index) in tableData['weighted']"
                            :key="index"
                        >
                            <td>A{{ index }}</td>
                            <td v-for="(child, key) in item" :key="key">
                                {{ parseFloat(child).toFixed(3) }}
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="divider my-10"></div>
            <div
                class="flex overflow-auto"
                v-if="tableData && tableData['normalization']"
            >
                <table class="table-pin-cols table">
                    <thead class="bg-base-300 text-center text-lg font-bold">
                        <tr>
                            <th style="width: 25%">Alternatif</th>
                            <th
                                v-for="(value, key) in tableData['matrix'][1]"
                                :key="key"
                                class="text-center"
                            >
                                C{{ key }}
                            </th>
                        </tr>
                    </thead>
                    <tbody class="text-center">
                        <tr
                            v-for="(item, index) in tableData['normalization']"
                            :key="index"
                        >
                            <td>A{{ index }}</td>
                            <td v-for="(child, key) in item" :key="key">
                                {{ parseFloat(child).toFixed(3) }}
                            </td>
                        </tr>
                    </tbody>
                </table>
            </div>
            <div class="divider my-10"></div>
            <div
                class="flex overflow-auto"
                v-if="tableData && tableData['ranking']"
            >
                <table class="table-pin-cols table">
                    <thead class="bg-base-300 text-center text-lg font-bold">
                        <tr>
                            <th style="width: 25%">Alternatif</th>
                            <th style="width: 25%">Nama</th>
                            <th style="width: 25%">Nilai</th>
                            <th style="width: 25%">Ranking</th>
                        </tr>
                    </thead>
                    <tbody class="text-center">
                        <tr
                            v-for="(item, index) in tableData['ranking']"
                            :key="item.id"
                        >
                            <td>A{{ index + 1 }}</td>
                            <td>{{ item.name }}</td>
                            <td>{{ parseFloat(item.score).toFixed(3) }}</td>
                            <td class="font-bold">{{ item.rank }}</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>
    </AuthenticatedLayout>
</template>
