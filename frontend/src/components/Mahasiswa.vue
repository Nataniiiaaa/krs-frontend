<template>
  <div>
    <nav class="navbar navbar-dark bg-dark fixed-top">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">KRS ADMIN</a>
        <button class="navbar-toggler" type="button" data-bs-toggle="offcanvas" data-bs-target="#offcanvasDarkNavbar" aria-controls="offcanvasDarkNavbar" aria-label="Toggle navigation">
          <span class="navbar-toggler-icon"></span>
        </button>
        <div class="offcanvas offcanvas-end text-bg-dark" tabindex="-1" id="offcanvasDarkNavbar" aria-labelledby="offcanvasDarkNavbarLabel">
          <div class="offcanvas-header">
            <h5 class="offcanvas-title" id="offcanvasDarkNavbarLabel">Menu</h5>
            <button type="button" class="btn-close btn-close-white" data-bs-dismiss="offcanvas" aria-label="Close"></button>
          </div>
          <div class="offcanvas-body">
            <ul class="navbar-nav justify-content-end flex-grow-1 pe-3">
              <li class="nav-item">
                <router-link class="nav-link" to="/dashboard">Dashboard</router-link>
              </li>
              <li class="nav-item">
                <router-link class="nav-link active" to="/datamahasiswa">Data Mahasiswa</router-link>
              </li>
              <li class="nav-item">
                <router-link class="nav-link" to="/matakuliah">Data Matakuliah</router-link>
              </li>
              <li class="nav-item dropdown">
                <router-link to="#" class="nav-link dropdown-toggle" role="button" data-bs-toggle="dropdown" aria-expanded="false"> Data KRS </router-link>
                <ul class="dropdown-menu dropdown-menu-dark">
                  <li><router-link to="/krs" class="dropdown-item">KRS</router-link></li>
                  <li><router-link to="/detilkrs" class="dropdown-item">Detail KRS</router-link></li>
                </ul>
              </li>

              <li class="d-flex justify-content-between my-3" style="text-align: left">
                <button type="button" class="btn btn-outline-danger" style="background-color: red; color: white" @click="logoutUser">Logout</button>
              </li>
            </ul>
          </div>
        </div>
      </div>
    </nav>
    <div class="container" style="padding-top: 70px; height: 100vh; overflow-y: auto">
      <div class="d-flex justify-content-between my-3">
        <h2>DATA MAHASISWA</h2>
        <router-link class="btn btn-primary" to="/tambahmahasiswa">Tambah Data</router-link>
      </div>
      <div class="table-responsive shadow p-3 mb-5 bg-white rounded">
        <DataTable removableSort class="shadow-sm rounded-lg overflow-clip" v-model:filters="filters" :value="getDataMahasiswa" paginator :rows="10" :rowsPerPageOptions="[5, 10, 20, 50]" tableStyle="min-width: 50rem">
          <template #header="slotProps">
            <div class="flex items-center gap-4">
              <div class="w-full">
                <label for="default-search" class="mb-2 text-sm font-semibold text-gray-900 sr-only dark:text-white">Search</label>
                <div class="relative">
                  <input
                    v-model="filters['global'].value"
                    type="text"
                    id="default-search"
                    class="block w-full p-4 ps-10 text-sm text-gray-900 border border-gray-300 rounded-lg bg-gray-50 focus:ring-blue-500 focus:border-blue-500 dark:bg-gray-700 dark:border-gray-600 dark:placeholder-gray-400 dark:text-white dark:focus:ring-blue-500 dark:focus:border-blue-500"
                    placeholder="Masukkan kata kunci..."
                  />
                </div>
              </div>
            </div>
          </template>
          <template #loading>
            <span class="dark:text-gray-300 font-bold"> Loading data, please wait. </span>
          </template>

          <Column field="nim" sortable header="NIM"></Column>
          <Column field="nama" sortable header="Nama Lengkap"></Column>
          <Column field="alamat" sortable header="Alamat"></Column>
          <Column field="lahir" sortable header="Tanggal Lahir"></Column>
          <Column field="agama" sortable header="Agama"> </Column>

          <Column header="Action">
            <template #body="slotProps">
              <div class="btn-group">
                <router-link
                  :to="{
                    name: 'EditMahasiswa',
                    params: { id: slotProps.data.id },
                  }"
                  class="btn btn-warning"
                  >Edit</router-link
                >
                <router-link
                  :to="{
                    name: 'DetailMahasiswa',
                    params: { id: slotProps.data.id },
                  }"
                  class="btn btn-info"
                  >Detail</router-link
                >
                <button type="button" class="btn btn-danger" @click="removeMahasiswa(slotProps.data)">Delete</button>
              </div>
            </template>
          </Column>
        </DataTable>
      </div>
    </div>
  </div>
</template>

<script>
import EditMahasiswa from '../components/EditMahasiswa.vue';
import axios from 'redaxios';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import { FilterMatchMode } from 'primevue/api';

export default {
  name: 'Mahasiswa',
  data() {
    return {
      allMahasiswa: [],
      agamaList: [],
      Mahasiswa: {
        id: '',
        nim: '',
        nama: '',
        alamat: '',
        lahir: '',
        agama_id: '',
      },
      filters: { global: { value: null, matchMode: FilterMatchMode.CONTAINS } },
      isLoading: true,
    };
  },
  created() {
    this.loadAllMahasiswa();
    this.loadAgamaList();
  },
  computed: {
    getDataMahasiswa() {
      const data = this.allMahasiswa;
      data.forEach((item) => {
        item.agama = this.getAgamaName(item.agama_id);
      });
      this.isLoading = false;
      return data;
    },
  },
  methods: {
    isValidIndex(index) {
      return Number.isInteger(index) && index >= 0;
    },
    loadAllMahasiswa() {
      var url = 'https://api-group7-prognet.manpits.xyz/api/mahasiswa';
      axios.get(url).then(({ data }) => {
        this.allMahasiswa = data;
        this.sortMahasiswa(); // Panggil fungsi sort setelah memuat data
      });
    },
    sortMahasiswa() {
      // Fungsi untuk menyortir array mahasiswa berdasarkan NIM
      this.allMahasiswa.sort((a, b) => {
        return a.nim.localeCompare(b.nim, undefined, {
          numeric: true,
          sensitivity: 'base',
        });
      });
    },
    loadAgamaList() {
      var agamaUrl = 'https://api-group7-prognet.manpits.xyz/api/agama';
      axios.get(agamaUrl).then(({ data }) => {
        this.agamaList = data;
      });
    },
    getAgamaName(agama_id) {
      const agama = this.agamaList.find((agama) => agama.id === agama_id);
      return agama ? agama.agama : 'Tidak Diketahui';
    },
    removeMahasiswa(mahasiswa) {
      var url = `https://api-group7-prognet.manpits.xyz/api/mahasiswa/${mahasiswa.id}`;
      axios
        .delete(url)
        .then(() => {
          console.log('Data Berhasil Dihapus !');
          this.loadAllMahasiswa(); // Memanggil kembali data setelah menghapus
        })
        .catch((error) => {
          console.error('Error deleting data:', error);
        });
    },
    editMahasiswa(Mahasiswa) {
      this.$router.push({
        name: 'EditMahasiswa',
        params: { id: Mahasiswa.id },
      });
    },
    logoutUser() {
      localStorage.removeItem('user');
      window.alert('Anda telah logout');
      this.$router.push('/login');
    },
  },
  components: {
    EditMahasiswa,
    DataTable,
    Column,
  },
};
</script>
