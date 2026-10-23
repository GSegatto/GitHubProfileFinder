<script setup>
import { ref, computed } from 'vue'

const busca = ref('')

// Os três estados distintos da requisição + o dado
const carregando = ref(false)
const naoEncontrado = ref(false)
const erro = ref('')
const perfil = ref(null)

// Fallback: se `name` vier vazio/null, usa o login
const nomeExibido = computed(() => perfil.value?.name || perfil.value?.login)

async function buscar() {
  const username = busca.value.trim()
  if (!username) return

  // Reseta tudo antes de cada busca: só um estado fica ativo por vez
  carregando.value = true
  naoEncontrado.value = false
  erro.value = ''
  perfil.value = null

  try {
    const response = await fetch(
      `https://api.github.com/users/${encodeURIComponent(username)}`
    )

    if (response.status === 404) {
      naoEncontrado.value = true 
      return
    }
    if (!response.ok) {
      throw new Error(`GitHub respondeu com status ${response.status}`)
    }

    perfil.value = await response.json()
    busca.value = ''

  } catch (e) {
    // chegar falhas inesperadas (rede fora, rate limit, etc.)
    erro.value = 'Não foi possível consultar o GitHub agora. Tente novamente.'
    console.error(e)
  } finally {
    carregando.value = false
  }
}
</script>



<template>
  <main class="pagina">
    <section class="painel-busca">
      <h1>Buscador de perfis GitHub</h1>

      <p class="subtitulo">
        Digite um nome de usuário para ver o perfil.
      </p>

      <form class="busca" @submit.prevent="buscar">
        <input
          v-model="busca"
          type="text"
          placeholder="Ex.: torvalds"
          aria-label="Nome de usuário do GitHub"
        />

        <button
          type="submit"
          :disabled="carregando || !busca.trim()"
        >
          {{ carregando ? 'Buscando...' : 'Buscar' }}
        </button>
      </form>

      <p v-if="carregando" class="status" role="status">
        <span class="spinner" aria-hidden="true"></span>
        Buscando perfil...
      </p>

      <p
        v-else-if="naoEncontrado"
        class="status aviso"
        role="alert"
      >
        Usuário "{{ busca.trim() }}" não encontrado.
        Confira o nome e tente novamente.
      </p>

      <p v-else-if="erro" class="status erro" role="alert">
        {{ erro }}
      </p>
    </section>

    <article v-if="perfil" class="card">
      <img
        :src="perfil.avatar_url"
        :alt="`Avatar de ${nomeExibido}`"
      />

      <div class="info">
        <h2>{{ nomeExibido }}</h2>

        <p class="login">@{{ perfil.login }}</p>

        <p v-if="perfil.bio" class="bio">
          {{ perfil.bio }}
        </p>

        <ul class="numeros">
          <li>
            <strong>{{ perfil.followers }}</strong>
            seguidores
          </li>

          <li>
            <strong>{{ perfil.public_repos }}</strong>
            repositórios
          </li>
        </ul>

        <a
          :href="perfil.html_url"
          target="_blank"
          rel="noopener noreferrer"
        >
          Ver perfil no GitHub
        </a>
      </div>
    </article>
  </main>
</template>



<style scoped>

:global(body) {
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
  font-family: system-ui, sans-serif;
  color: #f0f0f6;

  background:
    linear-gradient(
      135deg,
      rgba(9, 9, 15, 0.88),
      rgba(42, 17, 72, 0.78)
    ),
    url('/img/git-background.png') center / cover fixed no-repeat;

  background-color: #09090f;
}

:global(*) {
  box-sizing: border-box;
}

.pagina {
  width: 100%;
  min-height: 100vh;
  margin: 0;
  padding: 2rem 1.25rem;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 1.5rem;
}

.painel-busca {
  width: 100%;
  max-width: 560px;
  padding: 2.5rem 2rem;

  background: rgba(17, 17, 27, 0.78);
  border: 1px solid rgba(137, 87, 229, 0.4);
  border-radius: 20px;

  box-shadow: 0 16px 60px rgba(0, 0, 0, 0.35);
  backdrop-filter: blur(14px);

  text-align: center;
}

