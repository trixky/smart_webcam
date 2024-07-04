<!-- ================================================= SCRIPT -->
<script lang="ts">
  import "../style/reset.css";
  import "../style/tailwind.css";
  import "../style/main.css";

  import Header from "$lib/components/layout/header.svelte";
</script>

<!-- ================================================= CONTENT -->
<Header />
<main class="flex flex-col justify-center items-center p-4">
  <slot />
</main>

<!-- ================================================= CSS -->
<style lang="postcss">
</style>
