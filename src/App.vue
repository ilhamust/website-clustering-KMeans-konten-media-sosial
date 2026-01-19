<template>
  <div>

    <!-- HERO -->
    <section class="hero">
      <div class="container">
        <div class="hero-content">
          <div class="hero-badge">Machine Learning Project</div>
          <h1>Clustering Konten Media Sosial</h1>
          <p>
            Analisis pengelompokan konten berdasarkan tingkat engagement
            menggunakan algoritma K-Means untuk mengidentifikasi konten viral, normal, dan sepi engagement.
          </p>
          <button @click="loadData" class="btn-primary" :disabled="loading">
            <span v-if="loading">Memuat Data...</span>
            <span v-else>{{ data.length ? 'Refresh Analisis' : 'Mulai Analisis' }}</span>
          </button>
        </div>
      </div>
    </section>

    <!-- RINGKASAN -->
    <section v-if="data.length" class="summary-section">
      <div class="container">
        <h2 class="section-title">Ringkasan Clustering</h2>
        <div class="summary">
          <div class="card viral">
            <div class="card-icon">🚀</div>
            <div class="card-content">
              <h3>Viral</h3>
              <p class="card-number">{{ countByLabel('Viral') }}</p>
              <p class="card-label">Konten dengan engagement tinggi</p>
            </div>
          </div>
          <div class="card normal">
            <div class="card-icon">📊</div>
            <div class="card-content">
              <h3>Normal</h3>
              <p class="card-number">{{ countByLabel('Normal') }}</p>
              <p class="card-label">Konten dengan engagement sedang</p>
            </div>
          </div>
          <div class="card sepi">
            <div class="card-icon">📉</div>
            <div class="card-content">
              <h3>Sepi</h3>
              <p class="card-number">{{ countByLabel('Sepi') }}</p>
              <p class="card-label">Konten dengan engagement rendah</p>
            </div>
          </div>
        </div>

        <!-- Statistics -->
        <div class="stats-grid">
          <div class="stat-card">
            <div class="stat-label">Total Konten</div>
            <div class="stat-value">{{ data.length }}</div>
          </div>
          <div class="stat-card">
            <div class="stat-label">Rata-rata Likes</div>
            <div class="stat-value">{{ averageLikes }}</div>
          </div>
          <div class="stat-card">
            <div class="stat-label">Rata-rata Comments</div>
            <div class="stat-value">{{ averageComments }}</div>
          </div>
          <div class="stat-card">
            <div class="stat-label">Rata-rata Shares</div>
            <div class="stat-value">{{ averageShares }}</div>
          </div>
        </div>
      </div>
    </section>

    <!-- TABEL -->
    <section v-if="data.length" class="table-section">
      <div class="container">
        <div class="table-header">
          <h2 class="section-title">Detail Hasil Clustering</h2>
          <div class="table-controls">
            <input 
              v-model="searchQuery" 
              type="text" 
              placeholder="Cari Post ID..." 
              class="search-input"
            />
            <select v-model="filterCluster" class="filter-select">
              <option value="">Semua Cluster</option>
              <option value="Viral">Viral</option>
              <option value="Normal">Normal</option>
              <option value="Sepi">Sepi</option>
            </select>
          </div>
        </div>
        
        <div class="table-wrapper">
          <table>
            <thead>
              <tr>
                <th>Post ID</th>
                <th>Likes</th>
                <th>Comments</th>
                <th>Shares</th>
                <th>Total Engagement</th>
                <th>Cluster</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in filteredData" :key="item.post_id" class="table-row">
                <td><strong>#{{ item.post_id }}</strong></td>
                <td>{{ formatNumber(item.likes) }}</td>
                <td>{{ formatNumber(item.comments) }}</td>
                <td>{{ formatNumber(item.shares) }}</td>
                <td><strong>{{ formatNumber(item.likes + item.comments + item.shares) }}</strong></td>
                <td>
                  <span :class="['badge', item.cluster_label.toLowerCase()]">
                    {{ item.cluster_label }}
                  </span>
                </td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="table-footer">
          <p>Menampilkan {{ filteredData.length }} dari {{ data.length }} konten</p>
        </div>
      </div>
    </section>

    <!-- Footer -->
    <footer v-if="data.length" class="footer">
      <p>© 2026 Clustering Dashboard | Machine Learning Project</p>
    </footer>

  </div>
