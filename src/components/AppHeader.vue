<script setup>
import { ref, computed } from 'vue';
import { useRouter } from 'vue-router';

const searchQuery = ref('');
const router = useRouter();
const isDropdownOpen = ref(false);

const userData = computed(() => {
  try {
    const data = localStorage.getItem('userData');
    return data ? JSON.parse(data) : {};
  } catch (e) {
    console.error("Gagal mem-parsing data pengguna:", e);
    return {};
  }
});

const userInitials = computed(() => {
  const name = userData.value?.nama || 'A';
  const parts = name.trim().split(' ');
  if (parts.length > 1) {
    return (parts[0][0] + parts[parts.length - 1][0]).toUpperCase();
  }
  return name.substring(0, 2).toUpperCase();
});


function performSearch() {
  if (searchQuery.value) {
    console.log(`Mencari untuk: ${searchQuery.value}`);
  }
}

const toggleDropdown = () => {
  isDropdownOpen.value = !isDropdownOpen.value;
};

const logout = async () => {
  try {
    await fetch('http://localhost:3000/api/user/logout', {
      method: 'DELETE',
      credentials: 'include',
    });
  } catch (error) {
    console.error('API logout error (diabaikan):', error);
  } finally {
    localStorage.removeItem('userToken');
    localStorage.removeItem('userData');

    isDropdownOpen.value = false;
    
    router.push('/login');
  }
};
</script>

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
      
      <div class="avatar-wrapper">
        <div class="avatar" title="Profil Anda" @click="toggleDropdown">
          <span>{{ userInitials }}</span>
        </div>
        <div v-if="isDropdownOpen" class="avatar-dropdown">
          <div class="dropdown-header">
            <p class="user-name">{{ userData.nama || 'Nama Pengguna' }}</p>
            <p class="user-role">{{ userData.role || 'Member' }}</p>
          </div>
          <hr class="dropdown-divider" />
          <button @click="logout" class="logout-btn">
             <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="currentColor" class="icon">
                <path fill-rule="evenodd" d="M7.5 3.75A1.5 1.5 0 006 5.25v13.5a1.5 1.5 0 001.5 1.5h6a1.5 1.5 0 001.5-1.5V15a.75.75 0 011.5 0v3.75a3 3 0 01-3 3h-6a3 3 0 01-3-3V5.25a3 3 0 013-3h6a3 3 0 013 3V9A.75.75 0 0115 9V5.25a1.5 1.5 0 00-1.5-1.5h-6zm10.72 4.72a.75.75 0 011.06 0l3 3a.75.75 0 010 1.06l-3 3a.75.75 0 11-1.06-1.06l1.72-1.72H9a.75.75 0 010-1.5h10.94l-1.72-1.72a.75.75 0 010-1.06z" clip-rule="evenodd" />
             </svg>
            <span>Logout</span>
          </button>
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

.avatar-wrapper {
  position: relative;
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
  transition: transform 0.2s;
}

.avatar:hover {
    transform: scale(1.05);
}

.avatar-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  background-color: var(--color-light);
  border: 1px solid var(--color-border);
  border-radius: 8px;
  padding: 8px;
  min-width: 200px;
  box-shadow: 0 8px 16px rgba(0, 0, 0, 0.1);
  z-index: 1000;
  margin-top: 10px;
}

.dropdown-header {
  padding: 8px;
  line-height: 1.4;
}

.user-name {
  font-weight: 600;
  color: var(--color-dark);
  margin: 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.user-role {
  font-size: 13px;
  color: var(--color-text-muted);
  margin: 0;
  text-transform: capitalize;
}

.dropdown-divider {
  margin: 8px 0;
  border: none;
  border-top: 1px solid var(--color-border);
}

.logout-btn {
  width: 100%;
  padding: 10px 8px;
  border: none;
  background: none;
  text-align: left;
  font-family: 'Poppins', sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: var(--color-dark);
  cursor: pointer;
  transition: background-color 0.2s, color 0.2s;
  border-radius: 6px; 
  display: flex;
  align-items: center; 
  gap: 10px; 
}

.logout-btn:hover {
  background-color: rgba(220, 53, 69, 0.1);
  color: #dc3545;
}

.logout-btn .icon {
    width: 18px;
    height: 18px;
}
</style>