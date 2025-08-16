<script>
  let { currentTool = $bindable(), currentColor = $bindable(), penSize = $bindable(), eraserSize = $bindable(), onClear, onDownload } = $props();
  
  const colors = [
    '#000000', // 黒
    '#ffffff', // 白
    '#e11d48', // 赤系
    '#f59e0b', // オレンジ
    '#eab308', // 黄
    '#22c55e', // 緑
    '#06b6d4', // シアン
    '#3b82f6', // 青
    '#8b5cf6', // 紫
    '#ec4899', // ピンク
  ];
  
  function confirmClear() {
    if (confirm('キャンバスをすべて消去します。よろしいですか？')) {
      onClear();
    }
  }
</script>

<div class="toolbar" role="toolbar" aria-label="描画ツール">
  <div class="tool-section">
    <button
      class="tool-button"
      class:active={currentTool === 'pen'}
      onclick={() => currentTool = 'pen'}
      aria-pressed={currentTool === 'pen'}
      aria-label="ペンツール"
      title="ペンツール (P)"
    >
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M12 19l7-7 3 3-7 7-3-3z"/>
        <path d="M18 13l-1.5-7.5L2 2l3.5 14.5L13 18l5-5z"/>
        <path d="M2 2l7.586 7.586"/>
        <circle cx="11" cy="11" r="2"/>
      </svg>
      <span>ペン</span>
    </button>
    
    <button
      class="tool-button"
      class:active={currentTool === 'eraser'}
      onclick={() => currentTool = 'eraser'}
      aria-pressed={currentTool === 'eraser'}
      aria-label="消しゴムツール"
      title="消しゴムツール (E)"
    >
      <svg width="24" height="24" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M20 20H7l-4-4a1 1 0 010-1.414l9-9a1 1 0 011.414 0l7 7a1 1 0 010 1.414l-4 4z"/>
        <path d="M18 13l-5-5"/>
      </svg>
      <span>消しゴム</span>
    </button>
  </div>
  
  <div class="color-section">
    <label class="section-label">カラー</label>
    <div class="color-grid" role="radiogroup" aria-label="色選択">
      {#each colors as color}
        <button
          class="color-swatch"
          class:active={currentColor === color}
          style="background-color: {color}"
          onclick={() => currentColor = color}
          aria-label="色: {color}"
          aria-checked={currentColor === color}
          role="radio"
        >
          {#if currentColor === color}
            <svg class="check-icon" width="16" height="16" viewBox="0 0 24 24" fill="none">
              <path d="M5 12l5 5L20 7" stroke={color === '#ffffff' ? '#000' : '#fff'} stroke-width="3" stroke-linecap="round"/>
            </svg>
          {/if}
        </button>
      {/each}
    </div>
  </div>
  
  <div class="size-section">
    <label class="section-label">
      {currentTool === 'pen' ? 'ペン' : '消しゴム'}の太さ
    </label>
    <div class="size-control">
      {#if currentTool === 'pen'}
        <input
          type="range"
          class="size-slider"
          min={1}
          max={30}
          bind:value={penSize}
          aria-label="ペンの太さ"
        />
      {:else}
        <input
          type="range"
          class="size-slider"
          min={5}
          max={50}
          bind:value={eraserSize}
          aria-label="消しゴムの太さ"
        />
      {/if}
      <div class="size-preview">
        <div 
          class="size-circle"
          style="width: {currentTool === 'pen' ? penSize : eraserSize}px; height: {currentTool === 'pen' ? penSize : eraserSize}px; background-color: {currentTool === 'pen' ? currentColor : '#ccc'};"
        ></div>
        <span class="size-value">{currentTool === 'pen' ? penSize : eraserSize}px</span>
      </div>
    </div>
  </div>
  
  <div class="action-section">
    <button
      class="action-button danger"
      onclick={confirmClear}
      aria-label="キャンバスをクリア"
      title="キャンバスをクリア (C)"
    >
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M3 6h18"/>
        <path d="M19 6v14a2 2 0 01-2 2H7a2 2 0 01-2-2V6"/>
        <path d="M8 6V4a2 2 0 012-2h4a2 2 0 012 2v2"/>
      </svg>
      <span>クリア</span>
    </button>
    
    <button
      class="action-button primary"
      onclick={onDownload}
      aria-label="画像をダウンロード"
      title="画像をダウンロード (Ctrl/Cmd+S)"
    >
      <svg width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2">
        <path d="M21 15v4a2 2 0 01-2 2H5a2 2 0 01-2-2v-4"/>
        <polyline points="7 10 12 15 17 10"/>
        <line x1="12" y1="15" x2="12" y2="3"/>
      </svg>
      <span>保存</span>
    </button>
  </div>
</div>

<style>
  .toolbar {
    width: 88px;
    background-color: #ffffff;
    border-right: 1px solid #e5e5e5;
    display: flex;
    flex-direction: column;
    padding: 12px 8px;
    gap: 20px;
    overflow-y: auto;
    box-shadow: 2px 0 8px rgba(0, 0, 0, 0.05);
  }
  
  .tool-section {
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  
  .tool-button {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    padding: 8px;
    background: white;
    border: 2px solid #e5e5e5;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
    font-size: 11px;
    color: #666;
  }
  
  .tool-button:hover {
    background-color: #f5f5f5;
    border-color: #3b82f6;
  }
  
  .tool-button.active {
    background-color: #3b82f6;
    border-color: #3b82f6;
    color: white;
  }
  
  .tool-button span {
    font-weight: 500;
  }
  
  .section-label {
    font-size: 11px;
    font-weight: 600;
    color: #666;
    text-transform: uppercase;
    letter-spacing: 0.5px;
    margin-bottom: 8px;
    text-align: center;
  }
  
  .color-section {
    display: flex;
    flex-direction: column;
  }
  
  .color-grid {
    display: grid;
    grid-template-columns: repeat(2, 1fr);
    gap: 6px;
  }
  
  .color-swatch {
    width: 32px;
    height: 32px;
    border: 2px solid #e5e5e5;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    position: relative;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  
  .color-swatch:hover {
    transform: scale(1.1);
    border-color: #3b82f6;
  }
  
  .color-swatch.active {
    border-color: #3b82f6;
    border-width: 3px;
  }
  
  .check-icon {
    pointer-events: none;
  }
  
  .size-section {
    display: flex;
    flex-direction: column;
  }
  
  .size-control {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 8px;
  }
  
  .size-slider {
    width: 100%;
    transform: rotate(-90deg);
    transform-origin: center;
    margin: 20px 0;
  }
  
  .size-preview {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
  }
  
  .size-circle {
    border-radius: 50%;
    transition: all 0.2s;
  }
  
  .size-value {
    font-size: 10px;
    color: #666;
    font-weight: 500;
  }
  
  .action-section {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin-top: auto;
    padding-top: 20px;
  }
  
  .action-button {
    display: flex;
    flex-direction: column;
    align-items: center;
    gap: 4px;
    padding: 10px;
    background: white;
    border: 2px solid #e5e5e5;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s;
    font-size: 11px;
    font-weight: 500;
  }
  
  .action-button span {
    color: #666;
  }
  
  .action-button:hover {
    transform: translateY(-1px);
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  }
  
  .action-button.danger {
    border-color: #ef4444;
    color: #ef4444;
  }
  
  .action-button.danger:hover {
    background-color: #fef2f2;
  }
  
  .action-button.primary {
    border-color: #3b82f6;
    color: #3b82f6;
  }
  
  .action-button.primary:hover {
    background-color: #eff6ff;
  }
</style>