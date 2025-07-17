<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

// --- State Management ---
const email = ref('');
const password = ref('');
const errorMessage = ref('');
const router = useRouter();
const API_URL = 'http://localhost:3000/api'; // Pastikan URL ini benar

// --- Login Handler ---
const handleLogin = async () => {
  errorMessage.value = ''; // Reset pesan error setiap kali login dicoba
  try {
    const response = await fetch(`${API_URL}/user/login`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        email: email.value,
        password: password.value,
      }),
      // 'credentials: include' penting agar browser mengirim dan menerima cookies dari backend
      credentials: 'include',
    });

    const result = await response.json();

    // Jika response dari server tidak OK (misal: status 400, 404, 500)
    if (!response.ok) {
      throw new Error(result.msg || 'Login gagal. Periksa kembali email dan password Anda.');
    }

    // Backend menangani cookie, frontend hanya perlu memeriksa peran (role)
    // dan mengatur penanda login di localStorage.
    if (result.user && result.user.role === 'admin') {
      // Simpan data pengguna dan penanda login untuk UI dan router guards
      localStorage.setItem('userData', JSON.stringify(result.user));
      localStorage.setItem('userToken', 'true'); // Penanda sederhana bahwa user sudah login
      
      // Arahkan ke dashboard setelah login berhasil
      router.push('/dashboard');
    } else {
      // Jika user bukan admin atau tidak ada data user
      throw new Error('Akses ditolak. Akun ini bukan admin.');
    }

  } catch (error) {
    console.error('Login error:', error);
    // Tampilkan pesan error ke pengguna
    errorMessage.value = error.message;
  }
};
</script>

<template>
  <div class="login-page">
    <div class="login-visual">
      <div class="visual-content">
        <h2>PerpusKita - Admin Panel</h2>
        <p>Area khusus untuk manajemen dan administrasi sistem perpustakaan.</p>
      </div>
    </div>

    <div class="login-form-wrapper">
      <div class="login-form">
        <h1 class="form-title">Admin Login</h1>
        <p class="form-subtitle">Silakan masuk untuk mengakses dashboard.</p>
        
        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>
        
        <form @submit.prevent="handleLogin">
          <div class="form-group">
            <label for="email" class="form-label">Email</label>
            <input 
              type="email" 
              id="email" 
              v-model="email" 
              class="form-input" 
              placeholder="admin@email.com" 
              required 
            />
          </div>
          <div class="form-group">
            <label for="password" class="form-label">Password</label>
            <input 
              type="password" 
              id="password" 
              v-model="password" 
              class="form-input" 
              placeholder="••••••••" 
              required 
            />
          </div>
          <button type="submit" class="form-button">Login sebagai Admin</button>
        </form>

        <p class="switch-page-link">
          Bukan admin? <router-link to="/register">Daftar sebagai pengguna</router-link>
        </p>

      </div>
    </div>
  </div>
</template>

<style scoped>
/* Anda bisa menaruh semua style yang sudah Anda buat sebelumnya di sini */
.login-page {
  display: flex;
  height: 100vh;
  font-family: 'Poppins', sans-serif; /* Pastikan font ini diimpor */
}

.login-visual {
  flex: 1;
  background: linear-gradient(rgba(0, 0, 0, 0.5), rgba(0, 0, 0, 0.5)), url('https://images.unsplash.com/photo-1521587760476-6c12a4b040da?crop=entropy&cs=tinysrgb&fit=max&fm=jpg&ixid=MnwxfDB8MXxyYW5kb218MHx8fHx8fHx8MTY3OTg2MDYyMA&ixlib=rb-4.0.3&q=80&utm_campaign=api-credit&utm_medium=referral&utm_source=unsplash_source&w=1080');
  background-size: cover;
  background-position: center;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  padding: 2rem;
}

.visual-content h2 {
  font-size: 2.5rem;
  font-weight: 700;
  margin-bottom: 1rem;
}

.login-form-wrapper {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #f9fafb;
}

.login-form {
  width: 100%;
  max-width: 400px;
  padding: 2rem;
}

.form-title {
  font-size: 2rem;
  font-weight: 600;
  color: #1f2937;
  margin-bottom: 0.5rem;
}

.form-subtitle {
  color: #6b7280;
  margin-bottom: 2rem;
}

.form-group {
  margin-bottom: 1.5rem;
}

.form-label {
  display: block;
  font-weight: 500;
  color: #374151;
  margin-bottom: 0.5rem;
}

.form-input {
  width: 100%;
  padding: 0.75rem 1rem;
  border: 1px solid #d1d5db;
  border-radius: 8px;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.form-input:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.2);
}

.form-button {
  width: 100%;
  padding: 0.85rem;
  background-color: #3b82f6;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  cursor: pointer;
  transition: background-color 0.2s;
}

.form-button:hover {
  background-color: #2563eb;
}

.error-message {
  background-color: #fee2e2;
  color: #b91c1c;
  border: 1px solid #fecaca;
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 1.5rem;
  text-align: center;
  font-size: 0.9rem;
}

.switch-page-link {
  margin-top: 2rem;
  text-align: center;
  font-size: 0.9rem;
  color: #6b7280;
}

.switch-page-link a {
  color: #3b82f6;
  font-weight: 600;
  text-decoration: none;
}

.switch-page-link a:hover {
  text-decoration: underline;
}
</style>