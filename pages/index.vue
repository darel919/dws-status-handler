<script setup>
const route = useRoute()
const code = route.query.code || null
const origin = route.query.origin || null
</script>

<template>
    <!-- 400 Bad Request -->
    <section v-if="code == 400" class="px-4 sm:px-2 pb-28 bg-red-500 text-white h-screen w-screen flex flex-col sm:flex-row justify-center items-center">
        <img loading="eager" class="h-[40vh] sm:h-[50vh]" src="https://cdni.iconscout.com/illustration/premium/thumb/male-police-saying-stop-9465803-7693033.png"/>
        <div id="content" >
            <h1 class="text-6xl font-bold mb-4">Stop!</h1>
            <h4 ><b>{{ origin }}</b> rejected your connection due to a bad request.</h4>
            <footer>
                <p class="text-white font-bold mt-2 font-mono">400 Bad Request</p>
            </footer>
        </div>
    </section>
    <!-- 401 Unauthorized -->
    <section v-else-if="code == 401" class="px-4 sm:px-2 pb-28 bg-red-500 text-white h-screen w-screen flex flex-row justify-center items-center">
        <img loading="eager" class="h-[50vh]" src="https://cdni.iconscout.com/illustration/premium/thumb/male-police-saying-stop-9465803-7693033.png"/>
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Stop!</h1>
            <h4 class="my-4">Access to <b>{{ origin }}</b> requires authentication.</h4>
            <footer>
                <p class="text-white font-bold mt-2 font-mono">401 Unauthorized</p>
            </footer>
        </div>
    </section>
    <!-- 403 Forbidden -->
    <section v-else-if="code == 403" class="px-4 sm:px-2 pb-28 bg-red-500 text-white h-screen w-screen  flex flex-row justify-center items-center">
        <img loading="eager" class="h-[50vh]" src="https://cdni.iconscout.com/illustration/premium/thumb/male-police-saying-stop-9465803-7693033.png"/>
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Access Denied</h1>
            <h2 class="my-4">You do not have permission to access <b>{{ origin }}</b>.</h2>
            <footer>
                <p class="text-white font-bold mt-2 font-mono">403 Forbidden</p>
            </footer>
        </div>
    </section>
    <!-- Non-security related errors: blue screen -->
    <section v-else-if="code == 404" class="px-4 sm:px-2 pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
  <path stroke-linecap="round" stroke-linejoin="round" d="M9.879 7.519c1.171-1.025 3.071-1.025 4.242 0 1.172 1.025 1.172 2.687 0 3.712-.203.179-.43.326-.67.442-.745.361-1.45.999-1.45 1.827v.75M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Zm-9 5.25h.008v.008H12v-.008Z" />
</svg>

        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Page Not Found</h1>
            <h4>We couldn't find <b>{{ origin }}</b>.</h4>
            <h2 class="font-bold my-4">Check the URL and try again.</h2>
            <footer>
                <p class="text-red-400 mt-2 font-mono" >404 Not Found</p>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 429" class=" px-4 sm:px-2 pb-28 bg-black text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 9v3.75m-9.303 3.376c-.866 1.5.217 3.374 1.948 3.374h14.71c1.73 0 2.813-1.874 1.948-3.374L13.949 3.378c-.866-1.5-3.032-1.5-3.898 0L2.697 16.126ZM12 15.75h.007v.008H12v-.008Z" />
        </svg>

        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Slow Down!</h1>
            <h2 class="font-bold my-4">You're making requests too quickly.</h2>
            <h4>Please wait a moment before trying to access <b>{{origin}}</b> again.</h4>
            <footer>
                <p class="text-red-400 mt-2 font-mono">429 Too Many Requests</p>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 500" class="px-4 sm:px-2 pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">

        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="m9.75 9.75 4.5 4.5m0-4.5-4.5 4.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>

        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Oops!</h1>
            <h2 class="font-bold my-4">We're experiencing an internal error.</h2>
            <h4>The page is temporarily unavailable.</h4>
            <p><br> Please check back soon.</p>
            <!-- <p>This page will auto-refresh every 10 seconds.</p> -->
            <footer>
                <p class="text-red-400 mt-2 font-mono">500 Internal Server Error</p>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 502" class="px-4 sm:px-2 pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="m9.75 9.75 4.5 4.5m0-4.5-4.5 4.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>
        
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Uh-oh!</h1>
            <h2>There was a problem connecting to the server.</h2>
            <h4>The page is temporarily unavailable.</h4>
            <p><br> Please check back soon.</p>
            <!-- <p>This page will auto-refresh every 10 seconds.</p> -->
            <footer>
                <p class="text-red-400 mt-2 font-mono">502 Bad Gateway</p>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 503" class=" px-4 sm:px-2 pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="m9.75 9.75 4.5 4.5m0-4.5-4.5 4.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>
        
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Service Unavailable</h1>
            <h2>We're currently experiencing technical difficulties.</h2>
            <h4>The page you're trying to access is unavailable.</h4>
            <p><br> Please try again later.</p>
            <footer>
                <p class="text-red-400 mt-2 font-mono">503 Service Unavailable</p>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 504" class="pb-28 bg-blue-500  dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Timeout</h1>
            <h2>The server took too long to respond.</h2>
            <h4>The page is temporarily unavailable.</h4>
            <p><br> Please check back soon.</p>
            <footer>
                <p class="text-red-400 mt-2 font-mono">504 Gateway Timeout</p>
            </footer>
        </div>
    </section>
    <!-- Unknown error: blue screen -->
    <section v-else class=" pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Uh-oh!</h1>
            <h2>An unknown error occurred.</h2>
            <h4 class="font-bold my-4">The page might be unavailable.<br> Please check back soon.</h4>
            <footer>
                <p>{{code ? "Received " +code : 'Unknown error'}} from {{ origin }}</p>
            </footer>
        </div>
    </section>
    <footer class="bg-white dark:bg-black text-black dark:text-white text-center p-4 absolute bottom-0 w-full">
        <p class="text-sm">&copy; 2025 darelisme Web Service. All rights reserved.</p>
        <p class="mt-2 font-bold text-xs">For more information, visit <a href="https://status.darelisme.my.id">status page.</a></p>
        <p class="mt-2 text-xs">Powered by <a href="https://nuxtjs.org" class="text-blue-400">Nuxt.js</a></p>
    </footer>
</template>

<style scoped>
a:hover {
    color: #3b82f6;
    text-decoration: underline;
}
</style>