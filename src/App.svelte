<script>
  import { onMount, afterUpdate } from "svelte";
  import Searchbar from "./searchBar/index.svelte";
  import { foodIcons } from "./foodIcons";
  import Legend from "./legend.svelte";

  import { slide } from "svelte/transition";

  // -- Component lifecycle --
  onMount(() => {});

  afterUpdate(() => {});

  // d3
  import { scaleLinear } from "d3-scale";
  import { csv } from "d3-fetch";
  const d3 = { csv, scaleLinear };

  // colors
  let colors = ["#f1eac8", "#e5b9ad", "#d98994", "#d0587e"];

  // data
  let dataPath = "./data/whichFoodTypeMostCaloriesGer.csv";
  let data, dataLoaded;
  let sortedData;
  d3.csv(dataPath).then((d) => {
    data = d;
    console.log(`data: `, data);
    dataLoaded = true;
    sortedData = data.sort((a, b) => +a.total < +b.total);
    console.log(`sd: `, sortedData);
  });

  let countries;
  let all;
  $: if (dataLoaded) {
    // countries = data.map(e => e.country_de_clean)

    let c = data.reduce((acc, curr) => {
      acc.push({ name: curr["area"] });
      return acc;
    }, []);
    countries = c;
  }

  let selectedDataUnclean;
  let selectedDataClean;
  let selectionMade;

  let headers;
  $: if (selectedDataClean) {
    headers = {
      item_de_clean: "Typ",
      value: `Calories from ${selectedDataClean["item"]}<br><span style="font-size: .8rem">(% of total)</span>`,
      total:
        'Calories total <br><span style="font-size: .8rem">(per day)</span>',
      rank: "Rank",
    };
  }

  let selectedCountry;
  let selectedCountry_en;
  let countryCode;
  let rank;
  function chose(ele) {
    contentCollapsed = true;
    let {
      detail: { name: country },
    } = ele;

    selectedCountry = country;
    console.log(`country: `, country);
    rank = sortedData.map((e) => e.area).indexOf(country);
    console.log(`rank: `, rank);
    let selectedCountry_en_obj = data.find((e) => e.area == country);
    selectedCountry_en = selectedCountry_en_obj.area;
    countryCode = selectedCountry_en_obj.area_code_m49;

    selectedDataUnclean = data.find((e) => e.area == country);
    selectedDataClean = cleanData(selectedDataUnclean);
    selectionMade = true;
    content = getContent();
    contentLoaded = true;
  }

  function cleanData(dataUnclean) {
    // make copy
    let dataIntermediate = Object.assign({}, dataUnclean);
    let dataKeys = Object.keys(dataUnclean);
    let keysToKeep = ["item", "value", "total", "rank"];

    dataKeys.forEach((key, i) => {
      let d = dataUnclean[key];

      if (key === "item") {
        dataIntermediate[key] = dataUnclean[key];
      }

      if (key === "value") {
        let share = parseFloat(dataUnclean["share"]).toLocaleString("de", {
          maximumFractionDigits: 2,
        });
        let totalCalsFromProd = d;
        let res = `${totalCalsFromProd} <span style="font-size: .8rem">(${share} %)</span>`;
        dataIntermediate[key] = res;
      }
      if (key === "total") {
        dataIntermediate[key] = dataUnclean[key];
      }
      if (key === "rank") {
        dataIntermediate[key] = dataUnclean[key];
      }
    });

    // filter data
    let dataCleaned = Object.keys(dataIntermediate)
      .filter((key) => keysToKeep.includes(key))
      .reduce((obj, key) => {
        obj[key] = dataIntermediate[key];
        return obj;
      }, {});

    return dataCleaned;
  }

  let contentCollapsed = true;

  function closePanel() {
    selectionMade = false;
    contentCollapsed = true;
  }

  let content;
  let contentLoaded;

  function toggleContent() {
    contentCollapsed = !contentCollapsed;
  }

  function getContent() {
    let countryCodePadded = String(countryCode).padStart("3", "0");
    let path = `./data/countries/${countryCodePadded}.csv`;
    let dd = [];

    d3.csv(path).then((d) => {
      for (let item of d) {
        dd.push({
          item: item.item,
          value: item.value,
          share: +item.value * +item.total,
        });
      }
    });

    return dd;
  }

  // legend
  let xPos, xScale;
  let domain = [1500, 4000];
  $: if (selectedDataClean) {
    xScale = d3.scaleLinear().domain(domain).range([0, legendWidth]);
    xPos = xScale(selectedDataClean.total);
  }

  let width;
  let legendWidth, legend;
  $: if (legend) {
    legend;
    legendWidth = legend.clientWidth;
  }

  function resize() {
    if (legend) {
      legendWidth = legend.clientWidth;
    }
  }
</script>

