<script>
  import ActionBar from '../components/ActionBar.svelte'

  import { navigate } from 'svelte-native'

  import { Template } from 'svelte-native/components'
  import RoundData from './RoundData.svelte'
  import App from '~/App.svelte'

  export let players = []
  export let shufflerIdx = 0
  export let game = []

  $: MIPoints =
    game.length > 0
      ? game.map((e) => Number(e.mi) || 0).reduce((s, m) => s + m)
      : 0
  $: VIPoints =
    game.length > 0
      ? game.map((e) => Number(e.vi) || 0).reduce((s, m) => s + m)
      : 0
  $: (async () => {
    if (
      (MIPoints > 1000 && MIPoints > VIPoints) ||
      (VIPoints > 1000 && VIPoints > MIPoints)
    ) {
      let message = 'Igra je gotova!\n\nPobjednici: '
      if (MIPoints > VIPoints) {
        message += 'MI'
      } else {
        message += 'VI'
      }
      await alert({
        title: 'Gotovo!',
        message: message,
        okButtonText: 'Natrag na početak',
      }).then(() => {
        navigate({
          page: App,
        })
      })
    }
  })()

  function onDeleteRound(args) {
    let delround = game[args.object.id]
    MIPoints -= delround.mi
    VIPoints -= delround.vi
    game = game.filter((item) => item.idx !== args.object.id)
    shufflerIdx = game.length % 4
    players = players
    console.log(shufflerIdx)
  }

  function enterRound(args) {
    navigate({
      page: RoundData,
      props: {
        game: game,
        players: players,
      },
    })
  }
</script>

<style>
  .delete-btn {
    height: 25;
    width: 25;
    margin-bottom: 5;
    margin-right: 15;
    font-size: 15;
    padding-top: 2;
    color: white;
    background-color: maroon;
    border-radius: 2;
  }

  .round-item {
    padding-top: 0;
    padding-bottom: 0;
    padding-left: 0;
    padding-right: 0;

    margin-left: 0;
    margin-right: 0;
    margin-bottom: 0;
    margin-top: 0;
  }

  listView {
    separator-color: transparent;
  }
</style>

<page>
  <ActionBar />
  <gridLayout class="body-grid" rows="50,50,*,70">
    <label row="0" class="text-center">Dijeli: {players[shufflerIdx]}</label>
    <listView
      row="1"
      items={[[`MI (${MIPoints})`, `VI (${VIPoints})`, 'Pobriši']]}>
      <Template let:item>
        <gridLayout row="1" columns="*,*,100">
          <label col="0" text={item[0]} />
          <label col="1" text={item[1]} />
          <label class="text-right" col="2" text={item[2]} />
        </gridLayout>
      </Template>
    </listView>
    <listView row="2" items={game}>
      <Template let:item>
        <gridLayout class="round-item -separator" height="30" columns="*,*,100">
          <label col="0" text={item.mi} />
          <label col="1" text={item.vi} />
          <flexboxLayout col="2" class="justify-end">
            <label
              class="delete-btn text-center"
              id={item.idx}
              on:tap={onDeleteRound}>
              X
            </label>
          </flexboxLayout>
        </gridLayout>
      </Template>
    </listView>

    <button row="3" class="-primary" on:tap={enterRound}>Unesi bodove</button>
  </gridLayout>
</page>
