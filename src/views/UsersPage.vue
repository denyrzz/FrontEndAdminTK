<script setup>
import { ref, onMounted, reactive } from 'vue';
import Modal from '../components/Modal.vue';

// --- Konfigurasi & State ---
const API_URL = 'http://localhost:3000/api';
const users = ref([]);
const roles = ref([]); // Untuk menampung daftar role (Admin, User)
const isModalOpen = ref(false);
const isEditing = ref(false);

const currentUser = reactive({
  id_user: null,
  nama: '',
  email: '',
  password: '', // Dikosongkan saat edit, hanya diisi saat tambah/ubah
  role_id: 2, // Default ke 'User'
});

// --- Logika API ---
const fetchUsers = async () => {
  try {
    const response = await fetch(`${API_URL}/user`, { credentials: 'include' });
    if (!response.ok) throw new Error("Gagal mengambil data pengguna");
    const data = await response.json();
    users.value = data;
  } catch (error) {
    console.error("Error fetching users:", error);
  }
};

// Fungsi untuk mengambil daftar role (jika ada endpointnya)
const fetchRoles = async () => {
  try {
    // Asumsi Anda memiliki endpoint untuk mengambil semua role
    // const response = await fetch(`${API_URL}/roles`, { credentials: 'include' });
    // if (!response.ok) throw new Error("Gagal mengambil data role");
    // roles.value = await response.json();
    
    // Data contoh jika tidak ada API role
    roles.value = [
      { id_role: 1, nama: 'admin' },
      { id_role: 2, nama: 'user' },
    ];
  } catch (error) {
    console.error("Error fetching roles:", error);
  }
};

onMounted(() => {
  fetchUsers();
  fetchRoles();
});

const saveUser = async () => {
  try {
    let response;
    let url;
    let method;
    let body = {
      nama: currentUser.nama,
      email: currentUser.email,
      role_id: currentUser.role_id,
    };

    if (isEditing.value) {
      // Logika UPDATE
      url = `${API_URL}/user/${currentUser.id_user}`; // Asumsi endpoint update
      method = 'PUT';
      // Hanya tambahkan password ke body jika diisi
      if (currentUser.password) {
        body.password = currentUser.password;
      }
    } else {
      // Logika CREATE (menggunakan endpoint register)
      url = `${API_URL}/user/register`;
      method = 'POST';
      body.password = currentUser.password;
    }

    response = await fetch(url, {
      method: method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(body),
      credentials: 'include',
    });

    if (!response.ok) {
      const errorResult = await response.json();
      // Menangani error validasi dari backend
      const errorMsg = errorResult.errors ? errorResult.errors[0].msg : (errorResult.msg || "Gagal menyimpan data pengguna");
      throw new Error(errorMsg);
    }
    
    await fetchUsers();
    closeModal();
  } catch (error) {
    console.error("Error saving user:", error);
    alert(`Terjadi kesalahan: ${error.message}`);
  }
};

const deleteUser = async (id) => {
  if (confirm(`Apakah Anda yakin ingin menghapus pengguna dengan ID ${id}?`)) {
    try {
      // Asumsi Anda memiliki endpoint DELETE /user/:id
      const response = await fetch(`${API_URL}/user/${id}`, {
        method: 'DELETE',
        credentials: 'include',
      });
      if (!response.ok) throw new Error("Gagal menghapus pengguna");
      await fetchUsers();
    } catch (error) {
      console.error("Error deleting user:", error);
      alert("Terjadi kesalahan saat menghapus data.");
    }
  }
};

// --- Modal & Form Logic ---
const resetCurrentUser = () => {
  Object.assign(currentUser, {
    id_user: null, nama: '', email: '', password: '', role_id: 2
  });
};

const openAddModal = () => {
  isEditing.value = false;
  resetCurrentUser();
  isModalOpen.value = true;
};

const openEditModal = (user) => {
  isEditing.value = true;
  // Salin data pengguna ke form, kosongkan password
  Object.assign(currentUser, { ...user, password: '' });
  isModalOpen.value = true;
};

const closeModal = () => {
  isModalOpen.value = false;
};
</script>

<template>
  <div class="page-container">
    <div class="page-header">
      <h1>Kelola Pengguna</h1>
    </div>

    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>ID</th>
            <th>Nama</th>
            <th>Email</th>
            <th>Role</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="users.length === 0">
            <td colspan="5" class="text-center">Memuat data atau tidak ada data...</td>
          </tr>
          <tr v-for="user in users" :key="user.id_user">
            <td>{{ user.id_user }}</td>
            <td>{{ user.nama }}</td>
            <td>{{ user.email }}</td>
            <td>
              <span :class="['role-badge', user.role?.nama.toLowerCase()]">
                {{ user.role?.nama || 'N/A' }}
              </span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.role-badge {
  display: inline-block;
  padding: 4px 15px;
  border-radius: 12px;
  font-size: 0.9em;
  font-weight: 500;
  min-width: 80px;
  color: white;
  text-align: center;
  text-transform: capitalize;
}

.admin {
  background-color: #e74c3c; 
}

.user {
  background-color: #95a5a6;
}
</style>