<script>
    import "../app.css";

    import { page } from '$app/stores';
    import { env } from '$env/dynamic/public';

    export let API_PREFIX = env.PUBLIC_API_PREFIX;
    console.log('API_PREFIX', env, API_PREFIX);

    let slug;
    $: {
      //console.log($page);
      slug = $page.route.id.split('/')[1];
      console.log('slug', slug);
    }


</script>

<!--
  This example requires updating your template:

  ```
  <html class="h-full bg-gray-100">
  <body class="h-full">
  ```
-->
<div class="min-h-full">
  <div class="bg-gray-800 pb-32">
    <nav class="bg-gray-800">
      <div class="mx-auto max-w-7xl sm:px-6 lg:px-8">
        <div class="border-b border-gray-700">
          <div class="flex h-16 items-center justify-between px-4 sm:px-0">
            <div class="flex items-center">
              <div class="flex-shrink-0">
                <img class="h-8 w-32" src="/powerloom-logo.svg" alt="PowerLoom Logo">
              </div>
              <div class="hidden md:block">
                <div class="ml-10 flex items-baseline space-x-4">
                  <!-- Current: "bg-gray-900 text-white", Default: "text-gray-300 hover:bg-gray-700 hover:text-white" -->
                  <a href="/" class={slug == "" ? "bg-gray-900 text-white px-3 py-2 rounded-md text-sm font-medium" : "text-gray-300 hover:bg-gray-700 hover:text-white px-3 py-2 rounded-md text-sm font-medium"} aria-current="page">Dashboard</a>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </nav>
    <header class="py-10">
      <div class="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
        <h1 class="text-3xl font-bold tracking-tight text-white">{slug == "" ? "Dashboard" : slug}</h1>
      </div>
    </header>
  </div>

  <main class="-mt-32">
    <div class="mx-auto max-w-7xl px-4 pb-12 sm:px-6 lg:px-8">
      <!-- Replace with your content -->
      <div class="rounded-lg bg-white px-5 py-6 shadow sm:px-6">
        <div class="h-96 rounded-lg">
          <slot />
        </div>
      </div>
      <!-- /End replace -->
    </div>
  </main>
</div>

        