<script setup>
import { ref, onMounted, watch } from 'vue';
import { Chart, registerables } from 'chart.js';

Chart.register(...registerables);

const expenses = ref([
  { id: 1, description: 'Aluguel', amount: 1500, category: 'Moradia' },
  { id: 2, description: 'Compras', amount: 250, category: 'Alimentação' },
  { id: 3, description: 'Gasolina', amount: 180, category: 'Transporte' },
  { id: 4, description: 'Cinema', amount: 50, category: 'Lazer' },
]);

const newExpense = ref({
  description: '',
  amount: null,
  category: 'Alimentação',
});

const chartCanvas = ref(null);
const chartInstance = ref(null);

const editingExpenseId = ref(null);

const addExpense = () => {
    const description = newExpense.value.description;
    const amount = newExpense.value.amount;

    if (!description || description.trim() === '') {
        alert('Por favor, digite a descrição da despesa.');
        return;
    }

    if (amount === null || isNaN(amount) || amount <= 0) {
        alert('Por favor, digite um valor válido maior que zero.');
        return;
    }

    if (editingExpenseId.value) {
        const index = expenses.value.findIndex(exp => exp.id === editingExpenseId.value);
        if (index !== -1) {
            expenses.value[index].description = description.trim();
            expenses.value[index].amount = parseFloat(amount);
            expenses.value[index].category = newExpense.value.category;
        }
        editingExpenseId.value = null;
    } else {
        const expenseToAdd = {
            id: Date.now(),
            description: description.trim(),
            amount: parseFloat(amount),
            category: newExpense.value.category,
        };
        expenses.value.push(expenseToAdd);
    }
    
    newExpense.value.description = '';
    newExpense.value.amount = null;
};

const startEdit = (expense) => {
    editingExpenseId.value = expense.id;
    newExpense.value.description = expense.description;
    newExpense.value.amount = expense.amount;
    newExpense.value.category = expense.category;
};

const deleteExpense = (id) => {
    if (confirm('Tem certeza que deseja excluir esta despesa?')) {
        expenses.value = expenses.value.filter(expense => expense.id !== id);
    }
};

const createChartData = () => {
    const expensesByCategory = expenses.value.reduce((acc, expense) => {
        acc[expense.category] = (acc[expense.category] || 0) + expense.amount;
        return acc;
    }, {});

    const labels = Object.keys(expensesByCategory);
    const data = Object.values(expensesByCategory);

    return { labels, data };
};

onMounted(() => {
    if (!chartCanvas.value) return;
    const ctx = chartCanvas.value.getContext('2d');
    const { labels, data } = createChartData();
    chartInstance.value = new Chart(ctx, {
        type: 'doughnut',
        data: {
            labels: labels,
            datasets: [{
                data: data,
                backgroundColor: [
                    '#3498db', '#e74c3c', '#2ecc71', '#f1c40f', '#9b59b6'
                ],
                hoverOffset: 4
            }]
        },
        options: {
            responsive: true,
            plugins: {
                legend: {
                    position: 'top',
                },
                title: {
                    display: true,
                    text: 'Distribuição de Gastos por Categoria'
                }
            }
        }
    });
});

watch(expenses, () => {
    if (chartInstance.value) {
        const { labels, data } = createChartData();
        chartInstance.value.data.labels = labels;
        chartInstance.value.data.datasets[0].data = data;
        chartInstance.value.update();
    }
}, { deep: true });
</script>

<template>
  <div class="container">
    <h1>Organizador de Despesas</h1>

    <div class="main-content">
      <div class="form-section">
        <h2>{{ editingExpenseId ? 'Editar Despesa' : 'Adicionar Despesa' }}</h2>
        <form @submit.prevent="addExpense">
          <div class="input-group">
            <label for="description">Descrição</label>
            <input id="description" type="text" v-model="newExpense.description" placeholder="Ex: Conta de luz" required>
          </div>
          
          <div class="input-group">
            <label for="amount">Valor (R$)</label>
            <input id="amount" type="number" v-model.number="newExpense.amount" placeholder="Ex: 150.50" required>
          </div>

          <div class="input-group">
            <label for="category">Categoria</label>
            <select id="category" v-model="newExpense.category">
              <option value="Alimentação">Alimentação</option>
              <option value="Moradia">Moradia</option>
              <option value="Transporte">Transporte</option>
              <option value="Lazer">Lazer</option>
              <option value="Outros">Outros</option>
            </select>
          </div>

          <button type="submit">{{ editingExpenseId ? 'Salvar Alterações' : 'Adicionar' }}</button>
        </form>
      </div>

      <div class="list-section">
        <h2>Últimas Despesas</h2>
        <div class="expenses-list">
          <div v-for="expense in expenses" :key="expense.id" class="expense-item">
            <div class="expense-details">
              <span class="description">{{ expense.description }}</span>
              <span class="category">{{ expense.category }}</span>
            </div>
            <div class="actions">
              <span class="amount">R$ {{ expense.amount.toFixed(2) }}</span>
              <button @click="startEdit(expense)" class="edit-btn">Editar</button>
              <button @click="deleteExpense(expense.id)" class="delete-btn">Excluir</button>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <div class="chart-section">
      <h2>Análise de Gastos por Categoria</h2>
      <canvas id="expense-chart" ref="chartCanvas"></canvas>
    </div>
  </div>
</template>

<style scoped>
.container {
  font-family: 'Arial', sans-serif;
  max-width: 900px;
  margin: 40px auto;
  padding: 30px;
  background-color: #fff;
  border-radius: 10px;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

h1 {
  text-align: center;
  color: #2c3e50;
}

.main-content {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
  margin-top: 30px;
}

@media (max-width: 768px) {
  .main-content {
    grid-template-columns: 1fr;
  }
}

h2 {
  border-bottom: 2px solid #3498db;
  padding-bottom: 10px;
  color: #3498db;
}

.form-section, .list-section {
  padding: 20px;
  border: 1px solid #eee;
  border-radius: 8px;
}

.input-group {
  display: flex;
  flex-direction: column;
  margin-bottom: 15px;
}

label {
  font-weight: bold;
  margin-bottom: 5px;
}

input, select {
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  font-size: 1rem;
}

button {
  width: 100%;
  padding: 12px;
  background-color: #2c3e50;
  color: #fff;
  border: none;
  border-radius: 5px;
  font-size: 1.1rem;
  cursor: pointer;
}

.expenses-list {
    display: flex;
    flex-direction: column;
    gap: 10px;
}

.expense-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background-color: #f9f9f9;
  border-left: 5px solid #3498db;
  border-radius: 5px;
}

.expense-details {
  display: flex;
  align-items: center;
  gap: 10px;
}

.expense-details .description {
  font-weight: bold;
  color: #555;
}

.expense-details .category {
  background-color: #eaf2f8;
  color: #3498db;
  font-size: 0.8rem;
  padding: 3px 8px;
  border-radius: 12px;
}

.actions {
  display: flex;
  align-items: center;
  gap: 10px;
}

.amount {
  font-weight: bold;
  color: #e74c3c;
}

.edit-btn, .delete-btn {
  padding: 5px 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  color: #fff;
  font-size: 0.9rem;
}

.edit-btn {
  background-color: #f39c12;
}

.delete-btn {
  background-color: #c0392b;
}

.edit-btn:hover, .delete-btn:hover {
  opacity: 0.8;
}
</style>