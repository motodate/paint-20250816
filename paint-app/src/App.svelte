<script>
  import Toolbar from './components/Toolbar.svelte';
  import Canvas from './components/Canvas.svelte';
  
  let currentTool = $state('pen');
  let currentColor = $state('#000000');
  let penSize = $state(4);
  let eraserSize = $state(20);
  let showToolbar = $state(true);
  let canvasComponent = $state(null);
  
  function handleClear() {
    if (canvasComponent) {
      canvasComponent.clearCanvas();
    }
  }
  
  function handleDownload() {
    if (canvasComponent) {
      canvasComponent.downloadImage();
    }
  }
  
  function handleKeydown(event) {
    const key = event.key.toLowerCase();
    const isCtrlOrCmd = event.ctrlKey || event.metaKey;
    
    if (isCtrlOrCmd && key === 's') {
      event.preventDefault();
      handleDownload();
    } else if (key === 'p' && !isCtrlOrCmd) {
      currentTool = 'pen';
    } else if (key === 'e' && !isCtrlOrCmd) {
      currentTool = 'eraser';
    } else if (key === 'c' && !isCtrlOrCmd) {
      handleClear();
    } else if (key === 't' && !isCtrlOrCmd) {
      showToolbar = !showToolbar;
    }
  }
</script>

<svelte:window on:keydown={handleKeydown} />

<div class="app" class:toolbar-hidden={!showToolbar}>
  {#if showToolbar}
    <Toolbar
      bind:currentTool
      bind:currentColor
      bind:penSize
      bind:eraserSize
      onClear={handleClear}
      onDownload={handleDownload}
    />
  {/if}
  
  <div class="canvas-container">
    <Canvas
      bind:this={canvasComponent}
      {currentTool}
      {currentColor}
      currentSize={currentTool === 'pen' ? penSize : eraserSize}
    />
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    padding: 0;
    overflow: hidden;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
  }
  
  .app {
    display: flex;
    height: 100vh;
    width: 100vw;
    background-color: #f5f5f5;
  }
  
  .canvas-container {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
    overflow: hidden;
  }
  
  .toolbar-hidden .canvas-container {
    padding-left: 20px;
  }
</style>