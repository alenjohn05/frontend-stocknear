<script lang='ts'>
import { goto } from '$app/navigation';
import { screenWidth, numberOfUnreadNotification } from '$lib/store';
import logo from '$lib/images/top_winner_logo.png';
import { abbreviateNumber } from '$lib/utils';
import MiniPlot from '$lib/components/MiniPlot.svelte';

export let data;

const rawData = data?.getMiniPlotsIndex;

let priceDataSP500;
let priceDataNasdaq;
let priceDataDowJones;
let priceDataRussel2000;

let changeSP500, changeNasdaq, changeDowJones, changeRussel2000;
let previousCloseSP500, previousCloseNasdaq, previousCloseDowJones, previousCloseRussel2000;


function getCurrentDateFormatted() {
    // Get current date
    let date = new Date();
    
    // If today is Saturday or Sunday, move to the previous Friday
    if (date.getDay() === 6) { // Saturday
        date.setDate(date.getDate() - 1);
    } else if (date.getDay() === 0) { // Sunday
        date.setDate(date.getDate() - 2);
    }
    
    // Define months array for formatting
    const months = ["Jan", "Feb", "Mar", "Apr", "May", "Jun", "Jul", "Aug", "Sep", "Oct", "Nov", "Dec"];
    
    // Get formatted date components
    const month = months[date.getMonth()];
    const day = date.getDate();
    const year = date.getFullYear();
    
    // Return formatted date
    return `${month} ${day}, ${year}`;
}


// Assign values based on the symbol
rawData?.forEach(({ symbol, priceData, changesPercentage, previousClose }) => {
  switch (symbol) {
    case "SPY":
      priceDataSP500 = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
      priceDataSP500 = priceDataSP500?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
      changeSP500 = changesPercentage;
      previousCloseSP500 = previousClose;
      break;
    case "QQQ":
      priceDataNasdaq = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
      priceDataNasdaq = priceDataNasdaq?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
      changeNasdaq = changesPercentage;
      previousCloseNasdaq = previousClose;
      break;
    case "DIA":
      priceDataDowJones = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
      priceDataDowJones = priceDataDowJones?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
      changeDowJones = changesPercentage
      previousCloseDowJones = previousClose;
      break;
    case "IWM":
      priceDataRussel2000 = priceData?.map(({ time, value }) => ({ time: Date?.parse(time), value}));
      priceDataRussel2000 = priceDataRussel2000?.filter(item => item.value !== 0 && item.value !== null && item.value !== undefined)
      changeRussel2000 = changesPercentage;
      previousCloseRussel2000 = previousClose;
      break;
    default:
      // Handle unknown symbol
      break;
  }
});


let timePeriod = '1D';
let outputList = data?.getDailyGainerLoserActive;
let gainerLoserActive = outputList?.gainers[timePeriod]
let displaySection = 'gainer'
let order = 'highToLow';


let buttonText = 'Top Winners';


function changeOrder(state:string) {
    if (state === 'highToLow')
    {
      order = 'lowToHigh';
    }
    else {
      order = 'highToLow';
    }
  }

const sortByHighestChange = (tickerList) => {
    return tickerList?.sort(function(a, b) {
      if(order === 'highToLow')
      {
        return b?.changesPercentage - a?.changesPercentage;
      }
      else {
        return a?.changesPercentage - b?.changesPercentage;
      }
       
      });
  }
/*
const sortByLowestChange = (tickerList) => {
    return tickerList?.sort(function(a, b) {
      if(order === 'highToLow')
      {
        return a?.changesPercentage - b?.changesPercentage;
      }
      else {
        return b?.changesPercentage - a?.changesPercentage;
      }
       
      });
  }

const sortByVolume = (tickerList) => {
    return tickerList?.sort(function(a, b) {
      if(order === 'highToLow')
      {
        return b?.volume - a?.volume;
      }
      else {
        return a?.volume - b?.volume;
      }
       
      });
  }
*/
const sortByMarketCap = (tickerList) => {
  return tickerList?.sort(function(a, b) {
    if(order === 'highToLow')
    {
      return b?.marketCap - a?.marketCap;
    }
    else {
      return a?.marketCap - b?.marketCap;
    }
  });
}

  
function changeSection(state) {

    displaySection = state;
    timePeriod = '1D';
    
    if (state === 'gainer')
    {
      gainerLoserActive = outputList?.gainers[timePeriod];
      buttonText = 'Top Winners'
    }
    else if (state === 'loser')
    {
      gainerLoserActive = outputList?.losers[timePeriod];
      buttonText = 'Top Losers'
    }
    else if (state === 'active')
    {
      gainerLoserActive = outputList?.active[timePeriod];
      buttonText = 'Most Active'
    }

}



