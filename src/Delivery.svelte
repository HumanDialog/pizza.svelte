<script>
    import {reef} from '@humandialog/auth.svelte'
    import {Page, Kanban, KanbanSource, KanbanColumn, KanbanTitle, KanbanSummary, Button} from '@humandialog/forms.svelte'
    import { onMount } from 'svelte';
    import {STATE_PIZZA_WAITING_FOR_DELIVERY, STATE_PIZZA_IN_DELIVERY, STATE_PIZZA_PAID} from './main'

    let deliveryList;
    let kanban;
    let refreshIntevalId;

    onMount( async () => {
        await loadData();
        if(!refreshIntevalId)
            refreshIntevalId = setInterval(async () => refresh(), 10000)

        return () => {
            if(refreshIntevalId)
                clearInterval(refreshIntevalId)
        } 
    })


    async function loadData()
    {
        const res = await reef.post('app/ActiveDeliveryTasks/query',
                        {
                            Id: 1,
                            Name: "collector",
                            Tree:
                            [
                                {
                                    Id: 1,
                                    Association: '',
                                    SubTree:
                                    [
                                        {
                                            Id: 2,
                                            Association: 'Tasks',
                                        }
                                    ]
                                }
                            ]
                        });
        if(res)
            deliveryList =  res.TaskList;
    }

    async function refresh(selection = undefined)
    {
        await loadData();
        kanban.reload(deliveryList, selection);
    }

    
    async function deliverPizza(task)
    {
        await reef.post(`${task.$ref}/Process/DeliverPizza`, {})
        await refresh(task.Id);
    }

    async function receivePayment(task)
    {
        await reef.post(`${task.$ref}/Process/ReceivePayment`, {})
        await refresh(task.Id);
    }

    
</script>

<svelte:head>
    <title>Delivery panel</title>
</svelte:head>

{#if deliveryList}

<Page
		self={deliveryList}
		clearsContext="props sel"
        title={deliveryList.Name}>
    <h1 class="text-4xl font-extrabold dark:text-white mb-4 text-center">{deliveryList.Name}</h1>

    <section class="flex flex-row gap-4 justify-center">
        <Button context="props"
                action={deliverPizza}
                class=" bg-amber-700 hover:bg-amber-800 disabled:bg-stone-300
                        dark:bg-amber-800 dark:highlight-white/20 dark:hover:bg-amber-700 disabled:dark:bg-stone-800
                        mt-3 px-6
                        text-white font-semibold 
                        h-20 w-30 rounded-lg  
                        flex items-center justify-center
                        cursor-pointer disabled:cursor-default"
                    disabledFunc={(task) => task.State != STATE_PIZZA_WAITING_FOR_DELIVERY}>
                Take to delivery
        </Button>

        <Button context="props"
                action={receivePayment}
                class=" bg-sky-700 hover:bg-sky-800 disabled:bg-stone-300
                        dark:bg-sky-800 dark:highlight-white/20 dark:hover:bg-sky-700 disabled:dark:bg-stone-800
                        mt-3 px-6
                        text-white font-semibold 
                        h-20 w-30 rounded-lg  
                        flex items-center justify-center
                        cursor-pointer disabled:cursor-default"
                    disabledFunc={(task) => task.State != STATE_PIZZA_IN_DELIVERY}>
                Recive payment
        </Button>

        <!--Button action={refresh}
                class=" bg-lime-700 hover:bg-lime-800 disabled:bg-stone-300
                        dark:bg-lime-800 dark:highlight-white/20 dark:hover:bg-lime-700 disabled:dark:bg-stone-800
                        mt-3 px-6
                        text-white font-semibold 
                        h-20 w-30 rounded-lg  
                        flex items-center justify-center
                        cursor-pointer disabled:cursor-default">
                Refresh
        </Button-->

    </section>

    <Kanban class="grow-0" 
                bind:this={kanban}>

            <KanbanSource self={deliveryList}
                          a='Tasks'
                          stateAttrib='State'
                          orderAttrib='ListOrder'/>

            
            <KanbanColumn   title="To deliver" state={STATE_PIZZA_WAITING_FOR_DELIVERY}/>
            <KanbanColumn   title="In delivery" state={STATE_PIZZA_IN_DELIVERY}/>
            <KanbanColumn   title="Served" state={STATE_PIZZA_PAID} finishing/>
            
			<KanbanTitle a="Title"/>
			<KanbanSummary a="Summary" />

        </Kanban>

</Page>


{:else}
    <p>Loading delivery data...</p>
{/if}
