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

    <main class="container mt-4 card shadow p-5 mb-5 bg-light rounded">
      <div class="mb-4">
        <h2 class="text-center text mb-3">Informasi Mahasiswa</h2>
        <template v-if="isLoading">
          <div class="d-flex justify-content-center" style="padding-top: 70px; height: 100vh; overflow-y: auto">
            <div class="spinner-border" role="status">
              <span class="visually-hidden">Loading...</span>
            </div>
          </div>
        </template>
        <table v-if="!isLoading" class="table table-bordered">
          <tr>
            <th style="width: 150px">NIM</th>
            <td>{{ mahasiswa?.nim }}</td>
          </tr>
          <tr>
            <th>Nama</th>
            <td>{{ mahasiswa?.nama }}</td>
          </tr>
          <tr>
            <th>Alamat</th>
            <td>{{ mahasiswa?.alamat }}</td>
          </tr>
          <tr>
            <th>Tanggal Lahir</th>
            <td>{{ mahasiswa?.lahir }}</td>
          </tr>
          <tr>
            <th>Agama</th>
            <td>{{ mahasiswa?.agama?.agama }}</td>
          </tr>
        </table>
      </div>

      <template v-if="!isLoading">
        <div>
          <h2 class="text-center text mb-3">Kartu Hasil Studi</h2>

          <div class="pb-4">
            <select v-model="selectedSemester">
              <option value="99">Semua Semester</option>
              <template v-for="(semester, index) in semesterList" :key="index">
                <option :value="semester">Semester {{ semester }}</option>
              </template>
            </select>
            <h4 v-if="selectedSemester == 99" class="mt-2 text mb-3">IPK: {{ getIpk }}</h4>
          </div>

          <div v-for="(semester, index) in filteredSemester" :key="index">
            <div>
              <h5 class="text mb-2">
                {{ `Semester ${selectedSemester == 99 ? index : selectedSemester}` }}
              </h5>
            </div>

            <table class="table table-striped table-bordered">
              <thead>
                <tr>
                  <th>Kode Matakuliah</th>
                  <th>Nama Matakuliah</th>
                  <th>SKS</th>
                  <th>Nilai</th>
                  <th>Predikat</th>
                  <th>Bobot</th>
                  <th>Total (Bobot x SKS)</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="(detail, index) in semester" :key="index">
                  <td>{{ detail.matakuliah.kode }}</td>
                  <td>{{ detail.matakuliah.namamatakuliah }}</td>
                  <td>{{ detail.matakuliah.sks }}</td>
                  <td>{{ detail.nilai }}</td>
                  <td>{{ getPredikat(detail.nilai) }}</td>
                  <td>
                    {{ getBobot(getPredikat(detail.nilai)) }}
                  </td>
                  <td>
                    {{ getTotal(getBobot(getPredikat(detail.nilai)), Number(detail.matakuliah.sks)) }}
                  </td>
                </tr>

                <tr>
                  <td colspan="2" style="text-align: center">Total</td>
                  <td>{{ getTotalSksPerSemester(semester) }}</td>
                  <td></td>
                  <td></td>
                  <td>{{ getTotalBobot(semester) }}</td>
                  <td>
                    {{ getTotalSumInSemester(semester) }}
                  </td>
                </tr>
              </tbody>
            </table>
            <div>
              <p class="text mb-2">IPS:{{ (getTotalSumInSemester(semester) / getTotalSksPerSemester(semester)).toFixed(2) }}</p>
            </div>
          </div>
        </div>
      </template>

      <div class="mt-4">
        <router-link to="/datamahasiswa" class="btn btn-danger">Kembali</router-link>
      </div>
    </main>
  </div>
</template>

<script>
import axios from 'redaxios';

export default {
  name: 'ShowMahasiswaView',
  data() {
    return {
      MahasiswaId: this.$route.params.id,
      mahasiswa: null,
      detailBySemester: null,
      isLoading: true,
      selectedSemester: 99,
    };
  },
  created() {
    this.getDetailMahasiswa();
  },
  mounted() {},
  computed: {
    isMahasiswaHasMatakuliah() {
      return this.getMatakuliah(this.MahasiswaId).length > 0;
    },
    semesterList() {
      return Object.keys(this.detailBySemester);
    },
    filteredSemester() {
      if (this.selectedSemester == 99) {
        return this.detailBySemester;
      } else {
        return [this.detailBySemester[this.selectedSemester]];
      }
    },
    getTotalSumInSemester(semester) {
      return (semester) => {
        let total = 0;
        semester.forEach((detail) => {
          const bobot = this.getBobot(this.getPredikat(detail.nilai));
          const sks = Number(detail.matakuliah.sks);
          total += bobot * sks;
        });
        return total;
      };
    },
    getIpk() {
      let total = 0;
      let totalSks = 0;
      Object.keys(this.detailBySemester).forEach((semester) => {
        const semesterTotal = this.getTotalSumInSemester(this.detailBySemester[semester]);
        const semesterSks = this.getTotalSksPerSemester(this.detailBySemester[semester]);
        total += semesterTotal;
        totalSks += semesterSks;
      });
      return (total / totalSks).toFixed(2);
    },
  },
  methods: {
    async getDetailMahasiswa() {
      var url = `https://api-group7-prognet.manpits.xyz/api/mahasiswa/${this.MahasiswaId}/detail`;
      const response = await axios.get(url);
      this.mahasiswa = response.data;
      this.detailBySemester = this.groupDetailkrssBySemester(this.mahasiswa);
      this.isLoading = false;
      console.log(this.detailBySemester);
    },

    groupDetailkrssBySemester(data) {
      const groupedBySemester = {};
      data.detailkrss.forEach((detailkrs) => {
        const semester = detailkrs.matakuliah.semester;
        if (!groupedBySemester[semester]) {
          groupedBySemester[semester] = [];
        }
        groupedBySemester[semester].push(detailkrs);
      });
      return groupedBySemester;
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

    getBobot(nilai) {
      const nilaiMapping = {
        E: 0,
        D: 1,
        'D+': 1.5,
        C: 2,
        'C+': 2.5,
        B: 3,
        'B+': 3.5,
        A: 4,
      };
      return nilaiMapping[nilai];
    },

    getTotalSksPerSemester(semester) {
      let totalSks = 0;
      semester.forEach((detail) => {
        totalSks += Number(detail.matakuliah.sks);
      });
      return totalSks;
    },

    getTotal(bobot, sks) {
      const bobotNilai = bobot * sks;
      return bobotNilai;
    },

    getTotalBobot(semester) {
      let totalBobotNilai = 0;
      semester.forEach((detail) => {
        const angka = this.getBobot(this.getPredikat(detail.nilai));
        totalBobotNilai += angka;
      });
      return totalBobotNilai;
    },

    getIPS(TotalBobotNilai, totalSks) {
      const IPS = totalSks !== 0 ? TotalBobotNilai / totalSks : 0;
      return IPS.toFixed(2);
    },

    logoutUser() {
      localStorage.removeItem('user');
      window.alert('Anda telah logout');
      this.$router.push('/login');
    },
  },
};
</script>
