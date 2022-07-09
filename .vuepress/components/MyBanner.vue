<template>
  <section class="banner-wrapper" :style="{ ...bgImageStyle }">
    <div class="hero-content">
      <img
        v-if="heroImage"
        :src="heroImage"
        :style="{
          heroImageStyle,
        }"
        alt="heroImage"
      />
      <h1 
        v-if="frontmatter?.myBanner?.heroText"
  
      >
        {{ frontmatter?.myBanner?.heroText }}
      </h1>
      <p 
       v-if="frontmatter?.myBanner?.tagline"

      > 
      {{ frontmatter?.myBanner?.tagline }}
      </p>
    </div>
  </section>
</template>

<script setup lang="ts">
import { computed } from "vue";
import { usePageFrontmatter, withBase } from '@vuepress/client'

console.log("1232")
const frontmatter = usePageFrontmatter()
const heroImage = computed(() => {
  return frontmatter.value?.myBanner?.heroImage
    ? withBase(frontmatter.value?.myBanner?.heroImage)
    : null
})

const heroImageStyle = computed(
  () => frontmatter.value.myBanner.heroImageStyle || {}
)

const heroTextStyle = computed(
  () => frontmatter.value.myBanner.heroTextStyle || {}
)

const taglineStyle = computed(
  () => frontmatter.value.myBanner.taglineStyle || {}
)

const bgImageStyle = computed(() => {
  const { bgImageStyle, bgImage } = frontmatter.value?.myBanner || {}

  const initBgImageStyle = bgImage ? {
    textAlign: 'center',
    overflow: 'hidden',
    background: `url(${withBase(bgImage)}) center/cover no-repeat`
  } : {}

  return bgImageStyle ? { ...initBgImageStyle, ...bgImageStyle } : initBgImageStyle
})
// console.log("345")
</script>
