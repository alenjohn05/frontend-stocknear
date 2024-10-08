<script lang='ts'>
  import { stockTicker, displayCompanyName } from "$lib/store";
  import InfoModal from '$lib/components/InfoModal.svelte';
  
  export let data;
  export let marketMoods = {};
  
  let rawData = marketMoods;
  let mode = 'bullish';
  let showFullText = false;
  // Function to split text into paragraphs
  
  //DO NOT TOUCH THIS CODE - SAFARI 15.3 are not compatible with the previous code. The website will break.
  // CONSIDER MOVING THIS LOGIC TO THE BACKEND TO REMOVE THE RISK COMPLETELY
  function splitIntoParagraphs(text) {
    if (!text) return [];
    
    // Replace "U.S." and "vs." with placeholders
    const placeholderUS = '__US__';
    const placeholderVs = '__VS__';
    
    let tempText = text.replace(/U\.S\./g, placeholderUS)
                       .replace(/vs\./g, placeholderVs);
    
    // Split the text based on periods followed by spaces
    let paragraphs = tempText.split(/\. \s*/);

    // Restore the placeholders back to "U.S." and "vs."
    paragraphs = paragraphs.map(paragraph => 
        paragraph.replace(new RegExp(placeholderUS, 'g'), 'U.S.')
                 .replace(new RegExp(placeholderVs, 'g'), 'vs.')
    );

    return paragraphs;
}


  
  let paragraphs = splitIntoParagraphs(rawData?.bullSays);
  
  
  function handleMode(i) {
    activeIdx = i;
    if(activeIdx === 0) {
          paragraphs = splitIntoParagraphs(rawData?.bullSays);
      }
      else if (activeIdx === 1) {
          paragraphs = splitIntoParagraphs(rawData?.bearSays);
      }
  }
  
  const tabs = [
      {
        title: "Bull Case",
      },
      {
        title: "Bear Case",
      },
    ];
  
    let activeIdx = 0;
  
  $: {
      if($stockTicker && typeof window !== 'undefined') {
          rawData = marketMoods;
          mode = 'bullish';
          showFullText = false;
          paragraphs = splitIntoParagraphs(rawData?.bullSays);
      }
  }
      
  </script>
  
  
  {#if Object?.keys(marketMoods)?.length !== 0}
  
  <div class="space-y-3 overflow-hidden">  
      <!--Start Content-->
      <div class="w-auto lg:w-full p-1 flex flex-col m-auto">
  
        <div class="flex flex-col items-center w-full mb-6">
            <div class="flex flex-row justify-start mr-auto items-center">
              <!--<img class="h-10 inline-block mr-2" src={copilotIcon} />-->
              <div class="flex flex-row items-center">
                <label for="bullBearCase" class="mr-1 cursor-pointer flex flex-row items-center text-white text-xl sm:text-3xl font-bold">
                  Bull Case vs Bear Case
                </label>
                <InfoModal
                  title={"Bull Case vs Bear Case"}
                  content={`Before investing, examine both perspectives. We offer brief analyst report summaries, highlighting both positive ("Bulls Say") and negative ("Bears Say") viewpoints on ${$displayCompanyName}`}
                  id={"bullBearCase"}
                />
                </div>
  
            </div>
        </div>
  
      {#if data?.user?.tier === 'Pro'}
     <!--Start Header-->
     <div class="bg-[#313131] w-fit relative flex flex-wrap items-center justify-center rounded-lg p-1 -mt-2">
        {#each tabs as item, i}
          <button
            on:click={() => handleMode(i)}
            class="group relative z-[1] rounded-full px-6 py-1 {activeIdx === i
              ? 'z-0'
              : ''} "
          >
            {#if activeIdx === i}
                <div
                  class="absolute inset-0 rounded-lg bg-purple-600"
                ></div>
            {/if}
            <span
              class="relative text-sm block font-medium duration-200 text-white">
              {item.title}
            </span>
          </button>
        {/each}
    </div>
    
  
    <!--End Header-->
      <span class="text-gray-200 text-xs sm:text-[0.915rem] italic mt-6 sm:ml-auto">
        Updated {rawData?.date}
      </span>
      <div class="flex mt-5 h-auto">
      
        <div class="{activeIdx === 0 ? 'bg-[#10DB06]' : 'bg-[#FF2F1F]'} w-3.5 rounded-l-xl" />
        <span class="text-gray-100 ml-3 text-[1rem] ">
          {#if showFullText}
          {#each (showFullText ? paragraphs : paragraphs?.slice(0,1)) as paragraph, index}
          <p class="{index !== 0 ? 'mt-1' : ''} pr-1">{paragraph} {paragraphs?.length <= index+1 ? '' : '.'}</p>
          {/each}
          {:else}
          {paragraphs?.at(0)?.slice(0,250) + '...'}
          {/if}
        </span>
      </div>
      {:else}
      <div class="shadow-lg shadow-bg-[#000] bg-[#111112] sm:bg-opacity-[0.5] text-sm sm:text-[1rem] rounded-md w-full p-4 min-h-24 mt-4 text-white m-auto flex justify-center items-center text-center font-semibold">
        <svg class="mr-1.5 w-5 h-5 inline-block"xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="#A3A3A3" d="M17 9V7c0-2.8-2.2-5-5-5S7 4.2 7 7v2c-1.7 0-3 1.3-3 3v7c0 1.7 1.3 3 3 3h10c1.7 0 3-1.3 3-3v-7c0-1.7-1.3-3-3-3M9 7c0-1.7 1.3-3 3-3s3 1.3 3 3v2H9z"/></svg>
        Unlock content with <a class="inline-block ml-2 text-blue-400 hover:sm:text-white" href="/pricing">Pro Subscription</a>
      </div>
      {/if}
  
      </div>
  
      <label on:click={() => showFullText = !showFullText} class="{data?.user?.tier !== 'Pro' ? 'hidden' : ''} cursor-pointer m-auto flex justify-center items-center mt-5">
          <svg class="w-10 h-10 transform {showFullText ? 'rotate-180' : ''} " xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24"><path fill="#2A323C" d="M12 2C6.48 2 2 6.48 2 12s4.48 10 10 10s10-4.48 10-10S17.52 2 12 2zm0 13.5L7.5 11l1.42-1.41L12 12.67l3.08-3.08L16.5 11L12 15.5z"/></svg>
      </label>
  
  
    </div>
  {/if}