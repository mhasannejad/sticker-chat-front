<script>
    import {io} from "socket.io-client";
    import axios from "axios";
    import {onMount} from "svelte";
    import {adjectives, colors, uniqueNamesGenerator} from "unique-names-generator";
    import {closeModal, Modals, openModal} from "svelte-modals";
    import {baseSocketUrl, baseurl} from "../consts.js";
    import CreateSticker from "../modals/CreateSticker.svelte";
    import {userData} from "../stores/user_base.js";
    import {toast} from "@zerodevx/svelte-toast";


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
    let my_stickers = []
    const socket = io(baseSocketUrl)
    let room_name = shortName


    console.log(socket.readyState)
    socket.emit('room_join_req', {
        room: shortName
    })
    socket.emit('notification', {
        message: 'connected'
    })

    onMount(() => {
        getAllStickers()
        getMyStickers()
    })

    socket.on('message', text => {

        console.log(text)
        messages = [text, ...messages]
        console.log(messages)
    });
    socket.on('notification', text => {
        console.log(text)
        toast.push(text.message)
    })
    const sendMsg = () => {
        socket.emit('chat_message', {
            user: $userData.user,
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
        socket.emit('chat_message'+room_name, {
            user: $userData.user,
            message: {
                type: 'sticker',
                content: sticker
            }
        })
    }


    const changeName = () => {
        name = uniqueNamesGenerator(customConfig)
    }

    const saveSticker = (sticker) => {
        axios({
            url: `${baseurl}/sticker/save`,
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `bearer ${$userData.token}`
            },
            data: JSON.stringify({
                sticker_id: sticker.id
            })
        }).then((r) => {
            console.log(r)
            if (r.status === 201) {
                getMyStickers()
            }
        })
    }

    const getMyStickers = () => {
        axios({
            url: `${baseurl}/sticker/mine`,
            method: 'GET',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `bearer ${$userData.token}`
            },

        }).then((r) => {
            console.log(r)
            if (r.status === 200) {
                my_stickers = r.data
            }
        })
    }

    const joinRoom = () => {
        socket.emit('room_join_req', {
            room: room_name
        })
    }
</script>

<div class="row align-content-center">

    <div class="col-md-7 col-sm-12 ">
        <div class="my-2 row mx-3">
            <div class="col-md-3 "></div>
            <input bind:value={room_name} type="text" name="" class="form-control bg-dark col-md-6 w-50 text-white"
                   id="" >
            <button on:click={joinRoom} class="col-md-3">
                join
            </button>
        </div>


        <div class="message-list">
            {#each messages as msg}
                {#if msg.message.type === 'text'}
                    {#if msg.user.email === $userData.user.email}
                        <div class="row align-items-end message" style="align-items: end!important;">
                            <div class="col-md-8"></div>
                            <div class="col-md-4 align-self-end">
                                <p class="name is-sender">
                                    {msg.user.email}
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
                                    {msg.user.email}
                                </p>
                                <p class="body">
                                    {msg.message.content}
                                </p>
                            </div>
                            <div class="col-md-8"></div>
                        </div>
                    {/if}
                {:else}
                    {#if msg.user.email === $userData.user.email}
                        <div class="row align-items-end message" style="align-items: end!important;">
                            <div class="col-md-8"></div>
                            <div class="col-md-4 align-self-end">
                                <p class="name is-sender">
                                    {msg.user.email}
                                </p>
                                <img src={baseurl+'/'+msg.message.content.path} class="sticker-image" alt="">

                            </div>
                        </div>
                    {:else}
                        <div class="row align-items-start message">
                            <div class="col-md-4 align-self-start">
                                <p class="name">
                                    {msg.user.email}
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

    <div class="col-md-5 col-sm-12">
        <div class="accordion h-100" id="accordionExample">
            <div class="accordion-item">
                <h2 class="accordion-header" id="headingOne">
                    <button class="accordion-button" type="button" data-bs-toggle="collapse"
                            data-bs-target="#collapseOne" aria-expanded="true" aria-controls="collapseOne">
                        Explore Stickers
                    </button>
                </h2>
                <div id="collapseOne" class="accordion-collapse collapse show" aria-labelledby="headingOne"
                     data-bs-parent="#accordionExample">
                    <div class="accordion-body">
                        <div class="row justify-content-center mx-2">
                            <button class="w-100" on:click={getAllStickers}>refresh</button>
                        </div>
                        <div class="row">
                            <div class="col-md-4 my-1 sticker-cell col-sm-4 col-xs-4">
                                <button class="h-100 w-100" on:click={()=>{
                        openModal(CreateSticker, { title: "Alert", message: "This is an alert" })
                    }}>
                                    <div class=" h-100">
                                        <img src="/add_sticker.png" alt="" class="sticker-image">
                                    </div>
                                </button>
                            </div>
                            {#each stickers as sticker}
                                <div class="col-md-4 my-1 sticker-cell col-sm-4 col-xs-4 ">
                                    <button class="h-75 w-100" on:click={()=>{
                        sendMessage(sticker)
                    }}>
                                        <div class=" h-100">
                                            <img src={baseurl+'/'+sticker.path} alt="" class="sticker-image">
                                        </div>
                                    </button>
                                    <button class="w-100 my-1" on:click={()=>{saveSticker(sticker)}}>
                                        save
                                    </button>
                                </div>
                            {/each}

                        </div>
                    </div>
                </div>
            </div>
            <div class="accordion-item">
                <h2 class="accordion-header" id="headingTwo">
                    <button class="accordion-button collapsed" type="button" data-bs-toggle="collapse"
                            data-bs-target="#collapseTwo" aria-expanded="false" aria-controls="collapseTwo">
                        Saved Stickers
                    </button>
                </h2>
                <div id="collapseTwo" class="accordion-collapse collapse" aria-labelledby="headingTwo"
                     data-bs-parent="#accordionExample">
                    <div class="accordion-body">
                        <div class="row justify-content-center mx-2">
                            <button class="w-100" on:click={getAllStickers}>refresh</button>
                        </div>
                        <div class="row">

                            {#each my_stickers as sticker}
                                <div class="col-md-4 my-1 sticker-cell col-sm-4 col-xs-4 ">
                                    <button class="h-100 w-100" on:click={()=>{
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
            </div>

        </div>
        <!--<div class="row justify-content-center mx-2">
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

        </div>-->
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
</style>
