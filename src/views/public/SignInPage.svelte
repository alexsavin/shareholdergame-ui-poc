<script>
    import { onMount } from 'svelte';
    import {navigateTo} from 'svelte-router-spa'
    import {errorMessage} from "../../stores";
    import {authenticate} from '../../auth.js';

    export let currentRoute
    export let params = {}

    let userNameOrEmail = '';
    let password = '';

    function onSignIn() {
        authenticate(userNameOrEmail, password, function () {
            navigateTo('/secure/home')
        });
    }

    onMount(() => {
        errorMessage.set('')
    })
</script>

<svelte:head>
    <title>Game Stockholder - Sign In</title>
</svelte:head>

<div class="container-fluid">
    <div class="row justify-content-md-center">
        <div class="col-lg-4">
            <h2>Sign In</h2>
            {#if $errorMessage !== ''}
                <div class="alert alert-danger" role="alert">
                    {$errorMessage}
                </div>
            {/if}
            <div class="form-group">
                <label for="emailOrUserName">User name or Email</label>
                <input bind:value={userNameOrEmail} type="text" id="emailOrUserName" class="form-control"
                       placeholder="Enter your name or email"/>
            </div>
            <div class="form-group">
                <label for="userPassword">Password</label>
                <input bind:value={password} type="password" id="userPassword" class="form-control"
                       placeholder="Enter your password"/>
            </div>
            <button type="submit" class="btn btn-primary" on:click={onSignIn}>Sign In</button>
            <div class="flex-grow-1 text-right">
                <a href="/resetpassword">Forgot password?</a>
            </div>
        </div>
    </div>
</div>
