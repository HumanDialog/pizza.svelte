<script>
    import {reef} from '@humandialog/auth.svelte'
    import {Page, Kanban, KanbanSource, KanbanColumn, KanbanTitle, KanbanSummary, Button} from '@humandialog/forms.svelte'
    import { onMount } from 'svelte';
    import {STATE_WAITING_FOR_BAKING_PIZZA, STATE_BAKING_PIZZA, STATE_PIZZA_WAITING_FOR_DELIVERY, STATE_PIZZA_IN_DELIVERY, STATE_PIZZA_PAID, STATE_PIZZA_EVELUATED} from './main'

    let customersTasks;
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
        const res = await reef.post('app/ActiveCustomerTasks/query',
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
            customersTasks =  res.TaskList;
    }

    async function refresh(selection = undefined)
    {
        await loadData();
        kanban.reload(customersTasks, selection);
    }

    
</script>

<svelte:head>
    <title>Dashboard</title>
</svelte:head>

{#if customersTasks}

<Page
		self={customersTasks}
		clearsContext="props sel"
        title={customersTasks.Name}>
    <h1 class="text-4xl font-extrabold dark:text-white mb-4 text-center">{customersTasks.Name}</h1>

    <!--section class="flex flex-row gap-4 justify-center">
        <Button action={refresh}
                class=" bg-lime-700 hover:bg-lime-800 disabled:bg-stone-300
                        dark:bg-lime-800 dark:highlight-white/20 dark:hover:bg-lime-700 disabled:dark:bg-stone-800
                        mt-3 px-6
                        text-white font-semibold 
                        h-20 w-30 rounded-lg  
                        flex items-center justify-center
                        cursor-pointer disabled:cursor-default">
                Refresh
        </Button>

    </section-->

    <Kanban class="grow-0" 
                bind:this={kanban}>

            <KanbanSource self={customersTasks}
                          a='Tasks'
                          stateAttrib='State'
                          orderAttrib='ListOrder'/>

            <KanbanColumn   title="Waiting" state={STATE_WAITING_FOR_BAKING_PIZZA}/>
            <KanbanColumn   title="Baking" state={STATE_BAKING_PIZZA}/>
            <KanbanColumn   title="Waiting for delivery" state={STATE_PIZZA_WAITING_FOR_DELIVERY}/>
            <KanbanColumn   title="In delivery" state={STATE_PIZZA_IN_DELIVERY}/>
            <KanbanColumn   title="Eating" state={STATE_PIZZA_PAID}/>
            <KanbanColumn   title="Evaluated" state={STATE_PIZZA_EVELUATED} finishing/>
            
            <KanbanTitle a="Title"/>
			<KanbanSummary a="Summary" />

        </Kanban>

</Page>


{:else}
    <p>Loading clerk's data...</p>
{/if}
