<script>
    import { isOpenRoundDetails } from '../stores/roundData.store'

    export let details = {}

    let isOpen = false;
    const unsubscribe = isOpenRoundDetails.subscribe(value => {
        isOpen = value;
    });

    function closeDetails() {
        isOpenRoundDetails.set(false)
    }
</script>

<style>
    .overlay {
        width: 100%;
        height: 100%;
        z-index: 5;
        justify-content: center;
        align-items: center;
        background-color: rgba(0,0,0,0.5);
    }

    .body-grid {
        width: 90%;
        height: 90%;
        background-color: rgb(128,128,128);
    }

    .subtitle {
        font-size: 20;
    }
</style>

{#if isOpen}
<flexboxLayout class="overlay" width="100%" height="100%">
    <gridLayout class="body-grid" rows="125,*,70">
        <gridLayout row="0" columns="*,*">
            <gridLayout col="0" rows="50,50">
                <label row="0" class="subtitle">Zvali</label>
                <label row="1">{details.callers.toUpperCase()}</label>
            </gridLayout>
            {#if details.bela}
            <gridLayout col="1" rows="50,50">
                <label row="0" class="subtitle">Bela</label>
                <label row="1">{details.bela.toUpperCase()}</label>
            </gridLayout>
            {/if}
        </gridLayout>
        <gridLayout row="1" columns="*,*,*">
            <gridLayout col="0" rows="50,50,50">
                <label row="1">Bodovi</label>
                <label row="2">Zvanja</label>
            </gridLayout>
            {#each ['mi', 'vi'] as team, idx}
            <gridLayout col={idx+1} rows="50,50,50">
                <label row="0">{team.toUpperCase()}</label>
                <label row="1">{details[team].bodovi}</label>
                <label row="2">{details[team].zvanja}</label>
            </gridLayout>
            {/each}
        </gridLayout>
        <button row="2" text="Natrag" class="-primary" on:tap={closeDetails} />
    </gridLayout>
</flexboxLayout>
{/if}