<script lang="ts">
  import { onMount } from 'svelte';
  
  // Email-specific draggable elements
  const elements = [
    { 
      id: 'container', 
      type: 'container', 
      content: 'Container', 
      tag: 'table',
      style: 'width: 600px; margin: 0 auto; background: #ffffff;',
      properties: {
        width: '600px',
        background: '#ffffff'
      }
    },
    { 
      id: 'header', 
      type: 'header', 
      content: 'Header', 
      tag: 'tr',
      style: 'background: #f8f9fa; padding: 20px;',
      properties: {
        background: '#f8f9fa',
        padding: '20px',
        logo: {
          src: 'https://via.placeholder.com/150x50',
          alt: 'Logo'
        },
        nav: {
          items: ['Home', 'About', 'Contact'],
          position: 'right' // 'right' or 'center'
        }
      }
    },
    { 
      id: 'heading', 
      type: 'heading', 
      content: 'Heading', 
      tag: 'h1',
      style: 'color: #333333; font-family: Arial, sans-serif; font-size: 24px; margin: 0;',
      properties: {
        color: '#333333',
        fontSize: '24px'
      }
    },
    { 
      id: 'paragraph', 
      type: 'paragraph', 
      content: 'Paragraph', 
      tag: 'p',
      style: 'color: #666666; font-family: Arial, sans-serif; font-size: 16px; line-height: 1.5; margin: 0;',
      properties: {
        color: '#666666',
        fontSize: '16px'
      }
    },
    { 
      id: 'button', 
      type: 'button', 
      content: 'Button', 
      tag: 'a',
      style: 'display: inline-block; padding: 12px 24px; background: #007bff; color: #ffffff; text-decoration: none; border-radius: 4px; font-family: Arial, sans-serif;',
      properties: {
        backgroundColor: '#007bff',
        color: '#ffffff'
      }
    },
    { 
      id: 'image', 
      type: 'image', 
      content: 'Image', 
      tag: 'img',
      style: 'max-width: 100%; height: auto; display: block;',
      properties: {
        src: 'https://via.placeholder.com/600x300',
        alt: 'Placeholder'
      }
    },
    { 
      id: 'divider', 
      type: 'divider', 
      content: 'Divider', 
      tag: 'hr',
      style: 'border: none; border-top: 1px solid #eeeeee; margin: 20px 0;',
      properties: {
        color: '#eeeeee',
        margin: '20px 0'
      }
    },
    { 
      id: 'footer', 
      type: 'footer', 
      content: 'Footer', 
      tag: 'tr',
      style: 'background: #f8f9fa; padding: 20px; text-align: center;',
      properties: {
        background: '#f8f9fa',
        padding: '20px'
      }
    }
  ];

  // Drop zone layout
  let dropZone: HTMLElement;
  let dropZoneItems: any[] = [];
  let editingItem: any = null;
  let showEditor = false;
  let draggedItem: any = null;
  let imageFileInput: HTMLInputElement;
  
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

  // Reordering handlers
  function handleItemDragStart(event: DragEvent, item: any) {
    draggedItem = item;
    if (event.dataTransfer) {
      event.dataTransfer.effectAllowed = 'move';
      event.dataTransfer.setData('text/plain', item.id);
    }
  }

  function handleItemDragOver(event: DragEvent, item: any) {
    event.preventDefault();
    if (event.dataTransfer) {
      event.dataTransfer.dropEffect = 'move';
    }
    if (draggedItem && draggedItem.id !== item.id) {
      const draggedIndex = dropZoneItems.findIndex(i => i.id === draggedItem.id);
      const dropIndex = dropZoneItems.findIndex(i => i.id === item.id);
      const rect = (event.target as HTMLElement).getBoundingClientRect();
      const midY = rect.top + rect.height / 2;
      const dropPosition = event.clientY < midY ? 'before' : 'after';
      
      if (dropPosition === 'before' && draggedIndex > dropIndex) {
        dropZoneItems = dropZoneItems.filter(i => i.id !== draggedItem.id);
        dropZoneItems.splice(dropIndex, 0, draggedItem);
      } else if (dropPosition === 'after' && draggedIndex < dropIndex) {
        dropZoneItems = dropZoneItems.filter(i => i.id !== draggedItem.id);
        dropZoneItems.splice(dropIndex, 0, draggedItem);
      }
    }
  }

  function handleItemDragEnd() {
    draggedItem = null;
  }

  // Remove item from drop zone
  function removeItem(id: string) {
    dropZoneItems = dropZoneItems.filter(item => item.id !== id);
  }

  // Edit item properties
  function editItem(item: any) {
    editingItem = item;
    showEditor = true;
  }

  function updateItem() {
    if (editingItem) {
      dropZoneItems = dropZoneItems.map(item => 
        item.id === editingItem.id ? editingItem : item
      );
      showEditor = false;
      editingItem = null;
    }
  }

  // Add this new function to update header preview in real-time
  function updateHeaderPreview() {
    if (editingItem && editingItem.type === 'header') {
      dropZoneItems = dropZoneItems.map(item => 
        item.id === editingItem.id ? editingItem : item
      );
    }
  }

  function updateProperty(property: string, value: string) {
    if (editingItem) {
      editingItem.properties[property] = value;
      // Update style based on properties
      if (property === 'src' || property === 'alt') {
        editingItem.style = `max-width: 100%; height: auto; display: block;`;
      } else {
        const styleMap: Record<string, string> = {
          width: `width: ${value};`,
          background: `background: ${value};`,
          color: `color: ${value};`,
          fontSize: `font-size: ${value};`,
          backgroundColor: `background: ${value};`,
          padding: `padding: ${value};`,
          margin: `margin: ${value};`
        };
        editingItem.style = Object.entries(editingItem.properties)
          .map(([key, val]) => styleMap[key] || '')
          .filter(Boolean)
          .join(' ');
      }
    }
  }

  // Add this new function for handling image uploads
  function handleImageUpload(event: Event, item: any) {
    const file = (event.target as HTMLInputElement).files?.[0];
    if (file) {
      const reader = new FileReader();
      reader.onload = (e) => {
        if (e.target?.result) {
          item.properties.src = e.target.result as string;
          item.properties.alt = file.name;
          // Update the item in dropZoneItems immediately
          dropZoneItems = dropZoneItems.map(i => 
            i.id === item.id ? item : i
          );
        }
      };
      reader.readAsDataURL(file);
    }
  }

  // Add this new function for handling image resize
  function handleImageResize(event: Event, item: any) {
    const width = (event.target as HTMLInputElement).value;
    item.properties.width = width;
    item.style = `max-width: ${width}; height: auto; display: block;`;
  }

  // Generate email HTML
  function generateEmailHTML() {
    let html = `
      <!DOCTYPE html>
      <html>
      <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Email Template</title>
      </head>
      <body style="margin: 0; padding: 0; background: #f4f4f4;">
        <table cellpadding="0" cellspacing="0" border="0" style="width: 100%;">
          <tr>
            <td align="center" style="padding: 20px;">
              <table cellpadding="0" cellspacing="0" border="0" style="width: 600px; background: #ffffff;">
    `;

    dropZoneItems.forEach(item => {
      if (item.type === 'container') {
        html += `<tr><td style="${item.style}">${item.content}</td></tr>`;
      } else if (item.type === 'header') {
        html += `
          <tr>
            <td style="${item.style}">
              <table cellpadding="0" cellspacing="0" border="0" style="width: 100%;">
                <tr>
                  <td style="display: flex; justify-content: space-between; align-items: center;">
                    <div style="flex-shrink: 0;">
                      <img src="${item.properties.logo.src}" alt="${item.properties.logo.alt}" style="max-height: 50px;">
                    </div>
                    <div style="${item.properties.nav.position === 'center' ? 'flex: 1; text-align: center;' : ''}">
                      ${item.properties.nav.items.map((navItem: string) => 
                        `<a href="#" style="margin: 0 10px; color: #333; text-decoration: none;">${navItem}</a>`
                      ).join('')}
                    </div>
                  </td>
                </tr>
              </table>
            </td>
          </tr>`;
      } else if (item.type === 'footer') {
        html += `<tr><td style="${item.style}">${item.content}</td></tr>`;
      } else if (item.type === 'heading') {
        html += `<tr><td style="padding: 10px;"><h1 style="${item.style}">${item.content}</h1></td></tr>`;
      } else if (item.type === 'paragraph') {
        html += `<tr><td style="padding: 10px;"><p style="${item.style}">${item.content}</p></td></tr>`;
      } else if (item.type === 'button') {
        html += `<tr><td style="padding: 10px; text-align: center;"><a href="#" style="${item.style}">${item.content}</a></td></tr>`;
      } else if (item.type === 'image') {
        html += `<tr><td style="padding: 10px; text-align: center;"><img src="${item.properties.src}" alt="${item.properties.alt}" style="${item.style}"></td></tr>`;
      } else if (item.type === 'divider') {
        html += `<tr><td style="padding: 10px;"><hr style="${item.style}"></td></tr>`;
      }
    });

    html += `
              </table>
            </td>
          </tr>
        </table>
      </body>
      </html>
    `;

    return html;
  }

  // Download email template
  function downloadTemplate() {
    const html = generateEmailHTML();
    const blob = new Blob([html], { type: 'text/html' });
    const url = window.URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'email-template.html';
    a.click();
    window.URL.revokeObjectURL(url);
  }
