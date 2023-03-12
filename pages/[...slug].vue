<template>
  <main class="relative mx-auto max-w-max overflow-hidden bg-white py-16 px-4 sm:px-6 lg:px-8">
    <div class="mb-8">
      <nuxt-link class="mx-auto rounded-xl bg-teal-800 px-8 py-4 text-white" href="/#recent">
        Go Back
      </nuxt-link>
    </div>

    <!-- https://content.nuxtjs.org/api/components/content-doc -->

    <ContentDoc class="" >
      <template v-slot="{ doc }">
          <h2 class="text-2xl font-semibold break-normal">{{ doc.title }}</h2>
        <p class="mb-8 text-gray-500">
          by, {{ doc.author }}, {{ doc.date }}
        </p>
        <div
          class="prose prose-lg first-letter:text-7xl first-letter:text-blue-600"
        >
          <ContentRenderer :value="doc" />
        </div>
      </template>
      <template #not-found>
        <div class="grid h-screen place-content-center gap-3 text-center">
          <h1 class="text-5xl text-blue-500">Oops.. I have not wirtten that, yet.</h1>
          <p class="text-xl text-red-600">
            Check back soon or send me a request on a topic you would like me to cover.
          </p>
          <nuxt-link href="https://kcanamar-portfolio.vercel.app/contact" target="_blank" class="mx-auto rounded-xl bg-teal-800 px-20 py-4 text-white">
              Contact Me 
          </nuxt-link>
  </div>
      </template>
    </ContentDoc>
  </main>
</template>
