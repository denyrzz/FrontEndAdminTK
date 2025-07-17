<script setup>
import { ref, onMounted, reactive } from 'vue';
import Modal from '../components/Modal.vue'; // Asumsikan Anda punya komponen Modal

const API_URL = 'http://localhost:3000/api'; // Sesuaikan dengan URL API backend Anda
const returns = ref([]);
const loans = ref([]); // Untuk dropdown pilihan peminjaman
const isModalOpen = ref(false);
const isEditing = ref(false);
const isLoading = ref(false);
const errorMessage = ref('');

const currentReturn = reactive({
    id_pengembalian: null,
    peminjaman_id: null,
    tanggal_kembali: new Date().toISOString().split('T')[0], 
    kondisi_buku: 'Baik', 
    denda: 0,
});

const fetchReturns = async () => {
    isLoading.value = true;
    errorMessage.value = '';
    try {
        const response = await fetch(`${API_URL}/pengembalian`, { credentials: 'include' });
        const result = await response.json();

        if (!response.ok) {
            throw new Error(result.msg || 'Gagal mengambil data pengembalian.');
        }

        const formattedReturns = result.map(ret => ({
            ...ret,
            judul_buku: ret.Peminjaman?.Buku?.judul || 'Tidak Diketahui',
            nama_peminjam: ret.Peminjaman?.User?.nama || ret.Peminjaman?.User?.email?.split('@')[0] || 'Tidak Diketahui',
            nomor_resi: ret.Peminjaman?.KartuPustaka?.nomor_resi || 'N/A',
        }));
        returns.value = formattedReturns;

    } catch (error) {
        console.error('Error fetching returns:', error);
        errorMessage.value = error.message;
    } finally {
        isLoading.value = false;
    }
};

const fetchLoanOptions = async () => {
    try {
        const response = await fetch(`${API_URL}/peminjaman`, { credentials: 'include' });
        const result = await response.json();

        if (!response.ok) {
            throw new Error(result.msg || 'Gagal mengambil data peminjaman untuk opsi.');
        }

        loans.value = result.filter(loan => loan.status === 'Dipinjam').map(loan => ({
            id: loan.id_peminjaman,
            display: `${loan.Buku?.judul || 'Unknown Buku'} - ${loan.User?.nama || loan.User?.email?.split('@')[0] || 'Unknown User'} (ID: ${loan.id_peminjaman})`
        }));
    } catch (error) {
        console.error('Error fetching loan options:', error);
    }
};

onMounted(() => {
    fetchReturns();
    fetchLoanOptions();
});

const saveReturn = async () => {
    try {
        let url = `${API_URL}/pengembalian`;
        let method = 'POST';

        if (isEditing.value) {
            url = `${API_URL}/pengembalian/${currentReturn.id_pengembalian}`;
            method = 'PUT';
        }

        const response = await fetch(url, {
            method: method,
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify(currentReturn),
            credentials: 'include',
        });

        if (!response.ok) {
            const errorResult = await response.json();
            throw new Error(errorResult.msg || "Gagal menyimpan data pengembalian.");
        }

        await fetchReturns(); 
        closeModal();
    } catch (error) {
        console.error("Error saving return:", error);
        alert(`Terjadi kesalahan: ${error.message}`);
    }
};

const deleteReturn = async (id) => {
    if (confirm(`Apakah Anda yakin ingin menghapus data pengembalian ID ${id}?`)) {
        try {
            const response = await fetch(`${API_URL}/pengembalian/${id}`, {
                method: 'DELETE',
                credentials: 'include',
            });
            if (!response.ok) throw new Error("Gagal menghapus data pengembalian.");
            await fetchReturns(); // Refresh data
        } catch (error) {
            console.error("Error deleting return:", error);
            alert("Terjadi kesalahan saat menghapus data.");
        }
    }
};

const resetCurrentReturn = () => {
    Object.assign(currentReturn, {
        id_pengembalian: null,
        peminjaman_id: null,
        tanggal_kembali: new Date().toISOString().split('T')[0],
        kondisi_buku: 'Baik',
        denda: 0,
    });
};

