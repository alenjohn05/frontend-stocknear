<script lang='ts'>
  import { goto } from '$app/navigation';
  import { screenWidth } from '$lib/store';
  import { onMount } from 'svelte';

  export let data;
  let rawData = data?.getDividendAristocrats;
  let displayList = rawData?.slice(0,50);
  
  
  async function handleScroll() {
      const scrollThreshold = document.body.offsetHeight * 0.8; // 80% of the website height
      const isBottom = window.innerHeight + window.scrollY >= scrollThreshold;
      if (isBottom && displayList?.length !== rawData?.length) {
          const nextIndex = displayList?.length;
          const filteredNewResults = rawData?.slice(nextIndex, nextIndex + 50);
          displayList = [...displayList, ...filteredNewResults];
      }
    }
  
    onMount(async () => {
      window.addEventListener('scroll', handleScroll);
        return () => {
            window.removeEventListener('scroll', handleScroll);
        };
    })
  
  $: charNumber = $screenWidth < 640 ? 15 : 40;

  </script>
      
      <section class="w-full overflow-hidden m-auto">
              
            
          <div class="w-full sm:flex sm:flex-row sm:items-center m-auto text-gray-100 border border-gray-800 sm:rounded-lg h-auto p-5 mb-4">
              <svg class="w-5 h-5 inline-block sm:mr-2 flex-shrink-0" xmlns="http://www.w3.org/2000/svg" viewBox="0 0 256 256"><path fill="#a474f6" d="M128 24a104 104 0 1 0 104 104A104.11 104.11 0 0 0 128 24m-4 48a12 12 0 1 1-12 12a12 12 0 0 1 12-12m12 112a16 16 0 0 1-16-16v-40a8 8 0 0 1 0-16a16 16 0 0 1 16 16v40a8 8 0 0 1 0 16"/></svg>
              
              Dividend Aristocrats are companies in the S&P500 index that have been raising their dividends every year for 25 or more years. This is a list of all the dividend aristocrats.
            </div>
      
      
            <!-- Page wrapper -->
            <div class="flex justify-center w-full m-auto h-full overflow-hidden">
        
                
          
                <!-- Content area -->
                <div class="relative flex flex-col flex-1 overflow-x-scroll">
        
      
             
                  <table class="table table-sm sm:table-md table-compact rounded-none sm:rounded-md w-full border-bg-[#09090B] m-auto mt-4 ">
                    <thead>
                      <tr class="border border-slate-800">
                        <th class="text-white font-semibold text-sm sm:text-[1rem]">No.</th>
                        <th class="text-slate-200 hidden sm:table-cell sm:font-bold text-sm sm:text-[1rem] ">Symbol</th>
                        <th class="text-white font-semibold text-sm sm:text-[1rem] ">Company</th>
                        <th class="text-white font-semibold text-end text-sm sm:text-[1rem] ">Stock Price</th>
                        <th class="text-white font-semibold text-center text-sm sm:text-[1rem] ">% Change</th>
                        <th class="text-white font-semibold text-center text-sm sm:text-[1rem] ">Div. Yield</th>
                        <!--<th class="text-white font-semibold text-center text-sm sm:text-[1rem] ">Sector</th>-->
                        <th class="text-white font-semibold text-sm sm:text-[1rem]  text-end">Years</th>
                      </tr>
                    </thead>
                    <tbody>
                      {#each displayList as item,index}
                      <!-- row -->
                      <tr on:click={() => goto("/stocks/"+item?.symbol)}  class="sm:hover:bg-[#245073] sm:hover:bg-opacity-[0.2] odd:bg-[#27272A] border-b-[#09090B] shake-ticker cursor-pointer">
                        
                        <td class="text-white font-semibold sm:font-normal text-center text-sm sm:text-[1rem] border-b-[#09090B]">
                          {index+1}
                        </td>
                      
                        <td class="hidden sm:table-cell text-blue-400 font-normal text-[1rem] border-b-[#09090B]">
                          {item?.symbol}
                        </td>


                        <td class="text-gray-200 border-b-[#09090B] whitespace-nowrap">
                          <span class="hidden sm:inline-block text-white font-semibold sm:font-normal text-sm sm:text-[1rem]">{item?.name?.length > charNumber ? item?.name?.slice(0,charNumber) + "..." : item?.name}</span>
                          <div class="sm:hidden flex flex-row">
                            <div class="flex flex-col">
                              <span class="text-blue-400 font-medium">{item?.symbol}</span>
                              <span class="text-gray-200 font-medium">{item?.name?.length > charNumber ? item?.name?.slice(0,charNumber) + "..." : item?.name}</span>
                            </div>
                          </div>
                        </td>
    
                        <td class="text-white font-semibold sm:font-normal text-end text-sm sm:text-[1rem] border-b-[#09090B]">
                            {item?.price}
                        </td>
      
                        <td class="text-white font-semibold sm:font-normal text-center text-sm sm:text-[1rem] border-b-[#09090B]">
                          {#if item?.changesPercentage >=0}
                          <span class="text-[#10DB06]">+{item.changesPercentage?.toFixed(2)}%</span>
                          {:else}
                            <span class="text-[#FF2F1F]">{item.changesPercentage?.toFixed(2)}% </span> 
                          {/if}
                        </td>

                        <td class="text-white font-semibold sm:font-normal text-center text-sm sm:text-[1rem] border-b-[#09090B]">
                          {item?.dividiendYield}%
                        </td>
                        
                        <!--
                        <td class="text-white font-semibold sm:font-normal text-center text-sm sm:text-[1rem] border-b-[#09090B]">
                          {item?.sector}
                        </td>
                        -->
      
                        <td class="text-white  font-semibold sm:font-normal text-end text-sm sm:text-[1rem] border-b-[#09090B]">
                          {item?.years}
                        </td>
      
                   
        
        
                      </tr>
                      
                  
                      {/each}
                    </tbody>
                </table>
      
      
                      
            </div>
  
            
        
        </section>
        
        
