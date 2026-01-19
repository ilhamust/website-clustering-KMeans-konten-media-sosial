<template>
  <div class="container">
    <h1>Aplikasi Web Clustering Konten Media Sosial</h1>
    <p>Hasil analisis clustering menggunakan algoritma K-Means.</p>

    <button @click="loadData">Mulai Analisis</button>

    <table v-if="data.length">
      <thead>
        <tr>
          <th>Post ID</th>
          <th>Likes</th>
          <th>Comments</th>
          <th>Shares</th>
          <th>Cluster</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in data" :key="item.post_id">
          <td>{{ item.post_id }}</td>
          <td>{{ item.likes }}</td>
          <td>{{ item.comments }}</td>
          <td>{{ item.shares }}</td>
          <td>{{ item.cluster_label }}</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
export default {
  data() {
    return {
      data: []
    }
  },
  methods: {
    async loadData() {
      const response = await fetch('/clustering_result.json')
      this.data = await response.json()
    }
  }
}
</script>
<style>
.container {
  max-width: 900px;
  margin: 40px auto;
  font-family: Arial, sans-serif;
}

table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
}

th, td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: center;
}

th {
  background-color: #f4f4f4;
}
</style>
