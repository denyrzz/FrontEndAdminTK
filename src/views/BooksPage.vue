<script setup>
import { ref, onMounted, reactive } from 'vue';
import Modal from '../components/Modal.vue';

const API_URL = 'http://localhost:3000/api';
const books = ref([]);
const categories = ref([]);
const isModalOpen = ref(false);
const isEditing = ref(false); // Ref for reactive computed property for conditional rendering of button text.

const currentBook = reactive({
  id_buku: null,
  judul: '',
  penulis: '',
  penerbit: '',
  kategori_id: null,
  stok: 0,
  tahun_terbit: new Date().getFullYear(),
  cover: '',
  sinopsis: '',
});

const fetchBooks = async () => {
  try {
    const response = await fetch(`${API_URL}/buku`, { credentials: 'include' });
    if (!response.ok) {
      const errorData = await response.json();
      throw new Error(errorData.msg || 'Gagal mengambil data buku');
    }
    const dataFromApi = await response.json();
    const processedBooks = dataFromApi.map(book => ({
      id: book.id || book.id_buku, // Ensure consistency, use id_buku if id isn't present
      title: book.judul || book.title,
      author: book.penulis || book.author,
      category: book.category || book.kategori_nama, // Assuming kategori_nama comes with book list
      stok: book.stok,
      coverUrl: book.cover || 'https://placehold.co/100x150/e2e8f0/667085?text=No+Image',
    }));
    books.value = processedBooks;
  } catch (error) {
    console.error('Error fetching books:', error);
    alert('Gagal memuat data buku: ' + error.message);
  }
};

const fetchCategories = async () => {
  try {
    const response = await fetch(`${API_URL}/kategori`, { credentials: 'include' });
    if (!response.ok) {
      throw new Error(`Gagal mengambil data kategori: ${response.statusText}`);
    }
    const data = await response.json();
    if (!Array.isArray(data) || data.length === 0) {
      throw new Error('Tidak ada data kategori yang tersedia.');
    }
    categories.value = data;
  } catch (error) {
    console.error('Error fetching categories:', error);
    alert('Gagal memuat daftar kategori. Silakan coba lagi.');
  }
};

const saveBook = async () => {
  // Validate kategori_id
  if (!currentBook.kategori_id || isNaN(currentBook.kategori_id)) {
    alert('Silakan pilih kategori yang valid.');
    return;
  }
  const bookData = {
    judul: currentBook.judul,
    penulis: currentBook.penulis,
    penerbit: currentBook.penerbit,
    kategori_id: parseInt(currentBook.kategori_id), // Ensure integer
    tahun_terbit: currentBook.tahun_terbit,
    stok: currentBook.stok,
    cover: currentBook.cover || 'https://placehold.co/100x150/e2e8f0/667085?text=No+Image',
    sinopsis: currentBook.sinopsis,
  };
  try {
    let response;
    let url = `${API_URL}/buku`;
    let method = 'POST';
    if (isEditing.value) {
      url = `${API_URL}/buku/${currentBook.id_buku}`; // Use id_buku for update URL
      method = 'PUT';
    }
    response = await fetch(url, {
      method,
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(bookData),
      credentials: 'include',
    });
    if (!response.ok) {
      const errorResult = await response.json();
      throw new Error(errorResult.msg || 'Gagal menyimpan data buku');
    }
    await fetchBooks();
    closeModal();
    alert(isEditing.value ? 'Buku berhasil diperbarui.' : 'Buku berhasil ditambahkan.');
  } catch (error) {
    console.error('Error saving book:', error);
    alert(`Terjadi kesalahan saat menyimpan data: ${error.message}`);
  }
};

const deleteBook = async (id) => {
  if (!id || isNaN(id)) {
    alert('ID buku tidak valid.');
    return;
  }
  if (!confirm('Apakah Anda yakin ingin menghapus buku?')) return;
  try {
    const response = await fetch(`${API_URL}/buku/${id}`, {
      method: 'DELETE',
      credentials: 'include',
    });
    if (!response.ok) {
      const errorResult = await response.json();
      throw new Error(errorResult.msg || 'Gagal menghapus buku');
    }
    const result = await response.json();
    alert(result.msg || 'Buku berhasil dihapus.');
    await fetchBooks();
  } catch (error) {
    console.error('Error deleting book:', error);
    alert(`Terjadi kesalahan saat menghapus data: ${error.message}`);
  }
};

const resetCurrentBook = () => {
  Object.assign(currentBook, {
    id_buku: null,
    judul: '',
    penulis: '',
    penerbit: '',
    kategori_id: null, // Reset to null to show placeholder
    stok: 0,
    tahun_terbit: new Date().getFullYear(),
    cover: '',
    sinopsis: '',
  });
};

const openAddModal = async () => {
  isEditing.value = false;
  resetCurrentBook();
  await fetchCategories(); // Fetch categories to ensure the list is populated
  isModalOpen.value = true;
};

