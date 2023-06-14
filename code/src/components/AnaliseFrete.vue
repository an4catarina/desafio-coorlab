<template>
  <div class="d-flex align-items-center justify-content-center" style="height: 100vh;">
    <div class="custom-container input-container">
      <div class="col-md-40 d-flex flex-column align-items-center justify-content-center">
        <img src="@/assets/logo.png" alt="Logo" class="logo">
        <div class="form-group">
          <div class="input-label-container">
            <label for="city" class="input-label">
              <i class="bi bi-map icon"></i>
              <strong class="label-text">Insira o destino e o peso:</strong>
            </label>
          </div>
        </div>
        <div class="form-group">
          <label for="city" class="input-label">Destino:</label>
          <select id="city" v-model="city" class="form-control select-destination">
            <option value="">Selecione o destino</option>
            <option v-for="destination in destinations" :value="destination" :key="destination">{{ destination }}</option>
          </select>
        </div>
        <div class="form-group">
          <label for="weight" class="input-label">Peso da carga em Kg:</label>
          <input type="number" id="weight" v-model="weight" class="form-control" />
        </div>
        <div class="form-group">
          <button class="btn btn-primary" @click="analyzeFreight">Analisar</button>
        </div>
      </div>
    </div>
    <div class="custom-container output-container" style="width: 600px;">
      <div class="col-md-6" style="width: 100%;">
        <div v-if="bestPrice || fastest" class="row">
          <div class="result-section" v-if="bestPrice">
            <div class="result-icon">
              <i class="bi bi-currency-dollar"></i>
            </div>
            <div class="result-details">
              <p class="result-title">Frete com o menor valor</p>
              <p><strong>Transportadora:</strong> {{ bestPrice.name }}</p>
              <p><strong>Tempo estimado:</strong> {{ bestPrice.lead_time }}</p>
            </div>
            <div class="result-price">
              <p><strong>Preço</strong></p>
              <p>R${{ calculateTotalCost(bestPrice).toFixed(2) }}</p>
            </div>
          </div>
          <div class="result-section" v-if="fastest">
            <div class="result-icon">
              <i class="bi bi-clock"></i>
            </div>
            <div class="result-details">
              <p class="result-title">Frete mais rápido</p>
              <p><strong>Transportadora:</strong> {{ fastest.name }}</p>
              <p><strong>Tempo estimado:</strong> {{ fastest.lead_time }}</p>
            </div>
            <div class="result-price">
              <p><strong>Preço</strong></p>
              <p>R${{ calculateTotalCost(fastest).toFixed(2) }}</p>
            </div>
          </div>
          <div class="result-button">
            <button class="btn btn-primary" @click="clearData">Limpar</button>
          </div>
        </div>
        <div v-else>
          <p>Nenhum dado selecionado</p>
        </div>
        <div v-if="showWarning" class="error-message">
          Preencha todos os campos.
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import 'bootstrap-icons/font/bootstrap-icons.css';
import axios from 'axios';

export default {
  data() {
    return {
      weight: null,
      city: '',
      bestPrice: null,
      fastest: null,
      freightData: [],
      showWarning: false
    };
  },
  created() {
    this.getFreightData();
  },
  methods: {
    // Obter dados de frete
    async getFreightData() {
      try {
        const response = await axios.get('http://localhost:3000/transport');
        this.freightData = response.data;
      } catch (error) {
        console.log(error);
      }
    },

    // Analisar frete
    analyzeFreight() {
      if (!this.city || !this.weight) {
        this.showWarning = true;
        return;
      }

      this.showWarning = false;

      const filteredData = this.freightData.filter(item => item.city === this.city);
      if (filteredData.length > 0) {
        const { bestFreight, fastestFreight } = this.findBestPriceAndFastest(filteredData);
        this.bestPrice = bestFreight;
        this.fastest = fastestFreight;
      } else {
        this.bestPrice = null;
        this.fastest = null;
      }
    },

    // Encontrar melhor preço e frete mais rápido
    findBestPriceAndFastest(data) {
      let lowestCost = Infinity;
      let bestFreight = null;
      let lowestTime = Infinity;
      let fastestFreight = null;

      for (const freight of data) {
        const weight = this.getFreightWeight(freight);
        const totalCost = weight * this.weight;
        const time = parseInt(freight.lead_time);

        if (totalCost < lowestCost) {
          lowestCost = totalCost;
          bestFreight = freight;
        }

        if (time < lowestTime) {
          lowestTime = time;
          fastestFreight = freight;
        }
      }

      return { bestFreight, fastestFreight };
    },

    // Calcular custo total
    calculateTotalCost(freight) {
      const weight = this.getFreightWeight(freight);
      return weight * this.weight;
    },

    // Obter peso do frete
    getFreightWeight(freight) {
      return this.weight >= 100 ? parseFloat(freight.cost_transport_heavy.slice(3)) : parseFloat(freight.cost_transport_light.slice(3));
    },

    // Limpar dados
    clearData() {
      this.weight = null;
      this.city = '';
      this.bestPrice = null;
      this.fastest = null;
    },
  },
  computed: {
    destinations() {
      return [...new Set(this.freightData.map(item => item.city))];
    },
  },
};
</script>

<style>
.select-destination {
  margin-bottom: 3px;
  width: 15px;
}

.input-label-container {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.input-label {
  position: relative;
  margin-bottom: 0;
  margin-left: 10px;
}

.form-group {
  margin-bottom: 20px;
}

.custom-container {
  background-color: #f8f9fa;
  border-radius: 5px;
  padding: 20px;
}

.input-container {
  margin-right: 10px;
}

.icon {
  margin-right: 5px;
  width: 50px;
  position: relative;
  top: 1px;
}

.output-container {
  width: 600px; 
  height: 335px;
}

.custom-container.output-container {
  width: 30px; /* Set the desired width */
}
.result-section {
  display: flex;
  align-items: center;
  border: 2px solid #555;
  border-radius: 5px;
  margin-bottom: 10px;
  padding: 10px;
  width: 400px; /* Adjust width to fill available space */
}

.result-container {
  flex-grow: 1;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}

.result-icon {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 50px;
  height: 100%; 
  color: #000;
  background-color: #00aca6;
  border-radius: 5px;
}

.result-details {
  flex: auto;
  padding-left: 10px;
  align-items: center;
  justify-content: center;
  text-align: left;
  background-color: #dfdada;
  border-radius: 5px;
  font-size: 14px;
  margin-right: 10px;
  width: 100%;
}

.result-title {
  font-size: 18px;
  margin-bottom: 5px;
}

.result-info {
  margin-bottom: 5px;
}

.result-price {
  display: auto;
  padding-right: 10px;
  align-items: center;
  justify-content: center;
  text-align: left;
  background-color: #dfdada;
  color: #000;
  border-radius: 5px;
  font-size: 14px;
  margin-top: 5px;
}

.result-price p {
  margin-bottom: 0;
}

.result-details p {
  margin-bottom: 0;
}


.modal-content {
  text-align: center;
}

.btn.btn-primary {
  border-radius: 15px;
  margin-top: 10px;
  color: #fff;
  background-color: #00aca6;
  padding: 10px 20px;
  font-size: 16px;
  border: none;
}

.error-message {
  color: red;
  margin-top: 10px;
  font-size: 14px;
}

.logo {
  width: 100px;
  height: 100px;
  margin-bottom: 20px;
}
</style>