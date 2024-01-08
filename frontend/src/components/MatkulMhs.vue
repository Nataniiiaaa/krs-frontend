<template>
  <div>
    <nav class="navbar navbar-dark bg-dark">
      <div class="container-fluid">
        <a class="navbar-brand" href="#">DETAIL SEMESTER</a>
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
  </div>
  <div class="container">
    <div class="d-flex justify-content-between my-3">
      <h2>Detail Matakuliah Mahasiswa</h2>
    </div>
    <div class="d-flex justify-content-between my-3">
      <h5>Tahun : {{ krs?.tahun }}</h5>
    </div>
    <div class="d-flex justify-content-between my-3">
      <h5>Semester : {{ krs?.semester }}</h5>
    </div>
    <div class="d-flex justify-content-between my-3">
      <h5>NIM : {{ krs?.mahasiswa.nim }}</h5>
    </div>
    <div class="d-flex justify-content-between my-3">
      <h5>Nama : {{ krs?.mahasiswa.nama }}</h5>
    </div>
    <button class="btn btn-danger mb-4" @click="goBack">Kembali</button>
    <div class="table-responsive shadow p-3 mb-5 bg-white rounded">
      <table class="table table-bordered table-striped">
        <thead class="thead-dark">
          <tr>
            <th scope="col">Kode Matakuliah</th>
            <th scope="col">Nama Matakuliah</th>
            <th scope="col">Nilai</th>
            <th scope="col">Predikat</th>
            <th scope="col">Action</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(detail, index) in krs?.detilkrss" :key="index">
            <td>{{ detail?.matakuliah.kode }}</td>
            <td>{{ detail?.matakuliah.namamatakuliah }}</td>
            <td>{{ detail?.nilai }}</td>
            <td>{{ getPredikat(detail?.nilai) }}</td>
            <td>
              <button type="button" class="btn btn-primary" data-bs-toggle="modal" data-bs-target="#exampleModal" @click="openModalEdit(detail?.id, detail?.matakuliah.namamatakuliah, detail?.nilai)">Edit</button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="modal fade" id="exampleModal" tabindex="-1" aria-labelledby="exampleModalLabel" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h1 class="modal-title fs-5" id="exampleModalLabel">Edit Nilai {{ modal.matakuliahName }}</h1>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <div class="input-group mb-3">
              <span class="input-group-text" id="inputGroup-sizing-default">Nilai Angka</span>
              <input type="text" v-model="modal.nilaiAngka" class="form-control" aria-label="Sizing example input" aria-describedby="inputGroup-sizing-default" />
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
            <button @click="submitModalEdit" type="button" class="btn btn-primary" data-bs-dismiss="modal">Save changes</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'redaxios';

export default {
  name: 'DetailMatakuliah',
  data() {
    return {
      krsId: this.$route.params.id,
      krs: null,
      modal: {
        nilaiAngka: null,
        matakuliahName: null,
        idDetailKrs: null,
      },
    };
  },
  created() {
    this.getDetailKRS();
  },
  methods: {
    goBack() {
      this.$router.go(-1);
    },
    openModalEdit(idDetailKrs, matakuliahName, nilai) {
      this.modal.matakuliahName = matakuliahName;
      this.modal.nilaiAngka = nilai.toString();
      this.modal.idDetailKrs = idDetailKrs;
      console.log(this.modal);
    },
    async submitModalEdit() {
      var url = `https://api-group7-prognet.manpits.xyz/api/detilkrs/${this.modal.idDetailKrs}`;
      await axios.put(url, {
        nilai: this.modal.nilaiAngka,
      });
      this.getDetailKRS();
      alert('Data berhasil di update');
    },
    async getDetailKRS() {
      var url = `https://api-group7-prognet.manpits.xyz/api/krs/${this.krsId}/detail`;
      const response = await axios.get(url);
      console.log(response.data);
      this.krs = response.data;
    },
    getPredikat(nilai) {
      if (nilai > 0 && nilai < 45) return 'E';
      else if (nilai >= 45 && nilai < 50) return 'D';
      else if (nilai >= 50 && nilai < 55) return 'D+';
      else if (nilai >= 55 && nilai < 60) return 'C';
      else if (nilai >= 60 && nilai < 65) return 'C+';
      else if (nilai >= 65 && nilai < 75) return 'B';
      else if (nilai >= 75 && nilai < 80) return 'B+';
      else if (nilai >= 80 && nilai <= 100) return 'A';
    },
  },
};
</script>
