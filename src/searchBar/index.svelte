<script>
  import { onMount, afterUpdate, createEventDispatcher } from "svelte";
  import icons from "./icons";
  import Fuse from "fuse.js";

  export let searchable = [];
  export let keys = [];
  export let placeholder = "search for a country";
  let searchString = "";
  let suggestions = [];
  let selectedIndex = null;
  let fuse = null;

  let dispatch = createEventDispatcher();

  let options = {
    shouldSort: true,
    threshold: 0.6,
    location: 0,
    distance: 100,
    maxPatternLength: 32,
    minMatchCharLength: 1,
    keys,
  };

  $: {
    if (searchable.length > 0) fuse = new Fuse(searchable, options);
  }

  onMount(() => {});

  afterUpdate(() => {});

  function queryLocation(event) {
    searchString = event.target.value;

    if (event.key === "ArrowUp") {
      moveUp();
    }

    if (event.key === "ArrowDown") {
      moveDown();
    }

    if (event.key === "Enter") {
      goTo();
    }

    suggestions = fuse.search(searchString).slice(0, 5);
    selectedIndex = Math.min(suggestions.length - 1, selectedIndex);
    selectedIndex = selectedIndex >= 0 ? selectedIndex : null;
  }

  function moveDown() {
    selectedIndex = Math.min(suggestions.length - 1, selectedIndex + 1);
  }

  function moveUp() {
    selectedIndex = Math.max(0, selectedIndex - 1);
  }

  function goTo() {
    if (selectedIndex !== null) {
      chooseSuggestion(suggestions[selectedIndex]);
      clearSuggestions();
    }
  }

  function chooseSuggestion(suggestion) {
    searchString = suggestion.item[keys[0]];
    dispatch("choose", suggestion.item);
  }

  function clearSuggestions() {
    setTimeout(() => {
      suggestions = [];
    }, 300);
    selectedIndex = null;
    dispatch("blur", "nothing");
  }
</script>

<section class="relative">
  <section class="search relative">
    <input
      type="text"
      bind:value={searchString}
      on:keyup={queryLocation}
      on:input={queryLocation}
      on:blur={clearSuggestions}
      {placeholder}
      spellcheck="false"
    />
    {#if searchString !== ""}
      <button
        style="right: calc(0.4rem + 24px);"
        on:click={() => {
          searchString = "";
          dispatch("close");
        }}>{@html icons.clear}</button
      >
    {/if}
    <i style="right: 0.4rem;">{@html icons.search}</i>
  </section>
  <section class="suggestions absolute w-full">
    {#each suggestions as suggestion, j}
      <span
        class="block"
        class:selected={j === selectedIndex}
        on:click={() => {
          chooseSuggestion(suggestion);
        }}>{suggestion.item[keys[0]]}</span
      >
    {/each}
  </section>
</section>

<style src="./style.scss"></style>
