<script lang="ts" setup>
const route = useRoute()
const slug = route.params.slug as string

const { email, refresh, getEmail, getVueCode } = useEmail()

const emailTemplate = ref({
  vue: '',
  html: '',
  plainText: '',
})

await getEmail(slug)
const emailFile: any = email.value ? resolveComponent(email.value.resComponent!) : null

watch(refresh, async () => {
  await loadMarkups()
})

async function loadMarkups() {
  if (!emailFile || !email.value.component) return
  const vue = await getVueCode(email.value.component)
  const html = await useRender(emailFile, null, { pretty: true })
  const plainText = await useRender(emailFile, null, { plainText: true })

  emailTemplate.value = {
    vue,
    html,
    plainText,
  }
}

await loadMarkups()

useHead({
  title: email.value ? `${email.value.label}` : 'Email not found',
})
</script>

<template>
  <ClientOnly>
    <EmailPreview v-if="email" :slug="email.label" :template="emailTemplate" />
  </ClientOnly>
</template>
