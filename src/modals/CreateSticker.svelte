<script>
    import {closeModal} from 'svelte-modals'
    import axios from "axios";
    import {baseurl} from "../consts.js";
    import {toast} from "@zerodevx/svelte-toast";

    // provided by <Modals />
    export let isOpen

    export let title
    export let message

    let sticker, fileinput;

    const onFileSelected = (e) => {
        let image = e.target.files[0];
        let reader = new FileReader();
        reader.readAsDataURL(image);
        reader.onload = e => {
            sticker = e.target.result
        };
    }
    let stickerObj = {
        creator: '',
        description: '',
        sticker: sticker,
    }


    const uploadSticker = () => {
        let dataArray = new FormData();
        dataArray.append("sticker", sticker);
        dataArray.append("creator", stickerObj.creator);
        dataArray.append("description", stickerObj.description);
        console.log(...dataArray)
        console.log((stickerObj))

        /*const response = fetch(`${baseurl}/sticker/new`, {
            method: "POST",
            body: dataArray
        }).then(r=>{
            console.log(r)
        });*/
        /*axios.post(`${baseurl}/sticker/new`, JSON.stringify(stickerObj), {
            headers:{
                'Content-Type':'application/json'
            }
        }).then(r=>{
            console.log(r.data)
        });*/
        axios({
            url: `${baseurl}/sticker/new`,
            method: 'POST',
            headers: [['Content-Type','multipart/form-data']],
            body: dataArray,
            data:dataArray
        }).then(r => {
            if(r.data.status === true){
                closeModal()
                toast.push('uploaded')
            }
        })
    }
</script>

{#if isOpen}
    <div role="dialog" class="modal">
        <div class="contents ">
            <div class="mb-3">
                <label class="form-label">Creator name</label>
                <input bind:value={stickerObj.creator} type="text" class="form-control">
                <div class="form-text">stickers creator name</div>
            </div>
            <div class="mb-3">
                <label class="form-label">About this Sticker</label>
                <input bind:value={stickerObj.description} type="text" class="form-control">
                <div class="form-text">stickers creator name</div>
            </div>


            <input style="display:none" type="file" accept=".jpg, .jpeg, .png, .webp" on:change={(e)=>onFileSelected(e)}
                   bind:this={fileinput}/>

            <div class="d-flex justify-content-center">
                <img src={sticker || 'public/add_sticker.png'} class="sticker-select" alt=""
                     on:click={()=>{fileinput.click();}}>
            </div>
            <div class="actions">
                <button on:click={uploadSticker} class="w-100">Create</button>
            </div>
        </div>
    </div>
{/if}

<style>
    .modal {
        position: fixed;
        top: 0;
        bottom: 0;
        right: 0;
        left: 0;
        display: flex;
        justify-content: center;
        align-items: center;

        /* allow click-through to backdrop */
        pointer-events: none;
    }

    .contents {
        min-width: 360px;
        border-radius: 6px;
        padding: 16px;
        background: white;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        pointer-events: auto;
    }

    h2 {
        text-align: center;
        font-size: 24px;
    }

    p {
        text-align: center;
        margin-top: 16px;
    }

    .actions {
        margin-top: 32px;
        display: flex;
        justify-content: flex-end;
    }

</style>
