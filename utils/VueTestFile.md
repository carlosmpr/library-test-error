---
  title: 'VueTestFile.vue'
  description: 'component description'
  pubDate: 'August 1, 2024'
  author: 'Carlos Polanco'
  ---
  
  
  
  # VueTestFile.vue
  ## Explanation of the Code

### Template Section
- The template section contains a `div` element that serves as a drop zone for files. It has event listeners for `dragover` and `drop` events that call the `handleDragOver` and `handleDrop` methods respectively.
- Inside the `div`, there is an `input` element of type file that is hidden and allows multiple file selection. It has an `@change` event listener that triggers the `handleFileSelect` method.
- The `label` element is used to display the selected files or a message prompting the user to drag and drop files or click to select files.

### Script Section
- The script section imports two icons, `DocumentTextIcon` and `ArrowUpTrayIcon`, from the `@heroicons/vue/24/outline` library.
- The component's data contains an array of allowed file extensions, an array to store selected files, and a maximum number of files that can be selected.
- The component defines several methods:
  - `handleDragOver`: Prevents the default behavior of the dragover event.
  - `handleDrop`: Extracts files from the dropped items, validates them, and sets the selected files.
  - `extractFiles`: Extracts files from the dropped items, handling both files and directories recursively.
  - `traverseFileTree`: Recursively traverses a directory to extract files.
  - `isValidFile`: Checks if a file has a valid extension based on the allowed extensions.
  - `handleFileSelect`: Handles the selection of files using the file input element.
  - `validateAndSetFiles`: Validates selected files against allowed extensions and maximum file count.

### Example Usage
```html
<template>
  <!-- Your template code here -->
</template>

<script>
  export default {
    // Component definition here
  };
</script>

<style scoped>
  /* Component-specific styles here */
</style>
```

### External Libraries
- The code uses the `@heroicons/vue` library to import icons for display in the component.

This code snippet creates a file drop zone component that allows users to drag and drop files or click to select files with specific extensions. It enforces validation rules for file types and maximum file count.
  
  ## Component Code
  ```jsx
  <template>
    <div
      @dragover.prevent="handleDragOver"
      @drop.prevent="handleDrop"
      class="p-4 border-2 border-dashed border-gray-300 rounded-md text-center cursor-pointer mb-4 h-96 w-96 flex overflow-y-scroll items-center justify-center"
    >
      <input
        type="file"
        @change="handleFileSelect"
        style="display: none;"
        :accept="allowedExtensions.join(',')"
        id="fileUpload"
        multiple
      />
      <label for="fileUpload" class="cursor-pointer">
        <div v-if="selectedFiles.length">
          <div class="flex flex-wrap gap-10">
            <div v-for="file in selectedFiles" :key="file.name">
              <DocumentTextIcon class="mx-auto w-8" />
              <p>{{ file.name }}</p>
            </div>
          </div>
        </div>
        <div v-else>
          <ArrowUpTrayIcon class="mx-auto w-8" />
          <p>Drag & drop files or folders here, or click to select files</p>
        </div>
      </label>
    </div>
  </template>
  
  <script>
  import { DocumentTextIcon, ArrowUpTrayIcon } from "@heroicons/vue/24/outline";
  
  export default {
    components: {
      DocumentTextIcon,
      ArrowUpTrayIcon
    },
    data() {
      return {
        allowedExtensions: [
          ".js", ".jsx", ".ts", ".tsx", ".py", ".java", ".rb", ".php",
          ".html", ".css", ".cpp", ".c", ".go", ".rs", ".swift", ".kt",
          ".m", ".h", ".cs", ".json", ".xml", ".sh", ".yml", ".yaml",
          ".vue", ".svelte", ".qwik"
        ],
        selectedFiles: [],
        maxFiles: 4
      };
    },
    methods: {
      handleDragOver(event) {
        event.preventDefault();
      },
      handleDrop(event) {
        const items = event.dataTransfer.items;
        this.extractFiles(items).then(files => this.validateAndSetFiles(files));
      },
      async extractFiles(items) {
        const filesArray = [];
        for (let i = 0; i < items.length; i++) {
          const item = items[i].webkitGetAsEntry();
          if (item.isFile) {
            const file = await new Promise(resolve => item.file(resolve));
            if (this.isValidFile(file)) {
              filesArray.push(file);
            } else {
              this.$emit("setError", `Invalid file type. Only ${this.allowedExtensions.join(", ")} files are allowed.`);
            }
          } else if (item.isDirectory) {
            await this.traverseFileTree(item, filesArray);
          }
        }
        return filesArray;
      },
      traverseFileTree(item, filesArray) {
        return new Promise(resolve => {
          if (item.isFile) {
            item.file(file => {
              if (this.isValidFile(file)) {
                filesArray.push(file);
              }
              resolve();
            });
          } else if (item.isDirectory) {
            const dirReader = item.createReader();
            dirReader.readEntries(async entries => {
              for (const entry of entries) {
                await this.traverseFileTree(entry, filesArray);
              }
              resolve();
            });
          }
        });
      },
      isValidFile(file) {
        return this.allowedExtensions.some(ext => file.name.endsWith(ext));
      },
      handleFileSelect(event) {
        const files = Array.from(event.target.files);
        this.validateAndSetFiles(files);
      },
      validateAndSetFiles(files) {
        const filteredFiles = files.filter(file => this.isValidFile(file));
        if (filteredFiles.length > this.maxFiles) {
          this.$emit("setError", `You can only upload a maximum of ${this.maxFiles} files.`);
        } else {
          this.selectedFiles = filteredFiles.slice(0, this.maxFiles);
          this.$emit("setError", "");
        }
      }
    }
  };
  </script>
  
  <style scoped>
  /* Add any component-specific styles here */
  </style>
  ```
  