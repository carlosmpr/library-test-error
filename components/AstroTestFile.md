---
  title: 'AstroTestFile.astro'
  description: 'component description'
  pubDate: 'August 1, 2024'
  author: 'Carlos Polanco'
  ---
  
  
  
  # AstroTestFile.astro
  ## コードの概要

このコードは、ファイルをドラッグ＆ドロップまたはファイル選択ダイアログを使用してアップロードするためのファイルドロップゾーンコンポーネントを定義しています。許可されているファイルの拡張子と最大ファイル数が指定されており、選択されたファイルがこれらの条件を満たすかどうかが検証されます。

### 使用法

```jsx
<FileDropZone />
```

### 依存関係

- React
- @heroicons/react

### 注意

このコードはファイルのドラッグ＆ドロップ機能を実装するためのものであり、UI部分はTailwind CSSのクラスを使用してスタイリングされています。
  
  ## Component Code
  ```jsx
  ---
import { useState } from 'react';
import { DocumentTextIcon, ArrowUpTrayIcon } from '@heroicons/react/24/outline';

const allowedExtensions = [
  ".js", ".jsx", ".ts", ".tsx", ".py", ".java", ".rb", ".php",
  ".html", ".css", ".cpp", ".c", ".go", ".rs", ".swift", ".kt",
  ".m", ".h", ".cs", ".json", ".xml", ".sh", ".yml", ".yaml",
  ".vue", ".svelte", ".qwik"
];
const maxFiles = 4;

const FileDropZone = () => {
  const [selectedFiles, setSelectedFiles] = useState([]);
  const [error, setError] = useState('');

  const handleDragOver = (e) => {
    e.preventDefault();
  };

  const handleDrop = async (e) => {
    e.preventDefault();
    const items = e.dataTransfer.items;
    const filesArray = await extractFiles(items);
    validateAndSetFiles(filesArray);
  };

  const extractFiles = async (items) => {
    const filesArray = [];
    for (let i = 0; i < items.length; i++) {
      const item = items[i].webkitGetAsEntry();
      if (item.isFile) {
        const file = await new Promise((resolve) => item.file(resolve));
        if (isValidFile(file)) {
          filesArray.push(file);
        } else {
          setError(`Invalid file type. Only ${allowedExtensions.join(", ")} files are allowed.`);
        }
      } else if (item.isDirectory) {
        await traverseFileTree(item, filesArray);
      }
    }
    return filesArray;
  };

  const traverseFileTree = (item, filesArray) => {
    return new Promise((resolve) => {
      if (item.isFile) {
        item.file((file) => {
          if (isValidFile(file)) {
            filesArray.push(file);
          }
          resolve();
        });
      } else if (item.isDirectory) {
        const dirReader = item.createReader();
        dirReader.readEntries(async (entries) => {
          for (const entry of entries) {
            await traverseFileTree(entry, filesArray);
          }
          resolve();
        });
      }
    });
  };

  const isValidFile = (file) => allowedExtensions.some((ext) => file.name.endsWith(ext));

  const handleFileSelect = (e) => {
    const files = Array.from(e.target.files);
    validateAndSetFiles(files);
  };

  const validateAndSetFiles = (files) => {
    const filteredFiles = files.filter(isValidFile);
    if (filteredFiles.length + selectedFiles.length > maxFiles) {
      setError(`You can only upload a maximum of ${maxFiles} files.`);
    } else {
      setSelectedFiles((prev) => [...prev, ...filteredFiles].slice(0, maxFiles));
      setError("");
    }
  };

  return (
    <div
      onDragOver={handleDragOver}
      onDrop={handleDrop}
      class="p-4 border-2 border-dashed border-gray-300 rounded-md text-center cursor-pointer mb-4 h-96 w-96 flex overflow-y-scroll items-center justify-center"
    >
      <input
        type="file"
        onChange={handleFileSelect}
        style={{ display: "none" }}
        accept={allowedExtensions.join(",")}
        id="fileUpload"
        multiple
      />
      <label for="fileUpload" class="cursor-pointer">
        {selectedFiles.length > 0 ? (
          <div class="flex flex-wrap gap-10">
            {selectedFiles.map((file) => (
              <div key={file.name}>
                <DocumentTextIcon class="mx-auto w-8" />
                <p>{file.name}</p>
              </div>
            ))}
          </div>
        ) : (
          <div>
            <ArrowUpTrayIcon class="mx-auto w-8" />
            <p>Drag & drop files or folders here, or click to select files</p>
          </div>
        )}
      </label>
    </div>
  );
};

export default FileDropZone;
---

<FileDropZone />
  ```
  