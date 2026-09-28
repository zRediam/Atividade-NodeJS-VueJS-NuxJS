<script setup lang="ts">
type RegistrationForm = {
  fullName: string
  email: string
  area: string
  otherArea: string
  semester: string
  interests: string[]
  bio: string
}

type FormField = keyof RegistrationForm

const maxBioLength = 200
const interestOptions = ['Front-end', 'Back-end', 'Mobile', 'UI/UX', 'Dados', 'DevOps']
const createEmptyForm = (): RegistrationForm => ({
  fullName: '',
  email: '',
  area: '',
  otherArea: '',
  semester: '',
  interests: [],
  bio: '',
})

const form = reactive<RegistrationForm>(createEmptyForm())
const errors = reactive<Partial<Record<FormField, string>>>({})
const formElement = ref<HTMLFormElement | null>(null)
const submittedName = ref('')

function validateForm() {
  Object.keys(errors).forEach((key) => delete errors[key as FormField])

  if (form.fullName.trim().length < 3) {
    errors.fullName = 'Informe seu nome completo.'
  }
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(form.email.trim())) {
    errors.email = 'Informe um endereço de e-mail válido.'
  }
  if (!form.area) {
    errors.area = 'Selecione seu curso ou área de atuação.'
  }
  if (form.area === 'Outro' && !form.otherArea.trim()) {
    errors.otherArea = 'Informe sua área de atuação.'
  }
  if (!form.semester || !Number.isInteger(Number(form.semester)) || Number(form.semester) < 1 || Number(form.semester) > 20) {
    errors.semester = 'Informe um período entre 1 e 20.'
  }

  return Object.keys(errors).length === 0
}

async function submitForm() {
  submittedName.value = ''

  if (!validateForm()) {
    await nextTick()
    formElement.value?.querySelector<HTMLElement>('[aria-invalid="true"]')?.focus()
    return
  }

  const registration = {
    nomeCompleto: form.fullName.trim(),
    email: form.email.trim(),
    cursoArea: form.area === 'Outro' ? form.otherArea.trim() : form.area,
    semestrePeriodo: Number(form.semester),
    interessesHabilidades: [...form.interests],
    bio: form.bio.trim(),
  }

  console.log('Novo cadastro recebido:', registration)
  submittedName.value = form.fullName.trim().split(/\s+/)[0] ?? form.fullName.trim()
  Object.assign(form, createEmptyForm())
}
</script>

