# Browser APIs – Practical Notes (File Upload, Download, Copy, Drag & Drop)

These notes cover commonly used **browser APIs** in modern web apps (React / Next.js)
for handling files, clipboard actions, and downloads.

---

## 1. File Upload & Reading (FileReader API)

### Flow Overview
User selects or drops a file → browser provides a `File` object → `FileReader` reads it into memory.

---

### Basic Code Example
```ts
const reader = new FileReader();

reader.onload = (e) => {
  const content = e.target?.result as string;
  setCode(content);
};

reader.readAsText(file);
````

---

### Step-by-Step Explanation

#### `File` object

* Provided by browser via:

  * `<input type="file">`
  * Drag & drop (`dataTransfer.files`)
* Contains metadata:

  * `name`
  * `size`
  * `type`

---

#### `FileReader`

```ts
const reader = new FileReader();
```

* Browser API to read files **asynchronously**
* Does NOT block UI thread

---

#### Register `onload` callback

```ts
reader.onload = (e) => { ... }
```

* Runs after file is fully loaded into memory
* `e.target.result` contains file data

---

#### Start reading

```ts
reader.readAsText(file);
```

* Triggers the read process
* Browser loads file → fills `reader.result` → fires `onload`

---

### Common `FileReader` Methods

| Method              | Use Case                             |
| ------------------- | ------------------------------------ |
| `readAsText`        | `.js`, `.ts`, `.py`, `.csv`, `.json` |
| `readAsDataURL`     | Images                               |
| `readAsArrayBuffer` | Binary / zip files                   |

---

## 2. Drag & Drop File Handling

### Required Events

```tsx
<div
  onDragOver={(e) => e.preventDefault()}
  onDrop={handleDrop}
  onDragLeave={() => setIsDragging(false)}
>
```

---

### Why `preventDefault()` is required

* Without it, browser blocks drop
* Mandatory for `onDrop` to fire

---

### Drop Handler

```ts
const handleDrop = (e: React.DragEvent) => {
  e.preventDefault();
  const file = e.dataTransfer.files[0];
  if (file) handleFile(file);
};
```

---

### Key Concept

> Drag & drop and file picker should **both call the same file handler**

```ts
onDrop → handleFile
onChange → handleFile
```

This avoids duplicated logic.

Note:- Drag & Drop and Input handling both are different and we have to use both for good UX as user can also select the file instead of dragging and dropping.

---

## 3. Copy to Clipboard (Clipboard API)

### Basic Copy Logic

```ts
await navigator.clipboard.writeText(output);
```

---

### How it works

1. Browser writes text to system clipboard
2. Requires user interaction (click)
3. Secure context (HTTPS or localhost)

---

### Example with UX Feedback

```ts
await navigator.clipboard.writeText(text);
toast.success("Copied to clipboard!");
```

---

### Limitations

* Won’t work on insecure origins
* Can fail silently without permissions

---

## 4. Download File in Browser (Blob + URL API)

### Full Download Code

```ts
const blob = new Blob([output], { type: "text/plain" });
const url = URL.createObjectURL(blob);

const a = document.createElement("a");
a.href = url;
a.download = "generated.test.ts";
a.click();

URL.revokeObjectURL(url);
```

---

### Step-by-Step Breakdown

#### Create a Blob

```ts
new Blob([data], { type: "text/plain" });
```

* Converts data into a file-like object in memory

---

#### Create Object URL

```ts
URL.createObjectURL(blob);
```

* Generates a temporary browser URL
* Points to memory, not disk

---

#### Create `<a>` tag

```ts
document.createElement("a");
```

* Browsers allow downloads only via links or user actions

---

#### Set `href` and `download`

```ts
a.href = url;
a.download = "file.ts";
```

* Forces download
* Sets filename

---

#### Trigger click

```ts
a.click();
```

---

#### Cleanup

```ts
URL.revokeObjectURL(url);
```

* Prevents memory leaks

---

### Common MIME Types

| Type | MIME               |
| ---- | ------------------ |
| Text | `text/plain`       |
| JSON | `application/json` |
| CSV  | `text/csv`         |
| ZIP  | `application/zip`  |

---

## 5. Why We Don’t Use Backend for These

✔ Faster
✔ No server cost
✔ No permissions
✔ Works offline
✔ Fully client-side

---

## 6. Important Best Practices

### ✔ Always clean up object URLs

```ts
URL.revokeObjectURL(url);
```

### ✔ Centralize file logic

```ts
handleDrop → handleFile
onChange → handleFile
```

### ✔ Validate before processing

```ts
if (file.size > MAX_SIZE) return;
```

### ✔ Keep UI responsive

* FileReader is async
* Avoid blocking loops

---

## 7. Mental Models (One-liners)

* **Upload**: File → FileReader → State
* **Download**: Data → Blob → URL → `<a>` → Click
* **Copy**: Text → Clipboard API
* **Drag & Drop**: Browser → `dataTransfer.files`

---

## 8. Common Variations (Brief)

### Download JSON

```ts
new Blob([JSON.stringify(data, null, 2)], {
  type: "application/json"
});
```

### Copy fallback (older browsers)

```ts
document.execCommand("copy");
```

### Multi-file upload

```ts
Array.from(e.target.files)
```

---

## Summary

These browser APIs form the backbone of modern file-based UX.
Understanding them avoids:

* Unnecessary backend calls
* Performance issues
* Permission bugs

They are **safe**, **fast**, and **industry standard**.

---

```

---