function selectTimeInterval(interval) {
    timePeriod = interval;

    if (buttonText === 'Top Winners')
    {
      gainerLoserActive = outputList?.gainers[timePeriod];
    }
    else if (buttonText === 'Top Losers')
    {
      gainerLoserActive = outputList?.losers[timePeriod];
    }
    else if (buttonText === 'Most Active')
    {
      gainerLoserActive = outputList?.active[timePeriod];
    }
    
  }


  $: {
  if(order)
  {
    if(displaySection === 'gainer')
    {
      gainerLoserActive = sortByHighestChange(outputList?.gainers[timePeriod]);
    }
    else if(displaySection === 'loser')
    {
      gainerLoserActive = sortByHighestChange(outputList?.losers[timePeriod]);
    }
    else if(displaySection === 'active')
    {
      gainerLoserActive = sortByHighestChange(outputList?.active[timePeriod]);
    }
    
    // Add this condition for market cap sorting
    if (sortBy === 'marketCap') {
      gainerLoserActive = sortByMarketCap(gainerLoserActive);
    }
  }
}

let sortBy = 'change'; // Default sorting by change percentage
let charNumber = 30;

$: {
  if($screenWidth < 640)
  {
    charNumber = 20;
  }
  else {
    charNumber =30;
  }
}

</script>



<svelte:head>

<meta charset="utf-8" />
<meta name="viewport" content="width=device-width" />
<title>
  {$numberOfUnreadNotification > 0 ? `(${$numberOfUnreadNotification})` : ''} Today's Top Stock Gainers, Losers and Most Active · stocknear
</title>
<meta name="description" content={`A list of the stocks with the highest percentage gain, highest percentage loss and most active today. See stock price, volume, market cap and more.`} />

<!-- Other meta tags -->
<meta property="og:title" content={`Today's Top Stock Gainers, Losers and Most Active · stocknear`}/>
<meta property="og:description" content={`A list of the stocks with the highest percentage gain, highest percentage loss and most active today. See stock price, volume, market cap and more.`} />
<meta property="og:type" content="website"/>
<!-- Add more Open Graph meta tags as needed -->

<!-- Twitter specific meta tags -->
<meta name="twitter:card" content="summary_large_image"/>
<meta name="twitter:title" content={`Today's Top Stock Gainers, Losers and Most Active · stocknear`}/>
<meta name="twitter:description" content={`A list of the stocks with the highest percentage gain, highest percentage loss and most active today. See stock price, volume, market cap and more.`} />
<!-- Add more Twitter meta tags as needed -->

</svelte:head>
  
