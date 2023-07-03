<script>
    import { admin } from "../../stores";
    import { goto } from "$app/navigation";
    import jwt_decode from "jwt-decode";
    function checkToken() {
        if (localStorage.getItem("access_token")) {
            let decodedToken = jwt_decode(localStorage.getItem("access_token"));
            if (decodedToken) {
                if (Date.now() >= decodedToken.exp * 1000) {
                    localStorage.removeItem("access_token");
                    goto("/");
                } else {
                    admin.update((value) => decodedToken);
                }
            }
        } else {
            goto("/");
        }
    }

    let adminData;

    admin.subscribe((value) => (adminData = value));

    let email, password, username;

    function createAdmin() {
        checkToken();
        fetch("https://stay-withme-api.cyclic.app/admins/signUp", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                token: localStorage.getItem("access_token"),
                email: email,
                password: password,
                username: username,
                admin_role: adminData.role,
            }),
        })
            .then((res) => res.json())
            .then((jsonData) => {
                if (jsonData.status == 200) {
                    email = null;
                    password = null;
                    username = null;
                }
            });
    }
</script>

<main>
    <div class="center">
        <div class="container">
            <div class="headings">
                <h1>Create Admin</h1>
            </div>
            <div class="inputs">
                <h3>Username</h3>
                <input type="text" bind:value={username} />
            </div>
            <div class="inputs">
                <h3>EMAIL</h3>
                <input type="text" bind:value={email} />
            </div>
            <div class="inputs">
                <h3>PASSWORD</h3>
                <input type="text" bind:value={password} />
            </div>
            <button class="login" on:click={createAdmin}>Create Admin</button>
        </div>
    </div>
</main>


<style lang="scss">
    main{
        grid-template-columns: none;
    }
    
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
