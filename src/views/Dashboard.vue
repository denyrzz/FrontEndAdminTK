<script setup>
import { ref, onMounted, reactive } from 'vue';
import VueApexCharts from 'vue3-apexcharts';

const API_URL = 'http://localhost:3000/api';

const totalUsers = ref(0);
const totalBooks = ref(0);
const activeLoans = ref(0);
const overdueBooks = ref(0);

const userRegistrationChart = reactive({
  series: [{
    name: 'Pengguna Baru',
    data: []
  }],
  options: {
    chart: { type: 'area', height: 350, toolbar: { show: false } },
    colors: ['#007BFF'],
    dataLabels: { enabled: false },
    stroke: { curve: 'smooth' },
    xaxis: {
      type: 'datetime',
      categories: [],
    },
    tooltip: { x: { format: 'dd MMM yyyy' } },
  },
});

const bookCategoryChart = reactive({
  series: [],
  options: {
    chart: { type: 'donut' },
    labels: [],
    colors: ['#007BFF', '#00E396', '#FEB019', '#FF4560', '#775DD0', '#546E7A', '#E91E63'],
    legend: { position: 'bottom' },
    responsive: [{
      breakpoint: 480,
      options: {
        chart: { width: 200 },
        legend: { position: 'bottom' }
      }
    }]
  },
});

const recentLoans = ref([]);

const fetchDashboardData = async () => {
  try {
    const [usersRes, booksRes] = await Promise.all([
      fetch(`${API_URL}/user`, { credentials: 'include' }),
      fetch(`${API_URL}/buku`, { credentials: 'include' })
    ]);

    if (!usersRes.ok || !booksRes.ok) {
      throw new Error('Gagal mengambil data pengguna atau buku.');
    }

    const users = await usersRes.json();
    const books = await booksRes.json();

    totalUsers.value = users.length;
    totalBooks.value = books.length;
    processUserChart(users);
    processCategoryChart(books);

    try {
      const loansRes = await fetch(`${API_URL}/peminjaman`, { credentials: 'include' });
      if (loansRes.ok) {
        const loans = await loansRes.json();
        processLoanStats(loans);
        processRecentLoans(loans);
      } else {
        console.error('Gagal mengambil data peminjaman, endpoint mungkin belum siap.');
      }
    } catch (loanError) {
      console.error('Terjadi error saat mengambil data peminjaman:', loanError);
    }

  } catch (error) {
    console.error('Error fetching primary dashboard data:', error);
    alert('Gagal memuat data inti dasbor (pengguna dan buku).');
  }
};

const processLoanStats = (loans) => {
  const today = new Date();
  today.setHours(0, 0, 0, 0);

  activeLoans.value = loans.filter(peminjaman => peminjaman.status === 'Dipinjam').length;
  
  overdueBooks.value = loans.filter(peminjaman => 
    peminjaman.status === 'Dipinjam' && peminjaman.tanggal_kembali && new Date(peminjaman.tanggal_kembali) < today
  ).length;
};

const processUserChart = (users) => {
  const userCountsByDate = {};
  for (let i = 6; i >= 0; i--) {
    const d = new Date();
    d.setDate(d.getDate() - i);
    const dateString = d.toISOString().split('T')[0];
    userCountsByDate[dateString] = 0;
  }

  users.forEach(user => {
    if (user.createdAt) {
      const registrationDate = new Date(user.createdAt).toISOString().split('T')[0];
      if (userCountsByDate[registrationDate] !== undefined) {
        userCountsByDate[registrationDate]++;
      }
    }
  });

  userRegistrationChart.options.xaxis.categories = Object.keys(userCountsByDate);
  userRegistrationChart.series[0].data = Object.values(userCountsByDate);
};

const processCategoryChart = (books) => {
  const categoryCounts = {};
  books.forEach(buku => {
    const categoryName = buku.kategori?.nama_kategori || 'Tanpa Kategori';
    categoryCounts[categoryName] = (categoryCounts[categoryName] || 0) + 1;
  });

  bookCategoryChart.options.labels = Object.keys(categoryCounts);
  bookCategoryChart.series = Object.values(categoryCounts);
};