<template>
  <main class="registration-page">
    <div class="registration-shell">
      <NuxtLink class="back-link" to="/" aria-label="Voltar à página inicial">
        <span aria-hidden="true">←</span> Início
      </NuxtLink>

      <section class="registration-card" aria-labelledby="registration-title">
        <header class="registration-heading">
          <span class="heading-mark" aria-hidden="true">✳</span>
          <p class="eyebrow">COMUNIDADE NUXT</p>
          <h1 id="registration-title">Vamos nos conhecer<span>.</span></h1>
          <p>Preencha seus dados e dê o primeiro passo para fazer parte da comunidade.</p>
        </header>

        <div v-if="submittedName" class="success-message" role="status" aria-live="polite">
          <span class="success-check" aria-hidden="true">✓</span>
          <p><strong>Cadastro enviado, {{ submittedName }}!</strong><br>Obrigado por chegar junto. Seus dados foram registrados para esta demonstração.</p>
          <button class="success-dismiss" type="button" aria-label="Fechar mensagem" @click="submittedName = ''">×</button>
        </div>

        <form ref="formElement" class="registration-form" novalidate @submit.prevent="submitForm">
          <div class="form-field">
            <label for="full-name">Nome completo <span class="required-mark">*</span></label>
            <input id="full-name" v-model="form.fullName" type="text" autocomplete="name" placeholder="Ex.: Maria da Silva" :aria-invalid="!!errors.fullName" :aria-describedby="errors.fullName ? 'full-name-error' : undefined" @input="delete errors.fullName">
            <small v-if="errors.fullName" id="full-name-error" class="field-error">{{ errors.fullName }}</small>
          </div>

          <div class="form-field">
            <label for="email">E-mail <span class="required-mark">*</span></label>
            <input id="email" v-model="form.email" type="email" autocomplete="email" placeholder="voce@exemplo.com" :aria-invalid="!!errors.email" :aria-describedby="errors.email ? 'email-error' : undefined" @input="delete errors.email">
            <small v-if="errors.email" id="email-error" class="field-error">{{ errors.email }}</small>
          </div>

          <div class="form-grid">
            <div class="form-field">
              <label for="area">Curso / Área de atuação <span class="required-mark">*</span></label>
              <select id="area" v-model="form.area" :aria-invalid="!!errors.area" :aria-describedby="errors.area ? 'area-error' : undefined" @change="delete errors.area">
                <option disabled value="">Selecione uma opção</option>
                <option>Desenvolvimento de Software</option>
                <option>Ciência da Computação</option>
                <option>Análise e Desenvolvimento de Sistemas</option>
                <option>Design</option>
                <option>Marketing</option>
                <option>Outro</option>
              </select>
              <small v-if="errors.area" id="area-error" class="field-error">{{ errors.area }}</small>
            </div>

            <div class="form-field">
              <label for="semester">Semestre / Período <span class="required-mark">*</span></label>
              <input id="semester" v-model="form.semester" type="number" min="1" max="20" step="1" inputmode="numeric" placeholder="Ex.: 3" :aria-invalid="!!errors.semester" :aria-describedby="errors.semester ? 'semester-error' : undefined" @input="delete errors.semester">
              <small v-if="errors.semester" id="semester-error" class="field-error">{{ errors.semester }}</small>
            </div>
          </div>

          <div v-if="form.area === 'Outro'" class="form-field">
            <label for="other-area">Qual é sua área? <span class="required-mark">*</span></label>
            <input id="other-area" v-model="form.otherArea" type="text" placeholder="Conte pra gente" :aria-invalid="!!errors.otherArea" :aria-describedby="errors.otherArea ? 'other-area-error' : undefined" @input="delete errors.otherArea">
            <small v-if="errors.otherArea" id="other-area-error" class="field-error">{{ errors.otherArea }}</small>
          </div>

          <fieldset class="form-field interest-field">
            <legend>Interesses / Habilidades <span class="optional-mark">(opcional)</span></legend>
            <div class="interest-list">
              <label v-for="interest in interestOptions" :key="interest" class="interest-option">
                <input v-model="form.interests" type="checkbox" :value="interest">
                <span>{{ interest }}</span>
              </label>
            </div>
          </fieldset>

          <div class="form-field bio-field">
            <label for="bio">Mensagem / Bio curta <span class="optional-mark">(opcional)</span></label>
            <textarea id="bio" v-model="form.bio" rows="4" :maxlength="maxBioLength" placeholder="O que você está aprendendo ou gostaria de compartilhar?"></textarea>
            <small class="character-count" :class="{ 'character-limit': form.bio.length === maxBioLength }">{{ form.bio.length }}/{{ maxBioLength }}</small>
          </div>

          <button class="submit-button" type="submit">Enviar cadastro <span aria-hidden="true">↗</span></button>
          <p class="privacy-note">Demonstração sem backend: os dados são exibidos no console do navegador.</p>
        </form>
      </section>
      <p class="page-footer">Aprender em comunidade muda tudo.</p>
    </div>
  </main>
</template>

