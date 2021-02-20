<script>
  import { navigate } from 'svelte-native'
  import ActionBar from '../components/ActionBar.svelte'

  import Game from './Game.svelte'

  export let game = []
  export let players = []

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
      idx: game.length,
      mi: roundMI + (bela === 'mi' ? 20 : 0) + (roundMIZvanja || 0),
      vi: roundVI + (bela === 'vi' ? 20 : 0) + (roundVIZvanja || 0),
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

    emptyRound()
    navigate({
      page: Game,
      props: {
        shufflerIdx: (game.length + 1) % 4,
        game: [...game, newRound],
        players: players,
      },
    })
  }

  function emptyRound() {
    roundMI = ''
    roundMIZvanja = ''
    roundVI = ''
    roundVIZvanja = ''
  }

  function cancelEntry() {
    emptyRound()
    navigate({
      page: Game,
      props: {
        shufflerIdx: game.length % 4,
        game: game,
        players: players,
      },
    })
  }
</script>

<style>
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

<page>
  <ActionBar />
  <gridLayout class="body-grid" rows="125,*,70">
    <!-- <CheckBox checked="false" text="nesto" />
            <CheckBox text="CheckBox Label" checked="false" /> -->
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
        <stackLayout orientation="horizontal" class="checkbox-item" id="bela-mi" on:tap={changeBela}>
          <button id="bbela-mi" class={'checkbox ' + (bela === 'mi' ? '-primary' : '-outline-btn')} on:tap={changeBela} />
          <label>MI</label>
        </stackLayout>
        <stackLayout orientation="horizontal" class="checkbox-item" id="bela-vi" on:tap={changeBela}>
          <button id="bbela-vi" class={'checkbox ' + (bela === 'vi' ? '-primary' : '-outline-btn')} on:tap={changeBela} />
          <label>VI</label>
        </stackLayout>
      </flexboxLayout>
    </flexboxLayout>
    <gridLayout row="1" columns="*,*">
      <!-- MI -->
      <gridLayout class="left-col" col="0" rows="30,50,50">
        <label row="0">MI</label>
        <textField
          row="1"
          hint="Bodovi"
          id="mi"
          editable="true"
          class={roundMIError ? 'error' : ''}
          bind:text={roundMI}
          on:blur={changePoints} />
        <textField
          row="2"
          hint="Zvanja"
          editable="true"
          class={roundMIZvanjaError ? 'error' : ''}
          bind:text={roundMIZvanja} />
      </gridLayout>
      <gridLayout class="right-col" col="1" rows="30,50,50">
        <label row="0">VI</label>
        <textField
          row="1"
          hint="Bodovi"
          id="vi"
          editable="true"
          class={roundVIError ? 'error' : ''}
          bind:text={roundVI}
          on:blur={changePoints} />
        <textField
          row="2"
          hint="Zvanja"
          editable="true"
          class={roundVIZvanjaError ? 'error' : ''}
          bind:text={roundVIZvanja} />
      </gridLayout>
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
</page>
