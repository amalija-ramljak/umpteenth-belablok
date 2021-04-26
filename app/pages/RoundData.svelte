<script>
  import * as constants from '../constants/roundData.const';
  import { isOpenRoundEntry } from '../stores/roundData.store';

  export let newId = 0
  export let onAddRound = () => {}

  let isOpen;

  const unsubscribe = isOpenRoundEntry.subscribe(value => {
    isOpen = value;
  });

  const cleanTotal = 162
  const stiglja = 90
  const zvanjaVal = [0, 20, 40, 50, 100, 150, 200]

  let callers = 'mi'
  let bela = ''
  
  let roundMI = ''
  let roundMIZvanja = ''
  let roundVI = ''
  let roundVIZvanja = ''

  $: roundMIError = !isValidMV(roundMI)
  $: roundVIError = !isValidMV(roundVI)
  $: roundVIZvanjaError = !isValidZvanje(roundVIZvanja, roundMIZvanja)
  $: roundMIZvanjaError = !isValidZvanje(roundMIZvanja, roundVIZvanja)

  function isValidMV(val) {
    val = Number(val)
    return val >= 0 && val <= cleanTotal
  }

  function isValidZvanje(val, other) {
    let numVal = Number(val)
    let numOther = Number(other)
    if(numVal === 0) {
      return true
    }
    if (numOther > 0) {
      return false
    }
    for(let zvanje of zvanjaVal) {
      if(numVal === zvanje) {
        return true
      }
      for(let zvanje2 of zvanjaVal) {
        if((zvanje === 200 || zvanje === 150) && zvanje === zvanje2) {
          continue
        }
        if (numVal === zvanje + zvanje2) {
          return true
        }
      }
    }
    return false
  }

  function changePoints(args) {
    if (isValidMV(args.object.text)) {
      switch (args.object.id) {
        case 'mi':
          roundVI = cleanTotal - Number(args.object.text)
          break

        case 'vi':
          roundMI = cleanTotal - Number(args.object.text)
          break
      }
    }
  }

  function changeCallers(args) {
    callers = args.object.items[args.object.selectedIndex].toLowerCase()
  }

  function changeBela(args) {
    let newBela = args.object.id.split('-')[1]
    bela = newBela === bela ? '' : newBela
  }

  function addRound() {
    if (
      (!roundMI && !roundVI) ||
      roundMIZvanjaError ||
      roundMIError ||
      roundVIError ||
      roundVIZvanjaError ||
      !callers
    ) {
      return
    }
    if(roundMI === '') {
      roundVI = Number(roundVI)
      roundMI = cleanTotal - roundVI
    }
    if (roundVI === '') {
      roundMI = Number(roundMI)
      roundVI = cleanTotal - roundMI
    }

    roundMIZvanja = Number(roundMIZvanja)
    roundVIZvanja = Number(roundVIZvanja)

    
    let totalGame = cleanTotal + (roundMIZvanja || 0) + (roundVIZvanja || 0) + (bela ? 20 : 0)
    let absoluteWin = totalGame + stiglja

    let newRound = {
      id: newId,
      mi: roundMI + (bela === 'mi' ? 20 : 0) + (roundMIZvanja || 0),
      vi: roundVI + (bela === 'vi' ? 20 : 0) + (roundVIZvanja || 0),
      raw: {
        mi: {
          bodovi: roundMI,
          zvanja: roundMIZvanja
        },
        vi: {
          bodovi: roundVI,
          zvanja: roundVIZvanja
        },
        bela: bela,
        callers: callers,
      }
    }


    if (!roundMI || (callers === 'mi' && newRound.mi < totalGame / 2)) {
      newRound.vi = roundMI ? totalGame : absoluteWin
      newRound.mi = 0
    } else if (!roundVI || (callers === 'vi' && newRound.vi < totalGame / 2)) {
      newRound.mi = roundVI ? totalGame : absoluteWin
      newRound.vi = 0
    }


    if(!newRound.mi) newRound.mi = "-"
    if(!newRound.vi) newRound.vi = "-"

    onAddRound(newRound)
    isOpenRoundEntry.set(false)
    emptyRound()
  }

  function emptyRound() {
    roundMI = ''
    roundMIZvanja = ''
    roundVI = ''
    roundVIZvanja = ''
    bela = ''
  }

  function cancelEntry() {
    emptyRound()
    isOpenRoundEntry.set(false)
  }
</script>

<style>
  .overlay {
    justify-content: center;
    align-items: center;
    z-index: 10;
    background-color: rgba(0,0,0,0.5);
  }

  .body-grid {
    height: 90%;
    width: 90%;
    border-radius: 5;
    background-color: #808080;
  }
  .deleteButton {
    height: 40;
    width: 50;
  }

  .error {
    border-color: red;
    color: red;
  }
  .checkbox {
    height: 30;
    width: 25;
    margin: 0 10 0 0;
  }
  .checkbox-item {
    height: 30;
    align-items: center;
  }
  .checkbox-item > label {
      height: 25;
  }
</style>

{#if isOpen}
<flexboxLayout class="overlay" width="100%" height="100%">
  <gridLayout class="body-grid" rows="125,*,70">
    <flexboxLayout class="justify-even" row="0">
      <flexboxLayout class="justify-start flex-col">
        <label>Zvali:</label>
        <listPicker
          items={['MI', 'VI']}
          selectedIndex="0"
          height="70"
          on:selectedIndexChange={changeCallers} />
      </flexboxLayout>
      <flexboxLayout class="flex-col justify-start">
        <label>Bela:</label>
        {#each ['mi', 'vi'] as team}
        <stackLayout orientation="horizontal" class="checkbox-item" id={`bela-${team}`} on:tap={changeBela}>
          <button id={`bbela-${team}`} class={'checkbox ' + (bela === team ? '-primary' : '-outline-btn')} on:tap={changeBela} />
          <label>{team.toUpperCase()}</label>
        </stackLayout>
        {/each}
      </flexboxLayout>
    </flexboxLayout>
    <gridLayout row="1" columns="*,*">
      {#each constants.config_pointsGrid as team}
      <gridLayout class={team.class} col={team.col} rows="30,50,50">
        <label row="0">{team.id.toUpperCase()}</label>
        <textField
          row="1"
          hint="Bodovi"
          id={team.id}
          editable="true"
          class={(team.id === 'vi' && roundVIError || team.id === 'mi' && roundMIError) ? 'error' : ''}
          keyboardType="number"
          bind:text={team.id === 'mi' ? roundMI : roundVI}
          on:blur={changePoints} />
          <textField
          row="2"
          hint="Zvanja"
          editable="true"
          keyboardType="number"
          class={(team.id === 'vi' && roundVIZvanjaError || team.id === 'mi' && roundMIZvanjaError) ? 'error' : ''}
          bind:text={team.id === 'mi' ? roundMIZvanja : roundVIZvanja} />
      </gridLayout>
      {/each}
    </gridLayout>
    <gridLayout row="2" columns="*,*">
      <button col="0" text="Odustani" class="-primary" on:tap={cancelEntry} />
      <button
        col="1"
        text="Unesi"
        class="-primary"
        disabled={roundMIError || roundMIZvanjaError || roundVIError || roundVIZvanjaError}
        on:tap={addRound} />
    </gridLayout>
  </gridLayout>
</flexboxLayout>
{/if}