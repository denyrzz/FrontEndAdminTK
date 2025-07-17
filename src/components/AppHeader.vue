<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router'; 

const searchQuery = ref('');
const router = useRouter(); 
const isDropdownOpen = ref(false);

function performSearch() {
  if (searchQuery.value) {
    console.log(`Mencari untuk: ${searchQuery.value}`);
  }
}

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

// --- FUNGSI LOGOUT YANG DIPERBAIKI ---
const logout = async () => {
  try {
    // Anda tetap bisa memanggil API backend untuk invalidasi sesi server
    await fetch('http://localhost:3000/api/user/logout', {
      method: 'DELETE',
      credentials: 'include',
    });
    // Tidak perlu menunggu atau memeriksa respons jika tidak ada tindakan khusus
    
  } catch (error) {
    console.error('API logout error (diabaikan):', error);
  } finally {
    // --- INI BAGIAN PALING PENTING ---
    // Hapus state login dari sisi frontend (browser)
    localStorage.removeItem('userToken');
    localStorage.removeItem('userData'); // Hapus juga data pengguna jika ada

    // Tutup dropdown
    isDropdownOpen.value = false;
    
    // Arahkan ke halaman login.
    // Sekarang router guard akan melihat isLoggedIn sebagai false dan mengizinkan navigasi.
    router.push('/login');
  }
};
</script>

<!-- Template dan Style Anda tidak perlu diubah -->
<template>
  <header class="app-header">
    <div class="search-container">
      <svg class="icon search-icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
        <path fill-rule="evenodd" d="M10.5 3.75a6.75 6.75 0 100 13.5 6.75 6.75 0 000-13.5zM2.25 10.5a8.25 8.25 0 1114.59 5.28l4.69 4.69a.75.75 0 11-1.06 1.06l-4.69-4.69A8.25 8.25 0 012.25 10.5z" clip-rule="evenodd" />
      </svg>
      <input
        type="text"
        class="search-input"
        placeholder="Cari buku atau anggota..."
        v-model="searchQuery"
        @keyup.enter="performSearch"
      />
    </div>

    <div class="settings-container">
      <button class="icon-button" title="Notifikasi">
        <svg class="icon" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor">
          <path fill-rule="evenodd" d="M11.078 2.25c-.917 0-1.699.663-1.943 1.572L6.67 11.41c-.02.066-.033.132-.033.201v3.75a.75.75 0 00.75.75h6a.75.75 0 00.75-.75v-3.75c0-.069-.013-.135-.033-.201l-2.466-7.587A2.025 2.025 0 0012.922 2.25h-1.844zM12 1.5a3.525 3.525 0 013.402 2.58l2.466 7.587a.75.75 0 01.033.201v3.75a2.25 2.25 0 01-2.25 2.25h-6a2.25 2.25 0 01-2.25-2.25v-3.75a.75.75 0 01.033-.201l2.466-7.587A3.525 3.525 0 0112 1.5z" clip-rule="evenodd" />
        </svg>
      </button>
      <div class="avatar" title="Profil Anda" @click="toggleDropdown">
        <span>AS</span>
        <div v-if="isDropdownOpen" class="avatar-dropdown">
          <button @click="logout" class="logout-btn">Logout</button>
        </div>
      </div>
    </div>
  </header>
</template>

<style scoped>
.app-header {
  height: 72px;
  background-color: var(--color-light);
  border-bottom: 1px solid var(--color-border);
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 24px;
  gap: 1.5rem;
  flex-shrink: 0;
}

.search-container {
  display: flex;
  align-items: center;
  background-color: var(--color-grey-bg);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 0 12px;
  width: 100%;
  max-width: 500px;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.search-container:focus-within {
  border-color: var(--color-primary);
  box-shadow: 0 0 0 3px rgba(0, 123, 255, 0.1);
}

.search-icon {
  color: var(--color-text-muted);
  margin-right: 8px;
}

.search-input {
  border: none;
  background: none;
  outline: none;
  height: 40px;
  width: 100%;
  color: var(--color-dark);
  font-family: 'Poppins', sans-serif;
  font-size: 14px;
  font-weight: 500;
}

.search-input::placeholder {
  color: var(--color-text-muted);
}

.settings-container {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.icon {
  height: 22px;
  width: 22px;
}

.icon-button {
  background: none;
  border: none;
  cursor: pointer;
  padding: 6px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  color: var(--color-text-muted);
  transition: background-color 0.2s, color 0.2s;
}

.icon-button:hover {
  background-color: var(--color-grey-bg);
  color: var(--color-dark);
}

.avatar {
  height: 40px;
  width: 40px;
  background-color: var(--color-primary);
  color: var(--color-light);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 600;
  font-size: 14px;
  cursor: pointer;
  position: relative;
}

.avatar-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  background-color: var(--color-light);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 0.5rem 0;
  min-width: 120px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  margin-top: 5px;
}

.logout-btn {
  width: 100%;
  padding: 8px 16px;
  border: none;
  background: none;
  text-align: left;
  font-family: 'Poppins', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--color-dark);
  cursor: pointer;
  transition: background-color 0.2s;
}

.logout-btn:hover {
  background-color: var(--color-grey-bg);
}
</style>