</template>
<script>
export default {
  data() {
    return {
      data: [],
      loading: false,
      searchQuery: '',
      filterCluster: ''
    }
  },
  computed: {
    filteredData() {
      return this.data.filter(item => {
        const matchesSearch = item.post_id.toString().includes(this.searchQuery)
        const matchesFilter = !this.filterCluster || item.cluster_label === this.filterCluster
        return matchesSearch && matchesFilter
      })
    },
    averageLikes() {
      if (!this.data.length) return 0
      const sum = this.data.reduce((acc, item) => acc + item.likes, 0)
      return this.formatNumber(Math.round(sum / this.data.length))
    },
    averageComments() {
      if (!this.data.length) return 0
      const sum = this.data.reduce((acc, item) => acc + item.comments, 0)
      return this.formatNumber(Math.round(sum / this.data.length))
    },
    averageShares() {
      if (!this.data.length) return 0
      const sum = this.data.reduce((acc, item) => acc + item.shares, 0)
      return this.formatNumber(Math.round(sum / this.data.length))
    }
  },
  methods: {
    async loadData() {
      this.loading = true
      try {
        const res = await fetch('/clustering_result.json')
        this.data = await res.json()
      } catch (error) {
        console.error('Error loading data:', error)
      } finally {
        this.loading = false
      }
    },
    countByLabel(label) {
      return this.data.filter(d => d.cluster_label === label).length
    },
    formatNumber(num) {
      return new Intl.NumberFormat('id-ID').format(num)
    }
  }
}
</script>
<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html {
  margin: 0;
  padding: 0;
  overflow-x: hidden;
  width: 100%;
  max-width: 100%;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: #2563eb;
  color: #2c3e50;
  line-height: 1.6;
  margin: 0;
  padding: 0;
  min-height: 100vh;
  overflow-x: hidden;
  width: 100%;
  max-width: 100%;
}

#app {
  padding: 0;
  margin: 0;
  width: 100%;
  max-width: 100%;
  overflow-x: hidden;
}

/* Container */
.container {
  max-width: 1400px;
  margin: 0 auto;
  padding: 0 40px;
  width: 100%;
  box-sizing: border-box;
}

.container-wide {
  max-width: 1600px;
  margin: 0 auto;
  padding: 0 40px;
  width: 100%;
}

/* Hero Section */
.hero {
  background: transparent;
  color: white;
  padding: 60px 0 40px;
  text-align: center;
  position: relative;
  overflow: hidden;
  width: 100%;
  max-width: 100%;
}

.hero::before {
  display: none;
}

.hero-content {
  position: relative;
  z-index: 1;
  max-width: 800px;
  margin: 0 auto;
}

.hero-badge {
  display: inline-block;
  background: rgba(255, 255, 255, 0.25);
  padding: 8px 24px;
  border-radius: 50px;
  font-size: 13px;
  font-weight: 600;
  margin-bottom: 20px;
  backdrop-filter: blur(10px);
  letter-spacing: 0.5px;
}

.hero h1 {
  font-size: 42px;
  font-weight: 700;
  margin-bottom: 16px;
  letter-spacing: -0.5px;
}

.hero p {
  font-size: 17px;
  line-height: 1.7;
  margin-bottom: 28px;
  opacity: 0.95;
  max-width: 700px;
  margin-left: auto;
  margin-right: auto;
}

.btn-primary {
  background: white;
  color: #2563eb;
  border: none;
  padding: 14px 36px;
  border-radius: 8px;
  font-size: 15px;
  font-weight: 600;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.btn-primary:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.3);
}

.btn-primary:active:not(:disabled) {
  transform: translateY(0);
}

.btn-primary:disabled {
  opacity: 0.7;
  cursor: not-allowed;
}

/* Summary Section */
.summary-section {
  padding: 40px 0 60px;
  background: transparent;
  width: 100%;
  max-width: 100%;
  overflow: hidden;
}

.section-title {
  font-size: 28px;
  font-weight: 700;
  margin-bottom: 28px;
  text-align: center;
  color: white;
}

.summary {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 24px;
  margin-bottom: 40px;
}

.card {
  background: white;
  padding: 28px;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  border-left: 4px solid #ddd;
  display: flex;
  align-items: center;
  gap: 16px;
}

