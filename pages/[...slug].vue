<template>
  <main class="relative mx-auto max-w-prose overflow-hidden bg-white py-16 px-4 sm:px-6 lg:px-8">
    <div class="mb-8">
      <nuxt-link class="mx-auto rounded-xl bg-teal-800 px-8 py-4 text-white" href="/#recent">
        Go Back
      </nuxt-link>
    </div>

    <!-- https://content.nuxtjs.org/api/components/content-doc -->

    <ContentDoc class="" v-slot="{ doc }">
      <h2 class="text-2xl font-semibold">{{ doc.title }}</h2>
      <p class="mb-8 text-gray-500">
        by, {{ doc.author }}, {{ doc.date }}
      </p>
      <div
        class="prose prose-lg first-letter:text-7xl first-letter:text-blue-600"
      >
        <ContentRenderer :value="doc" />
      </div>
    </ContentDoc>
  </main>
</template>
