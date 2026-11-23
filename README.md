# Local Region Image Editing

Assignment 2 prototype for Intelligent Interactive Systems.

The idea is to let a designer select a specific region of an AI-generated image and edit only that area, so a single wrong detail can be fixed without regenerating the whole image. The user paints over the area, types a short instruction and gets back an image where only the painted area has changed. The app shows what changed, keeps a record of every step and lets the user go back.

**README prose word count:** 1744 (code and setup commands not included).

## Project status

The interface is finished and runs in a Kaggle notebook. The table below matches the system diagram in `evidence/`, using the same three labels: implemented, simulated and planned.

| Part | What it does | Status |
|---|---|---|
| Brush mask and paste-back | The painted strokes become a mask. The edit is pasted onto the original, so nothing outside the mask changes | Implemented |
| Precision check | Measures the average change outside the mask. Raw model output changed by about 5 to 6 on a 0 to 255 scale, and after paste-back it is 0.0 | Implemented |
| Instruction, seed and guidance | A text box for what should appear, a seed box and a guidance slider | Implemented |
| SDXL inpainting | Fills the painted area. It is the only random step | Implemented |
| Before and after, red highlight, status line | Shows both images, marks the changed area in red and reports what happened | Implemented |
| Mask check | Warns if the mask is tiny, huge or over plain background. It only advises and never blocks | Implemented |
| Three variants | Runs the same edit with the seed, seed + 1 and seed + 2, then the user picks one | Implemented |
| History, undo, back to original, keep editing | Thumbnails of every earlier step, click one to go back. Undo, Back to original and Keep editing this result also work | Implemented |
| Settings log and export | Records the instruction, seed, guidance and result check for every action, and exports them as `edit_log.json` | Implemented |
| Suggested regions and tick list | Hand-made boxes for the two demo images. The user ticks which ones to apply | Simulated |
| Object detector | Would find the suggested regions automatically | Planned |

**Earlier experiments (notebook 1):** LaMa removes objects, SDXL replaces or adds objects, and a font renderer gives more accurate text. Unsuccessful results are kept as failure evidence, including garbled poster text, a mug that stayed white and removals that invented new objects.

**Dropped:** Recolouring, after several failed attempts (see development record).

## Known limitations

- **Adding new objects to empty space is unreliable.** Replacing something that is already there works much better.
- **The mask check is a simple rule, so it can be wrong.** It marks the painted area as plain background when the detail level is low. On the cosy study it warned about an empty patch of desk, but adding a pen there still worked.
- **Edits run at 512 then 1024 for SDXL.** Full-detail 1024 made edits blend away. The cost is that the result keeps the smaller working size and loses fine detail.
- **Three variants take about three times as long** as one edit, because each one is a full SDXL run.
- **The suggestions are simulated.** The boxes are prepared by hand for the two demo images.
- **The share link is temporary.** The `gradio.live` link sometimes hangs or gives a 504 error, and the app only runs while the Kaggle session is open.

## Technology stack

- **Kaggle notebooks:** Main coding environment, using a Tesla T4 GPU.
- **Stable Diffusion 1.5 Inpainting** (`stable-diffusion-v1-5/stable-diffusion-inpainting`, via Diffusers): First inpainting model, used for the text and mug experiments.
- **SDXL Inpainting** (via Diffusers, model ID: `SDXL_ID`): Adds and replaces objects in the interface, with better style and lighting consistency.
- **LaMa** (`simple-lama-inpainting`): Object removal.
- **Segment Anything** (`facebook/sam-vit-base`, via Transformers): Object masks for the recolouring experiments.
- **PyTorch and Diffusers:** Load and run the inpainting models.
- **Pillow, NumPy and Matplotlib:** Image processing, masks, paste-back, font rendering and plots.
- **Gradio:** Python library that builds the interface. Its image editor provides the brush, and its blocks, galleries and buttons make up the rest of the layout. The app is opened through the temporary `gradio.live` link that Kaggle prints, because the local `127.0.0.1` link cannot be reached from Kaggle.
- **GitHub:** Repository and version control.

## Running or inspecting the work

**Inspect without running:** open `notebooks/01-inpainting.ipynb` for the experiments and `notebooks/02-interface.ipynb` for the app. Results are in `outputs/`. The storyboards, system diagram, screenshots and an example settings log are in `evidence/`.

**To rerun:**

1. Open the notebook in Kaggle.
2. Set the accelerator to GPU T4 and turn Internet on.
3. Upload the test images as a Kaggle dataset and update the image paths in the notebook.
4. Run the cells from top to bottom. The SDXL cells need the cell that loads `pipe_xl` to run first.
5. Run the final app cell last, then open the `gradio.live` link in the output. If the page hangs, restart the session and run it again for a new link.

**LaMa installation** (pip may show dependency warnings, which were present during setup):

```bash
pip install -q --no-deps simple-lama-inpainting
pip install -q opencv-python-headless fire
```

