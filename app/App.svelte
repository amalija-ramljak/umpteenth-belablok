<script>
  import { navigate } from 'svelte-native'
  import Players from './pages/Players.svelte'
  import ActionBar from './components/ActionBar.svelte'
  import Title from './components/Title.svelte'

  function onStartGame() {
    navigate({
      page: Players,
    })
  }
</script>

<page>
  <ActionBar />
  <gridLayout class="body-grid" rows="70,*,70">
    <Title row="0" text="Prošle igre" />
    <label row="1">Stare igre...</label>
    <button row="2" text="Nova igra" on:tap={onStartGame} class="-primary" />
  </gridLayout>
</page>
