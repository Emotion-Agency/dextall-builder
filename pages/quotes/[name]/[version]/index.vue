<script setup lang="ts">
import { ToastColor } from '~/composables/toasts'

import { useTransition } from '~/composables/transition'
import { useQoutesStories } from '~/composables/stories/quotes'
useTransition()

const { isAuth, isLoaded, isInEditor } = useAppState()
const { addToast } = useToasts()

const route = useRoute()

const { name, version } = route.params
const { story, listenStory } = await useQoutesStories(
  name as string,
  version as string
)

listenStory(version)

watch(isInEditor, () => {
  if (isInEditor.value) {
    isAuth.value = true
    isLoaded.value = true
  }
})

onMounted(() => {
  if (isInEditor.value) {
    isAuth.value = true
    isLoaded.value = true
    return
  }
  const isAuthData = localStorage.getItem('isAuth')
  const userData = JSON.parse(localStorage.getItem('user') || '[]')

  if (isAuthData === 'true') {
    isAuth.value = true
    addToast({
      color: ToastColor.success,
      id: Date.now().toString(),
      text: `Welcome back, ${userData.Name}`,
    })
  }
  isLoaded.value = true
})
</script>

<template>
  <Loader v-if="!isLoaded" />
  <div v-else-if="isAuth && isLoaded" data-page>
    <Main
      v-editable="story.content"
      :story="story.content"
      :address="story.content.address"
      :title="story.content.title"
      :scope="story.content.scope"
      :start-quote-date="story.content.start_quote_date"
      :end-quote-date="story.content.end_quote_date"
      :model="story.content.model"
    />
    <ImagesList />
    <Projects />
    <News />
    <Contacts />
  </div>
  <Login
    v-else
    :title="story.content.title"
    :start-quote-date="story.content.start_quote_date"
    :end-quote-date="story.content.end_quote_date"
  />
</template>
