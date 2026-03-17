<script setup lang="ts">
import { ref, onMounted } from 'vue'

const filmes = ref([])
const tituloCategoria = ref('Populares')

const buscarFilmes = async (generoId?: number, nomeGeneros?: string) => {
  const chave = 'ec332d19e6fed067df0160ce34067cc4'
  
  let url = `https://api.themoviedb.org/3/movie/popular?api_key=${chave}&language=pt-BR`
  
  if (generoId) {
    url = `https://api.themoviedb.org/3/discover/movie?api_key=${chave}&language=pt-BR&with_genres=${generoId}`
    tituloCategoria.value = nomeGeneros || ''
  } else {
    tituloCategoria.value = 'Populares'
  }

  const resposta = await fetch(url)
  const dados = await resposta.json()
  filmes.value = dados.results
}

onMounted(() => {
  buscarFilmes()
})
</script>

<template>
  <main>
    <nav class="filtros">
      <button @click="buscarFilmes()">LIMPAR</button>
      <button @click="buscarFilmes(14, 'Fantasia')">FANTASIA</button>
      <button @click="buscarFilmes(35, 'Comédia')">COMÉDIA</button>
      <button @click="buscarFilmes(12, 'Aventura')">AVENTURA</button>
      <button @click="buscarFilmes(80, 'Policial')">POLICIAL</button>
      <button @click="buscarFilmes(878, 'Ficção')">FICÇÃO</button>
      <button @click="buscarFilmes(10749, 'Romance')">ROMANCE</button>
    </nav>

    <h2>Exibindo: {{ tituloCategoria }}</h2>

    <div class="lista">
      <div v-for="item in filmes" :key="item.id" class="item">
        <img :src="`https://image.tmdb.org/t/p/w200${item.poster_path}`" v-if="item.poster_path">
        <p>{{ item.title }}</p>
      </div>
    </div>
  </main>
</template>

<style scoped>
.filtros {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}
button {
  cursor: pointer;
  padding: 10px 15px;
  border: none;
  background: #34495e;
  color: white;
  font-weight: bold;
  border-radius: 4px;
}
button:hover {
  background: #41b883;
}
.lista {
  display: flex;
  flex-wrap: wrap;
  gap: 15px;
}
.item {
  width: 150px;
  text-align: center;
  border: 1px solid #eee;
  padding: 10px;
  border-radius: 8px;
}
.item img {
  width: 100%;
  border-radius: 5px;
}
</style>