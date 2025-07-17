<script setup>
import { ref } from 'vue';
import { useRouter } from 'vue-router';

// --- State & Router ---
const nama = ref('');
const email = ref('');
const password = ref('');
const passwordConfirmation = ref('');
const errorMessage = ref('');
const router = useRouter();
const API_URL = 'http://localhost:3000/api'; // Pastikan URL ini benar

// --- Fungsi untuk menangani pendaftaran ---
const handleRegister = async () => {
  errorMessage.value = ''; // Reset pesan error

  // Validasi frontend sederhana: pastikan password cocok
  if (password.value !== passwordConfirmation.value) {
    errorMessage.value = 'Password dan konfirmasi password tidak cocok.';
    return;
  }

  try {
    // PENTING: Ganti '/register' dengan endpoint registrasi Anda yang sebenarnya
    const response = await fetch(`${API_URL}/register`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        nama: nama.value,
        email: email.value,
        password: password.value,
        // Backend Anda mungkin memerlukan role_id, sesuaikan jika perlu
        // role_id: 2, // Contoh untuk role 'user'
      }),
    });

    const result = await response.json();

    if (!response.ok) {
      // Menangkap error dari server (misal: email sudah terdaftar)
      throw new Error(result.message || 'Pendaftaran gagal. Silakan coba lagi.');
    }

    // --- Logika setelah pendaftaran berhasil ---
    alert('Pendaftaran berhasil! Anda akan diarahkan ke halaman login.');

    // Arahkan ke halaman login setelah berhasil mendaftar
    router.push('/login');

  } catch (error) {
    console.error('Registration error:', error);
    errorMessage.value = error.message;
  }
};
</script>

<template>
  <div class="auth-page">
    <div class="auth-visual">
      <div class="visual-content">
        <h2>Bergabung dengan Komunitas Kami</h2>
        <p>Daftarkan diri Anda dan mulailah perjalanan literasi tanpa batas.</p>
      </div>
    </div>

    <div class="auth-form-wrapper">
      <div class="auth-form">
        <h1 class="form-title">Buat Akun Baru</h1>
        <p class="form-subtitle">Hanya butuh beberapa langkah untuk memulai.</p>

        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>

        <form @submit.prevent="handleRegister">
          <div class="form-group">
            <label for="nama" class="form-label">Nama Lengkap</label>
            <input type="text" id="nama" v-model="nama" class="form-input" placeholder="Nama Anda" required>
          </div>
          <div class="form-group">
            <label for="email" class="form-label">Email</label>
            <input type="email" id="email" v-model="email" class="form-input" placeholder="contoh@email.com" required>
          </div>
          <div class="form-group">
            <label for="password" class="form-label">Password</label>
            <input type="password" id="password" v-model="password" class="form-input" placeholder="••••••••" required>
          </div>
          <div class="form-group">
            <label for="passwordConfirmation" class="form-label">Konfirmasi Password</label>
            <input type="password" id="passwordConfirmation" v-model="passwordConfirmation" class="form-input" placeholder="••••••••" required>
          </div>
          <button type="submit" class="form-button">Daftar</button>
        </form>

        <p class="switch-page-link">
          Sudah punya akun? <router-link to="/login">Masuk di sini</router-link>
        </p>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* Style ini hanya untuk pesan error. Style utama lainnya berasal dari file CSS global Anda. */
.error-message {
  background-color: #f8d7da;
  color: #721c24;
  border: 1px solid #f5c6cb;
  padding: 1rem;
  border-radius: 8px;
  margin-bottom: 1.5rem;
  text-align: center;
  font-size: 0.9rem;
}
</style>
