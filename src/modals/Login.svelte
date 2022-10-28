<script>

    // provided by <Modals />
    import axios from "axios";
    import {baseurl} from "../consts.js";
    import {toast} from "@zerodevx/svelte-toast";
    import {userData} from "../stores/user_base.js";
    import {closeModal} from "svelte-modals";

    export let isOpen

    let loginData = {

        email: '',
        password: ''
    }
    const loginSB = () => {
        axios({
            url: `${baseurl}/auth/login`,
            method: 'POST',
            data: JSON.stringify(loginData),
            headers: {
                'Content-Type': 'application/json'
            }
        }).then(r => {
            console.log(r)
            if (r.status === 200) {
                toast.push('Logged in Successfully , Welcome')
                $userData = r.data
                closeModal()
            } else {
                toast.push('Error Logging in, please check your input')
            }
        })
    }
</script>

{#if isOpen}
    <div role="dialog" class="modal">
        <div class="contents">
            <form>
                <!-- Email input -->
                <div class="form-outline mb-4">
                    <label class="form-label" for="form2Example1">Email address</label>

                    <input bind:value={loginData.email} type="email" id="form2Example1" class="form-control"/>
                </div>

                <!-- Password input -->
                <div class="form-outline mb-4">
                    <label class="form-label" for="form2Example2">Password</label>

                    <input bind:value={loginData.password} type="password" id="form2Example2" class="form-control"/>
                </div>
<!--
                &lt;!&ndash; 2 column grid layout for inline styling &ndash;&gt;
                <div class="row mb-4">
                    <div class="col d-flex justify-content-center">
                        &lt;!&ndash; Checkbox &ndash;&gt;
                        <div class="form-check">
                            <input class="form-check-input" type="checkbox" value="" id="form2Example31" checked/>
                            <label class="form-check-label" for="form2Example31"> Remember me </label>
                        </div>
                    </div>

                    <div class="col">
                        &lt;!&ndash; Simple link &ndash;&gt;
                        <a href="#!">Forgot password?</a>
                    </div>
                </div>-->

                <!-- Submit button -->
                <button on:click={loginSB} type="button" class="btn btn-primary btn-block mb-4 w-100">Sign in</button>


            </form>
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
        min-width: 400px;
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
