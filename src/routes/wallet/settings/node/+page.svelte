<script>
  import NodeSelector from '$lib/components/NodeSelector.svelte';
  import { node } from '$lib/stores/node.js';
  import { fade } from 'svelte/transition';
  import Button from '$lib/components/buttons/Button.svelte';
  import toast from 'svelte-french-toast';

  let showSelector = false;

  const changeNode = async (e) => {
    const change = await window.api.changeNode(e.detail.node);
    if (!change) {
      toast.error('Cannot connect to node.', {
        position: 'top-right',
        style:
          'border-radius: 5px; background: var(--toast-bg-color); border: 1px solid var(--toast-b-color); color: var(--toast-text-color);',
      });
      $node.loading = false;
    }
    $node.loading = false;
    $node.selectedNode = change;
    showSelector = false;
  };

  let progress;
  $: progress = Number((($node.walletBlockCount / $node.networkBlockCount) * 100).toFixed(2));
  $: if ($node.networkBlockCount === 0) {
    $node.nodeStatus = 'Node offline';
  }
  $: progressCheck = Number.isFinite(progress);
</script>

<div class="wrapper">
  <div class="gird">
    <div>
      <h4>Node</h4>
      <h3>{$node.selectedNode.url ?? ''}</h3>
    </div>
    <div>
      <h4>Wallet height</h4>
      <h3>{$node.walletBlockCount ? $node.walletBlockCount : '-'}</h3>
    </div>
    <div>
      <h4>Status</h4>
      <h3>{$node.nodeStatus}</h3>
    </div>
    <div>
      <h4>Status</h4>
      <h3>{progressCheck ? `${progress === '100.00' ? '100%' : `${progress}%`}` : 'Node offline'}</h3>
    </div>
    <div>
      <Button text="Change node" on:click={() => (showSelector = !showSelector)} />
    </div>
  </div>
</div>

{#if showSelector}
  <div class="overlay" out:fade>
    <NodeSelector on:connect={(e) => changeNode(e)} />
  </div>
{/if}

<style lang="scss">
  .wrapper {
    width: 100%;
    height: 100%;
    padding: 30px;
  }

  .gird {
    display: grid;
    grid-template-columns: repeat(2, minmax(0, 1fr));
    grid-gap: 2rem;

    div {
      display: flex;
      flex-direction: column;
      gap: 0.5rem;
    }

    h4 {
      opacity: 60%;
    }
  }

  .overlay {
    display: flex;
    justify-content: center;
    align-items: center;
    background-color: var(--backgound-color);
    z-index: 9;
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    border-radius: 15px;
  }
</style>