h1 {
  margin: 0 0 0.65rem;
  font-size: clamp(1.6rem, 4vw, 2rem);
  line-height: 1.25;
  letter-spacing: -0.5px;
}

.subtitulo {
  margin: 0 0 1.75rem;
  color: #b8b5c9;
  line-height: 1.6;
}

.busca {
  display: flex;
  gap: 0.65rem;
  margin-bottom: 1rem;
}

.busca input {
  flex: 1;
  min-width: 0;
  padding: 0.85rem 1rem;

  font-size: 1rem;
  color: #f0f0f6;
  background: rgba(9, 9, 15, 0.75);

  border: 1px solid #393347;
  border-radius: 10px;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.busca input::placeholder {
  color: #858096;
}

.busca input:focus {
  border-color: #8957e5;
  box-shadow: 0 0 0 3px rgba(137, 87, 229, 0.18);
}

.busca button {
  padding: 0.85rem 1.25rem;
  font-size: 0.95rem;
  font-weight: 600;
  color: #fff;

  background: linear-gradient(135deg, #8957e5, #6e40c9);
  border: 0;
  border-radius: 10px;

  cursor: pointer;
  transition: transform 0.2s, filter 0.2s;
}

.busca button:hover:not(:disabled) {
  transform: translateY(-2px);
  filter: brightness(1.12);
}

.busca button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.busca input:focus-visible,
.busca button:focus-visible,
.card a:focus-visible {
  outline: 2px solid #b99aff;
  outline-offset: 3px;
}

.status {
  display: flex;
  align-items: center;
  justify-content: center;
  flex-wrap: wrap;
  gap: 0.6rem;
  margin: 1rem 0 0;
  color: #d5d1e2;
}

.aviso {
  color: #f0c674;
}

.erro {
  color: #ff8b96;
}

.spinner {
  width: 1rem;
  height: 1rem;
  flex-shrink: 0;

  border: 2px solid #393347;
  border-top-color: #a780ff;
  border-radius: 50%;
  animation: girar 0.8s linear infinite;
}

@keyframes girar {
  to {
    transform: rotate(360deg);
  }
}

@media (prefers-reduced-motion: reduce) {
  .spinner {
    animation: none;
  }

  .busca button {
    transition: none;
  }
}

.card {
  display: flex;
  align-items: flex-start;
  gap: 1.25rem;

  width: 100%;
  max-width: 560px;
  padding: 1.5rem;

  color: #f0f0f6;
  background: rgba(17, 17, 27, 0.86);
  border: 1px solid rgba(137, 87, 229, 0.4);
  border-radius: 18px;

  box-shadow: 0 16px 60px rgba(0, 0, 0, 0.25);
  backdrop-filter: blur(14px);
}

.card img {
  width: 88px;
  height: 88px;
  flex-shrink: 0;
  object-fit: cover;
  border-radius: 50%;
  border: 2px solid #8957e5;
}

.info {
  min-width: 0;
}

.info h2 {
  margin: 0;
  font-size: 1.35rem;
  overflow-wrap: anywhere;
}

.login {
  margin: 0 0 0.75rem;
  color: #b8a2e8;
}

.bio {
  margin: 0 0 1rem;
  color: #d0cddd;
  overflow-wrap: anywhere;
}

.numeros {
  display: flex;
  flex-wrap: wrap;
  gap: 1rem;
  margin: 0 0 1rem;
  padding: 0;
  list-style: none;
  color: #d0cddd;
}

.numeros strong {
  color: #fff;
}

.card a {
  color: #b99aff;
  font-weight: 600;
  text-decoration: none;
}

.card a:hover {
  text-decoration: underline;
}

@media (max-width: 480px) {
  .pagina {
    padding: 1.25rem 1rem;
  }

  .painel-busca {
    padding: 1.75rem 1.1rem;
  }

  .busca {
    flex-direction: column;
  }

  .busca button {
    width: 100%;
  }

  .card {
    flex-direction: column;
    align-items: center;
    text-align: center;
  }

  .numeros {
    justify-content: center;
  }
}

</style>
