<script lang="ts">
    let id: number | null;
    let birthDate: string | null;
    let error = false;
    let errorMessage: string | null = null;
    let primarySelection: string | null = null;
    // let primarySelection: string | null = "Ökologie";
    let secondarySelection: string | null = null;
    let done = false;

    let data: object | null = null;
    // let data: object | null = JSON.parse(
    //     '{"id":"4009","stufe":"8","gebdat":"2005-11-16","erstwahl":"Ökologie","zweitwahl":null,"subjects":[{"fach":"Informatik"},{"fach":"\\u00d6kologie"},{"fach":"Cultural Studies"},{"fach":"Spanisch"},{"fach":"Franz\\u00f6sisch"}]}'
    // );
    let loading = false;
    $: secondarySelectionAvailable = ((data || {})["stufe"] || "") !== "6";

    const emojis = {
        Informatik: "🧑‍💻",
        Ökologie: "🌱",
        "Cultural Studies": "🇺🇸",
        Spanisch: "🇪🇸",
        Französisch: "🇫🇷",
        Latein: "📜",
    };

    async function fetchUserInfo() {
        error = false;
        errorMessage = null;
        loading = true;
        const res = await fetch("https://rz.gymhaan.de/q2/wahlen/wahlen.php", {
            method: "POST",
            headers: {
                "Content-Type": "application/json",
            },
            body: JSON.stringify({
                id,
                gebdat: birthDate,
            }),
        });

        data = await res.json();
        loading = false;

        // determine if request was successful
        if (!res.ok || !data["id"]) {
            error = true;
            data = null;
        }

        primarySelection = data["erstwahl"];
        secondarySelection = data["zweitwahl"];

        console.log(data);
    }

    async function submit() {
        try {
            loading = true;
            error = false;
            errorMessage = null;
            const res = await fetch(
                "https://rz.gymhaan.de/q2/wahlen/wahlen.php",
                {
                    method: "POST",
                    headers: {
                        "Content-Type": "application/json",
                    },
                    body: JSON.stringify({
                        id: data["id"],
                        gebdat: data["gebdat"],
                        erstwahl: primarySelection,
                        zweitwahl: secondarySelection,
                    }),
                }
            );
            loading = false;
            done = true;
        } catch (e) {
            error = true;
            errorMessage = e.message;
        }
    }
</script>

