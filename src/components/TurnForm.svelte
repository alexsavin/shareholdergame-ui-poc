<script>
    import {createEventDispatcher} from 'svelte';
    import {StepType} from '../scripts/constants'
    import BuySellForm from './BuySellForm.svelte'
    import ApplyCardForm from './ApplyCardForm.svelte'
    import {SHARES} from "../scripts/gameDescription";
    import BuySellStepItem from "./BuySellStepItem.svelte";
    import PriceChangeStepItem from "./PriceChangeStepItem.svelte";
    import CardItem from './CardItem.svelte'
    import {applyCard, calculateCanBuy, resetPrices, recalculateTotal} from "../scripts/playGamePageMediator";

    const dispatch = createEventDispatcher();

    export let currentPosition = {}
    export let currentStep = StepType.FIRST_BUY_SELL_STEP
    export let roundsNumber = 0

    function buildTurn() {
        return {
            firstBuySellStep: buildBuySellStep(),
            cardStep: buildCardStep(),
            lastBuySellStep: buildBuySellStep()
        }
    }

    function buildCardStep() {
        return {
            playerCardId: 0,
            operations: []
        }
    }

    function buildBuySellStep() {
        let step = {}
        for (const shareId of SHARES) {
            step[shareId] = 0
        }
        return step
    }

    function buildShares() {
        let shares = {}
        for (const shareId of SHARES) {
            shares[shareId] = {
                amount: 0
            }
        }
        return shares
    }

    function buildFirstStep() {
        return {
            shares: buildShares(),
            cash: 0
        }
    }

    function buildSharePrices() {
        let sharePrices = {}
        for (const shareId of SHARES) {
            sharePrices[shareId] = {
                price: 0
            }
        }
        return sharePrices
    }

    function buildPriceStep() {
        return {
            sharePrices: buildSharePrices(),
            cash: 0
        }
    }

    let turn = buildTurn()
    let cardId = 0
    let firstStep = buildFirstStep()
    let priceStep = buildPriceStep()

    let firstBuySellForm
    let lastBuySellForm
    let resetFirstBuySellStepBtn
    let resetLastBuySellStepBtn
    let doTurnBtn

    function isLastTurn() {
        return currentPosition.currentRound === roundsNumber
    }

    function copySharesAmount(shares, shares1) {
        for (const shareId of SHARES) {
            shares[shareId].amount = shares1[shareId].amount
        }
    }

    function copySharePrices(sharePrices, sharePrices1) {
        for (const shareId of SHARES) {
            sharePrices[shareId].price = sharePrices1[shareId].price
        }
    }

    function onApplyCard(event) {
        if (!isLastTurn()) {
            copySharesAmount(firstStep.shares, currentPosition.playerPositions[currentPosition.myTurnOrder].shares)
            firstStep.cash = currentPosition.playerPositions[currentPosition.myTurnOrder].cash
            lockShares(currentPosition.playerPositions[currentPosition.myTurnOrder].shares)
            resetFirstBuySellStepBtn.disabled = true
        }

        applyCard(event.detail.cardId, event.detail.operations, currentPosition);

        turn.cardStep.playerCardId = event.detail.playerCardId
        for (const shareId in event.detail.operations) {
            if (event.detail.operations.hasOwnProperty(shareId)) {
                turn.cardStep.operations.push({
                    priceOperationId: event.detail.operations[shareId].priceOperationId,
                    shareId: parseInt(shareId)
                })
            }
        }
        cardId = event.detail.cardId
        copySharePrices(priceStep.sharePrices, currentPosition.sharePrices)
        priceStep.cash = currentPosition.playerPositions[currentPosition.myTurnOrder].cash
        recalculateTotal(currentPosition)

        currentStep = StepType.LAST_BUY_SELL_STEP
        doTurnBtn.disabled = false

        dispatch('cardapplied', event.detail)
    }

    function lockShares(shares) {
        for (const shareId of SHARES) {
            if (shares.hasOwnProperty(shareId)) {
                if (shares[shareId].amount > shares[shareId].initialAmount) {
                    shares[shareId].lockedAmount = shares[shareId].amount - shares[shareId].initialAmount
                }
            }
        }
    }

    function resetLockedShares(shares) {
        for (const shareId of SHARES) {
            if (shares.hasOwnProperty(shareId)) {
                shares[shareId].lockedAmount = 0
            }
        }
    }

    function resetCard(event) {
        resetPrices(currentPosition)

        if (!isLastTurn()) {
            copySharesAmount(currentPosition.playerPositions[currentPosition.myTurnOrder].shares, firstStep.shares)
            resetLockedShares(currentPosition.playerPositions[currentPosition.myTurnOrder].shares)
            resetFirstBuySellStepBtn.disabled = false
        }
        currentPosition.playerPositions[currentPosition.myTurnOrder].cash = isLastTurn()
            ? currentPosition.playerPositions[currentPosition.myTurnOrder].initialCash : firstStep.cash

        for (const turnOrder in currentPosition.playerPositions) {
            if (currentPosition.playerPositions.hasOwnProperty(turnOrder)
                && currentPosition.playerPositions[turnOrder].turnOrder !== currentPosition.myTurnOrder) {
                for (const shareId of SHARES) {
                    if (currentPosition.playerPositions[turnOrder].shares[shareId].repurchased) {
                        currentPosition.playerPositions[turnOrder].shares[shareId].repurchased = false
                        currentPosition.playerPositions[turnOrder].shares[shareId].amount =
                            currentPosition.playerPositions[turnOrder].shares[shareId].initialAmount
                    }
                }
                currentPosition.playerPositions[turnOrder].cash =
                    currentPosition.playerPositions[turnOrder].initialCash
            }
        }

        turn.cardStep = buildCardStep()
        turn.lastBuySellStep = buildBuySellStep()
        if (!isLastTurn()) {
            lastBuySellForm.reset()
        }
        recalculateTotal(currentPosition)

        currentStep = StepType.FIRST_BUY_SELL_STEP
        doTurnBtn.disabled = true

        dispatch('resetcard')
    }

    function firstBuySell(event) {
        let buySellData = event.detail
        turn.firstBuySellStep[buySellData.shareId] =
            turn.firstBuySellStep[buySellData.shareId] + buySellData.buySellAmount

        resetFirstBuySellStepBtn.disabled = false

        dispatch('buysell')
    }

    function lastBuySell(event) {
        let buySellData = event.detail
        turn.lastBuySellStep[buySellData.shareId] =
            turn.lastBuySellStep[buySellData.shareId] + buySellData.buySellAmount

        resetLastBuySellStepBtn.disabled = false
        dispatch('buysell')
    }

    function resetBuySell(event) {
        let sharePrices = currentPosition.sharePrices
        for (const shareId of SHARES) {
            currentPosition.playerPositions[currentPosition.myTurnOrder].shares[shareId].amount =
                currentStep === StepType.FIRST_BUY_SELL_STEP
                    ? currentPosition.playerPositions[currentPosition.myTurnOrder].shares[shareId].initialAmount
                    : firstStep.shares[shareId].amount
        }
        currentPosition.playerPositions[currentPosition.myTurnOrder].cash =
            currentStep === StepType.FIRST_BUY_SELL_STEP
                ? currentPosition.playerPositions[currentPosition.myTurnOrder].initialCash
                : priceStep.cash
        for (const shareId of SHARES) {
            currentPosition.playerPositions[currentPosition.myTurnOrder].shares[shareId].canBuy =
                calculateCanBuy(currentPosition.playerPositions[currentPosition.myTurnOrder].cash, sharePrices[shareId].price)
            currentPosition.playerPositions[currentPosition.myTurnOrder].shares[shareId].total =
                currentPosition.playerPositions[currentPosition.myTurnOrder].shares[shareId].amount * sharePrices[shareId].price
        }
        if (currentStep === StepType.FIRST_BUY_SELL_STEP) {
            firstBuySellForm.reset()
            turn.firstBuySellStep = buildBuySellStep()
            resetFirstBuySellStepBtn.disabled = true
        } else if (currentStep === StepType.LAST_BUY_SELL_STEP) {
            lastBuySellForm.reset()
            turn.lastBuySellStep = buildBuySellStep()
            resetLastBuySellStepBtn.disabled = true
        }
        dispatch('reset')
    }

    function isNothingBoughtSolt() {
        let result = true
        let playerPosition = currentPosition.playerPositions[currentPosition.myTurnOrder]
        let shares = playerPosition.shares
        for (const shareId of SHARES) {
            if (shares[shareId].amount !== firstStep.shares[shareId].amount) {
                result = false
                break
            }
        }
        return result
    }

    function doTurn(event) {
        let readyToSendTurn
        if (isNothingBoughtSolt() && !isLastTurn()) {
            readyToSendTurn = confirm('You havent buy/sell anything on the third step. Do you want to continue?')
        } else {
            readyToSendTurn = confirm('Please confirm your turn?')
        }

        if (readyToSendTurn) {
            dispatch('doturn', turn)
        }
    }