## Evidence

- `evidence/system-architecture.png`: system diagram with every part labelled implemented, simulated or planned.
- `evidence/storyboard-1-successful-edit.png`, `evidence/storyboard-2-warning-and-variants.png` and `evidence/storyboard-3-suggestions-simulated.png`: the three storyboards, using screenshots from the final app.
- `evidence/edit_log.json`: an example settings log, with the model name and every edit, seed and guidance value.
- Comparison images and failure examples from the experiments.

## Development record

- **Kaggle accelerator:** The first run failed because the accelerator was disabled. Switched to GPU T4 and added a CPU fallback.
- **Garbled text:** Stable Diffusion 1.5 repeated the poster word and misspelled both copies. Kept as a failure example. Later text edits used a font renderer.
- **Pixel leakage:** Raw output changed areas outside the mask (about 5 to 6 against 0.0 after paste-back), so paste-back became a fixed step in every edit.
- **Object removal:** Stable Diffusion 1.5 kept inventing objects when removing the mug. Switched to LaMa, which fills the area from the surroundings, with a margin added to the mask.
- **Adding objects:** Switched to SDXL for better style and lighting. Replacing the mug in one step gave a cleaner result than removing it and then adding a new one. A first succulent prompt made the model remove the leaves and fill the area with background, and a more specific prompt that matched the scene's style worked in all four seeds.
- **Recolouring:** Tried prompting, hue-range selection and SAM masks with hue shifting. All had limits. Masks can cover more than the intended object, which shows the need for a preview and a way to correct selections.
- **Text editing:** Tested font rendering alone, font rendering plus an SDXL pass, and a printed-ink effect on the moon poster. Some boxes landed on the wrong surface, only noticed after drawing them over the original.
- **Interface, first version:** Built the Gradio upload, brush, instruction box, apply button, before and after view, red highlight, seed box, guidance slider and Keep editing this result button, then added undo, back to original, a status line and the simulated suggestions with a tick list. Checked the mask preview against what I painted.
- **Mask check:** Added a warning for masks that are tiny, huge or over plain background. The thresholds came from trying it on my test images. It only warns, because the pen test warned about plain background and still worked.
- **Variants:** Added three variants with different seeds and a picker. Each one goes through the same edit step, so the paste-back guarantee holds for all three. They take about three times as long, so the status line says so.
- **History:** Added a thumbnail strip so the user can click any earlier step. Going back drops the later steps, the same as pressing Undo several times.
- **Settings log:** Added a log of every edit, variant, suggestion, undo, jump and keep-editing action, with an export to `edit_log.json`. An edit that fails, for example with no mask, is not logged.
- **Share link:** The `gradio.live` link sometimes timed out with a 504 error even though the app had started. Restarting the session and launching again gave a fresh link.

**Key commits:** `COMMIT_LINKS`

**Submission release:** the `a2-submission` tag.

## AI assistance and attribution

**Test images:** The cosy study was generated with ChatGPT image generation. Version and date: `IMAGE_VERSION_DATE`. Prompt: "Generate a cosy, aesthetically pleasing study space with a desk, laptop, books, a mug, plants and decorative wall posters. Use warm sunset lighting, soft pastel colours and a detailed illustration style." Resized to 512 × 512 pixels, with no other edits. The underwater shipwreck scene: `UNDERWATER_IMAGE_SOURCE`.

**Coding and writing help:** I used Claude to help fix errors and to draft the report version. It also helped draft the rationale table, the storyboard ideas, and the first version of the system diagram, which I then edited. I ran everything, checked it against the outputs and changed what was wrong. Examples:

- **Hue-range recolouring:** The selection covered the whole bounding box (7,475 pixels) and made a solid rectangle, so I switched to SAM masks.
- **SAM mask choice:** The highest-scoring mask picked the wrong object. I inspected the overlays and chose candidate 2.
- **Gradio import error:** The brush cell failed with `NameError: name 'gr' is not defined` because the import was missing. I added it and reran.
- **Pip warning:** Gradio's install printed a `gym` version conflict. I checked that nothing in the project uses it and ignored it.
- **Mask check and log tests:** I tested the warnings on my own images and checked that failed edits did not add rows to the log.

**Other tools:** Grammarly for spelling and grammar.
**Digrams:** I created the digrams(sotryboard and artitecture digram) in Canva. 


**Reused material:** The brush and editor widget, and the gallery, button and table components, come from Gradio. The app design, the code that turns strokes into a mask, the paste-back and precision check, the model integration and the feedback and control features are my own work. The models and libraries are listed in the tech stack.

## Additional files

- `notebooks/01-inpainting.ipynb`: the image editing experiments.
- `notebooks/02-interface.ipynb`: the interface and its features.
- `evidence/`: the system diagram, storyboards, screenshots, an example settings log and saved comparison images used in the report.

## API key note

No Hugging Face token is needed at the moment. If one is needed later, it goes in an environment variable and is never committed.
