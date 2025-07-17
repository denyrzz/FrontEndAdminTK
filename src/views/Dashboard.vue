<script setup>
import { ref } from 'vue';
import VueApexCharts from 'vue3-apexcharts';

const totalUsers = ref('1,250');
const totalBooks = ref('8,930');
const activeLoans = ref('315');
const overdueBooks = ref('28');

const userRegistrationChart = {
  series: [{
    name: 'Pengguna Baru',
    data: [31, 40, 28, 51, 42, 109, 100]
  }],
  options: {
    chart: {
      height: 350,
      type: 'area',
      toolbar: { show: false }
    },
    colors: ['#007BFF'],
    dataLabels: { enabled: false },
    stroke: { curve: 'smooth' },
    xaxis: {
      type: 'datetime',
      categories: [
        "2025-07-11T00:00:00.000Z", "2025-07-12T00:00:00.000Z", "2025-07-13T00:00:00.000Z",
        "2025-07-14T00:00:00.000Z", "2025-07-15T00:00:00.000Z", "2025-07-16T00:00:00.000Z",
        "2025-07-17T00:00:00.000Z"
      ],
    },
    tooltip: {
      x: { format: 'dd MMM yyyy' }
    },
  },
};

const bookCategoryChart = {
  series: [44, 55, 41, 17, 15],
  options: {
    chart: { type: 'donut' },
    labels: ['Fiksi', 'Sains', 'Sejarah', 'Biografi', 'Komputer'],
    colors: ['#007BFF', '#00E396', '#FEB019', '#FF4560', '#775DD0'],
    legend: { position: 'bottom' },
    responsive: [{
      breakpoint: 480,
      options: {
        chart: { width: 200 },
        legend: { position: 'bottom' }
      }
    }]
  },
};

const recentLoans = ref([
  { id: 1, book: 'Pengantar Kecerdasan Buatan', user: 'Budi Santoso', status: 'Dipinjam' },
  { id: 2, book: 'Sejarah Dunia Modern', user: 'Citra Lestari', status: 'Dipinjam' },
  { id: 3, book: 'Dasar-dasar Vue.js', user: 'Ahmad Dahlan', status: 'Dikembalikan' },
  { id: 4, book: 'Novel Hujan Bulan Juni', user: 'Dewi Anggraini', status: 'Dipinjam' },
]);

</script>

<template>
  <div class="dashboard-container">
    <div class="stats-grid">
      <div class="stat-card">
        <div class="stat-icon users">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M4.5 6.375a.75.75 0 01.75-.75h9a.75.75 0 01.75.75v.75h-10.5v-.75zm10.5 0v.75a.75.75 0 001.5 0v-.75a.75.75 0 00-1.5 0z" /><path fill-rule="evenodd" d="M20.25 10.5a.75.75 0 01-.75.75H2.25a.75.75 0 010-1.5H19.5a.75.75 0 01.75.75zM3 15.75a.75.75 0 00.75-.75h16.5a.75.75 0 000-1.5H3.75a.75.75 0 00-.75.75z" clip-rule="evenodd" /></svg>
        </div>
        <div class="stat-info">
          <h4>Total Pengguna</h4>
          <p>{{ totalUsers }}</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon books">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M.75 3.75A.75.75 0 011.5 3h16.5a.75.75 0 01.75.75v16.5a.75.75 0 01-.75.75H1.5a.75.75 0 01-.75-.75V3.75zM9 6a.75.75 0 000 1.5h6a.75.75 0 000-1.5H9z" /></svg>
        </div>
        <div class="stat-info">
          <h4>Total Buku</h4>
          <p>{{ totalBooks }}</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon loans">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path d="M11.25 4.533A9.707 9.707 0 006 3a9.735 9.735 0 00-3.25.555.75.75 0 00-.5.707v14.25a.75.75 0 001 .707A9.735 9.735 0 006 21a9.707 9.707 0 005.25-1.533.75.75 0 000-1.327A8.235 8.235 0 016 18a8.235 8.235 0 01-4.25-.967V5.05A8.235 8.235 0 016 6a8.235 8.235 0 015.25.967.75.75 0 000-1.327z" /><path d="M12.75 3a.75.75 0 00-1.5 0v18a.75.75 0 001.5 0V3z" /></svg>
        </div>
        <div class="stat-info">
          <h4>Peminjaman Aktif</h4>
          <p>{{ activeLoans }}</p>
        </div>
      </div>
      <div class="stat-card">
        <div class="stat-icon overdue">
          <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor"><path fill-rule="evenodd" d="M12 2.25c-5.385 0-9.75 4.365-9.75 9.75s4.365 9.75 9.75 9.75 9.75-4.365 9.75-9.75S17.385 2.25 12 2.25zM12.75 6a.75.75 0 00-1.5 0v6c0 .414.336.75.75.75h4.5a.75.75 0 000-1.5h-3.75V6z" clip-rule="evenodd" /></svg>
        </div>
        <div class="stat-info">
          <h4>Buku Terlambat</h4>
          <p>{{ overdueBooks }}</p>
        </div>
      </div>
    </div>

    <div class="charts-grid">
      <div class="chart-card large">
        <h3>Pendaftaran Pengguna (7 Hari Terakhir)</h3>
        <VueApexCharts type="area" :options="userRegistrationChart.options" :series="userRegistrationChart.series" />
      </div>
      <div class="chart-card">
        <h3>Komposisi Kategori Buku</h3>
        <VueApexCharts type="donut" :options="bookCategoryChart.options" :series="bookCategoryChart.series" />
      </div>
      <div class="chart-card large table-card">
        <h3>Peminjaman Terbaru</h3>
        <table class="recent-loans-table">
          <thead>
            <tr>
              <th>Nama Buku</th>
              <th>Peminjam</th>
              <th>Status</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="loan in recentLoans" :key="loan.id">
              <td>{{ loan.book }}</td>
              <td>{{ loan.user }}</td>
              <td><span :class="['status-badge', loan.status.toLowerCase()]">{{ loan.status }}</span></td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>
