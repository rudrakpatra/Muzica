<script>
  import Muzica from "./lib/Muzica.svelte";
  let handleClick = _ => {
    document.getElementById("app").requestFullscreen();
  }
  let fullscreen = document.fullscreenElement;
  document.onfullscreenchange = _=> fullscreen = document.fullscreenElement;
</script>

{#if fullscreen}
  <Muzica/>
{:else}
  <!-- svelte-ignore a11y-click-events-have-key-events -->
  <div class="fs-prompt" on:click={handleClick}>
    <h1>MUZICA</h1>
    <h4>Digital Piano App - by Rudrak Patra</h4>
    <p>You must play in fullscreen mode</p>
    *for Touch Devices only
  </div>
{/if}
<style>
  .fs-prompt{
    user-select: none;
    width:100%;
    height:100%;
    display:grid;
    place-content:center;
  }
</style>
