<script>
  import { getData } from "./services/api";
  import Main from "./components/Main.svelte";
  import Virus from "./components/icons/Virus.svelte";
  import EarthIcon from "./components/icons/EarthIcon.svelte";
  import Error from "./components/icons/Error.svelte";
</script>

{#await getData()}
  <main>
    <div style="display: flex; gap: 10px">
      <Virus />
      <EarthIcon />
    </div>
    <p class="loading-text" />
  </main>
{:then data}
  <Main data={data.data} image={data.image} />
{:catch}
  <main>
    <Error />
    <p>Something went wrong</p>
  </main>
{/await}

<style>
  main {
    display: flex;
    min-height: 100vh;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    gap: 20px;
  }

  p {
    text-align: center;
  }

  .loading-text::after {
    content: "Fetching Covid Status";
    animation: load-ellipsis 2s infinite linear;
  }

  @keyframes load-ellipsis {
    0% {
      content: "Fetching Covid Status";
    }

    12.5% {
      content: "Fetching Covid Status.";
    }

    25% {
      content: "Fetching Covid Status..";
    }

    37.5% {
      content: "Fetching Covid Status...";
    }

    50% {
      content: "Building Earth Map";
    }

    52.5% {
      content: "Building Earth Map.";
    }

    75% {
      content: "Building Earth Map..";
    }

    87.5% {
      content: "Building Earth Map...";
    }

    100% {
      content: "Building Earth Map...";
    }
  }
</style>
