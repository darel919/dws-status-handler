<script setup>
const route = useRoute()
const code = route.query.code || null
const origin = route.query.origin || null
const isDev = process.env.NODE_ENV === 'development'
const isLoading = ref(false)
const statusMessage = ref('')
const retryCountdown = ref(0)

const getProtocolForOrigin = (url) => {
    if (url.endsWith('.server.drl')) {
        return 'http://'
    } else if (url.endsWith('.darelisme.my.id')) {
        return 'https://'
    } else {
        return 'https://'
    }
}

const checkOriginHealth = async (url) => {
    try {
        const response = await fetch(url, { 
            method: 'HEAD',
            mode: 'no-cors'
        })
        return true
    } catch {
        return false
    }
}

const handleRetry = async () => {
    if (!origin) return
    
    isLoading.value = true
    statusMessage.value = 'Checking...'
    
    let url = origin
    if (!origin.startsWith('http://') && !origin.startsWith('https://')) {
        const protocol = getProtocolForOrigin(origin)
        url = protocol + origin
    }
    
    const isHealthy = await checkOriginHealth(url)
    if (isHealthy) {
        statusMessage.value = `Redirecting to ${origin}...`
        navigateTo(url, {external: true})
    } else {
        statusMessage.value = 'Retrying in 5 seconds...'
        retryCountdown.value = 5
        
        const countdownInterval = setInterval(() => {
            retryCountdown.value--
            if (retryCountdown.value > 0) {
                statusMessage.value = `Retrying in ${retryCountdown.value} seconds...`
            } else {
                clearInterval(countdownInterval)
                statusMessage.value = ''
            }
        }, 1000)
        
        setTimeout(() => {
            isLoading.value = false
            handleRetry()
        }, 5000)
    }
}

const startHealthCheck = () => {
    if (origin) {
        const interval = setInterval(async () => {
            let url = origin
            if (!origin.startsWith('http://') && !origin.startsWith('https://')) {
                const protocol = getProtocolForOrigin(origin)
                url = protocol + origin
            }
            
            const isHealthy = await checkOriginHealth(url)
            if (isHealthy) {
                clearInterval(interval)
                statusMessage.value = `Redirecting to ${origin}...`
                navigateTo(url, {external: true})
            }
        }, 5000)
    }
}

onMounted(() => {
    startHealthCheck()
})

if(!isDev) {
    if (!code || !origin) {
        navigateTo('https://darelisme.my.id', {external: true})
    }
}
</script>

<template>
    <!-- 400 Bad Request -->
    <section v-if="code == 400" class="px-4 sm:px-2 pb-28 bg-red-500 text-white h-screen w-screen flex flex-col sm:flex-row justify-center items-center">
        <img loading="eager" class="h-[40vh] sm:h-[50vh]" src="https://cdni.iconscout.com/illustration/premium/thumb/male-police-saying-stop-9465803-7693033.png"/>
        <div id="content" >
            <h1 class="text-6xl font-bold mb-4">Stop!</h1>
            <h4 ><b>{{ origin }}</b> rejected your connection due to a bad request.</h4>
            <footer>
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">400 Bad Request</p>
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
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">401 Unauthorized</p>
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
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">403 Forbidden</p>
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
                <p class="mt-2 font-mono bg-black text-white w-fit p-1" >404 Not Found</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
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
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">429 Too Many Requests</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 500" class="px-4 sm:px-2 pb-28 bg-amber-400  dark:bg-amber-600 text-black dark:text-white h-screen w-screen flex flex-col justify-center items-center">
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
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">500 Internal Server Error</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 501" class="px-4 sm:px-2 pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Wow!</h1>
            <h2 class="font-bold my-4">Someone's got a bit too excited!</h2>
            <h4>The page is not available.</h4>
            <footer>
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">501 Not Implemented</p>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 502" class="px-4 sm:px-2 pb-28 bg-amber-400  dark:bg-amber-600 text-black dark:text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="m9.75 9.75 4.5 4.5m0-4.5-4.5 4.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>
        
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Oops!</h1>
            <h2>There was a problem connecting to <b>{{ origin }}</b></h2>
            <h4>The page is temporarily unavailable.</h4>
            <p><br> Please check back soon.</p>
            <!-- <p>This page will auto-refresh every 10 seconds.</p> -->            
             <footer>
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">502 Bad Gateway</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 503" class=" px-4 sm:px-2 pb-28 bg-black text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4 text-red-500">
            <path stroke-linecap="round" stroke-linejoin="round" d="m9.75 9.75 4.5 4.5m0-4.5-4.5 4.5M21 12a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>
        
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Service Unavailable</h1>
            <h2><b>{{origin}}</b> is unavailable.</h2>
            <h4>If you're keep seeing this repeatedly, this service might be switched off.</h4>
            <p><br> Please try again later.</p>            
            <footer>
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">503 Service Unavailable</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 714 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
            </footer>
        </div>
    </section>
    <section v-else-if="code == 504" class="pb-28 bg-amber-400  dark:bg-amber-600 text-black dark:text-white h-screen w-screen flex flex-col justify-center items-center">
        <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-24 mb-4">
            <path stroke-linecap="round" stroke-linejoin="round" d="M12 6v6h4.5m4.5 0a9 9 0 1 1-18 0 9 9 0 0 1 18 0Z" />
        </svg>

        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Timeout</h1>
            <h2 class="font-bold">{{origin}} took too long to respond.</h2>
            <h4>The page is temporarily unavailable.</h4>
            <p><br> Please check back soon.</p>            
            <footer>
                <p class="mt-2 font-mono bg-black text-white w-fit p-1">504 Gateway Timeout</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 714 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
            </footer>
        </div>
    </section>
    <!-- Unknown error: blue screen -->
    <section v-else-if="code" class=" pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Uh-oh!</h1>
            <h2>An unknown error occurred.</h2>
            <h4 class="font-bold my-4">The page might be unavailable.<br> Please check back soon.</h4>            <footer>
                <p>{{code ? "Received " +code : 'Unknown error'}} from {{ origin }}</p>
                <p v-if="statusMessage" class="mt-2 text-yellow-300">{{ statusMessage }}</p>
                <button @click="handleRetry" class="mt-4 flex items-center bg-white text-blue-500 hover:bg-blue-100 font-bold py-2 px-4 rounded-md" :disabled="isLoading">
                    <svg v-if="isLoading" class="animate-spin size-6 mr-2" viewBox="0 0 24 24">
                        <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" fill="none"></circle>
                        <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 714 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                    </svg>
                    <svg v-else xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke-width="1.5" stroke="currentColor" class="size-6 mr-2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M16.023 9.348h4.992v-.001M2.985 19.644v-4.992m0 0h4.992m-4.993 0 3.181 3.183a8.25 8.25 0 0 0 13.803-3.7M4.031 9.865a8.25 8.25 0 0 1 13.803-3.7l3.181 3.182m0-4.991v4.99" />
                    </svg>
                    <span>{{ isLoading ? 'Loading...' : 'Retry' }}</span>
                </button>
            </footer>
        </div>
    </section>
    <section v-else class=" pb-28 bg-blue-500 dark:bg-blue-900 text-white h-screen w-screen flex flex-col justify-center items-center">
        <div id="content">
            <h1 class="text-6xl font-bold mb-4">Uh-oh!</h1>
            <p>If you're seeing this page, it probably means the service is unavailable.</p>
        </div>
    </section>
    <footer class="bg-black dark:bg-white text-white dark:text-black text-center p-4 absolute bottom-0 w-full">
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