<svelte:window bind:innerWidth={width} on:resize={resize} />

<section class="mx-auto" style="max-width: 640px;">
  {#if countries}
    <div class="container">
      <Searchbar
        searchable={countries}
        keys={["name"]}
        on:choose={(e) => chose(e)}
        on:close={closePanel}
      />
      {#if selectionMade}
        <div class="selection" transition:slide>
          <div class="icon flex justify-center py-2" style="">
            <div
              class="type"
              style="display: flex; align-items: center; font-size: 2rem;"
            >
              <!-- <div class="result-content">🇦🇹</div> -->
            </div>
            <div
              class="type"
              style="display: flex; align-items: center; font-size: 2rem; text-decoration: underline"
            >
              <div class="result-content">
                {selectedDataClean["item"]}
              </div>
            </div>
          </div>
          <div class="results-outer-container relative" style="margin: .2rem">
            <div class="results">
              {#each Object.keys(selectedDataClean) as key, i}
                {#if key != "item"}
                  <div
                    class="result-item-box relative"
                    style="margin-bottom: 3rem; padding-bottom: 2rem;"
                  >
                    <div class="result-header">{@html headers[key]}</div>
                    <div class="result-content" style="font-size: 1.3rem;">
                      {@html selectedDataClean[key]}
                    </div>

                    {#if i === 2}
                      <div
                        bind:this={legend}
                        class="absolute"
                        style="bottom: 0px; left: 0; right: 0; height: 5px;"
                      >
                        <Legend {colors} {domain} barHeight="5" />
                        <div
                          class="absolute indicator"
                          style="bottom: 0px; left: {xPos}px;  height: 5px; background-color: black; width: 2px;"
                        />
                        <div
                          class="absolute country-label"
                          style:transform={xPos > legendWidth / 2
                            ? "translate(-100%, 5px)"
                            : "translate(0, 5px)"}
                          style="font-weight: 900;font-weight: 900;  font-size: .7rem; bottom: 0px; left: {xPos}px;  height: 5px; white-space: nowrap;"
                        >
                          {selectedCountry} (Rank: {rank})
                        </div>
                        <div
                          class="absolute labels"
                          style="bottom: 3px; left: 0px; display: flex; flex-direction: column;"
                        >
                          <span style="font-size: .6rem;"
                            >Zimbabwe<br />(1704 kcal)</span
                          >
                        </div>
                        <div
                          class="absolute labels"
                          style="bottom: 3px; right: 0px; display: flex; flex-direction: column;"
                        >
                          <span style="font-size: .6rem;"
                            >USA<br />(3816 kcal)</span
                          >
                        </div>
                      </div>
                    {/if}
                  </div>
                {/if}
              {/each}
            </div>
            <div
              class="arrow"
              style="position: absolute; left: 50%; bottom: 0; transform: translateX(-50%)"
            >
              <button on:click={toggleContent}
                >{contentCollapsed ? "more" : "less"}</button
              >
            </div>
            {#if !contentCollapsed && contentLoaded}
              <div
                class="content-more"
                style="padding: 1.4rem 0;"
                transition:slide
              >
                {#each Array(content.length + 1) as _, i}
                  <div
                    class="content-more-row"
                    style="display: flex; text-align: center"
                    style:border-bottom={i === 0 ? "1px solid black" : ""}
                    style:font-weight={i === 0 ? 800 : 200}
                  >
                    <!-- header -->
                    {#if i == 0}
                      <div class="name-content-more">Item</div>
                      <div class="value-content-more">Calories</div>
                    {:else}
                      <div class="name-content-more">{content[i - 1].item}</div>
                      <div class="value-content-more">
                        {content[i - 1].value}
                      </div>
                    {/if}
                  </div>
                {/each}
              </div>
            {/if}
          </div>
        </div>
      {/if}
    </div>
  {/if}
</section>

<style lang="scss">
  :root {
    font-size: 12px;
  }

  .container {
    box-sizing: border-box;
    & .results-outer-container {
      outline: 1px solid black;
      border-radius: 0.4rem;
      // padding: 0.3rem;
      & .results {
        display: flex;
        // flex-wrap: wrap;
        justify-content: space-between;
        flex-wrap: wrap;
        // outline: 1px solid black;
        // border-radius: 0.4rem;
        margin: 0.3rem;

        & > div {
          flex-grow: 1;
          padding: 1rem;
          flex-basis: 100px;
        }

        & .result-item-box {
          text-align: center;
          display: flex;
          min-height: 90px;
          flex-direction: column;
          justify-content: space-between;
        }
      }

      // @media screen and (max-width: 350px) {
      // 	& .results {
      // 		display: flex;
      // 		flex-direction: column;
      // 	}
      // }

      & .content-more-row {
        & > div {
          width: 50%;
          word-break: break-all;
        }
      }
    }
  }
</style>
