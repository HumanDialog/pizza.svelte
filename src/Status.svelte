<script>
    import { reef, session } from '@humandialog/auth.svelte';
    import {location, push, querystring} from 'svelte-spa-router'
    import {getStateMessage, STATE_PIZZA_PAID} from './main'
    import { onMount } from 'svelte';

    $: processChanged($location, querystring)

    let processRef
    let customerTask;
    let refreshIntevalId;
    async function processChanged(loc, qs)
    {
        const params = new URLSearchParams($querystring);
        processRef = params.get('process')
        await loadData();
    }

    async function loadData()
    {
        const res = await reef.get(`${processRef}/CustomerTask`)
        if(res)
            customerTask = res.Task;

        if(!refreshIntevalId)
            refreshIntevalId = setInterval(async () => loadData(), 10000)
    }

    onMount( () => {
        return () => {
            if(refreshIntevalId)
                clearInterval(refreshIntevalId)
        }})

    async function evaluatePizza(rating)
    {
        await reef.post(`${processRef}/EatPizza`, {evaluation: rating})
        await loadData();
    }

    function redirectToOrder()
    {
        push('/menu');
    }
</script>

<svelte:head>
    <title>Order status</title>
</svelte:head>

<div class="flex flex-col justify-center items-center">
    <h1 class="text-4xl font-extrabold dark:text-white mb-4 text-center">Pizza order staus</h1>

    {#if customerTask}
        <table class="mt-5 table-fixed">
            <tbody>
                <tr class="border-t">
                    <td class="p-2 text-xs uppercase font-semibold">Order name</td>
                    <td class="p-2">{customerTask.Title}</td>
                </tr>
                <tr class="border-t border-b">
                    <td class="p-2 text-xs uppercase font-semibold">Current status</td>
                    <td class="p-2">{getStateMessage(customerTask.State)}</td>
                </tr>
            </tbody>
        </table>
        {#if customerTask.State < STATE_PIZZA_PAID}
            <!--button  class="bg-stone-400 hover:bg-stone-500 
                            dark:bg-stone-600 dark:highlight-white/20 dark:hover:bg-stone-500
                            mt-3 px-6
                            text-white font-semibold 
                            h-6 rounded-lg  
                            flex items-center justify-center "
                    on:click={loadData}>
                    Refresh
            </button-->
        {:else if customerTask.State == STATE_PIZZA_PAID}
            <p class="mt-5">Rate your ordering process:</p>
            <div class="flex flex-row gap-2">
                {#each [1, 2, 3, 4, 5, 6] as rate}
                    <button  class="bg-stone-400 hover:bg-stone-500 
                                dark:bg-stone-600 dark:highlight-white/20 dark:hover:bg-stone-500
                                mt-3 px-6
                                text-white font-semibold 
                                h-6 rounded-lg  
                                flex items-center justify-center "
                        on:click={() => evaluatePizza(rate)}>
                        {rate}
                    </button>
                {/each}
            </div>
        {:else}
            <p class="mt-5 text-sm font-semibold">Thank you!</p>
            <button  class="bg-stone-400 hover:bg-stone-500 
                            dark:bg-stone-600 dark:highlight-white/20 dark:hover:bg-stone-500
                            mt-3 px-6
                            text-white font-semibold 
                            h-6 rounded-lg  
                            flex items-center justify-center "
                    on:click={redirectToOrder}>
                    Order again
            </button>
        {/if}
    {:else}
        <p>Loading order status...</p>
    {/if}
</div>