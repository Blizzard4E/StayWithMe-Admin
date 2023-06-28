<script>
    import { goto } from "$app/navigation";
    import jwt_decode from "jwt-decode";
    import { onMount } from "svelte";
    import { admin } from "../stores";

    let email, password;
    let failed = false;

    async function login() {
        fetch("http://localhost:3000/admins/login", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({ email: email, password: password }),
        })
            .then(async (res) => {
                const jsonData = await res.json();
                if (jsonData.status == 200) {
                    console.log("work");
                    localStorage.setItem("access_token", jsonData.accessToken);
                    let adminData = jwt_decode(
                        localStorage.getItem("access_token")
                    );
                    admin.update((value) => adminData);
                    location.href = "logged";
                } else failed = true;
                console.log(jsonData);
            })
            .catch((error) => console.error("Error:", error));
    }

    onMount(() => {
        let decodedToken = jwt_decode(localStorage.getItem("access_token"));
        if (decodedToken) {
            if (Date.now() >= decodedToken.exp * 1000) {
            } else {
                goto("/logged");
            }
        }
    });
</script>

<div class="center">
    <div class="container">
        <div class="headings">
            <h1>Welcome Back!</h1>
            <h2>We're excited to have you back!</h2>
        </div>
        <div class="inputs">
            <h3>EMAIL</h3>
            <input type="text" bind:value={email} />
        </div>
        <div class="inputs">
            <h3>PASSWORD</h3>
            <input type="text" bind:value={password} />
        </div>
        {#if failed}
            <p>Wrong Credentials</p>
        {/if}
        <button class="login" on:click={login}>Log In</button>
    </div>
</div>

<style lang="scss">
    .center {
        height: 100vh;
        display: grid;
        place-items: center;

        .container {
            background-color: $bg_light;
            padding: 1.8rem;
            padding-top: 1rem;
            height: max-content;
            width: 30rem;
            overflow: auto;
            box-shadow: 0 0 0.2rem rgba(0, 0, 0, 0.705);
            border-radius: 0.5rem;

            .headings {
                text-align: center;
                h1 {
                    font-size: 1.8rem;
                    font-weight: normal;
                }

                h2 {
                    font-size: 1.2rem;
                    font-weight: normal;
                    color: $txt_nav;
                }
            }

            .inputs {
                margin-top: 1rem;
                h3 {
                    font-size: 0.8rem;
                    font-weight: normal;
                    margin: 0.1rem 0;
                }

                input {
                    padding: 0 2.5%;
                    box-shadow: 0 0 0.2rem black;
                    border-radius: 0.5rem;
                    background-color: $bg_input;
                    height: 2.5rem;
                    width: 95%;
                }
            }

            button {
                cursor: pointer;
                margin: 0.5rem 0;
                font-weight: normal;
                font-size: 0.8rem;
                color: $forget;

                &:hover {
                    text-decoration: underline;
                }
            }

            .login {
                font-size: 1.2rem;
                color: white;
                background-color: $login;
                border-radius: 0.5rem;
                height: 3rem;
                width: 100%;
                &:hover {
                    text-decoration: none;
                    background-color: #9247c4;
                }
            }
        }
    }
</style>
