# 🌐 Cloudonix Doc

The official documentation portal for the **Cloudonix Upload SDK**. Built with React, Vite, and Tailwind CSS v4, this portal provides developers with an interactive, bilingual, and responsive guide on how to integrate and use the Cloudonix storage services.

---

## 🚀 Live Links

- **Developer Upload Portal:** [https://cloudonix.shivam-75.workers.dev](https://cloudonix.shivam-75.workers.dev)
- **Official Package (NPM):** [https://www.npmjs.com/package/cloudonix](https://www.npmjs.com/package/cloudonix)

---

## 📦 Cloudonix Upload SDK (The Package)

The `cloudonix` library is a developer-friendly client SDK designed to facilitate uploading images, videos, and PDFs to the Cloudonix Cloud Storage Server. It supports both CommonJS (`require`) and ES Modules (`import`) systems, and works natively in Node.js, Bun, and frontend environments.

### 📥 Package Installation

You can install the SDK in your project using any of the major package managers:

```bash
# Using npm
npm install cloudonix

# Using yarn
yarn add cloudonix

# Using pnpm
pnpm add cloudonix
```

### ⚡ Quick Usage Examples

#### 1. ES Modules (ESM) / Bun
```javascript
import cloudonix from 'cloudonix';

// Initialize the uploader client
const uploader = new cloudonix({
  api_key: 'YOUR_CLOUDONIX_API_KEY' // Obtain from your dashboard profile
});

// Upload standard File or Blob
const response = await uploader.upload(file);

if (response.success) {
  console.log('Uploaded image URL:', response.data.data.imageUrl);
} else {
  console.error('Upload error:', response.error);
}
```

#### 2. CommonJS (CJS) / Node.js
```javascript
const cloudonix = require('cloudonix');
const fs = require('node:fs');

const uploader = new cloudonix({
  api_key: 'YOUR_CLOUDONIX_API_KEY'
});

async function main() {
  const buffer = fs.readFileSync('./my-image.png');
  const file = new File([buffer], 'my-image.png', { type: 'image/png' });

  const response = await uploader.upload(file);
  console.log('Upload response:', response)
}
main();
```

#### 3. Express.js API Route (with Multer Uploads)
Use this setup to handle file uploads in an Express.js backend using Multer (disk storage):

```bash
npm install express multer
```

```javascript
const express = require('express');
const multer = require('multer');
const cloudonix = require('cloudonix');
const fs = require('node:fs');
const path = require('node:path');

const app = express();

// Configure multer to save files on disk
const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    const uploadDir = './uploads';
    // Ensure the upload directory exists
    if (!fs.existsSync(uploadDir)) {
      fs.mkdirSync(uploadDir, { recursive: true });
    }
    cb(null, uploadDir);
  },
  filename: (req, file, cb) => {
    // Generate a unique filename using timestamp and random number
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1e9);
    cb(null, file.fieldname + '-' + uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({ storage: storage });

const uploader = new cloudonix({
  api_key: 'YOUR_CLOUDONIX_API_KEY'
});

app.post('/upload', upload.single('file'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ success: false, error: 'No file uploaded' });
    }

    // Read the uploaded file from disk and convert to standard File object
    const buffer = fs.readFileSync(req.file.path);
    const file = new File([buffer], req.file.originalname, {
      type: req.file.mimetype
    });

    // Upload to Cloudonix Vault
    const response = await uploader.upload(file);

    if (response.success) {
      res.status(200).json({
        success: true,
        message: 'Uploaded successfully!',
        url: response.data.data.imageUrl // URL of the uploaded file
      });
    } else {
      res.status(500).json({ success: false, error: response.error });
    }
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  } finally {
    // Always clean up the temporary file from the disk after upload/error
    if (req.file && req.file.path && fs.existsSync(req.file.path)) {
      try {
        fs.unlinkSync(req.file.path);
      } catch (cleanupError) {
        console.error('Error deleting temporary file:', cleanupError);
      }
    }
  }
});

app.listen(3000, () => {
  console.log('Server is running on http://localhost:3000');
});
```

---

## ✨ Features of the Documentation Site

- 🌐 **Bilingual Translation Support (EN / HI):** Completely localized in English and Hindi. Toggle languages instantly via the navigation bar.
- 🌗 **Dynamic Theme Toggler:** Supports seamless transitions between Dark Mode (custom dark slate aesthetics) and Light Mode. Settings persist via `localStorage`.
- 📱 **Fluid & Responsive Layout:** Fully optimized for mobile, tablet, and desktop screens with custom hamburger slide-out navigation for smaller viewports.
- 📋 **Zero-Dependency Copyable Code Blocks:** Code view blocks with interactive "Copy" functionality and micro-interactions.
- 🗺️ **Seamless Page Routing:** Structured navigation using React Router v7 detailing:
  - **Introduction:** SDK Overview & key advantages of Cloudonix Vault.
  - **Installation:** Package manager options and basic setup guidelines.
  - **Usage & API:** Developer guide for Node.js, Bun, and frontend implementations.
  - **Response & Error Handling:** Detailed response schemas and HTTP error resolutions.
  - **Best Practices:** API key security and image compression strategies.