.card:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 16px rgba(0, 0, 0, 0.15);
}

.card-icon {
  font-size: 48px;
  line-height: 1;
}

.card-content {
  flex: 1;
}

.card h3 {
  font-size: 18px;
  font-weight: 600;
  margin-bottom: 8px;
  color: #2c3e50;
}

.card-number {
  font-size: 36px;
  font-weight: 700;
  margin-bottom: 4px;
  line-height: 1;
}

.card-label {
  font-size: 14px;
  color: #7f8c8d;
  margin: 0;
}

.card.viral {
  border-left-color: #10b981;
}

.card.viral .card-number {
  color: #10b981;
}

.card.normal {
  border-left-color: #f59e0b;
}

.card.normal .card-number {
  color: #f59e0b;
}

.card.sepi {
  border-left-color: #6b7280;
}

.card.sepi .card-number {
  color: #6b7280;
}

/* Stats Grid */
.stats-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 20px;
}

.stat-card {
  background: white;
  padding: 24px;
  border-radius: 10px;
  text-align: center;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
}

.stat-card:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.12);
}

.stat-label {
  font-size: 14px;
  color: #7f8c8d;
  margin-bottom: 8px;
  font-weight: 500;
}

.stat-value {
  font-size: 28px;
  font-weight: 700;
  color: #2563eb;
}

/* Table Section */
.table-section {
  padding: 40px 0 60px;
  background: transparent;
  width: 100%;
  max-width: 100%;
  overflow: hidden;
}

.table-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 32px;
  flex-wrap: wrap;
  gap: 20px;
}

.table-header .section-title {
  color: white;
  margin-bottom: 0;
}

.table-controls {
  display: flex;
  gap: 12px;
  flex-wrap: wrap;
}

.search-input,
.filter-select {
  padding: 12px 20px;
  border: 2px solid #e5e7eb;
  border-radius: 8px;
  font-size: 14px;
  transition: all 0.3s ease;
  background: white;
  font-family: inherit;
  color: #1f2937;
  font-weight: 500;
}

.search-input {
  min-width: 250px;
}

.search-input::placeholder {
  color: #9ca3af;
}

.search-input:focus,
.filter-select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.filter-select {
  cursor: pointer;
  min-width: 150px;
}

.table-wrapper {
  overflow-x: auto;
  border-radius: 12px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  background: white;
}

table {
  width: 100%;
  border-collapse: collapse;
  background: white;
}

thead {
  background: #1e40af;
  color: white;
}

thead th {
  padding: 16px;
  text-align: left;
  font-weight: 600;
  font-size: 14px;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

tbody td {
  padding: 16px;
  border-bottom: 1px solid #f3f4f6;
  font-size: 15px;
}

tbody tr.table-row {
  transition: all 0.2s ease;
}

tbody tr.table-row:hover {
  background: #f9fafb;
}

tbody tr:last-child td {
  border-bottom: none;
}

.badge {
  display: inline-block;
  padding: 6px 16px;
  border-radius: 20px;
  font-size: 13px;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.5px;
}

.badge.viral {
  background: #d1fae5;
  color: #065f46;
}

.badge.normal {
  background: #fef3c7;
  color: #92400e;
}

.badge.sepi {
  background: #e5e7eb;
  color: #374151;
}

.table-footer {
  margin-top: 20px;
  text-align: center;
  color: #7f8c8d;
  font-size: 14px;
}

/* Footer */
.footer {
  background: rgba(0, 0, 0, 0.2);
  color: white;
  padding: 30px 0;
  text-align: center;
  width: 100%;
  margin: 40px 0 0 0;
}

.footer p {
  margin: 0;
  opacity: 0.9;
}

/* Responsive Design */
@media (max-width: 768px) {
  .hero h1 {
    font-size: 32px;
  }

  .hero p {
    font-size: 16px;
  }

  .summary {
    grid-template-columns: 1fr;
  }

  .table-header {
    flex-direction: column;
    align-items: stretch;
  }

  .table-controls {
    flex-direction: column;
  }

  .search-input,
  .filter-select {
    width: 100%;
  }

  .section-title {
    font-size: 24px;
  }

  thead th,
  tbody td {
    padding: 12px 8px;
    font-size: 13px;
  }
}
</style>
