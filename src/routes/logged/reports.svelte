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

    let reports = [];
    let filteredReports = [];
    let selectedUser;

    function selectUser(report) {
        selectedUser = report.user_id;
        selectedUser.report_id = report.id;
    }

    function filterReports() {
        filteredReports = [];
        reports.forEach((report) => {
            if (report.user_id.banned == false) {
                filteredReports.push(report);
            }
        });
    }

    async function banUser(user_id, report_id) {
        checkToken();
        fetch("https://stay-withme-api.cyclic.app/admins/banUser", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                token: localStorage.getItem("access_token"),
                user_id: user_id,
                report_id: report_id,
            }),
        })
            .then((res) => res.json())
            .then((jsonData) => {
                if (jsonData.status == 200) {
                    selectedUser = null;
                    getReports();
                }
                console.log(jsonData);
            });
    }

    async function getReports() {
        checkToken();
        fetch("https://stay-withme-api.cyclic.app/admins/reports", {
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
                    reports = jsonData.reportsData;
                    filterReports();
                }
                console.log(jsonData);
            });
    }

    onMount(() => {
        getReports();
    });
</script>

<main>
    <div class="reports">
        <h1>Reports - {filteredReports.length}</h1>
        <ul>
            {#each filteredReports as report}
                <li on:click={() => selectUser(report)}>
                    <div class="profile">
                        <img src="/favicon.png" alt="" />
                        <div class="info">
                            <h1>{report.user_id.username}</h1>
                            <h2>{report.review_id.feedback}</h2>
                        </div>
                    </div>
                    <button on:click={() => banUser(report.user_id.id, report.id)}
                        >Ban</button
                    >
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
                <button
                    on:click={() =>
                        banUser(selectedUser.id, selectedUser.report_id)}
                    >Ban</button
                >
            </div>
        {/if}
    </div>
</main>

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
                    img {
                        margin: auto 1rem;
                        height: 3rem;
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
                    padding: 0 0.5rem;
                    margin: auto;
                    font-size: 1rem;
                    background: rgb(204, 48, 48);
                    border-radius: 10%;
                    height: 1.8rem;
                    width: 2.5rem;
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

            button {
                color: white;
                font-size: 1rem;
                float: right;
                border-radius: 10%;
                height: 1.8rem;
                width: 2.5rem;
                padding: 0 0.5rem;
                background-color: rgb(204, 48, 48);
                cursor: pointer;
                margin: 0.8rem 0;
            }
        }
    }
</style>
