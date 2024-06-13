<script>
    import {reef} from '@humandialog/auth.svelte'
    import {Page, Kanban, KanbanSource, KanbanColumn, KanbanCallbacks, KanbanTitle, KanbanSummary, Button} from '@humandialog/forms.svelte'
    import { onMount } from 'svelte';
    import {STATE_WAITING_FOR_BAKING_PIZZA, STATE_BAKING_PIZZA, STATE_PIZZA_BAKED} from './main'

    let chefList;
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
       const res = await reef.post('app/ActivePizzaChefTasks/query',
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
            chefList =  res.TaskList;
    }

    async function refresh(selection = undefined)
    {
        await loadData();
        kanban.reload(chefList, selection);
    }

    async function startBaking(task)
    {
        await reef.post(`${task.$ref}/Process/StartBakingPizza`, {})
        await refresh(task.Id);
    }

    async function endBaking(task)
    {
        await reef.post(`${task.$ref}/Process/EndBakingPizza`, {})
        await refresh(task.Id);
    }

  

</script>

<svelte:head>
    <title>Chef's panel</title>
</svelte:head>

{#if chefList}

<Page
		self={chefList}
		clearsContext="props sel"
        title={chefList.Name}>
    <h1 class="text-4xl font-extrabold dark:text-white mb-4 text-center">{chefList.Name}</h1>

    <section class="flex flex-row gap-4 justify-center">
        <Button context="props"
                action={startBaking}
                class=" bg-amber-700 hover:bg-amber-800 disabled:bg-stone-300
                        dark:bg-amber-800 dark:highlight-white/20 dark:hover:bg-amber-700 disabled:dark:bg-stone-800
                        mt-3 px-6
                        text-white font-semibold 
                        h-20 w-30 rounded-lg  
                        flex items-center justify-center
                        cursor-pointer disabled:cursor-default"
                    disabledFunc={(task) => task.State != STATE_WAITING_FOR_BAKING_PIZZA}>
                Start baking
        </Button>

        <Button context="props"
                action={endBaking}
                class=" bg-sky-700 hover:bg-sky-800 disabled:bg-stone-300
                        dark:bg-sky-800 dark:highlight-white/20 dark:hover:bg-sky-700 disabled:dark:bg-stone-800
                        mt-3 px-6
                        text-white font-semibold 
                        h-20 w-30 rounded-lg  
                        flex items-center justify-center
                        cursor-pointer disabled:cursor-default"
                    disabledFunc={(task) => task.State != STATE_BAKING_PIZZA}>
                End baking
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

            <KanbanSource self={chefList}
                          a='Tasks'
                          stateAttrib='State'
                          orderAttrib='ListOrder'/>

            
            <KanbanColumn   title="Waiting" state={STATE_WAITING_FOR_BAKING_PIZZA}/>
            <KanbanColumn   title="Baking" state={STATE_BAKING_PIZZA}/>
            <KanbanColumn   title="Baked" state={STATE_PIZZA_BAKED} finishing/>
            
			<KanbanTitle a="Title"/>
			<KanbanSummary a="Summary" />

        </Kanban>

</Page>


{:else}
    <p>Loading chef's data...</p>
{/if}
