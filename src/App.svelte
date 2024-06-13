<script>
    import {Layout} from '@humandialog/forms.svelte'
    import {reef} from '@humandialog/auth.svelte'
    import Main from './Main.svelte'
    import Menu from './Menu.svelte'
    import Status from './Status.svelte'
    import Kitchen from './Kitchen.svelte'
    import Delivery from './Delivery.svelte'
    import Dashboard from './Dashboard.svelte'
    import Nav from './Nav.svelte'
    import AppIcon from './appicon.svelte'
    
    reef.configure( {
                    mode: 'local',
                    local: {
                        api:    "http://127.0.0.1:1996/",
                        users:
                        [
                            "Alice",
                            "Bob",
                            "Chef",
                            "Courier",
                            "Clerk"
                        ],
                        apiVersion: "v001"}
                   });


    const layout = {
                sidebar : {
                    'TOC': {
                        authorized :true,
                        icon: AppIcon,
                        component: Nav
                    }
                },
                mainContent : {
                    routes : {
                        '/' :           { component: Main},
                        '/menu':        { component: Menu},
                        '/status':      { component: Status},
                        '/dashboard':   { component: Dashboard},
                        '/kitchen':     { component: Kitchen },
                        '/delivery':    { component: Delivery }
                    }
                },
                mainToolbar : {
                    signin: true
                },
                dark:
                {
                    optional: true,
                    default: false
                }
    }

    
</script>


<Layout {layout} />

      