</script>

<div class="row">
    <div class="col-md-1 text-center color-turn-{currentPosition.currentTurn} bgcolor-turn-{currentPosition.currentTurn}">
        <strong>{currentPosition.currentRound}.{currentPosition.currentTurn}</strong>
    </div>
    <div class="col-md-3 align-top color-turn-{currentPosition.currentTurn} bgcolor-turn-{currentPosition.currentTurn}">
        {#if !isLastTurn()}
            {#if currentStep === StepType.FIRST_BUY_SELL_STEP && currentPosition.hasOwnProperty('sharePrices')}
                <BuySellForm bind:this={firstBuySellForm}
                             step={StepType.FIRST_BUY_SELL_STEP}
                             sharePrices={currentPosition.sharePrices}
                             playerPosition={currentPosition.playerPositions[currentPosition.myTurnOrder]}
                             on:buysell={firstBuySell}/>
            {:else}
                <BuySellStepItem step="{firstStep}"/>
            {/if}
            <div class="text-center p-2">
                <button bind:this={resetFirstBuySellStepBtn} on:click={resetBuySell} disabled class="btn btn-secondary form-control-sm">Reset</button>
            </div>
        {/if}
    </div>
    <div class="col-md-1 bgcolor-turn-{currentPosition.currentTurn}">
        {#if currentStep === StepType.LAST_BUY_SELL_STEP}
            <CardItem cardId={cardId}/>
        {/if}
    </div>
    <div class="col-md-3 color-turn-{currentPosition.currentTurn} bgcolor-turn-{currentPosition.currentTurn}">
        {#if currentStep === StepType.FIRST_BUY_SELL_STEP && currentPosition.hasOwnProperty('playerPositions')}
            <div id="mobile-only">
                <table class="w-100 mb-1">
                    <tr>
                        {#each SHARES as shareId}
                            <td class="w-20 text-center game-card-cell-color share-color-{shareId}">{currentPosition.sharePrices[shareId].price}</td>
                        {/each}
                        <td class="w-20"></td>
                    </tr>
                </table>
            </div>
            <ApplyCardForm playerCards={currentPosition.playerPositions[currentPosition.myTurnOrder].playerCards}
                           on:cardapplied={onApplyCard}/>
        {:else}
            <PriceChangeStepItem step="{priceStep}"/>
            <div class="text-center p-2">
                <button id="resetCardBtn" on:click={resetCard} class="btn btn-secondary form-control-sm">Reset</button>
            </div>
        {/if}
    </div>
    <div class="col-md-3 color-turn-{currentPosition.currentTurn} bgcolor-turn-{currentPosition.currentTurn} align-top">
        {#if currentStep === StepType.LAST_BUY_SELL_STEP && !isLastTurn() && currentPosition.hasOwnProperty('playerPositions')}
            <div id="mobile-only-2">
                <table class="w-100 mb-1">
                    <tr>
                        {#each SHARES as shareId}
                            <td class="w-20 text-center game-card-cell-color share-color-{shareId}">{currentPosition.sharePrices[shareId].price}</td>
                        {/each}
                        <td class="w-20"></td>
                    </tr>
                </table>
            </div>
            <BuySellForm bind:this={lastBuySellForm}
                         step={StepType.LAST_BUY_SELL_STEP}
                         sharePrices={currentPosition.sharePrices}
                         playerPosition={currentPosition.playerPositions[currentPosition.myTurnOrder]}
                         on:buysell={lastBuySell}/>
            <div class="text-center p-2">
                <button bind:this={resetLastBuySellStepBtn} on:click={resetBuySell} disabled class="btn btn-secondary form-control-sm">Reset</button>
            </div>
        {/if}
    </div>
    <div class="col-md-1 color-turn-{currentPosition.currentTurn} bgcolor-turn-{currentPosition.currentTurn} text-center">
        {#if currentPosition.hasOwnProperty('playerPositions')}
            {currentPosition.playerPositions[currentPosition.myTurnOrder].total}
        {/if}
    </div>
</div>
<div class="row mb-3 bgcolor-turn-{currentPosition.currentTurn}">
    <div class="col-md-10 text-right">
        <button bind:this={doTurnBtn} on:click={doTurn} class="btn btn-primary" disabled>Do Turn</button>
    </div>
</div>