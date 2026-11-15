# Local Region Image Editing

Assignment 2 prototype for Intelligent Interactive Systems.

The idea is to let a designer select a specific region of an AI-generated image and edit only that area. A single wrong detail can then be fixed without regenerating the whole image.

**README prose word count:** 968 (code and setup commands not included).

## Project status

**Completed (Kaggle notebook)**

- **Paste-back step:** The edited region is copied back onto the original image, so nothing outside the mask changes.
- **Precision check:** The raw model output changed pixels outside the mask by 6.086 (poster text), 5.451 (mug) and 5.214 (added object) on a 0 to 255 scale. After paste-back the change was 0.0 in all three tests.
- **Removal:** LaMa removes objects.
- **Replacement and addition:** SDXL inpainting replaces an object or adds a new one.
- **Text editing:** A font renderer gives more accurate text.
- **Failure evidence:** Unsuccessful results are kept, including garbled poster text, a mug that stayed white and removals that invented new objects.
- **Gradio interface (partly built):** Image upload and a brush work. The painted strokes are turned into a black and white mask, shown as a preview.

**Not implemented yet (planned)**

- Instruction box and apply button connected to the inpainting step.
- Before and after comparison.
- Highlight of the changed area.
- Undo and keep original.
- Simulated suggested regions with a tick list. These will be prepared boxes and labels, not real AI detection.

**Dropped:** Recolouring, after several failed attempts (see development record).

## Technology stack

- **Kaggle notebooks:** Main coding environment, using a Tesla T4 GPU.
- **Stable Diffusion 1.5 Inpainting** (`stable-diffusion-v1-5/stable-diffusion-inpainting`, via Diffusers): First inpainting model, used for the text and mug experiments.
- **SDXL Inpainting** (via Diffusers, model ID: `SDXL_ID`): Adds and replaces objects, with better style and lighting consistency.
- **LaMa** (`simple-lama-inpainting`): Object removal.
- **Segment Anything** (`facebook/sam-vit-base`, via Transformers): Object masks for the recolouring experiments.
- **PyTorch and Diffusers:** Load and run the inpainting models.
- **Pillow, NumPy and Matplotlib:** Image processing, masks, paste-back, font rendering and plots.
- **Gradio:** Python library that builds the interface. Its image editor provides the brush widget. The app is opened through the temporary `gradio.live` link that Kaggle prints, because the local `127.0.0.1` link cannot be reached from Kaggle.
- **GitHub:** Repository and version control.

## Running or inspecting the work

**Inspect without running:** open `notebooks/01-inpainting.ipynb`. Results are in `outputs/` and comparison images are in `evidence/`.

**To rerun:**

1. Open the notebook in Kaggle.
2. Set the accelerator to GPU T4 and turn Internet on.
3. Upload `testImage.png` as a Kaggle dataset and update `image_path` in the notebook.
4. Run the cells from top to bottom. The SDXL cells need the cell that loads `pipe_xl` to run first.
5. For the interface, run the Gradio cells and open the `gradio.live` link in the output.

**LaMa installation** (pip may show dependency warnings, which were present during setup):

```bash
pip install -q --no-deps simple-lama-inpainting
pip install -q opencv-python-headless fire
```

## Development record

- **Kaggle accelerator:** The first run failed because the accelerator was disabled. Switched to GPU T4 and added a CPU fallback.
- **Garbled text:** Stable Diffusion 1.5 repeated the poster word and misspelled both copies. Kept as a failure example. Later text edits used a font renderer.
- **Pixel leakage:** Raw output changed areas outside the mask (6.086 against 0.0 after paste-back), so paste-back became a fixed step in every edit.
- **Object removal:** Stable Diffusion 1.5 kept inventing objects when removing the mug. Switched to LaMa, which fills the area from the surroundings, with a margin added to the mask.
- **Adding objects:** Switched to SDXL for better style and lighting. Replacing the mug in one step gave a cleaner result than removing it and then adding a new one.
- **Recolouring:** Tried prompting, hue-range selection and SAM masks with hue shifting. All had limits. Masks can cover more than the intended object, which shows the need for a preview and a way to correct selections.
- **Text editing:** Tested font rendering alone, font rendering plus an SDXL pass, and a printed-ink effect on the moon poster. Some boxes landed on the wrong surface, only noticed after drawing them over the original.
- **Interface, brush:** Built the Gradio upload and brush in small sections and checked the mask preview against what I painted. Chose the Kaggle share link for hosting.

**Key commits:** `COMMIT_LINKS`

## AI assistance and attribution

**Test image:** Generated with ChatGPT image generation. Version and date: `IMAGE_VERSION_DATE`. Prompt: "Generate a cosy, aesthetically pleasing study space with a desk, laptop, books, a mug, plants and decorative wall posters. Use warm sunset lighting, soft pastel colours and a detailed illustration style." Resized to 512 × 512 pixels, with no other edits.

**Coding help:** I used Claude to help fix errors and to write the first version of the Gradio code, one section at a time. I ran everything, checked it against the outputs and changed what was wrong. Examples:

- **Hue-range recolouring:** The selection covered the whole bounding box (7,475 pixels) and made a solid rectangle, so I switched to SAM masks.
- **SAM mask choice:** The highest-scoring mask picked the wrong object. I inspected the overlays and chose candidate 2.
- **Gradio import error:** The brush cell failed with `NameError: name 'gr' is not defined` because the import was missing. I added it and reran.
- **Pip warning:** Gradio's install printed a `gym` version conflict. I checked that nothing in the project uses it and ignored it.

**Other tools:** Grammarly for spelling and grammar.

**Reused material:** The brush and editor widget come from Gradio. The app design, the code that turns strokes into a mask, the model integration and the planned feedback and control features are my own work. The models and libraries are listed in the tech stack.

## Additional files

- `notebooks/01-inpainting.ipynb`: the image editing experiments and interface code.
- `evidence/`: saved comparison images used in the report.

## API key note

No Hugging Face token is needed at the moment. If one is needed later, it goes in an environment variable and is never committed.
