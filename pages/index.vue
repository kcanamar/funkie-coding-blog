<script setup>
// create a query variable looking into the content/blog directory
const contentQuery = await queryContent('blog').sort({ date: -1 }).find()

</script>

<template>
    <div class="grid h-screen place-content-center gap-3 text-center">
            <h1 class="text-5xl text-blue-500">Welcome to Funkie Coding</h1>
            <p class="text-xl text-red-600">
                Check out the latest blog posts from KCanamar
            </p>
            <nuxt-link href="#recent" class="mx-auto rounded-xl bg-teal-800 px-20 py-4 text-white">
                Go to Blog
            </nuxt-link>
    </div>
    <main  class="h-screen bg-white px-4 pt-16 pb-20 sm:px-6 lg:px-8 lg:pt-24 lg:pb-28">
        <div class="mx-auto max-w-lg lg:max-w-7xl">
            <div class="border-b border-b-blue-200 pb-6">
                <h2 id="recent" class="text-3xl font-semibold tracking-tight text-grey-900 sm:text-4xl">
                    Recent Posts
                </h2>
            </div>

            <!-- reset the query param -->
            <!-- <div class="grid my-4 place-content-center text-center">
                <nuxt-link href="#recent" class="mx-auto rounded-xl bg-teal-800 px-20 py-4 text-white">
                    Go Back
                </nuxt-link>
            </div> -->

            <div class="mt-12 grid gap-16 lg:grid-cols-3 lg:gap-x-5 lg:gap-y-12">

                <!-- https://content.nuxtjs.org/api/components/content-list  -->
                <!-- https://content.nuxtjs.org/api/composables/query-content -->

                    <!-- creates a loop over all content -->
                    <div 
                        v-for="article in contentQuery" 
                        :key="article._path" 
                        class="flex flex-col justify-between rounded-lg border border-gray-300 p-4"
                    >
                        <nuxt-link :href="article._path">
                        <p class="text-xl text-gray-900 ">{{ article.title }}</p>
                        <p>{{ article.description }}</p>
                        </nuxt-link> 

                        <div class="mt-6">
                            <div class="flex justify-between">
                                <p class="text-sm font-medium text-gray-900">
                                    {{article.author}}
                                </p>
                                <p class="text-sm font-medium text-gray-900">
                                    {{article.lang}}
                                </p>
                            </div>
                            <div class="text-sm text-gray-500">
                                <time datetime="2023-03-06">{{article.date}}</time>
                            </div>
                        </div>
                    </div>
            </div>
        </div>
    </main>
</template>