const openAddModal = () => {
    isEditing.value = false;
    resetCurrentReturn();
    isModalOpen.value = true;
};

const openEditModal = (ret) => {
    isEditing.value = true;
    Object.assign(currentReturn, {
        ...ret,
        peminjaman_id: ret.peminjaman_id,
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
            <h1>Kelola Pengembalian Peminjaman</h1>
            <button @click.stop="openAddModal" class="btn btn-primary">Tambah Pengembalian</button>
        </div>

        <div class="table-container">
            <table>
                <thead>
                    <tr>
                        <th>ID</th>
                        <th>Buku</th>
                        <th>Peminjam</th>
                        <th>No. Resi</th>
                        <th>Tgl Kembali</th>
                        <th>Kondisi Buku</th>
                        <th>Denda (Rp)</th>
                        <th>Aksi</th>
                    </tr>
                </thead>
                <tbody>
                    <tr v-if="isLoading">
                        <td colspan="8" class="text-center">Memuat data pengembalian...</td>
                    </tr>
                    <tr v-else-if="errorMessage">
                        <td colspan="8" class="text-center text-danger">Error: {{ errorMessage }}</td>
                    </tr>
                    <tr v-else-if="returns.length === 0">
                        <td colspan="8" class="text-center">Tidak ada data pengembalian yang tersedia.</td>
                    </tr>
                    <template v-else>
                        <tr v-for="ret in returns" :key="ret.id_pengembalian">
                            <td>{{ ret.id_pengembalian }}</td>
                            <td>{{ ret.judul_buku }}</td>
                            <td>{{ ret.nama_peminjam }}</td>
                            <td>{{ ret.nomor_resi }}</td>
                            <td>{{ ret.tanggal_kembali }}</td>
                            <td>
                                <span :class="['status-badge', ret.kondisi_buku.toLowerCase().replace(/\s/g, '-')]">{{ ret.kondisi_buku }}</span>
                            </td>
                            <td>Rp {{ ret.denda.toLocaleString('id-ID') }}</td>
                            <td class="action-buttons">
                                <button @click.stop="openEditModal(ret)" class="btn btn-secondary" title="Edit">E</button>
                                <button @click.stop="deleteReturn(ret.id_pengembalian)" class="btn btn-danger" title="Delete">D</button>
                            </td>
                        </tr>
                    </template>
                </tbody>
            </table>
        </div>

        <Modal :show="isModalOpen" @close="closeModal">
            <template #header>
                <h3>{{ isEditing ? 'Edit Pengembalian' : 'Tambah Pengembalian Baru' }}</h3>
            </template>
            <template #body>
                <div class="form-group">
                    <label for="peminjaman">Peminjaman</label>
                    <select id="peminjaman" v-model="currentReturn.peminjaman_id" class="form-input" :disabled="isEditing">
                        <option :value="null" disabled>Pilih Peminjaman</option>
                        <option v-for="loan in loans" :key="loan.id" :value="loan.id">{{ loan.display }}</option>
                    </select>
                    <small v-if="isEditing" class="text-info">Peminjaman tidak bisa diubah setelah dibuat.</small>
                </div>
                <div class="form-group">
                    <label for="tanggal_kembali">Tanggal Kembali</label>
                    <input type="date" id="tanggal_kembali" v-model="currentReturn.tanggal_kembali" class="form-input">
                </div>
                <div class="form-group">
                    <label for="kondisi_buku">Kondisi Buku</label>
                    <select id="kondisi_buku" v-model="currentReturn.kondisi_buku" class="form-input">
                        <option>Baik</option>
                        <option>Rusak Ringan</option>
                        <option>Rusak Berat</option>
                        <option>Hilang</option>
                    </select>
                </div>
                <div class="form-group">
                    <label for="denda">Denda (Rp)</label>
                    <input type="number" id="denda" v-model.number="currentReturn.denda" class="form-input" min="0">
                </div>
            </template>
            <template #footer>
                <button @click="closeModal" class="btn btn-secondary">Batal</button>
                <button @click="saveReturn" class="btn btn-primary">Simpan</button>
            </template>
        </Modal>
    </div>
</template>