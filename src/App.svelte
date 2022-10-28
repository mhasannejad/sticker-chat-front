<script>

    import {closeModal, Modals} from "svelte-modals";
    import {SvelteToast} from "@zerodevx/svelte-toast";
    import Navbar from "./lib/Navbar.svelte";
    import Messager from "./lib/Messager.svelte";
</script>
<SvelteToast  />
<Modals>
    <div
            slot="backdrop"
            class="backdrop"
            on:click={closeModal}
    ></div>
</Modals>
<Navbar />
<div class="px-1">
    <Messager />
</div>

<style>


    .message {
        height: 140px;
        border-radius: 10px;
        border-width: 1px !important;
        border-color: #646cff !important;
        background-color: #242424;
        margin: 10px;
    }

    .name {
        text-align: start;
        font-size: 12px;
        color: white;
    }

    .body {
        font-size: 20px;
        color: white;
        text-align: start;
    }

    .is-sender {
        text-align: end !important;
    }

    .message-list {
        height: 550px !important;
        max-height: 550px !important;
        width: 100%;
        place-content: end;
        overflow-y: scroll !important;
    }


    .backdrop {
        position: fixed;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
        background: rgba(0, 0, 0, 0.50)
    }

</style>
