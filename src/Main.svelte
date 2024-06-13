<script>
    import {Authorized, reef, session} from '@humandialog/auth.svelte'
    import {onMount } from 'svelte';
    import { push } from 'svelte-spa-router';

    onMount( async () =>
    {
        let viewPath = await reef.post('user/GetView', {})
        if(viewPath)
            push(viewPath);
    })
</script>

<p>Loading right view...</p>



