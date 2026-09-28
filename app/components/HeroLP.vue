<template>
  <div class="hero bg-base-200 min-h-screen">
    <div class="hero-content text-center">
      <div class="max-w-md">
        <h1 class="text-5xl font-bold">Aprender junto <span class="text-primary">faz a diferença.</span></h1>
        <p class="py-6">
          Um espaço para aprender, compartilhar ideias e crescer na tecnologia. Faça seu cadastro e venha
          conhecer a comunidade.
        </p>
        <div class="flex flex-wrap justify-center gap-3">
          <NuxtLink class="btn btn-primary" to="/cadastro">Quero fazer parte</NuxtLink>
          <button class="btn btn-outline" type="button" @click="goToExample()">Ver exemplo</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
const router = useRouter()

function goToExample () {
  router.push('/example')
}
</script>