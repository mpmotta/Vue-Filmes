<script setup lang="ts">
import { ref, onMounted } from 'vue'

const filmes = ref([])
const selecionado = ref(null)
const atores = ref([])
const tituloCategoria = ref('Populares')
const textoBusca = ref('')
const chave = 'ec332d19e6fed067df0160ce34067cc4'

const buscarFilmes = async (generoId?: number, nomeGeneros?: string) => {
  selecionado.value = null
  textoBusca.value = ''
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

const pesquisarFilme = async () => {
  if (textoBusca.value === '') return
  selecionado.value = null

  const url = `https://api.themoviedb.org/3/search/movie?api_key=${chave}&language=pt-BR&query=${textoBusca.value}`
  const resposta = await fetch(url)
  const dados = await resposta.json()
  filmes.value = dados.results
  tituloCategoria.value = `Resultados para: ${textoBusca.value}`
}

const abrirDetalhes = async (filme: any) => {
  const res = await fetch(`https://api.themoviedb.org/3/movie/${filme.id}/credits?api_key=${chave}&language=pt-BR`)
  const dados = await res.json()
  atores.value = dados.cast.slice(0, 10)
  selecionado.value = filme
}

onMounted(() => {
  buscarFilmes()
})
</script>

<template>
  <main>
    <div v-if="!selecionado">
      <div class="controles">
        <nav class="filtros">
          <button @click="buscarFilmes()">LIMPAR</button>
          <button @click="buscarFilmes(14, 'Fantasia')">FANTASIA</button>
          <button @click="buscarFilmes(35, 'Comédia')">COMÉDIA</button>
          <button @click="buscarFilmes(12, 'Aventura')">AVENTURA</button>
          <button @click="buscarFilmes(80, 'Policial')">POLICIAL</button>
          <button @click="buscarFilmes(878, 'Ficção')">FICÇÃO</button>
          <button @click="buscarFilmes(10749, 'Romance')">ROMANCE</button>
        </nav>

        <div class="busca">
          <input v-model="textoBusca" @keyup.enter="pesquisarFilme" placeholder="Digite o nome de um filme...">
          <button @click="pesquisarFilme">BUSCAR</button>
        </div>
      </div>

      <h2>{{ tituloCategoria }}</h2>

      <div class="lista">
        <div v-for="item in filmes" :key="item.id" class="item" @click="abrirDetalhes(item)">
          <img :src="`https://image.tmdb.org/t/p/w200${item.poster_path}`" v-if="item.poster_path">
          <div v-else class="sem-foto">Sem Foto</div>
          <p>{{ item.title }}</p>
        </div>
      </div>
    </div>

    <div v-else class="detalhes">
      <button @click="selecionado = null">VOLTAR</button>
      <div class="infos">
        <img :src="`https://image.tmdb.org/t/p/w500${selecionado.poster_path}`" class="poster-g">
        <div class="texto">
          <h1>{{ selecionado.title }}</h1>
          <p>Nota: {{ selecionado.vote_average }}</p>
          <p>{{ selecionado.overview }}</p>
          <h3>Atores:</h3>
          <div class="atores-lista">
             <div v-for="a in atores" :key="a.id" class="ator">
               <img :src="`https://image.tmdb.org/t/p/w200${a.profile_path}`" v-if="a.profile_path">
               <p>{{ a.name }}</p>
             </div>
          </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
.controles { background: #f4f4f4; padding: 15px; border-radius: 8px; margin-bottom: 20px; }
.filtros { display: flex; gap: 8px; margin-bottom: 15px; flex-wrap: wrap; }
.busca { display: flex; gap: 10px; }
input { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 4px; }
button { cursor: pointer; padding: 10px 15px; border: none; background: #34495e; color: white; font-weight: bold; border-radius: 4px; }
button:hover { background: #41b883; }
.lista { display: flex; flex-wrap: wrap; gap: 15px; }
.item { width: 150px; text-align: center; cursor: pointer; }
.item img { width: 100%; border-radius: 5px; }
.sem-foto { width: 100%; height: 225px; background: #ddd; display: flex; align-items: center; justify-content: center; border-radius: 5px; }
.infos { display: flex; gap: 20px; margin-top: 20px; }
.poster-g { width: 300px; border-radius: 10px; }
.atores-lista { display: flex; gap: 10px; overflow-x: auto; }
.ator { min-width: 80px; font-size: 10px; text-align: center; }
.ator img { width: 100%; border-radius: 50%; }
</style>