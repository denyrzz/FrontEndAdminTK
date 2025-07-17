<script setup>
import { ref, onMounted, reactive } from 'vue';
import Modal from '../components/Modal.vue';

const API_URL = 'http://localhost:3000/api';
const loans = ref([]);
const users = ref([]);
const books = ref([]);
const isModalOpen = ref(false);
const isEditing = ref(false);
const isLoading = ref(false);
const errorMessage = ref('');

const currentLoan = reactive({
  id_peminjaman: null,
  user_id: null,
  buku_id: null,
  kartu_pustaka_id: null,
  tanggal_pinjam: new Date().toISOString().split('T')[0],
  tanggal_kembali: '',
  status: 'Dipinjam',
});

const fetchLoans = async () => {
  try {
    isLoading.value = true;
    errorMessage.value = '';
    const response = await fetch(`${API_URL}/peminjaman`, { credentials: 'include' });
    const result = await response.json();

    if (!response.ok) {
      throw new Error(result.msg || 'Gagal mengambil data peminjaman');
    }

    const formattedLoans = result.map(loan => ({
      id_peminjaman: loan.id_peminjaman,
      user_id: loan.user_id || 'N/A',
      buku_id: loan.buku_id || 'N/A',
  User: {
    email: loan.User?.email?.split('@')[0] || 'Unknown',
    nama: loan.User?.nama || loan.User?.email?.split('@')[0] || 'Unknown',
  },
  Buku: {
    judul: loan.Buku.judul || 'Unknown',
  },
  KartuPustaka: { 
    nomor_resi: loan.KartuPustaka?.nomor_resi || 'N/A',
  },

      tanggal_pinjam: loan.tanggal_pinjam,
      tanggal_kembali: loan.tanggal_kembali,
      status: loan.status || 'Unknown',
    }));

    loans.value = formattedLoans;
  } catch (error) {
    console.error('Error fetching loans:', error);
    errorMessage.value = error.message;
  } finally {
    isLoading.value = false;
  }
};
const fetchSelectOptions = async () => {
    try {
        // users.value = await (await fetch(`${API_URL}/user`, {credentials: 'include'})).json();
        // books.value = await (await fetch(`${API_URL}/buku`, {credentials: 'include'})).json();
        
        // Data contoh
        users.value = [{id_user: 1, nama: 'Budi Santoso'}, {id_user: 2, nama: 'Citra Lestari'}];
        books.value = [{id_buku: 1, judul: 'Vue.js untuk Pemula'}, {id_buku: 2, judul: 'Mastering MySQL'}];
    } catch(error) {
        console.error("Error fetching select options:", error);
    }
};

onMounted(() => {
  fetchLoans();
  fetchSelectOptions();
});

const saveLoan = async () => {
  try {
    let url = `${API_URL}/peminjaman`;
    let method = 'POST';

    if (isEditing.value) {
      url = `${API_URL}/peminjaman/${currentLoan.id_peminjaman}`;
      method = 'PUT';
    }

    const response = await fetch(url, {
      method: method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(currentLoan),
      credentials: 'include',
    });

    if (!response.ok) {
      const errorResult = await response.json();
      throw new Error(errorResult.msg || "Gagal menyimpan data peminjaman");
    }
    
    await fetchLoans();
    closeModal();
  } catch (error) {
    console.error("Error saving loan:", error);
    alert(`Terjadi kesalahan: ${error.message}`);
  }
};

const deleteLoan = async (id) => {
  if (confirm(`Apakah Anda yakin ingin menghapus data peminjaman ID ${id}?`)) {
    try {
      const response = await fetch(`${API_URL}/peminjaman/${id}`, {
        method: 'DELETE',
        credentials: 'include',
      });
      if (!response.ok) throw new Error("Gagal menghapus data peminjaman");
      await fetchLoans();
    } catch (error) {
      console.error("Error deleting loan:", error);
      alert("Terjadi kesalahan saat menghapus data.");
    }
  }
};

