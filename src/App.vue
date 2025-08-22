<script setup>
import { ref } from 'vue';

// Estado para a lista de despesas
const expenses = ref([
  { id: 1, description: 'Aluguel', amount: 1500, category: 'Moradia' },
  { id: 2, description: 'Compras', amount: 250, category: 'Alimentação' },
]);

// Estado para os campos do formulário
const newExpense = ref({
  description: '',
  amount: null,
  category: 'Alimentação',
});

const addExpense = () => {
  if (!newExpense.value.description || !newExpense.value.amount) {
    alert('Por favor, preencha todos os campos.');
    return;
  }

  const expenseToAdd = {
    id: Date.now(), // ID único
    description: newExpense.value.description,
    amount: parseFloat(newExpense.value.amount),
    category: newExpense.value.category,
  };

  expenses.value.push(expenseToAdd);

  // Limpar o formulário
  newExpense.value.description = '';
  newExpense.value.amount = null;
};
</script>

<template>
  <div class="container">
    <h1>Organizador de Despesas</h1>

    <div class="main-content">
      <div class="form-section">
        <h2>Adicionar Despesa</h2>
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

          <button type="submit">Adicionar</button>
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
            <span class="amount">R$ {{ expense.amount.toFixed(2) }}</span>
          </div>
        </div>
      </div>
    </div>

    <div class="chart-section">
      <h2>Análise de Gastos por Categoria</h2>
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

.expense-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
  background-color: #f9f9f9;
  border-left: 5px solid #3498db;
  border-radius: 5px;
  margin-bottom: 10px;
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
  margin-left: 10px;
}

.amount {
  font-weight: bold;
  color: #e74c3c;
}
</style>