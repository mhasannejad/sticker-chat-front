<script>
    import {io} from "socket.io-client";
    import {baseSocketUrl, baseurl} from "./consts.js";
    import axios from "axios";
    import {onMount} from "svelte";
    import {adjectives, colors, uniqueNamesGenerator} from "unique-names-generator";
    import {closeModal, Modals, openModal} from "svelte-modals";
    import CreateSticker from "./modals/CreateSticker.svelte";
    import { SvelteToast } from '@zerodevx/svelte-toast'
    console.log(baseurl)
    const customConfig = {
        dictionaries: [adjectives, colors],
        separator: '-',
        length: 2,
    };
    const shortName = uniqueNamesGenerator(customConfig);
    let message = ''
    let name = shortName
    let messages = []

    let stickers = []
    const socket = io(baseSocketUrl)
    console.log(socket.readyState)
    socket.emit('message', {
        message: 'connected '
    })

    onMount(() => {
        getAllStickers()
    })

    socket.on('message', text => {

        console.log(text)
        messages = [text, ...messages]
        console.log(messages)
    });

    const sendMsg = () => {
        socket.emit('chat_message', {
            name: name,
            message: {
                type: 'text',
                content: message
            }
        })
        message = ''
    }

    const getAllStickers = () => {
        axios({
            url: `${baseurl}/sticker/all`,
            method: 'GET',
            headers: {
                'Content-Type': 'application/json'
            },

        }).then((r) => {
            stickers = r.data
            console.log(stickers)
        })
    }

    const sendMessage = (sticker) => {
        socket.emit('chat_message', {
            name: name,
            message: {
                type: 'sticker',
                content: sticker
            }
        })
    }


    const changeName = () => {
        name = uniqueNamesGenerator(customConfig)
    }
</script>
<SvelteToast  />
<Modals>
    <div
            slot="backdrop"
            class="backdrop"
            on:click={closeModal}
    ></div>
</Modals>

<div class="row align-content-center">

    <div class="col-md-8 col-sm-12 ">
        <div class="my-2 row mx-3">
            <div class="col-md-3"></div>
            <p class="text-white col-md-6">
                {name}
            </p>
            <button on:click={changeName} class="col-md-3">
                rename
            </button>
        </div>


        <div class="message-list">
            {#each messages as msg}
                {#if msg.message.type === 'text'}
                    {#if msg.name === name}
                        <div class="row align-items-end message" style="align-items: end!important;">
                            <div class="col-md-8"></div>
                            <div class="col-md-4 align-self-end">
                                <p class="name is-sender">
                                    {msg.name}
                                </p>
                                <p class="body is-sender">
                                    {msg.message.content}
                                </p>
                            </div>
                        </div>
                    {:else}
                        <div class="row align-items-start message">
                            <div class="col-md-4 align-self-start">
                                <p class="name">
                                    {msg.name}
                                </p>
                                <p class="body">
                                    {msg.message.content}
                                </p>
                            </div>
                            <div class="col-md-8"></div>
                        </div>
                    {/if}
                {:else}
                    {#if msg.name === name}
                        <div class="row align-items-end message" style="align-items: end!important;">
                            <div class="col-md-8"></div>
                            <div class="col-md-4 align-self-end">
                                <p class="name is-sender">
                                    {msg.name}
                                </p>
                                <img src={baseurl+'/'+msg.message.content.path} class="sticker-image" alt="">

                            </div>
                        </div>
                    {:else}
                        <div class="row align-items-start message">
                            <div class="col-md-4 align-self-start">
                                <p class="name">
                                    {msg.name}
                                </p>

                                <img src={baseurl+'/'+msg.message.content.path} class="sticker-image" alt="">
                            </div>
                            <div class="col-md-8"></div>
                        </div>
                    {/if}
                {/if}

            {/each}
        </div>
        <!-- please dont mess with the concept of app and dont change display on this  -->
        <div class="my-3" style="display:none;">
            <div class="input-group">
                <input type="text" placeholder="message" bind:value={message} class="form-control">
                <button on:click={sendMsg} class="">
                    send
                </button>
            </div>
        </div>
    </div>

    <div class="col-md-4 col-sm-12">
        <div class="row justify-content-center mx-2">
            <button class="w-100" on:click={getAllStickers}>refresh</button>
        </div>
        <div class="row">
            <div class="col-md-4 my-1 sticker-cell col-sm-4 col-xs-4">
                <button class="h-100" on:click={()=>{
                        openModal(CreateSticker, { title: "Alert", message: "This is an alert" })
                    }}>
                    <div class=" h-100">
                        <img src="/add_sticker.png" alt="" class="sticker-image">
                    </div>
                </button>
            </div>
            {#each stickers as sticker}
                <div class="col-md-4 my-1 sticker-cell col-sm-4 col-xs-4">
                    <button class="h-100" on:click={()=>{
                        sendMessage(sticker)
                    }}>
                        <div class=" h-100">
                            <img src={baseurl+'/'+sticker.path} alt="" class="sticker-image">
                        </div>
                    </button>
                </div>
            {/each}

        </div>
    </div>
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
