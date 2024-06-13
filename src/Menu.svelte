<script>
    import {reef} from '@humandialog/auth.svelte'
    import { onMount } from 'svelte';
    import { push } from 'svelte-spa-router';


    let menuItems = [];
    let orderingProcess = null;

    onMount( async () => {
        const res = await reef.get('app/PizzaMenuItems')
        if(res)
            menuItems = res.PizzaMenuItem

        if(!orderingProcess)
        {
            const res = await reef.post('app/SelectPizza', {})
            if(res)
                orderingProcess = res.PizzaOrder
        }

        return () => {}
    })

    async function selectPizza(pizza)
    {
        if(!orderingProcess)
            return;

        const result = await reef.post(`${orderingProcess.$ref}/OrderPizza`, 
                                        {
                                            pizzaItem: pizza.$ref
                                        });
        if(result)
        {
            push('/status?process=' + encodeURIComponent(orderingProcess.$ref));
        }
    }

</script>

<svelte:head>
    <title>Menu</title>
</svelte:head>

<div class="flex flex-col justify-center items-center">
    <h1 class="text-4xl font-extrabold dark:text-white mb-4 text-center">Menu</h1>

    {#if menuItems.length > 0}
        <div class=" grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3 max-w-6xl">
            {#each menuItems as pizza}
                <section class="border rounded-lg cursor-pointer group" on:click={() => selectPizza(pizza)}>
                    <img class="rounded-t-lg opacity-80 group-hover:opacity-100" src="{pizza.Picture}"/>
                    <h2 class="text-center uppercase text-xl mt-2 mb-3 font-semibold dark:text-white">{pizza.Name}</h2>
                    <p class="text-right mr-2 text-lg">Price: ${pizza.Price}</p>
                </section>
            {/each}
        </div>
    {:else}
        <p>Loading menu...</p>
    {/if}
</div>