// --- Modal & Form Logic ---
const resetCurrentLoan = () => {
  Object.assign(currentLoan, {
    id_peminjaman: null, user_id: null, buku_id: null, kartu_pustaka_id: null,
    tanggal_pinjam: new Date().toISOString().split('T')[0], tanggal_kembali: '', status: 'Dipinjam'
  });
};

const openAddModal = () => {
  isEditing.value = false;
  resetCurrentLoan();
  isModalOpen.value = true;
};

const openEditModal = (loan) => {
  isEditing.value = true;
  Object.assign(currentLoan, {
      ...loan,
      user_id: loan.User?.id_user, // Sesuaikan dengan struktur data Anda
      buku_id: loan.Buku?.id_buku, // Sesuaikan dengan struktur data Anda
  });
  isModalOpen.value = true;
};

const closeModal = () => {
  isModalOpen.value = false;
};
</script>

<template>
  <div class="page-container">
    <div class="page-header">
      <h1>Kelola Peminjaman</h1>
      <button @click.stop="openAddModal" class="btn btn-primary">Tambah Peminjaman</button>
    </div>

    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Nama Peminjam</th>
            <th>Judul Buku</th>
            <th>Nomor Resi</th>
            <th>Tgl Pinjam</th>
            <th>Tgl Kembali</th>
            <th>Status</th>
            <th>Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="loans.length === 0">
            <td colspan="7" class="text-center">Memuat data atau tidak ada data...</td>
          </tr>
          <tr v-for="loan in loans" :key="loan.id_peminjaman">
            <td>{{ loan.id_peminjaman }}</td>
            <td>{{ loan.User?.nama || 'N/A' }}</td>
            <td>{{ loan.Buku?.judul || 'N/A' }}</td>
            <td>{{ loan.nomor_resi?.nomor_resi || 'N/A' }}</td>
            <td>{{ loan.tanggal_pinjam }}</td>
            <td>{{ loan.tanggal_kembali || '-' }}</td>
            <td>
              <span :class="['status-badge', loan.status.toLowerCase()]">{{ loan.status }}</span>
            </td>
            <td class="action-buttons">
              <button @click.stop="openEditModal(loan)" class="btn btn-secondary" title="Edit">E</button>
              <button @click.stop="deleteLoan(loan.id_peminjaman)" class="btn btn-danger" title="Delete">D</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <Modal :show="isModalOpen" @close="closeModal">
      <template #header>
        <h3>{{ isEditing ? 'Edit Peminjaman' : 'Tambah Peminjaman Baru' }}</h3>
      </template>
      <template #body>
        <div class="form-group">
          <label for="user">Peminjam</label>
          <select id="user" v-model="currentLoan.user_id" class="form-input">
            <option disabled :value="null">Pilih Pengguna</option>
            <option v-for="user in users" :key="user.id_user" :value="user.id_user">{{ user.nama }}</option>
          </select>
        </div>
        <div class="form-group">
          <label for="buku">Buku</label>
          <select id="buku" v-model="currentLoan.buku_id" class="form-input">
             <option disabled :value="null">Pilih Buku</option>
            <option v-for="book in books" :key="book.id_buku" :value="book.id_buku">{{ book.judul }}</option>
          </select>
        </div>
        <div class="form-group-row">
          <div class="form-group">
            <label for="tanggal_pinjam">Tanggal Pinjam</label>
            <input type="date" id="tanggal_pinjam" v-model="currentLoan.tanggal_pinjam" class="form-input">
          </div>
          <div class="form-group">
            <label for="tanggal_kembali">Tanggal Kembali</label>
            <input type="date" id="tanggal_kembali" v-model="currentLoan.tanggal_kembali" class="form-input">
          </div>
        </div>
         <div class="form-group">
          <label for="status">Status</label>
          <select id="status" v-model="currentLoan.status" class="form-input">
            <option>Dipinjam</option>
            <option>Dikembalikan</option>
            <option>Terlambat</option>
          </select>
        </div>
      </template>
      <template #footer>
        <button @click="closeModal" class="btn btn-secondary">Batal</button>
        <button @click="saveLoan" class="btn btn-primary">Simpan</button>
      </template>
    </Modal>
  </div>
</template>
