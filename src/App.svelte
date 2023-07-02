<script>
  import Muzica from "./lib/Muzica.svelte";

  let handleClick = _ => {
    document.getElementById("app").requestFullscreen();
  }
  let fullscreen = document.fullscreenElement;
  document.onfullscreenchange = _=> fullscreen = document.fullscreenElement;

  //chords
  let M=(x)=>[x,x+4,x+7];
  let m=(x)=>[x,x+3,x+7];
  let M7=(x)=>[x,x+4,x+7,x+11];
  let m7=(x)=>[x,x+3,x+7,x+10];
  let dim=(x)=>[x,x+3,x+6];
  let aug=(x)=>[x,x+4,x+8];
  let sus2=(x)=>[x,x+2,x+7];
  let sus4=(x)=>[x,x+5,x+7];

  let Tr=(x,d)=>Array.isArray(x)?x.map((x)=>Tr(x,d)):x+d;

  let major=(x)=>[x,x+2,x+4,x+5,x+7,x+9,x+11];
  let minor=(x)=>[x,x+2,x+3,x+5,x+7,x+8,x+10];
  let penta=(x)=>[x,x+2,x+4,x+7,x+9];
  let blues=(x)=>[x,x+3,x+5,x+6,x+7,x+10];
  let augmented=(x)=>[x,x+3,x+4,x+7,x+8,x+11];

  let C4=48,C5=60;
  let A3=45,A4=57;

  let sharps=(x)=>[[],x+1,x+3,[],x+6,x+8,x+10,[],];
  let defaultLayout=[
    sharps(C4),//sharps
    major(C4)//C major
  ]

  let defaultLayoutExtended=[
    [...sharps(C4),...sharps(C5)],//sharps
    [...major(C4),...major(C5)]//C major
  ]

  //get the address of the current page
  let url = window.location.href;
  //find the part after layout=
  let urlLayout = url.split("layout=")[1];
  console.log(urlLayout);
  //find the row array for the current page
  let layout = urlLayout? JSON.parse(urlLayout):defaultLayout;
  console.log(layout);
</script>

{#if fullscreen}
  <Muzica {layout}/>
{:else}
  <div class="fs-prompt">
<pre class="gradient-loop">
888b     d888 888     888 8888888888P 8888888 .d8888b.        d8888
8888b   d8888 888     888       d88P    888  d88P  Y88b      d88888
88888b.d88888 888     888      d88P     888  888    888     d88P888
888Y88888P888 888     888     d88P      888  888           d88P 888
888 Y888P 888 888     888    d88P       888  888          d88P  888
888  Y8P  888 888     888   d88P        888  888    888  d88P   888
888   "   888 Y88b. .d88P  d88P         888  Y88b  d88P d8888888888
888       888  "Y88888P"  d8888888888 8888888 "Y8888P" d88P     888
</pre>
    <h3>Digital Piano App - by Rudrak Patra</h3>
    <p>You must play in fullscreen<br/>
      *for Touch Devices only
    </p>
      <hr/>
    {#if urlLayout}
      <div>using custom layout</div>
      <pre class="layout">{JSON.stringify(layout)}</pre>
      <button on:click={handleClick}>Play Custom Layout</button>
    {:else}
      <button on:click={handleClick}>Play Default Layout</button>
      <div class="customize">
        <h2>How to customize the layout</h2>
        <div>Add this to the page link <pre class="layout">/layout=[[[48,52,55],51,52,53,54,55]]</pre></div>
        <div>e.g. 48 is C4, 49 is C#5 an so on.</div>
      </div>
    {/if}
      <hr/>
      <p>Examples below</p>
      <ul>
        <li><a href="/">default</a></li>
        <li><a href="/?layout={JSON.stringify(defaultLayoutExtended)}">extended</a></li>
        <li><a href="/?layout={JSON.stringify([major(C4)])}" >Major</a></li>
        <li><a href="/?layout={JSON.stringify([minor(C4)])}" >Minor</a></li>
        <li><a href="/?layout={JSON.stringify([blues(C4)])}" >Blues</a></li>
        <li><a href="/?layout={JSON.stringify([penta(C4)])}" >Pentatonic</a></li>
        <li><a href="/?layout={JSON.stringify([augmented(C4)])}" >Augmented</a></li>
        <li><a href="/?layout={JSON.stringify([
                [M(C4+5),M(C4),M(C4-5),...major(C4+12),...major(C4+24)],
                [m(A3+5),m(A3),m(A3-5),...major(A3+3),...major(A3+12+3)],
              ])}">complex</a>
        </li>
      </ul>
  </div>
{/if}
<style>
  .fs-prompt{
    height:100%;
    padding:20px;
    overflow-y: scroll;
    text-align: center;
  }
  button{
    margin:auto;
    width:fit-content;
    padding: 8px 12px;
    border-radius: 5px;
    color:gold;
    border: 1px solid goldenrod;
    background-color: #0000;
    opacity: 0.9;
    font-size: 1.2em;
    cursor: pointer;
  }
  ul{
    list-style-type: none;
    padding: 0;
    display:flex;
    justify-content: space-evenly;
    gap: 1em;
    flex-wrap: wrap;
  }
  .layout{
    background-color: #222;
    overflow-x : auto;
    margin:1rem auto;
    padding:0.5rem;
    max-width:fit-content;
  }
  hr{
    opacity: 0.5;
  }
  .customize{
    font-size: 80%;
    opacity: 0.8;
  }
</style>
