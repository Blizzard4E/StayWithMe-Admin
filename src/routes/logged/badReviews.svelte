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

    let ratings = [];
    let filteredRatings = [];
    let selectedHotel;

    function selectHotel(hotel) {
        selectedHotel = hotel;
    }

    function filterRatings() {
        filteredRatings = [];
        ratings.forEach((rating) => {
            if (rating.hotel_id.banned == false) {
                filteredRatings.push(rating);
            }
        });
        console.log(filteredRatings);
    }

    async function banHotel(hotel_id) {
        checkToken();
        fetch("http://localhost:3000/admins/banHotel", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                token: localStorage.getItem("access_token"),
                hotel_id: hotel_id,
            }),
        })
            .then((res) => res.json())
            .then((jsonData) => {
                if (jsonData.status == 200) {
                    selectedHotel = null;
                    getRatings();
                }
                console.log(jsonData);
            });
    }

    async function getRatings() {
        checkToken();
        fetch("http://localhost:3000/admins/ratings", {
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
                    ratings = jsonData.ratingsData;
                    filterRatings();
                }
                console.log(jsonData);
            });
    }

    onMount(() => {
        getRatings();
    });
</script>

<div class="reviews">
    <h1>Reviews - {filteredRatings.length}</h1>
    <ul>
        {#each filteredRatings as rating}
            <li on:click={() => selectHotel(rating.hotel_id)}>
                <div class="profile">
                    <img src={rating.hotel_id.images[4]} alt="" />
                    <div class="info">
                        <h1>{rating.hotel_id.name}</h1>
                        <div class="stars">
                            {#each Array(rating.ratings) as _, i}
                                <div class="star">
                                    <img src="star.png" alt="" />
                                </div>
                            {/each}
                        </div>
                        <h2>{rating.feedback}</h2>
                    </div>
                </div>
                <button on:click={() => banHotel(rating.hotel_id.id)}
                    >Ban</button
                >
            </li>
        {/each}
    </ul>
</div>

<div class="details">
    <h1>Details</h1>
    {#if selectedHotel}
        <div class="info">
            <img src={selectedHotel.images[4]} alt="" />
            <h1>{selectedHotel.name}</h1>
            <h3>{selectedHotel.email}</h3>

            <h2>Description</h2>
            <h3>{selectedHotel.description}</h3>

            <div class="location">
                <h2>Country:</h2>
                <h3>{selectedHotel.country}</h3>
            </div>
            <div class="googleMaps">
                <h2>Google Map:</h2>
                <h3>{selectedHotel.googleMap}</h3>
            </div>
            <!-- svelte-ignore a11y-missing-attribute -->
            <iframe
                src={"https://maps.google.com/maps?&q=" +
                    encodeURIComponent(selectedHotel.googleMap) +
                    "&output=embed"}
                width="100%"
                height="200"
                style="border:0;"
                allowfullscreen=""
                loading="lazy"
                referrerpolicy="no-referrer-when-downgrade"
            />

            <h2>Images</h2>
            <div class="images">
                <img src={selectedHotel.images[0]} alt="" />
                <img src={selectedHotel.images[1]} alt="" />
                <img src={selectedHotel.images[2]} alt="" />
                <img src={selectedHotel.images[3]} alt="" />
            </div>

            <button on:click={() => banHotel(selectedHotel.id)}>Ban</button>
        </div>
    {/if}
</div>

<style lang="scss">
    .reviews {
        overflow: auto;
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
                        width: 70px;
                        border-radius: 50%;
                        aspect-ratio: 1/1;
                        object-fit: cover;
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
                    .stars {
                        display: flex;
                        column-gap: 0.25rem;
                        .star {
                            width: 16px;
                            img {
                                margin: 0;
                                width: 100%;
                                height: auto;
                                aspect-ratio: 1/1;
                            }
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
                width: 120px;
                aspect-ratio: 1/1;
                border-radius: 50%;
                object-fit: cover;
            }

            h1 {
                font-weight: bold;
                font-size: 1.4rem;
            }

            h2 {
                font-weight: bold;
                font-size: 1rem;
                margin-top: 1rem;
            }

            h3 {
                font-weight: lighter;
                font-size: 0.9rem;
            }

            .location,
            .googleMaps {
                align-items: center;
                display: flex;
                margin-top: 1rem;
                h2 {
                    margin: 0;
                }
                h3 {
                    font-size: 1rem;
                    margin: 0 0.3rem;
                }
            }

            iframe {
                margin: 0.5rem 0;
                align-items: center;
                height: 10rem;
                width: 100%;
            }

            .images {
                margin-top: 0.5rem;
                display: grid;
                grid-template-columns: 1fr 1fr;
                place-items: center;
                column-gap: 1rem;
                row-gap: 1rem;
                img {
                    border-radius: 0;
                    width: 100%;
                    aspect-ratio: 2/1.5;
                    object-fit: cover;
                }
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
