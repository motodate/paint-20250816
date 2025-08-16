<script>
  import { onMount } from 'svelte';
  
  let { currentTool, currentColor, currentSize } = $props();
  
  let canvasElement = $state(null);
  let ctx = $state(null);
  let isDrawing = $state(false);
  let lastX = $state(0);
  let lastY = $state(0);
  let dpr = 1;
  
  const CANVAS_WIDTH = 1280;
  const CANVAS_HEIGHT = 720;
  
  onMount(() => {
    if (!canvasElement) return;
    
    ctx = canvasElement.getContext('2d');
    dpr = window.devicePixelRatio || 1;
    
    setupCanvas();
    
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  });
  
  function setupCanvas() {
    if (!canvasElement || !ctx) return;
    
    canvasElement.width = CANVAS_WIDTH * dpr;
    canvasElement.height = CANVAS_HEIGHT * dpr;
    
    canvasElement.style.width = `${CANVAS_WIDTH}px`;
    canvasElement.style.height = `${CANVAS_HEIGHT}px`;
    
    ctx.scale(dpr, dpr);
    
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    ctx.imageSmoothingEnabled = true;
    
    clearCanvas();
  }
  
  function handleResize() {
    const imageData = ctx.getImageData(0, 0, canvasElement.width, canvasElement.height);
    setupCanvas();
    ctx.putImageData(imageData, 0, 0);
  }
  
  function startDrawing(event) {
    isDrawing = true;
    const coords = getCoordinates(event);
    lastX = coords.x;
    lastY = coords.y;
    
    ctx.beginPath();
    ctx.moveTo(lastX, lastY);
  }
  
  function draw(event) {
    if (!isDrawing) return;
    
    const coords = getCoordinates(event);
    
    if (currentTool === 'pen') {
      ctx.globalCompositeOperation = 'source-over';
      ctx.strokeStyle = currentColor;
      ctx.lineWidth = currentSize;
    } else if (currentTool === 'eraser') {
      ctx.globalCompositeOperation = 'destination-out';
      ctx.lineWidth = currentSize;
    }
    
    ctx.beginPath();
    ctx.moveTo(lastX, lastY);
    ctx.lineTo(coords.x, coords.y);
    ctx.stroke();
    
    lastX = coords.x;
    lastY = coords.y;
  }
  
  function stopDrawing() {
    if (!isDrawing) return;
    isDrawing = false;
    ctx.beginPath();
  }
  
  function getCoordinates(event) {
    const rect = canvasElement.getBoundingClientRect();
    let clientX, clientY;
    
    if (event.touches && event.touches.length > 0) {
      clientX = event.touches[0].clientX;
      clientY = event.touches[0].clientY;
    } else if (event.changedTouches && event.changedTouches.length > 0) {
      clientX = event.changedTouches[0].clientX;
      clientY = event.changedTouches[0].clientY;
    } else {
      clientX = event.clientX;
      clientY = event.clientY;
    }
    
    return {
      x: clientX - rect.left,
      y: clientY - rect.top
    };
  }
  
  export function clearCanvas() {
    if (!ctx || !canvasElement) return;
    ctx.clearRect(0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
  }
  
  export function downloadImage() {
    if (!canvasElement) return;
    
    const tempCanvas = document.createElement('canvas');
    const tempCtx = tempCanvas.getContext('2d');
    
    tempCanvas.width = CANVAS_WIDTH;
    tempCanvas.height = CANVAS_HEIGHT;
    
    tempCtx.drawImage(canvasElement, 0, 0, canvasElement.width, canvasElement.height, 0, 0, CANVAS_WIDTH, CANVAS_HEIGHT);
    
    tempCanvas.toBlob((blob) => {
      const url = URL.createObjectURL(blob);
      const a = document.createElement('a');
      const now = new Date();
      const timestamp = now.toISOString().replace(/[-:T]/g, '').slice(0, 14);
      a.download = `paint-${timestamp}.png`;
      a.href = url;
      a.click();
      URL.revokeObjectURL(url);
      
      showToast('画像を保存しました');
    }, 'image/png');
  }
  
  function showToast(message) {
    const toast = document.createElement('div');
    toast.className = 'toast';
    toast.textContent = message;
    toast.style.cssText = `
      position: fixed;
      bottom: 20px;
      right: 20px;
      background: #333;
      color: white;
      padding: 12px 20px;
      border-radius: 8px;
      font-size: 14px;
      z-index: 1000;
      animation: slideIn 0.3s ease;
    `;
    document.body.appendChild(toast);
    
    setTimeout(() => {
      toast.style.animation = 'slideOut 0.3s ease';
      setTimeout(() => toast.remove(), 300);
    }, 2000);
  }
  
  function handlePointerMove(event) {
    const coords = getCoordinates(event);
    updateCursor(coords.x, coords.y);
    draw(event);
  }
  
  function updateCursor(x, y) {
    if (!canvasElement) return;
    canvasElement.style.cursor = 'none';
    
    const cursor = document.getElementById('custom-cursor');
    if (cursor) {
      cursor.style.left = `${x}px`;
      cursor.style.top = `${y}px`;
      cursor.style.width = `${currentSize}px`;
      cursor.style.height = `${currentSize}px`;
      cursor.style.borderColor = currentTool === 'pen' ? currentColor : '#666';
    }
  }
  
  function handlePointerLeave() {
    const cursor = document.getElementById('custom-cursor');
    if (cursor) {
      cursor.style.display = 'none';
    }
  }
  
  function handlePointerEnter() {
    const cursor = document.getElementById('custom-cursor');
    if (cursor) {
      cursor.style.display = 'block';
    }
  }
</script>

<div class="canvas-wrapper">
  <canvas
    bind:this={canvasElement}
    onpointerdown={startDrawing}
    onpointermove={handlePointerMove}
    onpointerup={stopDrawing}
    onpointercancel={stopDrawing}
    onpointerleave={handlePointerLeave}
    onpointerenter={handlePointerEnter}
    class="drawing-canvas"
  ></canvas>
  <div 
    id="custom-cursor"
    class="custom-cursor"
    style="width: {currentSize}px; height: {currentSize}px; border-color: {currentTool === 'pen' ? currentColor : '#666'};"
  ></div>
</div>

<style>
  .canvas-wrapper {
    position: relative;
    background: white;
    border-radius: 8px;
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    overflow: hidden;
  }
  
  .drawing-canvas {
    display: block;
    touch-action: none;
    user-select: none;
    background: white;
  }
  
  .custom-cursor {
    position: absolute;
    border: 2px solid;
    border-radius: 50%;
    pointer-events: none;
    transform: translate(-50%, -50%);
    transition: none;
    mix-blend-mode: difference;
  }
  
  @keyframes slideIn {
    from {
      opacity: 0;
      transform: translateY(20px);
    }
    to {
      opacity: 1;
      transform: translateY(0);
    }
  }
  
  @keyframes slideOut {
    from {
      opacity: 1;
      transform: translateY(0);
    }
    to {
      opacity: 0;
      transform: translateY(20px);
    }
  }
</style>