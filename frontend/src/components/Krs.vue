<template>
  <div>
    <nav class="navbar navbar-dark bg-dark fixed-top">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">KRS</a>
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
                <router-link class="nav-link" to="/datamahasiswa">Data Mahasiswa</router-link>
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
        <h2>DATA KRS</h2>
        <router-link class="btn btn-primary" to="/tambahkrs">Tambah Data</router-link>
      </div>
      <div class="table-responsive shadow p-3 mb-5 bg-white rounded">
        <table class="table table-bordered table-striped">
          <thead class="thead-dark">
            <tr>
              <th scope="col">Tahun</th>
              <th scope="col">Semester</th>
              <th scope="col">Aksi</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="KRS in allKRS" :key="KRS.id">
              <td>{{ KRS.tahun }}</td>
              <td>{{ KRS.semester }}</td>
              <td>
                <div class="btn-group">
                  <router-link :to="{ name: 'EditKrs', params: { id: KRS.id } }" class="btn btn-warning">Edit</router-link>
                  <router-link :to="{ name: 'AddDetilKrs', params: { id: KRS.id } }" class="btn btn-info">Add Detil KRS</router-link>
                  <router-link :to="{ name: 'DetKrs', params: { id: KRS.id } }" class="btn btn-primary">Detail KRS</router-link>
                  <button type="button" class="btn btn-danger" @click="removeKrs(KRS)">Delete</button>
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'redaxios';

export default {
  name: 'KRS',
  data() {
    return {
      allKRS: [],
      KRS: {
        id: '',
        tahun: '',
        semester: '',
      },
    };
  },
  created() {
    console.log('Created !');
    this.loadAllKrs();
  },
  mounted() {
    console.log('Mounted !');
  },
  methods: {
    loadAllKrs() {
      var url = 'https://api-group7-prognet.manpits.xyz/api/krs';
      axios.get(url).then(({ data }) => {
        console.log(data);
        this.allKRS = data;
        this.sortByYearAndSemester();
      });
    },
    sortByYearAndSemester() {
      const ganjilFirst = (a, b) => {
        if (a.semester % 2 === 0 && b.semester % 2 !== 0) {
          return 1; // Jika b ganjil dan a genap, a di atas
        }
        if (a.semester % 2 !== 0 && b.semester % 2 === 0) {
          return -1; // Jika a ganjil dan b genap, b di atas
        }
        return 0; // Jika keduanya ganjil atau keduanya genap, biarkan urutan seperti itu
      };

      this.allKRS.sort((a, b) => {
        if (a.tahun !== b.tahun) {
          return a.tahun - b.tahun; // Mengurutkan tahun terlebih dahulua
        } else {
          return ganjilFirst(a, b); // Mengurutkan mulai dari Ganjil
        }
      });
    },
    removeKrs(KRS) {
      var url = `https://api-group7-prognet.manpits.xyz/api/krs/${KRS.id}`;
      axios
        .delete(url)
        .then(() => {
          console.log('Data Berhasil Dihapus !');
          this.loadAllKrs();
        })
        .catch((error) => {
          console.error('Error deleting data:', error);
        });
    },
    logoutUser() {
      localStorage.removeItem('user');
      window.alert('Anda telah logout');
      this.$router.push('/login');
    },
  },
};
</script>
