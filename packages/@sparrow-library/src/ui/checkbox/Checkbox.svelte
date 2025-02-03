<script lang="ts">
  export let isActive: boolean = false;
  export let onToggle: () => void;

  function handleKeyPress(event: KeyboardEvent) {
    if (event.key === 'Enter' || event.key === ' ') {
      event.preventDefault();
      console.log('key pressed');
      isActive = !isActive;
      onToggle();
    }
  }
  $: console.log(isActive);
</script>

<div 
  class="toggle-container"
  role="switch"
  aria-checked={isActive}
  tabindex="0"
  on:click={onToggle}
  on:keydown={handleKeyPress}
>
  <div class="toggle-track">
    <div class="toggle-knob" class:active={isActive}></div>
  </div>
</div>

<style>
  .toggle-container {
    display: flex;
    max-width: 38px;
    padding: 4px 0;
    flex-direction: column;
    justify-content: center;
    cursor: pointer;
  }

  .toggle-track {
    border-radius: 16px;
    background-color: rgba(39, 41, 53, 1);
    display: flex;
    width: 100%;
    padding: 2px 16px 2px 2px;
    align-items: center;
    justify-content: flex-start;
    height: 16px;
    transition: background-color 0.2s ease-in-out;
  }

  .toggle-knob {
    box-shadow: 0 2px 4px rgba(0, 35, 11, 0.2);
    align-self: stretch;
    display: flex;
    margin: auto 0;
    width: 12px;
    flex-direction: column;
    overflow: hidden;
    border-radius: 50%;
    background-color: #ffffff;
    height: 12px;
    transition: transform 0.2s ease-in-out;
  }

  .toggle-knob.active {
    transform: translateX(16px);
  }

</style>
