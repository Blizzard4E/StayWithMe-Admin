<script>
    import reports from "./reports.svelte";
    import badReviews from "./badReviews.svelte";
    import unbanUser from "./unbanUser.svelte";
    import unbanHotel from "./unbanHotel.svelte/";
    import { goto } from "$app/navigation";
    import jwt_decode from "jwt-decode";
    import CreateAdmin from "./createAdmin.svelte";
    import { admin } from "../../stores";
    import { onMount } from "svelte";

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
    let adminList = [];

    async function getAdmins() {
        checkToken();
        fetch("http://localhost:3000/admins/all", {
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
                    adminList = jsonData.data;
                }
                console.log(jsonData);
            });
    }

    function logOut() {
        localStorage.removeItem("access_token");
        admin.update((value) => null);
        goto("/");
    }

    admin.subscribe((value) => {
        adminData = value;
    });
    const webpages = [
        { name: "Reports", component: reports },
        { name: "Low Ratings", component: badReviews },
        { name: "Unban Users", component: unbanUser },
        { name: "Unban Hotels", component: unbanHotel },
        { name: "Create Admin", component: CreateAdmin },
    ];

    // Loads an object in webpages array
    let selectedPage = webpages[0];
    $: console.dir(selectedPage);

    // Have to use obj as arg. so value can be a class
    // @ts-ignore
    const tab = (obj) => (selectedPage = obj);
    let currentPage = 0;

    onMount(() => {
        getAdmins();
    });
</script>

<nav>
    <div class="tabs">
        <h1>Stay WithMe</h1>

        <button
            class="tablink {currentPage === 0 ? 'active' : ''}"
            title={webpages[0].name}
            on:click={() => {
                tab(webpages[0]);
                currentPage = 0;
            }}
        >
            <div class="tab">
                <img src="/favicon.png" alt="" />
                <h3>{webpages[0].name}</h3>
            </div>
        </button>

        <button
            class="tablink {currentPage === 1 ? 'active' : ''}"
            title={webpages[1].name}
            on:click={() => {
                tab(webpages[1]);
                currentPage = 1;
            }}
        >
            <div class="tab">
                <img src="/favicon.png" alt="" />
                <h3>{webpages[1].name}</h3>
            </div>
        </button>

        <button
            class="tablink {currentPage === 2 ? 'active' : ''}"
            title={webpages[2].name}
            on:click={() => {
                tab(webpages[2]);
                currentPage = 2;
            }}
        >
            <div class="tab">
                <img src="/favicon.png" alt="" />
                <h3>{webpages[2].name}</h3>
            </div>
        </button>

        <button
            class="tablink {currentPage === 3 ? 'active' : ''}"
            title={webpages[3].name}
            on:click={() => {
                tab(webpages[3]);
                currentPage = 3;
            }}
        >
            <div class="tab">
                <img src="/favicon.png" alt="" />
                <h3>{webpages[3].name}</h3>
            </div>
        </button>
        {#if adminData}
            {#if adminData.role == 1}
                <button
                    class="tablink {currentPage === 4 ? 'active' : ''}"
                    title={webpages[4].name}
                    on:click={() => {
                        tab(webpages[4]);
                        currentPage = 4;
                    }}
                >
                    <div class="tab">
                        <img src="/favicon.png" alt="" />
                        <h3>{webpages[4].name}</h3>
                    </div>
                </button>
            {/if}
        {/if}
    </div>

    <div class="admins">
        <h1>Other Admins</h1>
        {#each adminList as admin}
            <div class="admin">
                <div class="info">
                    <h2>{admin.username}</h2>
                    <h3>
                        {#if admin.role == 1}
                            Head Admin
                        {:else}
                            Moderator
                        {/if}
                    </h3>
                </div>
            </div>
        {/each}
    </div>

    <div class="profile">
        {#if adminData}
            <div class="about">
                <h2>{adminData.username}</h2>
                <h3>{adminData.email}</h3>
            </div>
            <div class="logout">
                <button on:click={logOut}>Log Out</button>
            </div>
        {/if}
    </div>
</nav>

<main>
    <svelte:component this={selectedPage.component} />
</main>

<style lang="scss">
    .logout {
        display: flex;
        align-items: center;

        button {
            padding: 0.5rem;
            background-color: rgb(204, 48, 48);
        }
    }
    nav {
        overflow: auto;
        background-color: $bg_dark;
        display: grid;
        grid-template-rows: 46% 46% 8%;

        .tabs {
            display: grid;
            grid-template-rows: repeat(6, 3rem);
            padding-top: 1rem;
            padding-left: 1rem;

            h1 {
                padding: 0 0.6rem;
            }

            .tablink {
                .tab {
                    display: flex;
                    justify-content: start;
                    cursor: pointer;
                    padding: 0.5rem 0;

                    img {
                        margin: auto 0;
                        height: 1.8rem;
                        padding: 0 0.6rem;
                    }
                    h3 {
                        margin: auto 0;
                        font-weight: normal;
                        font-size: 1rem;
                        color: $txt_nav;
                    }
                }
            }

            .tablink.active {
                h3 {
                    color: white;
                }
                background-color: #393c43;
                border-bottom-left-radius: 9%;
                border-top-left-radius: 9%;
            }
        }

        .admins {
            overflow: auto;
            padding-left: 1rem;

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
                font-weight: lighter;
                font-size: 1rem;
                color: $txt_nav;
            }

            .admin {
                display: flex;
                margin: 0.2rem 0;

                img {
                    margin: auto 0;
                    height: 1.8rem;
                    padding: 0 0.6rem;
                }

                .info {
                    display: grid;
                    margin: auto 0;
                    margin-left: 0.3rem;
                    h2 {
                        margin: auto 0;
                        font-weight: normal;
                        font-size: 0.85rem;
                        color: $txt_nav;
                    }
                    h3 {
                        margin: auto 0;
                        font-weight: bold;
                        font-size: 0.7rem;
                        word-break: break-all;
                        color: white;
                    }
                }
            }
        }

        .profile {
            background-color: $logged_acc;
            display: flex;
            justify-content: space-between;
            padding: 0rem 1rem;
            height: 100%;

            img {
                margin: auto 0;
                height: 2rem;
                padding: 0 0.6rem;
                border-radius: 50%;
            }

            .about {
                display: grid;
                margin: auto 0;
                margin-left: 0.3rem;
                h2 {
                    margin: auto 0;
                    font-weight: bold;
                    font-size: 1rem;
                    color: white;
                }
                h3 {
                    margin: auto 0;
                    font-weight: normal;
                    font-size: 0.9rem;
                    word-break: break-all;
                    color: $txt_nav;
                }
            }
        }
    }

    main {
        padding-top: 0.5%;
        background-color: $bg_light;
        display: grid;
        grid-template-columns: 70% 30%;
        overflow: hidden;
    }
</style>