<style scoped>
.registration-page{min-height:100vh;padding:34px 20px 24px;background:radial-gradient(ellipse at 50% 0%,#d9f4e9 0%,#edf8f3 24%,#f4f7f5 58%,#f7f8f7 100%);color:#182c25;font-family:Arial,sans-serif}.registration-shell{width:min(100%,680px);margin:0 auto}.back-link{display:inline-flex;align-items:center;gap:9px;margin-bottom:17px;color:#365448;font-size:13px;font-weight:600;text-decoration:none}.back-link span{font-size:18px;transition:transform .2s}.back-link:hover span{transform:translateX(-3px)}.registration-card{padding:34px 46px 32px;border:1px solid #deebe3;border-radius:18px;background:#fff;box-shadow:0 22px 65px #214a3212}.registration-heading{text-align:center;margin-bottom:25px}.heading-mark{width:43px;height:43px;display:grid;place-items:center;margin:0 auto 14px;border-radius:14px;background:#e5f7ee;color:#168355;font-size:22px}.eyebrow{margin:0 0 7px;color:#168355;font-size:10px;font-weight:700;letter-spacing:1.8px}.registration-heading h1{margin:0;font-size:clamp(27px,5vw,36px);line-height:1.2;letter-spacing:-1.5px}.registration-heading h1 span{color:#168355}.registration-heading>p:last-child{max-width:420px;margin:10px auto 0;color:#76867d;font-size:13px;line-height:1.65}.registration-form{display:grid;gap:16px}.form-field{min-width:0;display:flex;flex-direction:column;gap:7px}.form-field>label,.interest-field legend{color:#263a31;font-size:12px;font-weight:700}.required-mark{color:#cf5353}.optional-mark{color:#8c9992;font-size:10px;font-weight:400}.form-field input:not([type=checkbox]),.form-field select,.form-field textarea{width:100%;min-height:43px;padding:0 12px;border:1px solid #dce7e0;border-radius:8px;outline:none;background:#fff;color:#25372e;font:inherit;font-size:12px;transition:border-color .18s,box-shadow .18s}.form-field textarea{min-height:94px;padding:11px 12px;resize:vertical;line-height:1.6}.form-field input::placeholder,.form-field textarea::placeholder{color:#a2afa8}.form-field select{color:#384a40}.form-field input:focus,.form-field select:focus,.form-field textarea:focus{border-color:#38a978;box-shadow:0 0 0 3px #1683551a}.form-field [aria-invalid=true]{border-color:#d85f65!important}.form-grid{display:grid;grid-template-columns:1.4fr .6fr;gap:13px}.interest-field{margin:0;padding:0;border:0}.interest-field legend{margin-bottom:9px}.interest-list{display:flex;flex-wrap:wrap;gap:8px}.interest-option{position:relative;cursor:pointer}.interest-option input{position:absolute;width:1px;height:1px;opacity:0}.interest-option span{min-height:32px;padding:0 11px;display:inline-flex;align-items:center;border:1px solid #dce7e0;border-radius:7px;background:#fff;color:#607167;font-size:11px;transition:background .18s,border-color .18s,color .18s}.interest-option input:checked+span{border-color:#52ad80;background:#e9f7ef;color:#176843}.interest-option input:focus-visible+span{outline:3px solid #16835538}.bio-field{position:relative}.character-count{align-self:flex-end;margin-top:-3px;color:#8b9991;font-size:10px}.character-limit{color:#bb4d53}.field-error{color:#bc4148;font-size:10px}.submit-button{min-height:47px;padding:0 17px;display:flex;align-items:center;justify-content:center;gap:14px;border:0;border-radius:8px;background:#168355;color:#fff;font-size:13px;font-weight:700;cursor:pointer;transition:background .18s,transform .18s}.submit-button:hover{transform:translateY(-1px);background:#106c45}.privacy-note{margin:-7px 0 0;text-align:center;color:#8d9992;font-size:10px;line-height:1.5}.success-message{position:relative;display:flex;align-items:flex-start;gap:12px;margin-bottom:19px;padding:13px 35px 13px 13px;border:1px solid #a9dec0;border-radius:9px;background:#edfaf2;color:#285d40}.success-check{flex:none;width:25px;height:25px;display:grid;place-items:center;border-radius:50%;background:#d5f1e0;color:#218255;font-weight:700}.success-message p{margin:0;font-size:11px;line-height:1.65}.success-dismiss{position:absolute;top:7px;right:10px;border:0;background:none;color:#477658;font-size:19px;cursor:pointer}.page-footer{margin:17px 0 0;text-align:center;color:#829389;font-size:11px}@media(max-width:560px){.registration-page{padding:20px 13px}.registration-card{padding:27px 19px 23px}.form-grid{grid-template-columns:1fr;gap:16px}.registration-heading{margin-bottom:21px}.registration-form{gap:14px}}
</style>
