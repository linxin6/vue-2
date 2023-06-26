<template>
  <div class="main-container">
    <h1>查询消费记录</h1>
    <div class="balance">
      <p class="balance">当前余额: {{ balance }}</p>
    </div>
    <div class="controls">
      <input type="text" v-model="key" placeholder="请输入 key" class="key-input">
      <button @click="query">查询</button>
      <button @click="purchase">购买 key</button>
    </div>
    <div class="results">
      <table v-if="paginatedRecords.length > 0" class="results-table">
        <thead>
        <tr>
          <th v-for="header in headers" :key="header" :class="{ 'date-col': header === 'Date' }">
            {{ header }}
          </th>
        </tr>
        </thead>
        <tbody>
        <tr v-for="record in paginatedRecords" :key="record.id">
          <td v-for="(item, index) in record.slice(1)" :key="index" :class="{ 'date-col': headers[index] === 'Date' }">
            {{ item }}
          </td>
        </tr>
        </tbody>
      </table>
      <div v-if="numPages > 1" class="pagination">
        <button @click="prevPage">上一页</button>
        <p>当前页: {{ currentPage + 1 }} / {{ numPages }}</p>
        <button @click="nextPage">下一页</button>
      </div>
      <p v-else>未找到消费记录</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      key: '',
      balance: '',
      records: [],
      headers: ['Key', '模型(Model)', '消费', '时间', '问题(prompt tokens)', '回答(completion tokens)'],
      currentPage: 0,
      itemsPerPage: 10,
    }
  },
  computed: {
    paginatedRecords() {
      const start = this.currentPage * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.records.slice(start, end);
    },
    numPages() {
      return Math.ceil(this.records.length / this.itemsPerPage);
    },
  },
  methods: {
    async getBalance() {
      const apiUrl = `https://zcl3.icu/get_key_balance?key=${this.key}`;
      try {
        const response = await fetch(apiUrl);
        const data = await response.json();
        if (data && data.balance) {
          this.balance = data.balance;
        } else {
          alert('无法获取余额');
        }
      } catch (error) {
        alert('查询余额过程中出错，请稍后再试');
      }
    },
    async query() {
      this.getBalance();
      this.records = []; // 清空记录
      const apiUrl = `https://zcl3.icu/get_transactions?key=${this.key}`;
      try {
        const response = await fetch(apiUrl);
        const data = await response.json();
        if (data && data.length > 0) {
          this.records = data;
        } else {
          alert('未找到消费记录');
        }
      } catch (error) {
        alert('查询过程中出错，请稍后再试');
      }
    },

    purchase() {
      window.location.href = 'purchase.html';
    },
    nextPage() {
      if (this.currentPage < this.numPages - 1) {
        this.currentPage++;
      }
    },
    prevPage() {
      if (this.currentPage > 0) {
        this.currentPage--;
      }
    },
  },
}
</script>

<style scoped>
.main-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 10px;
}

.controls {
  display: flex;
  align-items: center;
  justify-content: center;
}
.controls input,
.controls button {
  margin-right: 10px;
}

.key-input {
  width: 200%;
}

.balance {
  margin-bottom: 10px;
}

.results-table {
  border-collapse: collapse;
  width: 100%;
  text-align: center;
}

.results-table thead th {
  background-color: #f2f2f2;
  padding: 10px;
}

.results-table tbody td {
  border: 1px solid #ddd;
  padding: 8px;
}

.results-table tbody tr:nth-child(even) {
  background-color: #f2f2f2;
}

.results p {
  text-align: center;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 20px 0;
}

button {
  margin-top: 10px;
  margin-left: 10px;
  padding: 10px;
  background-color: #4CAF50;
  color: white;
  border: none;
  cursor: pointer;
  border-radius: 5px;
}

button:hover {
  background-color: #45a049;
}
</style>
