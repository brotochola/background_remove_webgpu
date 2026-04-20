# background_remove_webgpu

Remove image backgrounds in the browser. No upload to a server: models run locally via WebGPU or WebAssembly (ONNX). The app is a single static page (`index.html`) with an English UI.

## Requirements

- A **modern Chromium-based browser** (Chrome, Edge, Brave, etc.) with **WebGPU** enabled for the fastest path (MODNet). Safari on iOS uses the WASM fallback only.
- **Network access** on first use: the page loads `@huggingface/transformers` from a CDN and downloads model weights from Hugging Face (on the order of tens to ~100 MB depending on the model).
- **HTTPS or localhost** is recommended for WebGPU and for fewer mixed-content issues; serving over plain `file://` may break module loading or model fetch behavior depending on the browser.

## Run locally

Serve this folder with **any static HTTP server** (nginx, Caddy, `serve`, your IDE’s “live preview”, etc.). Do not open `index.html` via `file://`, or ES modules and remote assets may fail to load.

From the project directory, examples:

```bash
npx --yes serve -p 8080
```

```bash
python -m http.server 8080
```

Then open the URL the server prints (often `http://localhost:8080`). Point the server’s document root at this folder if you configure a virtual host or reverse proxy.

## How to use

1. Open the app in the browser as above.
2. Optional: choose **RMBG-1.4 (WASM)** or, if WebGPU is available and you are not on iOS, **MODNet (WebGPU)**.
3. Drag and drop images onto the drop zone, or click to select one or more files (JPG, PNG, WebP). The model runs once per image; the raw mask is kept in memory for that result.
4. Wait for the first-time model download if prompted; later runs use cached assets when possible.
5. **Matte tuning (per image):** after each result appears, open **Matte tuning** directly under that preview. Sliders affect only that image and update the cutout **live** — no re-upload. Pipeline: **levels → contrast → gamma → feather**:
   - **Alpha black / white level** — remaps the matte like levels in an image editor: raise black to remove leftover background haze; lower white to soften a too-hard edge.
   - **Matte contrast** — pushes semitransparent pixels toward full transparency or full opacity (1.0 = no change).
   - **Preserve foreground (gamma)** — curve on opacity after the above; values below 1 weaken the foreground slightly; above 1 strengthen it.
   - **Edge feather** — softens the alpha boundary by a few pixels (box blur on the alpha channel only).
6. Download each result as PNG from the card (files are named `*-no-bg.png`), or use **Clear all** to remove every result.

## Behavior and models

- **MODNet (`Xenova/modnet`, WebGPU)**  
  Used when WebGPU is available and the device is not treated as iOS. Faster on supported GPUs.

- **RMBG-1.4 (`briaai/RMBG-1.4`, ONNX WASM)**  
  Used on iOS, when WebGPU is missing, or when MODNet initialization fails. Runs on CPU via WASM; can be slower but is broadly compatible.

The UI may show a badge indicating whether WebGPU was detected.

## Privacy

Images are processed in your browser. They are not sent to this project’s server (there is no backend). Third-party CDNs and Hugging Face still receive requests for scripts and model files when you load or update models.

## Troubleshooting

- **Blank page or module errors**  
  Confirm you are using `http://localhost/...` or another HTTP URL, not `file:///...`.

- **WebGPU option missing or fallback message**  
  Update the browser, enable WebGPU if your build requires a flag, or use the WASM model.

- **Slow first run**  
  Expected while models download; keep the tab open until loading finishes.

- **Out of memory on very large images**  
  Try smaller resolution inputs.

## License

Refer to the licenses of [Transformers.js](https://github.com/huggingface/transformers.js), the **MODNet** and **RMBG-1.4** model cards on Hugging Face, and your chosen browser/runtime.
