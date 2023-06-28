<script>
    import { goto } from "$app/navigation";
    import jwt_decode from "jwt-decode";
    import { onMount } from "svelte";
    import { admin } from "../../stores";
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

    let bannedUsers = [];
    let selectedUser;

    function selectUser(user) {
        selectedUser = user;
    }

    async function unBanUser(user_id) {
        checkToken();
        fetch("http://localhost:3000/admins/unBanUser", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                token: localStorage.getItem("access_token"),
                user_id: user_id,
            }),
        })
            .then((res) => res.json())
            .then((jsonData) => {
                if (jsonData.status == 200) {
                    selectedUser = null;
                    getBannedUsers();
                }
                console.log(jsonData);
            });
    }

    async function getBannedUsers() {
        checkToken();
        fetch("http://localhost:3000/admins/bannedUsers", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                token: localStorage.getItem("access_token"),
            }),
        })
            .then((res) => res.json())
            .then((jsonData) => {
                if (jsonData.status == 200) {
                    bannedUsers = jsonData.data;
                }
                console.log(jsonData);
            });
    }

    onMount(() => {
        getBannedUsers();
    });
</script>

<div class="reports">
    <h1>Banned Users - {bannedUsers.length}</h1>
    <ul>
        {#each bannedUsers as ban}
            <li on:click={() => selectUser(ban)}>
                <div class="profile">
                    <img src={ban.profile_pic} alt="" />
                    <div class="info">
                        <h1>{ban.username}</h1>
                        <h2>{ban.bio}</h2>
                    </div>
                </div>
                <div>
                    <button on:click={() => unBanUser(ban.id)}>Unban</button>
                </div>
            </li>
        {/each}
    </ul>
</div>

<div class="details">
    {#if selectedUser}
        <h1>Details</h1>
        <div class="info">
            <img src={selectedUser.profile_pic} alt="" />
            <h1>{selectedUser.username}</h1>
            <h3>{selectedUser.email}</h3>
            <p>{selectedUser.bio}</p>
            <div>
                <button on:click={() => unBanUser(selectedUser.id)}
                    >Unban</button
                >
            </div>
        </div>
    {/if}
</div>

<style lang="scss">
    .reports {
        overflow: auto;
        /* width */
        &::-webkit-scrollbar {
            width: 5px;
        }

        /* Track */
        &::-webkit-scrollbar-track {
            background: #646464;
        }

        /* Handle */
        &::-webkit-scrollbar-thumb {
            background: #161616;
        }

        /* Handle on hover */
        &::-webkit-scrollbar-thumb:hover {
            background: rgb(46, 46, 46);
        }
        h1 {
            color: grey;
            font-weight: normal;
            font-size: 0.95rem;
            margin: 0.5rem 1rem;
        }

        ul {
            margin: 0 2%;

            li {
                padding: 0.4rem 0;
                display: grid;
                grid-template-columns: 90% 10%;
                border-top: 2px solid rgba(255, 255, 255, 0.3);

                min-height: 4rem;

                .profile {
                    display: flex;
                    column-gap: 1rem;
                    img {
                        width: 64px;
                        aspect-ratio: 1/1;
                        object-fit: cover;
                        border-radius: 50%;
                    }

                    .info {
                        display: grid;
                        h1 {
                            color: $name_pf;
                            font-weight: bold;
                            font-size: 1.2rem;
                            margin: auto 0;
                        }
                        h2 {
                            margin: auto 0;
                            color: $sub_pf;
                            font-weight: normal;
                            font-size: 1rem;
                            word-break: break-all;
                        }
                    }
                }

                button {
                    margin-top: 1.25rem;
                    padding: 0 0.25rem;
                    font-size: 1rem;
                    background: rgb(48, 134, 204);
                    border-radius: 10%;
                    cursor: pointer;
                }
            }
        }
    }

    .details {
        background-color: $bg_dark;
        display: flex;
        flex-direction: column;
        overflow: auto;
        padding: 5% 6%;
        h1 {
            font-size: xx-large;
        }
        .info {
            background-color: $details;
            padding: 0.5rem 5%;

            img {
                height: 6rem;
                border-radius: 50%;
            }

            h1 {
                font-weight: bold;
                font-size: 1.4rem;
            }

            h3,
            p {
                font-weight: lighter;
                font-size: 1rem;
            }
            div {
                display: flex;
                justify-content: end;
            }
            button {
                color: white;
                font-size: 1rem;
                border-radius: 10%;
                padding: 0 0.5rem;
                background-color: rgb(48, 134, 204);
                cursor: pointer;
            }
        }
    }
</style>
