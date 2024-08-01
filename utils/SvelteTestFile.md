---
  title: 'SvelteTestFile.svelte'
  description: 'component description'
  pubDate: 'August 1, 2024'
  author: 'Carlos Polanco'
  ---
  
  
  
  # SvelteTestFile.svelte
  ## Code Explanation

### Libraries Used
- **Svelte**: The code is written using the Svelte framework, which is a reactive web framework that compiles components into efficient JavaScript code.

### Variables
- **allowedExtensions**: An array containing a list of file extensions that are allowed to be uploaded.
- **maxFiles**: Maximum number of files that can be uploaded.
- **selectedFiles**: An array to store the selected files.
- **dispatch**: A function from Svelte used to dispatch events.

### Functions
1. **handleDragOver(event)**:
   - Prevents the default behavior of dragover event.

2. **handleDrop(event)**:
   - Prevents the default behavior of drop event.
   - Extracts files from the dropped items and validates them.

3. **extractFiles(items)**:
   - Extracts files from the dropped items asynchronously.
   - Validates each file and adds it to the filesArray if valid.

4. **traverseFileTree(item, filesArray)**:
   - Recursively traverses a file tree asynchronously.
   - Adds valid files to the filesArray.

5. **isValidFile(file)**:
   - Checks if the file has a valid extension based on allowedExtensions.

6. **handleFileSelect(event)**:
   - Handles the file selection from the input element.
   - Validates and sets the selected files.

7. **validateAndSetFiles(files)**:
   - Filters out invalid files from the selected files.
   - Checks if the total number of files exceeds the maximum allowed.
   - Updates the selectedFiles array and dispatches an error if needed.

### Event Handlers
- **on:dragover|preventDefault={handleDragOver}**: Calls handleDragOver function on dragover event and prevents the default behavior.
- **on:drop|preventDefault={handleDrop}**: Calls handleDrop function on drop event and prevents the default behavior.
- **on:change={handleFileSelect}**: Calls handleFileSelect function when files are selected.

### HTML Elements
- **<div>**: Represents a container for drag and drop functionality.
- **<input>**: Hidden file input element for selecting files.
- **<label>**: Label for the file input element.
- **{#if}/{:else}**: Conditional rendering based on the number of selected files.
- **{#each}**: Loops through the selectedFiles array to display each file.

### Example Usage
```html
<!-- Example usage of the drag and drop file uploader -->
<script>
  // Include the code provided above
</script>

<div
  on:dragover|preventDefault={handleDragOver}
  on:drop|preventDefault={handleDrop}
  class="p-4 border-2 border-dashed border-gray-300 rounded-md text-center cursor-pointer mb-4 h-96 w-96 flex overflow-y-scroll items-center justify-center"
>
  <!-- Input and label elements for file selection -->
</div>
```
  
  ## Component Code
  ```jsx
  <script>
  import { createEventDispatcher } from 'svelte';

  const allowedExtensions = [
    ".js", ".jsx", ".ts", ".tsx", ".py", ".java", ".rb", ".php",
    ".html", ".css", ".cpp", ".c", ".go", ".rs", ".swift", ".kt",
    ".m", ".h", ".cs", ".json", ".xml", ".sh", ".yml", ".yaml",
    ".vue", ".svelte", ".qwik"
  ];
  const maxFiles = 4;

  let selectedFiles = [];
  const dispatch = createEventDispatcher();

  function handleDragOver(event) {
    event.preventDefault();
  }

  async function handleDrop(event) {
    event.preventDefault();
    const items = event.dataTransfer.items;
    const filesArray = await extractFiles(items);
    validateAndSetFiles(filesArray);
  }

  async function extractFiles(items) {
    const filesArray = [];
    for (let i = 0; i < items.length; i++) {
      const item = items[i].webkitGetAsEntry();
      if (item.isFile) {
        const file = await new Promise(resolve => item.file(resolve));
        if (isValidFile(file)) {
          filesArray.push(file);
        } else {
          dispatch('setError', `Invalid file type. Only ${allowedExtensions.join(", ")} files are allowed.`);
        }
      } else if (item.isDirectory) {
        await traverseFileTree(item, filesArray);
      }
    }
    return filesArray;
  }

  function traverseFileTree(item, filesArray) {
    return new Promise(resolve => {
      if (item.isFile) {
        item.file(file => {
          if (isValidFile(file)) {
            filesArray.push(file);
          }
          resolve();
        });
      } else if (item.isDirectory) {
        const dirReader = item.createReader();
        dirReader.readEntries(async entries => {
          for (const entry of entries) {
            await traverseFileTree(entry, filesArray);
          }
          resolve();
        });
      }
    });
  }

  function isValidFile(file) {
    return allowedExtensions.some(ext => file.name.endsWith(ext));
  }

  function handleFileSelect(event) {
    const files = Array.from(event.target.files);
    validateAndSetFiles(files);
  }

  function validateAndSetFiles(files) {
    const filteredFiles = files.filter(file => isValidFile(file));
    if (filteredFiles.length + selectedFiles.length > maxFiles) {
      dispatch('setError', `You can only upload a maximum of ${maxFiles} files.`);
    } else {
      selectedFiles = [...selectedFiles, ...filteredFiles].slice(0, maxFiles);
      dispatch('setError', "");
    }
  }
</script>

<div
  on:dragover|preventDefault={handleDragOver}
  on:drop|preventDefault={handleDrop}
  class="p-4 border-2 border-dashed border-gray-300 rounded-md text-center cursor-pointer mb-4 h-96 w-96 flex overflow-y-scroll items-center justify-center"
>
  <input
    type="file"
    on:change={handleFileSelect}
    style="display: none;"
    accept={allowedExtensions.join(",")}
    id="fileUpload"
    multiple
  />
  <label for="fileUpload" class="cursor-pointer">
    {#if selectedFiles.length > 0}
      <div class="flex flex-wrap gap-10">
        {#each selectedFiles as file}
          <div>
            <DocumentTextIcon class="mx-auto w-8" />
            <p>{file.name}</p>
          </div>
        {/each}
      </div>
    {:else}
      <div>
        <ArrowUpTrayIcon class="mx-auto w-8" />
        <p>Drag & drop files or folders here, or click to select files</p>
      </div>
    {/if}
  </label>
</div>

<style>
/* Add any component-specific styles here */
</style>
  ```
  