<main>
    <a id="logo" href="./">
        <img src="./logo.svg" alt="Logo" />
        <h1>Gymnasium Haan</h1>
    </a>
    <div id="form" class:expanded-form={secondarySelectionAvailable && data}>
        {#if error}
            <div id="error">
                {#if errorMessage}
                    {errorMessage}
                {:else}
                    Joa nee eher nicht bro
                {/if}
            </div>
        {/if}
        {#if loading}
            Loading...
        {:else if data}
            {#if done}
                <h1>Fertig 🎉</h1>
                <a href="/" id="back-link">Zurück</a>
            {:else}
                <h1>Wahlen für Stufe {data["stufe"]}</h1>
                <div id="former-choices">
                    {#if data["erstwahl"] || data["zweitwahl"]}
                        <h2>Bisher</h2>
                    {/if}
                    {#if data["erstwahl"]}
                        <div>
                            Erstwahl: {data["erstwahl"]}
                            {emojis[data["erstwahl"]]}
                        </div>
                    {/if}
                    {#if data["zweitwahl"]}
                        <div>
                            Zweitwahl: {data["zweitwahl"]}
                            {emojis[data["zweitwahl"]]}
                        </div>
                    {/if}
                </div>
                <div id="choices">
                    <div class="choice">
                        {#if secondarySelectionAvailable}
                            <h2>Erstwahl</h2>
                        {/if}
                        <div id="subject-choices">
                            {#each data["subjects"] as subject}
                                <button
                                    class="subject"
                                    on:click={() => {
                                        primarySelection = subject["fach"];
                                        if (
                                            secondarySelection ===
                                            primarySelection
                                        ) {
                                            secondarySelection = null;
                                        }
                                    }}
                                    class:selected={primarySelection ===
                                        subject["fach"]}
                                >
                                    <div class="subject-icon">
                                        {emojis[subject["fach"]] || "?"}
                                    </div>
                                    <div class="subject-name">
                                        {subject["fach"]}
                                    </div>
                                </button>
                            {/each}
                        </div>
                    </div>
                    {#if secondarySelectionAvailable}
                        <div class="choice">
                            <h2>Zweitwahl</h2>
                            <div id="subject-choices">
                                {#each data["subjects"] as subject}
                                    <button
                                        class="subject"
                                        on:click={() => {
                                            secondarySelection =
                                                subject["fach"];
                                            if (
                                                secondarySelection ===
                                                primarySelection
                                            ) {
                                                primarySelection = null;
                                            }
                                        }}
                                        class:selected={secondarySelection ===
                                            subject["fach"]}
                                    >
                                        <div class="subject-icon">
                                            {emojis[subject["fach"]] || "?"}
                                        </div>
                                        <div class="subject-name">
                                            {subject["fach"]}
                                        </div>
                                    </button>
                                {/each}
                            </div>
                        </div>
                    {/if}
                </div>
                <button
                    id="submit-button"
                    disabled={primarySelection === null ||
                        (secondarySelection === null &&
                            secondarySelectionAvailable)}
                    on:click={() => {
                        if (
                            primarySelection !== null &&
                            (secondarySelection !== null ||
                                !secondarySelectionAvailable)
                        ) {
                            submit();
                        }
                    }}>Weiter</button
                >
            {/if}
        {:else}
            <h1>Wahlen Wahlpflichtfächer</h1>
            <form on:submit|preventDefault={fetchUserInfo}>
                <label for="id">Schüler-ID</label>
                <input
                    name="id"
                    type="number"
                    placeholder="1264"
                    bind:value={id}
                />
                <label for="birthDate">Geburtsdatum</label>
                <input
                    name="birthDate"
                    type="date"
                    placeholder="13.12.2007"
                    bind:value={birthDate}
                />
                <input
                    type="submit"
                    value="Anmelden"
                    disabled={!(birthDate && id)}
                />
            </form>
        {/if}
    </div>
</main>

<style lang="scss">
    #logo {
        margin-bottom: auto;

        display: flex;
        flex-direction: row;
        align-items: center;

        height: 2.8rem;
        gap: 1rem;
        color: #1e293b;
        text-decoration: none;

        img {
            height: 100%;
        }

        h1 {
            font-weight: 400;
            margin: 0;
            text-transform: uppercase;

            font-size: 1.5rem;
        }
    }

    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        min-height: 100vh;
        padding: 2rem;
        gap: 1rem;
        background-color: #e2e8f0;

        color: #1e293b;
        font-size: 18px;
    }

    #form {
        margin-bottom: auto;

        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        width: 100%;
        max-width: 600px;
        padding: 3rem;
        border-radius: 1rem;
        background-color: #f8fafc;
        gap: 1rem;

        filter: drop-shadow(0 25px 25px rgb(0 0 0 / 0.15));

        &.expanded-form {
            max-width: 1200px;
            #choices {
                display: flex;
                flex-direction: row;
                gap: 4rem;
                .choice {
                    flex: 1;
                }
            }

            #submit-button {
                margin-top: 2rem;
                width: 50%;
            }
        }

        * {
            width: 100%;
        }

        h1 {
            font-size: 2rem;
            font-weight: 400;
            text-align: center;
            text-transform: uppercase;
        }

        h2 {
            font-weight: 400;
            margin: 0.5rem 0;
        }

        input[type="submit"] {
            background-color: #6d28d9;
            color: #ede9fe;
            border: none;
            border-radius: 0.5rem;
            font-weight: bold;
            text-transform: uppercase;

            padding: 0.8rem;
        }
        // disabled
        input[type="submit"]:disabled {
            background-color: #f1f5f9;
        }
    }

    #error {
        background-color: #ef4444;
        width: 100%;

        border-radius: 0.5rem;
        padding: 1rem;

        color: #fef2f2;
    }

    #subject-choices {
        display: grid;
        grid-template-columns: calc(50% - 0.5rem) calc(50% - 0.5rem);
        gap: 1rem;
        overflow: hidden;
        .subject {
            width: 100%;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            border-radius: 0.5rem;
            text-align: center;
            border: 3px solid #e2e8f0;
            padding: 0.5rem;
            background-color: #f8fafc;
            transition: border-color 0.2s ease-in-out;
            margin: 0;

            cursor: pointer;

            &.selected {
                border-color: #6d28d9 !important;
            }

            .subject-icon {
                font-size: 3rem;
                color: #94a3b8; // fallback for the question mark
            }
        }
    }
    #submit-button {
        background-color: #6d28d9;
        color: #ede9fe;
        border: none;
        border-radius: 0.5rem;
        font-weight: bold;
        text-transform: uppercase;
        transition: background-color 0.2s ease-in-out;

        padding: 0.8rem;
        cursor: pointer;
    }
    // disabled
    #submit-button:disabled {
        background-color: #f1f5f9;
        cursor: not-allowed;
    }

    #back-link {
        display: block;
        background-color: #6d28d9;
        color: #ede9fe;
        font-weight: bold;
        border-radius: 0.5rem;
        text-transform: uppercase;
        padding: 1rem;
        text-align: center;
    }

    @media (max-width: 600px) {
        #logo {
            img {
                height: 2.4rem;
            }

            h1 {
                font-size: 1.2rem;
            }
        }

        main {
            padding: 1rem;
        }

        #form {
            h1 {
                font-size: 1.5rem;
            }

            padding: 1.5rem;
        }

        #subject-choices {
            gap: 0.5rem;
        }

        .expanded-form {
            #choices {
                flex-direction: column !important;
                gap: 2rem !important;
                .choice {
                    flex: 1;
                    gap: 1rem;
                }
            }

            #submit-button {
                width: 100%;
            }
        }
    }
</style>
