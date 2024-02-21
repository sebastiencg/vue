<template>
  <div>
    <!-- Loader -->
    <div v-if="loading" class="cs-loader">
      <div class="cs-loader-inner">
        <label>●</label>
        <label>●</label>
        <label>●</label>
        <label>●</label>
        <label>●</label>
        <label>●</label>
      </div>
    </div>
    <!-- Form -->
    <div class="container mt-5 d-flex justify-content-center align-items-center" v-if="!loading">
      <form @submit.prevent="handleSubmit" class="w-100">
        <div class="mb-3">
          <label for="inputText" class="form-label">Entrez votre texte :</label>
          <input v-model="userInput" id="inputText" type="text" class="form-control" placeholder="Saisissez quelque chose..." />
        </div>
        <button type="submit" class="btn btn-primary btn-block">Soumettre</button>
      </form>
    </div>

    <!-- Response -->
    <div class="container mt-3" v-if="responseData.message && !loading">
      <div v-if="responseData.congratulations">
        Félicitations, tu as atteint 100% de similarité !
      </div>
      <div v-else>
        Tu es à {{ responseData.message }} % de similarité
      </div>
    </div>

    <!-- Historique -->
    <div class="container mt-3" v-if="historique.length > 0 && !loading" >
      <h2>Historique</h2>
      <ul>
        <li v-for="(item, index) in historique" :key="index">
          <strong>Texte saisi:</strong> {{ item.userInput }} <br>
          <strong>Réponse:</strong> tu es a {{ item.responseData }}  % de similarité
          <br><br>
        </li>
      </ul>
    </div>
  </div>
</template>
<script>
import axios from "axios";
import '../assets/input.css'

export default {
  data() {
    return {
      userInput: '',
      responseData: {},
      loading: false,
      historique: []
    };
  },
  methods: {
    async fetchData(word) {
      try {
        this.loading = true;
        const response = await axios.get(`https://127.0.0.1:8000/vecteur/${word}`);
        this.handleResponse(response.data.message);
        this.addToHistorique({ userInput: word, responseData: response.data.message });
      } catch (error) {
        console.error('Erreur lors de la requête GET', error);
      } finally {
        this.loading = false;
      }
    },
    handleSubmit() {
      this.fetchData(this.userInput);
    },
    handleResponse(data) {
      this.responseData = { message: data };
      if (data === 100) {
        console.log(data)
        this.responseData.congratulations = true;
      } else {

        this.responseData.congratulations = false;
      }
    },
    addToHistorique(item) {

      let historique = JSON.parse(localStorage.getItem("historique")) || [];

      historique.unshift(item);

      historique.sort((a, b) => b.responseData - a.responseData);

      localStorage.setItem("historique", JSON.stringify(historique));

      this.historique = historique;
    }
  },
  created() {
    this.historique = JSON.parse(localStorage.getItem("historique")) || [];
  }
}
</script>

<style scoped>
/* Ajoutez vos styles si nécessaire */
</style>
