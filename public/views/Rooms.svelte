<script>
  import { onMount } from 'svelte';
  import Fab from '../components/Fab.svelte';
  import Modal from '../components/Modal.svelte';
  import { navigate } from 'svelte-routing';

  let data = JSON.parse(localStorage.getItem('blaze'));
  let rooms = data.rooms;
  let newRoomModal = {
    roomName: '',
    isOpen: false
  };
  let onLine = navigator.onLine;

  function removeRoom(room) {
    rooms = rooms.filter(roomName => roomName !== room);

    data = {
      ...data,
      rooms
    };

    localStorage.setItem('blaze', JSON.stringify(data));
  }

  function joinRoom() {
    const room = newRoomModal.roomName.toLowerCase();

    // Close the modal
    newRoomModal.isOpen = false;
    // Send the user to that room
    navigate(`/app/t/${room}`);
  }

  /**
   * Update the state when user goes online or offline
   */
  function networkStatus() {
    onLine = navigator.onLine;
  }

  onMount(() => {
    /**
     * Show the new room modal when there are no recent rooms
     */
    if (!rooms.length)
      newRoomModal.isOpen = true;

  });

</script>

<svelte:window on:online={networkStatus} on:offline={networkStatus} />
<div id="app">

  <header>
    <h1 class="title">Salas recientes</h1>
  </header>

  <main>

    {#if onLine}
      <ul class="recent-rooms">
        {#if rooms && rooms.length}
          {#each rooms as room}
            <li
              role="link"
              on:click={() => navigate(`/app/t/${room}`)}
              tabindex="0"
            >
              <div>{room}</div>
              <button
                class="thin icon icon-cancel"
                aria-label="Remove room"
                on:click|stopPropagation={() => removeRoom(room)}
              />
            </li>
          {/each}
        {/if}
      </ul>
      <Fab
        icon="icon-add"
        text="Nueva sala"
        on:click={() => newRoomModal.isOpen = true}
      />
    {:else}
      <div class="message">Connect to the internet to start sharing files</div>
    {/if}
  
  </main>

</div>

<!-- Modal to allow users to join a new room -->
<Modal isOpen={newRoomModal.isOpen}>
  <form on:submit|preventDefault={joinRoom} class="join-room">
  
    <input
      type="text"
      maxlength="10"
      required
      placeholder="Nombre de Sala"
      pattern="[A-Za-z0-9]+"
      bind:value={newRoomModal.roomName}
      style="margin-top: 0"
    >
    <button type="submit" class="wide">
      Entrar en sala
    </button>
  </form>
</Modal>
