<script>
    import {onMount} from "svelte";

    const licensesEndpoint = "/licenses.json";
    const licenses = [];
    let licensesFetched = false;
    onMount(async () => {
        const response = await fetch(licensesEndpoint);

        const licensesData = await response.json();
        console.log(licensesData);
        Object.entries(licensesData).forEach(([key, value]) => {
            licenses.push({
                name: key,
                licenses: value.licenses,
                publisher: value.publisher,
                repository: value.repository,
            });
        });

        licensesFetched = true;
    });
</script>

<div class="flex gap-3 items-baseline justify-between">
    <h1 class="text-3xl font-bold">Licencje</h1>
    {#if !licensesFetched}
        <span class="loading loading-spinner"/>
    {/if}
</div>

<div class="flex flex-col gap-3 mt-3">
    {#if licensesFetched}
        {#each licenses as license}
            <div class="flex gap-3 justify-between">
                <div class="flex gap-3 items-center">
                    <h2 class="text-xl">{license.name}</h2>
                    <span class="text-slate-600">{license.publisher}</span>
                </div>
                <div class="flex gap-3 items-center">
                    <strong class="text-sm text-slate-500">{license.licenses}</strong>
                    <button
                            class="btn btn-sm"
                            on:click={() => window.open(license.repository)}>Repository
                    </button
                    >
                </div>
            </div>
        {/each}
    {:else}
        {#each {length: 30} as _}
            <div class="flex gap-3 justify-between animate-pulse">
                <div class="flex gap-3 items-center">
                    <div class="rounded-full bg-slate-600 h-6 w-80"/>
                    <div class="rounded-full bg-slate-700 h-4 w-32"/>
                </div>
                <div class="flex gap-2">
                    <div class="rounded-full bg-slate-600 h-4 w-8"/>
                    <button class="btn btn-sm" disabled="disabled">Repository</button>
                </div>
            </div>
        {/each}
    {/if}
</div>