</script>

<div class="builder-container">
  <!-- Elements Panel -->
  <div class="elements-panel">
    <h2>Email Elements</h2>
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
    <div class="template-header">
      <h2>Email Template Builder</h2>
      <button class="download-btn" on:click={downloadTemplate}>Download Template</button>
    </div>
    <div class="template-container">
      {#each dropZoneItems as item}
        <div 
          class="template-item"
          draggable="true"
          on:dragstart={(e) => handleItemDragStart(e, item)}
          on:dragover={(e) => handleItemDragOver(e, item)}
          on:dragend={handleItemDragEnd}
        >
          <div class="item-content" style={item.style}>
            {#if item.type === 'container'}
              <div class="container-preview">
                <div contenteditable="true" bind:textContent={item.content}>Container</div>
              </div>
            {:else if item.type === 'header' || item.type === 'footer'}
              {#if item.type === 'header'}
                <div class="section-preview header-preview">
                  <div class="header-content" style="display: flex; justify-content: space-between; align-items: center;">
                    <div class="header-logo">
                      <img 
                        src={item.properties.logo.src} 
                        alt={item.properties.logo.alt}
                        style="max-height: 50px;"
                      />
                    </div>
                    <div class="header-nav" style={item.properties.nav.position === 'center' ? 'flex: 1; text-align: center;' : ''}>
                      {#each item.properties.nav.items as navItem}
                        <a href="#" style="margin: 0 10px; color: #333; text-decoration: none;">{navItem}</a>
                      {/each}
                    </div>
                  </div>
                </div>
              {:else}
                <div class="section-preview">
                  <div contenteditable="true" bind:textContent={item.content}>{item.content}</div>
                </div>
              {/if}
            {:else if item.type === 'heading'}
              <h1 contenteditable="true" bind:textContent={item.content}>{item.content}</h1>
            {:else if item.type === 'paragraph'}
              <p contenteditable="true" bind:textContent={item.content}>{item.content}</p>
            {:else if item.type === 'button'}
              <a href="#" contenteditable="true" bind:textContent={item.content}>{item.content}</a>
            {:else if item.type === 'image'}
              <div class="image-container">
                <img 
                  src={item.properties.src} 
                  alt={item.properties.alt} 
                  style="width: {item.properties.width || 'auto'}px; height: {item.properties.height || 'auto'}px; display: block;"
                />
                <div class="image-controls">
                  <input
                    type="file"
                    accept="image/*"
                    on:change={(e) => handleImageUpload(e, item)}
                    style="display: none"
                    bind:this={imageFileInput}
                  />
                  <button class="upload-btn" on:click={() => imageFileInput.click()}>
                    Upload Image
                  </button>
                  <div class="resize-control">
                    <div class="dimension-input">
                      <label>Width:</label>
                      <input 
                        type="number" 
                        min="100" 
                        max="600" 
                        value={item.properties.width || 300}
                        on:input={(e) => {
                          item.properties.width = (e.target as HTMLInputElement).value;
                          item.style = `width: ${item.properties.width}px; height: ${item.properties.height || 'auto'}; display: block;`;
                        }}
                      />
                    </div>
                    <div class="dimension-input">
                      <label>Height:</label>
                      <input 
                        type="number" 
                        min="100" 
                        max="600" 
                        value={item.properties.height || 'auto'}
                        on:input={(e) => {
                          item.properties.height = (e.target as HTMLInputElement).value;
                          item.style = `width: ${item.properties.width || 'auto'}; height: ${item.properties.height}px; display: block;`;
                        }}
                      />
                    </div>
                  </div>
                </div>
              </div>
            {:else if item.type === 'divider'}
              <hr />
            {/if}
          </div>
          <div class="item-controls">
            <button class="edit-btn" on:click={() => editItem(item)}>Edit</button>
            <button class="remove-btn" on:click={() => removeItem(item.id)}>×</button>
          </div>
        </div>
      {/each}
    </div>
  </div>

  <!-- Property Editor Modal -->
  {#if showEditor && editingItem}
    <div class="modal-overlay" on:click={() => showEditor = false}>
      <div class="modal-content" on:click|stopPropagation>
        <h3>Edit {editingItem.type}</h3>
        {#if editingItem.type === 'image'}
          <div class="image-editor">
            <div class="form-group">
              <label>Image URL:</label>
              <input type="text" bind:value={editingItem.properties.src} />
            </div>
            <div class="form-group">
              <label>Alt Text:</label>
              <input type="text" bind:value={editingItem.properties.alt} />
            </div>
            <div class="form-group">
              <label>Upload Local Image:</label>
              <input 
                type="file" 
                accept="image/*" 
                on:change={(e) => handleImageUpload(e, editingItem)}
              />
            </div>
            <div class="form-group">
              <label>Width (px):</label>
              <input 
                type="number" 
                min="100" 
                max="600" 
                value={editingItem.properties.width || 300}
                on:input={(e) => {
                  editingItem.properties.width = (e.target as HTMLInputElement).value;
                  editingItem.style = `width: ${editingItem.properties.width}px; height: ${editingItem.properties.height || 'auto'}; display: block;`;
                }}
              />
            </div>
            <div class="form-group">
              <label>Height (px):</label>
              <input 
                type="number" 
                min="100" 
                max="600" 
                value={editingItem.properties.height || 'auto'}
                on:input={(e) => {
                  editingItem.properties.height = (e.target as HTMLInputElement).value;
                  editingItem.style = `width: ${editingItem.properties.width || 'auto'}; height: ${editingItem.properties.height}px; display: block;`;
                }}
              />
            </div>
            <img 
              src={editingItem.properties.src} 
              alt={editingItem.properties.alt} 
              class="preview"
            />
          </div>
        {:else if editingItem.type === 'container'}
          <div class="form-group">
            <label>Width:</label>
            <input type="text" bind:value={editingItem.properties.width} />
          </div>
          <div class="form-group">
            <label>Background Color:</label>
            <input type="color" bind:value={editingItem.properties.background} />
          </div>
        {:else if editingItem.type === 'header'}
          <div class="header-editor">
            <div class="form-group">
              <label>Logo:</label>
              <div class="logo-upload">
                <img 
                  src={editingItem.properties.logo.src} 
                  alt={editingItem.properties.logo.alt}
                  class="logo-preview"
                />
                <input 
                  type="file" 
                  accept="image/*" 
                  on:change={(e) => {
                    const file = (e.target as HTMLInputElement).files?.[0];
                    if (file) {
                      const reader = new FileReader();
                      reader.onload = (e) => {
                        if (e.target?.result) {
                          editingItem.properties.logo.src = e.target.result as string;
                          editingItem.properties.logo.alt = file.name;
                          // Update the item in dropZoneItems immediately
                          dropZoneItems = dropZoneItems.map(item => 
                            item.id === editingItem.id ? editingItem : item
                          );
                        }
                      };
                      reader.readAsDataURL(file);
                    }
                  }}
                />
              </div>
            </div>
            <div class="form-group">
              <label>Navigation Position:</label>
              <select 
                bind:value={editingItem.properties.nav.position}
                on:change={() => {
                  // Update the item in dropZoneItems immediately
                  dropZoneItems = dropZoneItems.map(item => 
                    item.id === editingItem.id ? editingItem : item
                  );
                }}
              >
                <option value="right">Right</option>
                <option value="center">Center</option>
              </select>
            </div>
            <div class="form-group">
              <label>Navigation Items:</label>
              <div class="nav-items">
                {#each editingItem.properties.nav.items as item, index}
                  <div class="nav-item">
                    <input 
                      type="text" 
                      bind:value={editingItem.properties.nav.items[index]} 
                      placeholder="Nav item"
                      on:input={() => {
                        // Update the item in dropZoneItems immediately
                        dropZoneItems = dropZoneItems.map(item => 
                          item.id === editingItem.id ? editingItem : item
                        );
                      }}
                    />
                    <button 
                      class="remove-nav-item" 
                      on:click={() => {
                        editingItem.properties.nav.items = editingItem.properties.nav.items.filter((_: string, i: number) => i !== index);
                        // Update the item in dropZoneItems immediately
                        dropZoneItems = dropZoneItems.map(item => 
                          item.id === editingItem.id ? editingItem : item
                        );
                      }}
                    >×</button>
                  </div>
                {/each}
                <button 
                  class="add-nav-item" 
                  on:click={() => {
                    editingItem.properties.nav.items = [...editingItem.properties.nav.items, 'New Item'];
                    // Update the item in dropZoneItems immediately
                    updateHeaderPreview();
                  }}
                >
                  Add Item
                </button>
              </div>
            </div>
          </div>
        {:else}
          <div class="form-group">
            <label>Content:</label>
            <input type="text" bind:value={editingItem.content} />
          </div>
          {#if editingItem.properties.color}
            <div class="form-group">
              <label>Color:</label>
              <input type="color" bind:value={editingItem.properties.color} />
            </div>
          {/if}
          {#if editingItem.properties.fontSize}
            <div class="form-group">
              <label>Font Size:</label>
              <input type="text" bind:value={editingItem.properties.fontSize} />
            </div>
          {/if}
          {#if editingItem.properties.background}
            <div class="form-group">
              <label>Background Color:</label>
              <input type="color" bind:value={editingItem.properties.background} />
            </div>
          {/if}
        {/if}
        <div class="modal-actions">
          <button class="save-btn" on:click={updateItem}>Save</button>
          <button class="cancel-btn" on:click={() => showEditor = false}>Cancel</button>
        </div>
      </div>
    </div>
  {/if}
</div>

<style>
  .builder-container {
    display: flex;
    gap: 20px;
    padding: 20px;
    height: 100vh;
    background: #f4f4f4;
  }

  .elements-panel {
    width: 250px;
    background: white;
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
    background: #f8f9fa;
    border: 1px solid #ddd;
    border-radius: 4px;
    cursor: move;
    transition: all 0.2s;
  }

  .element-item:hover {
    background: #e9ecef;
  }

  .drop-zone {
    flex: 1;
    background: white;
    padding: 20px;
    border: 2px dashed #ccc;
    border-radius: 8px;
    min-height: 500px;
  }

  .template-header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 20px;
  }

  .download-btn {
    background: #28a745;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
    font-family: Arial, sans-serif;
  }

  .download-btn:hover {
    background: #218838;
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
    cursor: move;
    transition: all 0.2s;
    background: white;
  }

  .template-item:hover {
    background: #f8f9fa;
  }

  .template-item.dragging {
    opacity: 0.5;
    background: #e9ecef;
  }

  .template-item.drag-over {
    border: 2px dashed #007bff;
  }

  .item-content {
    min-height: 30px;
  }

  .item-controls {
    position: absolute;
    top: 5px;
    right: 5px;
    display: flex;
    gap: 5px;
  }

  .edit-btn {
    background: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    padding: 4px 8px;
    cursor: pointer;
    font-size: 12px;
  }

  .edit-btn:hover {
    background: #0056b3;
  }

  .container-preview {
    background: #f8f9fa;
    padding: 20px;
    border: 2px dashed #ccc;
    text-align: center;
  }

  .section-preview {
    background: #e9ecef;
    padding: 20px;
    text-align: center;
    font-weight: bold;
  }

  .remove-btn {
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

  h1, p, a {
    margin: 0;
    padding: 5px;
    font-family: Arial, sans-serif;
  }

  img {
    max-width: 100%;
    height: auto;
    display: block;
  }

  [contenteditable="true"]:focus {
    outline: 2px solid #007bff;
    padding: 2px;
  }

  .modal-overlay {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;
    z-index: 1000;
  }

  .modal-content {
    background: white;
    padding: 20px;
    border-radius: 8px;
    min-width: 300px;
  }

  .form-group {
    margin-bottom: 15px;
  }

  .form-group label {
    display: block;
    margin-bottom: 5px;
    font-weight: bold;
  }

  .form-group input {
    width: 100%;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
  }

  .modal-actions {
    display: flex;
    justify-content: flex-end;
    gap: 10px;
    margin-top: 20px;
  }

  .save-btn {
    background: #28a745;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
  }

  .cancel-btn {
    background: #6c757d;
    color: white;
    border: none;
    padding: 8px 16px;
    border-radius: 4px;
    cursor: pointer;
  }

  .save-btn:hover {
    background: #218838;
  }

  .cancel-btn:hover {
    background: #5a6268;
  }

  .image-container {
    position: relative;
    margin: 10px 0;
    background: #f8f9fa;
    padding: 10px;
    border-radius: 4px;
  }

  .image-container img {
    max-width: 100%;
    height: auto;
    display: block;
    margin: 0 auto;
  }

  .image-controls {
    position: absolute;
    bottom: -30px;
    left: 0;
    right: 0;
    display: flex;
    gap: 10px;
    align-items: center;
    background: rgba(255, 255, 255, 0.9);
    padding: 5px;
    border-radius: 4px;
    box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  }

  .upload-btn {
    background: #007bff;
    color: white;
    border: none;
    padding: 4px 8px;
    border-radius: 4px;
    cursor: pointer;
    font-size: 12px;
  }

  .upload-btn:hover {
    background: #0056b3;
  }

  .resize-control {
    display: flex;
    align-items: center;
    gap: 8px;
  }

  .resize-control label {
    font-size: 12px;
    color: #666;
  }

  .resize-control input[type="range"] {
    width: 100px;
  }

  .resize-control span {
    font-size: 12px;
    color: #666;
    min-width: 45px;
  }

  .image-editor .preview {
    max-width: 100%;
    height: auto;
    margin: 10px 0;
    border: 1px solid #ddd;
    border-radius: 4px;
  }

  .header-preview {
    padding: 15px;
  }

  .header-content {
    max-width: 600px;
    margin: 0 auto;
  }

  .header-logo img {
    max-height: 50px;
    width: auto;
  }

  .header-nav {
    display: flex;
    align-items: center;
  }

  .header-nav a {
    color: #333;
    text-decoration: none;
    padding: 5px 10px;
    font-family: Arial, sans-serif;
  }

  .header-nav a:hover {
    color: #007bff;
  }

  .header-editor {
    display: flex;
    flex-direction: column;
    gap: 15px;
  }

  .logo-upload {
    display: flex;
    align-items: center;
    gap: 15px;
  }

  .logo-preview {
    max-width: 150px;
    height: auto;
    border: 1px solid #ddd;
    border-radius: 4px;
  }

  .nav-items {
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .nav-item {
    display: flex;
    gap: 10px;
    align-items: center;
  }

  .nav-item input {
    flex: 1;
  }

  .remove-nav-item {
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

  .add-nav-item {
    background: #28a745;
    color: white;
    border: none;
    padding: 5px 10px;
    border-radius: 4px;
    cursor: pointer;
    align-self: flex-start;
  }

  .add-nav-item:hover {
    background: #218838;
  }

  select {
    width: 100%;
    padding: 8px;
    border: 1px solid #ddd;
    border-radius: 4px;
    background: white;
  }

  .dimension-input {
    display: flex;
    align-items: center;
    gap: 5px;
  }

  .dimension-input label {
    font-size: 12px;
    color: #666;
    white-space: nowrap;
  }

  .dimension-input input {
    width: 60px;
    padding: 4px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 12px;
  }
</style> 