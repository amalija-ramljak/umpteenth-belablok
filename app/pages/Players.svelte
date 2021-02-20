<script>
  import ActionBar from '../components/ActionBar.svelte'

  import { navigate } from 'svelte-native'
  import Game from './Game.svelte'
  import Title from '~/components/Title.svelte'

  let players = ['Jedan', 'Dva', 'Tri', 'Četiri']

  function addPlayer(args) {
    players[Number(args.object.id)] = args.object.text
  }

  function startGame(args) {
    navigate({
      page: Game,
      props: {
        players: players,
      },
    })
  }
</script>

<page>
  <ActionBar />
  <gridLayout class="body-grid" rows="70,*,70">
    <Title row="0" text="Unesi imena igrača" />
    <gridLayout row="1" columns="*,*">
      <!-- MI -->
      <gridLayout col="0" class="left-col" rows="30,50,50">
        <label row="0">MI</label>
        <textField
          id="0"
          row="1"
          hint="Jedan"
          on:textChange={addPlayer} />
          id="2"
        <textField
          row="2"
          hint="Tri"
          on:textChange={addPlayer} />
      </gridLayout>

      <!-- VI -->
      <gridLayout col="1" class="right-col" rows="30,50,50">
        <label row="0">VI</label>
        <textField
          id="1"
          row="1"
          hint="Dva"
          on:textChange={addPlayer} />
        <textField
          id="3"
          row="2"
          hint="Četiri"
          on:textChange={addPlayer} />
      </gridLayout>
    </gridLayout>

    <button row="2" class="-primary" on:tap={startGame}>Započni igru</button>
  </gridLayout>
</page>
