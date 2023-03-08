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
        const res = await fetch("https://rz.gymhaan.de/q2/json/wahlen.php", {
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
                "https://rz.gymhaan.de/q2/json/wahlen.php",
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
    <div id="form">
        {#if error}
            <div id="error">Joa nee eher nicht bro</div>
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
                        <p>
                            Erstwahl: {data["erstwahl"]}
                            {emojis[data["erstwahl"]]}
                        </p>
                    {/if}
                    {#if data["zweitwahl"]}
                        <p>
                            Zweitwahl: {data["zweitwahl"]}
                            {emojis[data["zweitwahl"]]}
                        </p>
                    {/if}
                </div>
                <h2>Erstwahl</h2>
                <div id="subject-choices">
                    {#each data["subjects"] as subject}
                        <button
                            class="subject"
                            on:click={() => {
                                primarySelection = subject["fach"];
                                if (secondarySelection === primarySelection) {
                                    secondarySelection = null;
                                }
                            }}
                            class:selected={primarySelection ===
                                subject["fach"]}
                        >
                            <div class="subject-icon">
                                {emojis[subject["fach"]] || "?"}
                            </div>
                            <div class="subject-name">{subject["fach"]}</div>
                        </button>
                    {/each}
                </div>
                <h2>Zweitwahl</h2>
                <div id="subject-choices">
                    {#each data["subjects"] as subject}
                        <button
                            class="subject"
                            on:click={() => {
                                secondarySelection = subject["fach"];
                                if (secondarySelection === primarySelection) {
                                    primarySelection = null;
                                }
                            }}
                            class:selected={secondarySelection ===
                                subject["fach"]}
                        >
                            <div class="subject-icon">
                                {emojis[subject["fach"]] || "?"}
                            </div>
                            <div class="subject-name">{subject["fach"]}</div>
                        </button>
                    {/each}
                </div>
                <button
                    id="selection-step-advance"
                    disabled={primarySelection === null ||
                        secondarySelection === null}
                    on:click={() => {
                        if (
                            primarySelection !== null &&
                            secondarySelection !== null
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
                <input name="id" type="number" placeholder="1264" bind:value={id} />
                <label for="birthDate">Geburtsdatum</label>
                <input name="birthDate" type="date" placeholder="13.12.2007" bind:value={birthDate} />
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
    main {
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 100vh;
        padding: 2rem;
        background-color: #e2e8f0;

        color: #1e293b;
        font-size: 18px;
    }

    #form {
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
            margin: 0;
        }

        h3 {
            font-weight: 400;
            margin: 0;
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
        grid-template-columns: 50% 50%;
        gap: 1rem;
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
    #selection-step-advance {
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
    #selection-step-advance:disabled {
        background-color: #f1f5f9;
        cursor: not-allowed;
    }

    #former-choices {
        p {
            margin: 0;
        }
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
    }
</style>
