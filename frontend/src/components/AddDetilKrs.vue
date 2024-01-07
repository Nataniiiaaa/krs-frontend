<template>
    <div class="container mt-4">
        <div class="card shadow p-3 mb-5 bg-white rounded">
            <div class="card-body">
                <h2 class="card-title">TAMBAH DETIL KRS</h2>
                <form @submit.prevent="addDetilKrs">
                  <div class="mb-3">
                      <label for="krs_id" class="form-label">Tahun - Semester</label>
                      <select class="form-select" v-model="KrsId">
                          <option v-for="krs in KRSList" :key="krs.id" :value="krs.id">
                              {{ krs.tahun }} - {{ krs.semester }}
                          </option>
                      </select>
                  </div>
                    <div class="mb-3">
                        <label for="kodeMatkul" class="form-label">Kode Matakuliah</label>
                        <select class="form-select" v-model="detilKrs.matakuliah_id" required>
                        <option v-for="matakuliah in MatakuliahList" :key="matakuliah.id" :value="matakuliah.id">
                            {{ matakuliah.kode }} - {{ matakuliah.namamatakuliah }}
                        </option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label for="nim" class="form-label">NIM Mahasiswa</label>
                        <select class="form-select" v-model="detilKrs.mahasiswa_id" required>
                            <option v-for="mahasiswa in MahasiswaList" :key="mahasiswa.id" :value="mahasiswa.id">
                                {{ mahasiswa.nim }} - {{ mahasiswa.nama }}
                            </option>
                        </select>
                    </div>
                    <div class="mb-3">
                        <label for="nilai" class="form-label">Nilai</label>
                        <input type="text" class="form-control" v-model="detilKrs.nilai" required>
                    </div>

                    <div class="btn-group">
                        <button type="submit" class="btn btn-primary ">Tambah Data</button>
                        <router-link class="btn btn-danger" to="/krs">Kembali</router-link>
                    </div>
                </form>
            </div>
        </div>
    </div>
</template>
  
  <script>
  import axios from 'redaxios';
  
  export default {
    data() {
      return {
        KrsId: '',
        detilKrs: {
          'krs_id': '',
          'matakuliah_id': '',
          'mahasiswa_id': '',
          'nilai': '',
        },
        MatakuliahList: [],
        MahasiswaList: [],
        KRSList: [],
      };
    },
    created() {
      this.loadMatakuliahList();
      this.loadMahasiswaList();
      this.loadKrsList();
      this.KrsId = this.$route.params.id;
    },
    methods: {
      loadMatakuliahList() {
        var matkulurl = 'https://api-group7-prognet.manpits.xyz/api/matakuliah';
        axios.get(matkulurl).then(({ data }) => {
          this.MatakuliahList = data;
        });
      },
      loadMahasiswaList() {
        var mhsurl = 'https://api-group7-prognet.manpits.xyz/api/mahasiswa';
        axios.get(mhsurl).then(({ data }) => {
          this.MahasiswaList = data;
        });
      },
      loadKrsList() {
        var krsurl = 'https://api-group7-prognet.manpits.xyz/api/krs';
        axios.get(krsurl).then(({ data }) => {
          this.KRSList = data;
        });
      },
      addDetilKrs() {
        this.detilKrs.krs_id = this.KrsId;
        console.log('Data to be submitted:', this.detilKrs);
        var url = 'https://api-group7-prognet.manpits.xyz/api/detilkrs';
        axios.post(url, this.detilKrs).then(() => {
          console.log('Data Berhasil Ditambahkan!');
          // Setelah berhasil menambahkan, bisa kembali ke halaman list atau lakukan aksi lainnya
          this.$router.push('/detilkrs');
        }).catch((error) => {
          console.error('Error adding data:', error);
        });
      },
    },
  };
  </script>
  