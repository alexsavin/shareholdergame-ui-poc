<script>
    import RoundItem from './RoundItem.svelte'
    import RoundListHeader from './RoundListHeader.svelte'

    export let rounds = []
</script>

<RoundListHeader/>
<div class="overflow-y mb-3">
    {#each rounds as round}
        {#if round.round !== 0}
            <RoundItem round="{round}"/>
        {/if}
    {/each}
</div>
