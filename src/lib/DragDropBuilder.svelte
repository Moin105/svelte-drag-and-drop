<script lang="ts">
  import { onMount } from 'svelte';
  
  // Draggable elements
  const elements = [
    { id: 'heading', type: 'heading', content: 'Heading', tag: 'h1' },
    { id: 'paragraph', type: 'paragraph', content: 'Paragraph', tag: 'p' },
    { id: 'button', type: 'button', content: 'Button', tag: 'button' },
    { id: 'image', type: 'image', content: 'Image', tag: 'img' },
    { id: 'divider', type: 'divider', content: 'Divider', tag: 'hr' }
  ];

  // Drop zone layout
  let dropZone: HTMLElement;
  let dropZoneItems: any[] = [];
  
  // Drag and drop handlers
  function handleDragStart(event: DragEvent, element: any) {
    if (event.dataTransfer) {
      event.dataTransfer.setData('text/plain', JSON.stringify(element));
    }
  }

  function handleDragOver(event: DragEvent) {
    event.preventDefault();
  }

  function handleDrop(event: DragEvent) {
    event.preventDefault();
    if (event.dataTransfer) {
      const elementData = JSON.parse(event.dataTransfer.getData('text/plain'));
      dropZoneItems = [...dropZoneItems, { ...elementData, id: `${elementData.id}-${Date.now()}` }];
    }
  }

  // Remove item from drop zone
  function removeItem(id: string) {
    dropZoneItems = dropZoneItems.filter(item => item.id !== id);
  }
</script>

<div class="builder-container">
  <!-- Elements Panel -->
  <div class="elements-panel">
    <h2>Elements</h2>
    <div class="elements-list">
      {#each elements as element}
        <div
          class="element-item"
          draggable="true"
          role="button"
          on:dragstart={(e) => handleDragStart(e, element)}
          on:keydown={(e) => {
            if (e.key === 'Enter' || e.key === ' ') {
              handleDragStart(new DragEvent('dragstart'), element);
            }
          }}
          tabindex="0"
        >
          {element.content}
        </div>
      {/each}
    </div>
  </div>

  <!-- Drop Zone -->
  <div
    class="drop-zone"
    bind:this={dropZone}
    role="region"
    aria-label="Email template drop zone"
    on:dragover={handleDragOver}
    on:drop={handleDrop}
  >
    <h2>Email Template</h2>
    <div class="template-container">
      {#each dropZoneItems as item}
        <div class="template-item">
          <div class="item-content">
            {#if item.type === 'heading'}
              <h1 contenteditable="true">{item.content}</h1>
            {:else if item.type === 'paragraph'}
              <p contenteditable="true">{item.content}</p>
            {:else if item.type === 'button'}
              <button contenteditable="true">{item.content}</button>
            {:else if item.type === 'image'}
              <img src="https://via.placeholder.com/150" alt="Placeholder" />
            {:else if item.type === 'divider'}
              <hr />
            {/if}
          </div>
          <button class="remove-btn" on:click={() => removeItem(item.id)}>×</button>
        </div>
      {/each}
    </div>
  </div>
</div>

<style>
  .builder-container {
    display: flex;
    gap: 20px;
    padding: 20px;
    height: 100vh;
  }

  .elements-panel {
    width: 250px;
    background: #f5f5f5;
    padding: 20px;
    border-radius: 8px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .elements-list {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .element-item {
    padding: 10px;
    background: white;
    border: 1px solid #ddd;
    border-radius: 4px;
    cursor: move;
    transition: all 0.2s;
  }

  .element-item:hover {
    background: #f0f0f0;
  }

  .drop-zone {
    flex: 1;
    background: white;
    padding: 20px;
    border: 2px dashed #ccc;
    border-radius: 8px;
    min-height: 500px;
  }

  .template-container {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .template-item {
    position: relative;
    padding: 10px;
    border: 1px solid #eee;
    border-radius: 4px;
  }

  .item-content {
    min-height: 30px;
  }

  .remove-btn {
    position: absolute;
    top: 5px;
    right: 5px;
    background: #ff4444;
    color: white;
    border: none;
    border-radius: 50%;
    width: 20px;
    height: 20px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 14px;
  }

  .remove-btn:hover {
    background: #cc0000;
  }

  h1, p, button {
    margin: 0;
    padding: 5px;
  }

  img {
    max-width: 100%;
    height: auto;
  }

  [contenteditable="true"]:focus {
    outline: 2px solid #007bff;
    padding: 2px;
  }
</style> 