const processRecentLoans = (loans) => {
  const sortedLoans = loans.sort((a, b) => b.id_peminjaman - a.id_peminjaman);
  const today = new Date();
  today.setHours(0, 0, 0, 0);
  
  recentLoans.value = sortedLoans.slice(0, 5).map(loan => {
    const isOverdue = loan.status === 'dipinjam' && loan.tanggal_kembali && new Date(loan.tanggal_kembali) < today;
    return {
      id: loan.id_peminjaman,
      book: loan.buku?.judul || 'Buku Tidak Ditemukan',
      user: loan.user?.nama || 'Pengguna Tidak Ditemukan',
      status: loan.status || 'Tidak Diketahui',
      isOverdue: isOverdue
    };
  });
};

onMounted(() => {
  fetchDashboardData();
});

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
            <tr v-if="recentLoans.length === 0">
              <td colspan="3" style="text-align: center;">Data peminjaman belum tersedia</td>
            </tr>
            <tr v-for="loan in recentLoans" :key="loan.id">
              <td>{{ loan.book }}</td>
              <td>{{ loan.user }}</td>
              <td>
                <span v-if="loan.isOverdue" class="status-badge terlambat">
                  Terlambat
                </span>
                <span v-else :class="['status-badge', loan.status.toLowerCase().replace(' ', '-')]">
                  {{ loan.status }}
                </span>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dashboard-container {
  padding: 24px;
  background-color: #f4f7f6;
}

.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(220px, 1fr));
  gap: 24px;
  margin-bottom: 24px;
}

.stat-card {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
  transition: transform 0.2s;
}

.stat-card:hover {
    transform: translateY(-5px);
}

.stat-icon {
  width: 48px;
  height: 48px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.stat-icon svg {
    width: 24px;
    height: 24px;
}

.stat-icon.users { background-color: #007BFF; }
.stat-icon.books { background-color: #28a745; }
.stat-icon.loans { background-color: #ffc107; }
.stat-icon.overdue { background-color: #dc3545; }

.stat-info h4 {
  margin: 0;
  font-size: 14px;
  color: #6c757d;
  font-weight: 500;
}

.stat-info p {
  margin: 4px 0 0;
  font-size: 24px;
  font-weight: 700;
  color: #343a40;
}

.charts-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
}

.chart-card {
  background-color: #fff;
  border-radius: 8px;
  padding: 20px;
  box-shadow: 0 4px 6px rgba(0,0,0,0.05);
}

.chart-card.large {
  grid-column: span 2;
}

.chart-card h3 {
  margin-top: 0;
  margin-bottom: 20px;
  font-size: 18px;
  font-weight: 600;
}

.recent-loans-table {
  width: 100%;
  border-collapse: collapse;
}

.recent-loans-table th, .recent-loans-table td {
  padding: 12px 8px;
  text-align: left;
  border-bottom: 1px solid #e9ecef;
}

.recent-loans-table th {
    font-size: 14px;
    font-weight: 600;
    color: #495057;
}

.recent-loans-table tbody tr:last-child td {
    border-bottom: none;
}

.status-badge {
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: 600;
  text-transform: capitalize;
}

.status-badge.dipinjam {
  background-color: rgba(255, 193, 7, 0.2);
  color: #b98900;
}

.status-badge.dikembalikan {
  background-color: rgba(40, 167, 69, 0.2);
  color: #28a745;
}

.status-badge.terlambat {
  background-color: rgba(220, 53, 69, 0.2);
  color: #dc3545;
}

@media (max-width: 1200px) {
  .charts-grid {
    grid-template-columns: 1fr 1fr;
  }
  .chart-card.large.table-card {
      grid-column: span 2;
  }
}
@media (max-width: 768px) {
  .charts-grid {
    grid-template-columns: 1fr;
  }
  .chart-card.large {
    grid-column: span 1;
  }
}
</style>
