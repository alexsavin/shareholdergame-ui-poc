<script>
    import { SHARES } from '../scripts/gameDescription'
    import PriceScale from './PriceScale.svelte'
    import PlayerNameLink from './PlayerNameLink.svelte'

    export let currentPosition = {}

    export function updateCurrentPosition(cp) {
        currentPosition = cp
    }
</script>

<table class="w-100">
    <tr>
        <th class="text-center bg-light" colspan="2">Price</th>
        {#if currentPosition.hasOwnProperty('playerPositions')}
            {#each Object.entries(currentPosition.playerPositions) as [turnOrder, playerPosition]}
                <th class="text-center bgcolor-turn-{turnOrder}" colspan="2">
                    <PlayerNameLink name="{playerPosition.name}"/>
                </th>
            {/each}
        {/if}
    </tr>
    {#if currentPosition.hasOwnProperty('sharePrices')}
        {#each SHARES as shareId}
            <tr>
                <td class="w-1 bg-light text-right pr-1">
                    {currentPosition.sharePrices[shareId].price}
                </td>
                <td class="w-50 bg-light">
                    <PriceScale shareId={shareId} currentPosition="{currentPosition}"/>
                </td>
                {#if currentPosition.hasOwnProperty('playerPositions')}
                    {#each Object.entries(currentPosition.playerPositions) as [turnOrder, playerPosition]}
                        <td class="text-right color-turn-{turnOrder} bgcolor-turn-{turnOrder}">
                            {playerPosition.shares[shareId].amount}
                        </td>
                        <td class="text-center color-turn-{turnOrder} bgcolor-turn-{turnOrder}">
                            <span class="text-black-50"><small><em>{playerPosition.shares[shareId].total}</em></small></span>
                        </td>
                    {/each}
                {/if}
            </tr>
        {/each}
    {/if}
    <tr>
        <td class="text-right bg-light pr-4" colspan="2">
            Cash
        </td>
        {#if currentPosition.hasOwnProperty('playerPositions')}
            {#each Object.entries(currentPosition.playerPositions) as [turnOrder, playerPosition]}
                <td class="text-center color-turn-{turnOrder} bgcolor-turn-{turnOrder}" colspan="2">
                    <em>{playerPosition.cash}</em>
                </td>
            {/each}
        {/if}
    </tr>
    <tr>
        <td class="text-right pr-4 bg-light" colspan="2">
            Total
        </td>
        {#if currentPosition.hasOwnProperty('playerPositions')}
            {#each Object.entries(currentPosition.playerPositions) as [turnOrder, playerPosition]}
                <td class="text-center color-turn-{turnOrder} bgcolor-turn-{turnOrder}" colspan="2">
                    {playerPosition.total}
                </td>
            {/each}
        {/if}
    </tr>
</table>
