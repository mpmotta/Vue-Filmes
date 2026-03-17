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
  atores.value = dados.cast.slice(0, 12)
  selecionado.value = filme
}

onMounted(() => buscarFilmes())
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
          <input v-model="textoBusca" @keyup.enter="pesquisarFilme" placeholder="Pesquisar...">
          <button @click="pesquisarFilme">BUSCAR</button>
        </div>
      </div>

      <div class="lista">
        <div v-for="item in filmes" :key="item.id" class="item" @click="abrirDetalhes(item)">
          <div class="capa-container">
            <img :src="`https://image.tmdb.org/t/p/w500${item.poster_path}`" v-if="item.poster_path">
          </div>
          <p>{{ item.title }}</p>
        </div>
      </div>
    </div>

    <div v-else class="detalhes">
      <button @click="selecionado = null" class="btn-voltar">VOLTAR</button>
      
      <div class="topo-detalhes">
        <div class="foto-fixa">
          <img :src="`https://image.tmdb.org/t/p/w500${selecionado.poster_path}`">
        </div>

        <div class="texto-fixo">
          <h1>{{ selecionado.title }}</h1>
          <p class="meta">⭐ {{ selecionado.vote_average?.toFixed(1) }} | {{ selecionado.release_date?.split('-')[0] }}</p>
          <h3>Sinopse</h3>
          <p class="resenha">{{ selecionado.overview || 'Sem sinopse.' }}</p>
        </div>
      </div>

      <div class="elenco-fixo">
        <h3>Elenco Principal</h3>
        <div class="elenco-wrap">
           <div v-for="a in atores" :key="a.id" class="ator-item">
             <img :src="a.profile_path ? `https://image.tmdb.org/t/p/w200${a.profile_path}` : 'https://via.placeholder.com/80x80'">
             <p>{{ a.name }}</p>
           </div>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
main { padding: 20px; font-family: sans-serif; }
.controles { background: #f4f4f4; padding: 15px; border-radius: 8px; margin-bottom: 20px; }
.filtros { display: flex; gap: 8px; margin-bottom: 15px; flex-wrap: wrap; }
.busca { display: flex; gap: 10px; }
input { flex: 1; padding: 10px; border: 1px solid #ccc; border-radius: 4px; }
button { cursor: pointer; padding: 10px 15px; border: none; background: #34495e; color: white; font-weight: bold; border-radius: 4px; }

.lista { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 20px; }
.item { text-align: center; cursor: pointer; }
.capa-container { height: 280px; overflow: hidden; border-radius: 8px; }
.capa-container img { width: 100%; height: 100%; object-fit: cover; }

/* REGRAS RÍGIDAS DE DIMENSÃO */
.topo-detalhes { 
  display: flex; 
  gap: 30px; 
  margin-bottom: 30px; 
  align-items: flex-start;
}

.foto-fixa { 
  width: 500px; /* Travado em 500px */
  flex-shrink: 0;
}
.foto-fixa img { width: 100%; border-radius: 12px; }

.texto-fixo { 
  max-width: 700px; /* Travado em no máximo 700px */
}
.texto-fixo h1 { margin-top: 0; font-size: 2.2rem; }
.resenha { line-height: 1.6; font-size: 1.1rem; }

.elenco-fixo { 
  width: 1200px; /* Travado em 1200px (500px foto + 700px texto) */
  border-top: 1px solid #eee;
  padding-top: 20px;
}
.elenco-wrap { 
  display: flex; 
  flex-wrap: wrap; 
  gap: 15px; 
}
.ator-item { width: 90px; text-align: center; }
.ator-item img { width: 70px; height: 70px; border-radius: 50%; object-fit: cover; }

/* Ajuste para telas menores que 1200px para não quebrar o layout */
@media (max-width: 1250px) {
  .elenco-fixo { width: 100%; }
  .topo-detalhes { flex-direction: column; align-items: center; text-align: center; }
  .foto-fixa { width: 100%; max-width: 500px; }
}
</style>