const openEditModal = async (book) => {
  try {
    // Fetch full book data using the ID from the list
    const response = await fetch(`${API_URL}/buku/${book.id}`, { credentials: 'include' });
    if (!response.ok) throw new Error('Gagal mengambil detail buku');
    const fullBookData = await response.json();
    isEditing.value = true;
    await fetchCategories(); // Fetch categories to ensure the list is populated
    Object.assign(currentBook, {
      id_buku: fullBookData.id_buku, 
      judul: fullBookData.judul || fullBookData.title || '',
      penulis: fullBookData.penulis || fullBookData.author || '',
      penerbit: fullBookData.penerbit ,
      kategori_id: fullBookData.kategori_id || null,
      stok: fullBookData.stok || 0,
      tahun_terbit: fullBookData.tahun_terbit || new Date().getFullYear(),
      cover: fullBookData.cover || '',
      sinopsis: fullBookData.sinopsis || '',
    });
    isModalOpen.value = true;
  } catch (error) {
    console.error('Error fetching book details:', error);
    alert('Tidak dapat memuat data detail buku.');
  }
};

const closeModal = () => {
  isModalOpen.value = false;
};

onMounted(() => {
  fetchCategories();
  fetchBooks();
});
</script>
<template>
  <div class="page-container">
    <div class="page-header">
      <h1>Kelola Buku</h1>
      <button @click="openAddModal" class="btn btn-primary">Tambah Buku</button>
    </div>
    <div class="table-container">
      <table>
        <thead>
          <tr>
            <th>Cover</th>
            <th>ID</th>
            <th>Judul</th>
            <th>Penulis</th>
            <th>Kategori</th>
            <th>Stok</th>
            <th>Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr v-if="books.length === 0">
            <td colspan="6" class="text-center">Memuat data atau tidak ada data...</td>
          </tr>
          <tr v-for="book in books" :key="book.id">
            <td>
              <img :src="book.coverUrl" :alt="book.title" class="book-cover-thumbnail" 
                   onerror="this.onerror=null;this.src='https://placehold.co/100x150/e2e8f0/667085?text=Error';">
            </td>
            <td>{{ book.id }}</td>
            <td>{{ book.title }}</td>
            <td>{{ book.author }}</td>
            <td>{{ book.category }}</td>
            <td>{{ book.stok }}</td>
            <td class="action-buttons">
              <button @click.stop="openEditModal(book)" class="btn btn-secondary" title="Edit">E</button>
              <button @click="deleteBook(book.id)" class="btn btn-danger" title="Delete">D</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
    <Modal :show="isModalOpen" @close="closeModal">
      <template #header>
        <h3>{{ isEditing ? 'Edit Buku' : 'Tambah Buku Baru' }}</h3>
      </template>
      <template #body>
        <div class="form-group">
          <label for="judul">Judul</label>
          <input type="text" id="judul" v-model="currentBook.judul" class="form-input" required>
        </div>
        <div class="form-group">
          <label for="penulis">Penulis</label>
          <input type="text" id="penulis" v-model="currentBook.penulis" class="form-input" required>
        </div>
        <div class="form-group">
          <label for="penerbit">Penerbit</label>
          <input type="text" id="penerbit" v-model="currentBook.penerbit" class="form-input" required>
        </div>
        <div class="form-group">
          <label for="kategori">Kategori</label>
          <select id="kategori" v-model="currentBook.kategori_id" class="form-input" required>
            <option value="" disabled>Pilih Kategori</option>
            <option v-for="category in categories" :key="category.id_kategori" :value="category.id_kategori">
              {{ category.nama_kategori }}
            </option>
          </select>
          <small v-if="categories.length === 0" class="error-text">Tidak ada kategori tersedia. Silakan periksa koneksi API.</small>
        </div>
        <div class="form-group-row">
          <div class="form-group">
            <label for="tahun_terbit">Tahun Terbit</label>
            <input type="number" id="tahun_terbit" v-model.number="currentBook.tahun_terbit" class="form-input" required>
          </div>
          <div class="form-group">
            <label for="stok">Stok</label>
            <input type="number" id="stok" v-model.number="currentBook.stok" class="form-input" required>
          </div>
        </div>
        <div class="form-group">
          <label for="cover">URL Gambar Sampul</label>
          <input type="text" id="cover" v-model="currentBook.cover" class="form-input" placeholder="Masukkan URL gambar">
        </div>
        <div class="form-group">
          <label for="sinopsis">Sinopsis</label>
          <textarea id="sinopsis" v-model="currentBook.sinopsis" class="form-input" rows="4"></textarea>
        </div>
      </template>
      <template #footer>
        <button @click="closeModal" class="btn btn-secondary">Batal</button>
        <button @click="saveBook" class="btn btn-primary">Simpan</button>
      </template>
    </Modal>
  </div>
</template>

<style scoped>
.error-text {
  color: red;
  font-size: 0.8rem;
  margin-top: 0.25rem;
}
</style>