<section class="w-full max-w-3xl sm:max-w-screen-xl overflow-hidden min-h-screen pt-5 pb-40">
    

  <div class="text-sm sm:text-[1rem] breadcrumbs ml-4">
    <ul>
      <li><a href="/" class="text-gray-300">Home</a></li> 
      <li class="text-gray-300">
        Market Movers
      </li>
    </ul>
  </div>



  <div class="w-full m-auto sm:bg-[#27272A] sm:rounded-xl h-auto sm:p-10 mt-8 mb-4 sm:mb-8">
    <div class="grid grid-cols-1 sm:grid-cols-2 gap-10">
  
      <!-- Start Column -->
      <div>
        <div class="flex flex-row justify-center items-center">
          <h1 class="text-3xl sm:text-4xl text-white text-center font-bold mb-5">
            Market Movers
          </h1>
        </div>

        <span class="hidden sm:block text-white text-md font-medium text-center flex justify-center items-center ">
          Explore top-performing, underperforming & most active traded stocks across different time frames.
        </span>


      </div>
      <!-- End Column -->
  
      <!-- Start Column -->
      <div class="hidden sm:block relative m-auto mb-5 mt-5 md:mb-0 md:mt-0">
        <svg class="w-32 -my-4" viewBox="0 0 200 200" xmlns="http://www.w3.org/2000/svg">
          <defs>
            <filter id="glow">
              <feGaussianBlur stdDeviation="5" result="glow"/>
              <feMerge>
                <feMergeNode in="glow"/>
                <feMergeNode in="SourceGraphic"/>
              </feMerge>
            </filter>
          </defs>
          <path fill="#1E40AF" d="M57.6,-58.7C72.7,-42.6,81.5,-21.3,82,0.5C82.5,22.3,74.7,44.6,59.7,60.1C44.6,75.6,22.3,84.3,0,84.3C-22.3,84.2,-44.6,75.5,-61.1,60.1C-77.6,44.6,-88.3,22.3,-87.6,0.7C-86.9,-20.8,-74.7,-41.6,-58.2,-57.7C-41.6,-73.8,-20.8,-85.2,0.2,-85.4C21.3,-85.6,42.6,-74.7,57.6,-58.7Z" transform="translate(100 100)" filter="url(#glow)" />
        </svg>
        
        
        <div class="z-1 absolute -top-3 right-1">
          <img class="w-28" src={logo} alt="logo" loading="lazy">
        </div>
      </div>
      <!-- End Column -->
    </div>

  </div>

  <div class="text-white text-xs sm:text-sm pb-5 sm:pb-2 pl-3 sm:pl-0">
    Stock Indexes - {getCurrentDateFormatted()}
  </div>

  <div class="w-full -mt-4 sm:mt-0 mb-8 m-auto flex justify-center items-center p-3 sm:p-0">
    <div class="w-full grid grid-cols-2 md:grid-cols-3 lg:grid-cols-5 gap-y-3 lg:gap-y-0 gap-x-3 ">
    <MiniPlot title="S&P500" priceData = {priceDataSP500} changesPercentage={changeSP500} previousClose={previousCloseSP500}/>
    <MiniPlot title="Nasdaq" priceData = {priceDataNasdaq} changesPercentage={changeNasdaq} previousClose={previousCloseNasdaq}/>
    <MiniPlot title="Dow" priceData = {priceDataDowJones} changesPercentage={changeDowJones} previousClose={previousCloseDowJones}/>
    <MiniPlot title="Russel" priceData = {priceDataRussel2000} changesPercentage={changeRussel2000} previousClose={previousCloseRussel2000}/>
    </div>
  </div>


    <div class="w-full m-auto mb-10 bg-[#09090B] pl-3 pr-3 sm:pl-0 sm:pr-0">
     
      

        <div class="tabs flex flex-row justify-between sm:justify-start items-center w-full pl-3 pr-3 sm:pl-0 sm:pr-0">
          <button class="w-fit text-xl mr-0 sm:mr-10 rounded-md transition font-semibold hover:text-white {displaySection === 'gainer' ? ' text-white' : 'text-[#9A9996]'}" on:click={() => (changeSection('gainer'))} >
            Gainers
            <div class="{displaySection === 'gainer' ? 'bg-[#75D377]' : 'bg-[#09090B]'} mt-1 h-[3px] rounded-full w-[5rem] rounded-full" />
          </button> 
          
          <button class="w-fit text-xl mr-0 sm:mr-10 rounded-md transition font-semibold hover:text-white {displaySection === 'loser' ? ' text-white' : 'text-[#9A9996]'}" on:click={() => (changeSection('loser'))} >
            Losers
            <div class="{displaySection === 'loser' ? 'bg-[#75D377]' : 'bg-[#09090B]'} mt-1 h-[3px] rounded-full w-[5rem] rounded-full" />
          </button> 
          
          <button class="w-fit text-xl mr-0 sm:mr-10 rounded-md transition font-semibold hover:text-white {displaySection === 'active' ? ' text-white' : 'text-[#9A9996]'}" on:click={() => (changeSection('active'))} >
            Active
            <div class="{displaySection === 'active' ? 'bg-[#75D377]' : 'bg-[#09090B]'} mt-1 h-[3px] rounded-full w-[5rem] rounded-full" />
          </button> 
        </div>

      </div>

    
    <!--Start Top Winners/Losers-->
    <div class="flex flex-col justify-center items-center overflow-hidden">
      

        
      <div class="pl-2 sm:pl-0 flex justify-start items-center mb-10 w-full">
        <label for="timePeriodModal" class="px-4 py-3 rounded-lg cursor-pointer bg-[#27272A] flex flex-row items-center">
          <span class="text-white text-sm sm:text-md mr-2">
            Time Period: 
          </span>
            <div class="flex flex-row items-center">
                <span class="text-sm sm:text-md m-auto font-medium text-white">
                  {timePeriod}
                </span>
                <svg class="inline-block w-4 h-4 ml-1 mt-1" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 1024 1024">
                    <g transform="rotate(180 512 512)">
                        <path fill="#fff" d="m488.832 344.32l-339.84 356.672a32 32 0 0 0 0 44.16l.384.384a29.44 29.44 0 0 0 42.688 0l320-335.872l319.872 335.872a29.44 29.44 0 0 0 42.688 0l.384-.384a32 32 0 0 0 0-44.16L535.168 344.32a32 32 0 0 0-46.336 0z"/>
                    </g>
                </svg>
            </div>
          </label>

      </div>
    


    <div class="w-full overflow-x-scroll no-scrollbar">
      <table class="table table-sm table-compact rounded-none sm:rounded-md w-full bg-[#09090B] border-bg-[#09090B]">
        <thead>
          <tr>
            <th class="text-white font-semibold text-sm whitespace-nowrap">Company</th>
            <th on:click={() => { sortBy = 'change'; changeOrder(order); }} class="whitespace-nowrap cursor-pointer text-white font-semibold text-sm text-end">
              % Change
              <svg class="w-5 h-5 inline-block {order === 'highToLow' && sortBy === 'change' ? '' : 'rotate-180'}" viewBox="0 0 20 20" fill="currentColor" style="max-width:40px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
            <th class="text-white font-semibold text-end text-sm">Price</th>
            <th on:click={() => { sortBy = 'marketCap'; changeOrder(order); }} class="whitespace-nowrap cursor-pointer text-white font-semibold text-sm text-end">
              Market Cap
              <svg class="w-5 h-5 inline-block {order === 'highToLow' && sortBy === 'marketCap' ? 'rotate-180' : ''}" viewBox="0 0 20 20" fill="currentColor" style="max-width:40px"><path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd"></path></svg>
            </th>
            <th class="text-white font-semibold text-sm text-end ">Volume</th>
          </tr>
        </thead>
        <tbody>
          {#each gainerLoserActive as item, index}
          <tr on:click={() => goto("/stocks/"+item?.symbol)} class="sm:hover:bg-[#245073] sm:hover:bg-opacity-[0.2] odd:bg-[#27272A] cursor-pointer">
    
          <td class="border-b-[#09090B] text-sm sm:text-[1rem] whitespace-nowrap">
            <div class="flex flex-col">
              <span class="text-blue-400 font-medium">
                {item?.symbol}
              </span>
              <span class="text-white font-medium border-b-[#09090B]">
                {item?.name?.length > charNumber ? item?.name?.slice(0,charNumber) + "..." : item?.name}
              </span>
            </div>
          </td>

          <td class="text-white text-end text-sm sm:text-[1rem] font-medium border-b-[#09090B]">
              {#if item?.changesPercentage >=0}
                <span class="text-[#10DB06]">+{item?.changesPercentage >= 1000 ? abbreviateNumber(item?.changesPercentage) : item?.changesPercentage?.toFixed(2)}%</span>
              {:else}
                <span class="text-[#FF2F1F]">{item?.changesPercentage <= -1000 ? abbreviateNumber(item?.changesPercentage) : item?.changesPercentage?.toFixed(2)}% </span> 
              {/if}
          </td>

          <td class="text-white font-medium text-sm sm:text-[1rem] text-end border-b-[#09090B]">
            ${item?.price?.toFixed(2)}
          </td>

          <td class="text-white text-sm sm:text-[1rem] font-medium border-b-[#09090B] text-end">
            {item?.marketCap !== null ? abbreviateNumber(item?.marketCap, true) : '-'}
          </td>

          <td class="text-white text-sm sm:text-[1rem] font-medium border-b-[#09090B] text-end">
            {item?.volume !== null ? abbreviateNumber(item?.volume) : '-'}
          </td>





          </tr>
          {/each}
        </tbody>
      </table>

    </div>



<!--
    {#if !fullList}
    <label on:click={showFullList} class="mt-10 btn btn-sm btn-outline btn-ghost text-white text-xs m-auto normal-case ">
        Show more
    </label>
    {/if}
-->

  </div>



</section>





  <!--Start Time Period Modal-->
  <input type="checkbox" id="timePeriodModal" class="modal-toggle" />
      
  <dialog id="timePeriodModal" class="modal modal-bottom sm:modal-middle ">
  
  
    <label id="timePeriodModal" for="timePeriodModal"  class="cursor-pointer modal-backdrop bg-[#09090B] bg-opacity-[0.5]"></label>
    
    
    <div class="modal-box w-full bg-[#09090B] sm:border sm:border-slate-800">
  
  
  
    <label for="timePeriodModal" class="cursor-pointer absolute right-5 top-2 bg-[#09090B] text-[1.8rem] text-white">
      ✕
    </label>
  
      <div class="text-white">
        <h3 class="font-medium text-lg sm:text-2xl mb-10">
          Time Period
        </h3>
  
  
        <div class="flex flex-col items-center w-full max-w-3xl bg-[#09090B]">
  

          <label for="timePeriodModal" on:click={() => selectTimeInterval('1D')} class="cursor-pointer w-full flex flex-row justify-start items-center mb-5">
  
            <div class="flex flex-row items-center w-full bg-[#303030] p-3 rounded-lg {timePeriod === '1D' ? 'ring-2 ring-[#04E000]' : ''}">
              
              <span class="ml-1 text-white font-medium mr-auto">
                1 Day
              </span>

              <div class="rounded-full w-8 h-8 relative border border-[#737373]">
                {#if timePeriod === '1D'}
                  <svg class="w-full h-full rounded-full" viewBox="0 0 48 48" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" fill="#09090B000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools --> <title>ic_fluent_checkmark_circle_48_filled</title> <desc>Created with Sketch.</desc> <g id="🔍-Product-Icons" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"> <g id="ic_fluent_checkmark_circle_48_filled" fill="#04E000" fill-rule="nonzero"> <path d="M24,4 C35.045695,4 44,12.954305 44,24 C44,35.045695 35.045695,44 24,44 C12.954305,44 4,35.045695 4,24 C4,12.954305 12.954305,4 24,4 Z M32.6338835,17.6161165 C32.1782718,17.1605048 31.4584514,17.1301307 30.9676119,17.5249942 L30.8661165,17.6161165 L20.75,27.732233 L17.1338835,24.1161165 C16.6457281,23.6279612 15.8542719,23.6279612 15.3661165,24.1161165 C14.9105048,24.5717282 14.8801307,25.2915486 15.2749942,25.7823881 L15.3661165,25.8838835 L19.8661165,30.3838835 C20.3217282,30.8394952 21.0415486,30.8698693 21.5323881,30.4750058 L21.6338835,30.3838835 L32.6338835,19.3838835 C33.1220388,18.8957281 33.1220388,18.1042719 32.6338835,17.6161165 Z" id="🎨-Color"> </path> </g> </g> </g></svg>
                {/if}
              </div>

            </div>
           
        </label>
  
          <label for="timePeriodModal" on:click={() => selectTimeInterval('1W')} class="cursor-pointer w-full flex flex-row justify-start items-center mb-5">
  
              <div class="flex flex-row items-center w-full bg-[#303030] p-3 rounded-lg {timePeriod === '1W' ? 'ring-2 ring-[#04E000]' : ''}">
                
                <span class="ml-1 text-white font-medium mr-auto">
                  1 Week
                </span>
  
                <div class="rounded-full w-8 h-8 relative border border-[#737373]">
                  {#if timePeriod === '1W'}
                    <svg class="w-full h-full rounded-full" viewBox="0 0 48 48" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" fill="#09090B000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools --> <title>ic_fluent_checkmark_circle_48_filled</title> <desc>Created with Sketch.</desc> <g id="🔍-Product-Icons" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"> <g id="ic_fluent_checkmark_circle_48_filled" fill="#04E000" fill-rule="nonzero"> <path d="M24,4 C35.045695,4 44,12.954305 44,24 C44,35.045695 35.045695,44 24,44 C12.954305,44 4,35.045695 4,24 C4,12.954305 12.954305,4 24,4 Z M32.6338835,17.6161165 C32.1782718,17.1605048 31.4584514,17.1301307 30.9676119,17.5249942 L30.8661165,17.6161165 L20.75,27.732233 L17.1338835,24.1161165 C16.6457281,23.6279612 15.8542719,23.6279612 15.3661165,24.1161165 C14.9105048,24.5717282 14.8801307,25.2915486 15.2749942,25.7823881 L15.3661165,25.8838835 L19.8661165,30.3838835 C20.3217282,30.8394952 21.0415486,30.8698693 21.5323881,30.4750058 L21.6338835,30.3838835 L32.6338835,19.3838835 C33.1220388,18.8957281 33.1220388,18.1042719 32.6338835,17.6161165 Z" id="🎨-Color"> </path> </g> </g> </g></svg>
                  {/if}
                </div>
  
              </div>
             
          </label>
  
  
          <label for="timePeriodModal" on:click={() => selectTimeInterval('1M')} class="cursor-pointer w-full flex flex-row justify-start items-center mb-5">
  
            <div class="flex flex-row items-center w-full bg-[#303030] p-3 rounded-lg {timePeriod === '1M' ? 'ring-2 ring-[#04E000]' : ''}">
              
              <span class="ml-1 text-white font-medium mr-auto">
                1 Month
              </span>
  
              <div class="rounded-full w-8 h-8 relative border border-[#737373]">
                {#if timePeriod === '1M'}
                <svg class="w-full h-full rounded-full" viewBox="0 0 48 48" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" fill="#09090B000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools --> <title>ic_fluent_checkmark_circle_48_filled</title> <desc>Created with Sketch.</desc> <g id="🔍-Product-Icons" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"> <g id="ic_fluent_checkmark_circle_48_filled" fill="#04E000" fill-rule="nonzero"> <path d="M24,4 C35.045695,4 44,12.954305 44,24 C44,35.045695 35.045695,44 24,44 C12.954305,44 4,35.045695 4,24 C4,12.954305 12.954305,4 24,4 Z M32.6338835,17.6161165 C32.1782718,17.1605048 31.4584514,17.1301307 30.9676119,17.5249942 L30.8661165,17.6161165 L20.75,27.732233 L17.1338835,24.1161165 C16.6457281,23.6279612 15.8542719,23.6279612 15.3661165,24.1161165 C14.9105048,24.5717282 14.8801307,25.2915486 15.2749942,25.7823881 L15.3661165,25.8838835 L19.8661165,30.3838835 C20.3217282,30.8394952 21.0415486,30.8698693 21.5323881,30.4750058 L21.6338835,30.3838835 L32.6338835,19.3838835 C33.1220388,18.8957281 33.1220388,18.1042719 32.6338835,17.6161165 Z" id="🎨-Color"> </path> </g> </g> </g></svg>
                {/if}
              </div>
  
            </div>
           
          </label>
  
      
  
  
          <label for="timePeriodModal" on:click={() => selectTimeInterval('3M')} class="cursor-pointer w-full flex flex-row justify-start items-center mb-5">
            <div class="flex flex-row items-center w-full bg-[#303030] p-3 rounded-lg {timePeriod === '3M' ? 'ring-2 ring-[#04E000]' : ''}">
              <span class="ml-1 text-white font-medium mr-auto">
                3 Months
              </span>
              <div class="rounded-full w-8 h-8 relative border border-[#737373]">
                {#if timePeriod === '3M'}
                  <svg class="w-full h-full rounded-full" viewBox="0 0 48 48" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" fill="#09090B000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools --> <title>ic_fluent_checkmark_circle_48_filled</title> <desc>Created with Sketch.</desc> <g id="🔍-Product-Icons" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"> <g id="ic_fluent_checkmark_circle_48_filled" fill="#04E000" fill-rule="nonzero"> <path d="M24,4 C35.045695,4 44,12.954305 44,24 C44,35.045695 35.045695,44 24,44 C12.954305,44 4,35.045695 4,24 C4,12.954305 12.954305,4 24,4 Z M32.6338835,17.6161165 C32.1782718,17.1605048 31.4584514,17.1301307 30.9676119,17.5249942 L30.8661165,17.6161165 L20.75,27.732233 L17.1338835,24.1161165 C16.6457281,23.6279612 15.8542719,23.6279612 15.3661165,24.1161165 C14.9105048,24.5717282 14.8801307,25.2915486 15.2749942,25.7823881 L15.3661165,25.8838835 L19.8661165,30.3838835 C20.3217282,30.8394952 21.0415486,30.8698693 21.5323881,30.4750058 L21.6338835,30.3838835 L32.6338835,19.3838835 C33.1220388,18.8957281 33.1220388,18.1042719 32.6338835,17.6161165 Z" id="🎨-Color"> </path> </g> </g> </g></svg>
                {/if}
              </div>
            </div>
          </label>
  
          <label for="timePeriodModal" on:click={() => selectTimeInterval('6M')} class="cursor-pointer w-full flex flex-row justify-start items-center mb-5">
            <div class="flex flex-row items-center w-full bg-[#303030] p-3 rounded-lg {timePeriod === '6M' ? 'ring-2 ring-[#04E000]' : ''}">
              <span class="ml-1 text-white font-medium mr-auto">
                6 Months
              </span>
              <div class="rounded-full w-8 h-8 relative border border-[#737373]">
                {#if timePeriod === '6M'}
                <svg class="w-full h-full rounded-full" viewBox="0 0 48 48" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" fill="#09090B000"><g id="SVGRepo_bgCarrier" stroke-width="0"></g><g id="SVGRepo_tracerCarrier" stroke-linecap="round" stroke-linejoin="round"></g><g id="SVGRepo_iconCarrier"> <!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools --> <title>ic_fluent_checkmark_circle_48_filled</title> <desc>Created with Sketch.</desc> <g id="🔍-Product-Icons" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd"> <g id="ic_fluent_checkmark_circle_48_filled" fill="#04E000" fill-rule="nonzero"> <path d="M24,4 C35.045695,4 44,12.954305 44,24 C44,35.045695 35.045695,44 24,44 C12.954305,44 4,35.045695 4,24 C4,12.954305 12.954305,4 24,4 Z M32.6338835,17.6161165 C32.1782718,17.1605048 31.4584514,17.1301307 30.9676119,17.5249942 L30.8661165,17.6161165 L20.75,27.732233 L17.1338835,24.1161165 C16.6457281,23.6279612 15.8542719,23.6279612 15.3661165,24.1161165 C14.9105048,24.5717282 14.8801307,25.2915486 15.2749942,25.7823881 L15.3661165,25.8838835 L19.8661165,30.3838835 C20.3217282,30.8394952 21.0415486,30.8698693 21.5323881,30.4750058 L21.6338835,30.3838835 L32.6338835,19.3838835 C33.1220388,18.8957281 33.1220388,18.1042719 32.6338835,17.6161165 Z" id="🎨-Color"> </path> </g> </g> </g></svg>
              {/if}
              </div>
  
            </div>
           
          </label>
  
  
        </div>
         
      </div>
  
  
          
        </div>
    </dialog>
  <!--End Time Period Modal-->

