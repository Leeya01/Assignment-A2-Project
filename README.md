**Local Region Image Editing**

Assignment 2 prototype for Intelligent Interactive Systems. The idea: a designer selects one region of an AI-generated image and the AI edits only that area, so one wrong detail can be fixed without regenerating the whole image.

README prose word count: 721 (code and setup commands not counted)

**Project status**

Done (notebook experiments, Kaggle):

Paste-back step: the edited area is copied back onto the original, so nothing outside the mask changes
Precision check: raw model output changed pixels outside the mask by 6.086 (poster text), 5.451 (mug) and 5.214 (added object) on a 0 to 255 scale. After paste-back the change was 0.0
Removal with LaMa, replacing an object with SDXL inpainting, and exact text with a font renderer
Failed edits kept as evidence (garbled poster text, mug that stayed white, removal that invented objects)

Not built yet: the Gradio app, brush, before and after view, changed-area highlight, undo, and the simulated suggested regions with a tick list. These are planned, not implemented.

Dropped: recolouring. Prompting failed twice, colour selection tinted the whole box, and SAM masks recoloured whole books including their pages.

**Technology stack**
Kaggle notebook, Tesla T4 GPU: where everything runs
Stable Diffusion 1.5 inpainting (stable-diffusion-v1-5/stable-diffusion-inpainting, via diffusers): first model, used for the text and mug tests
SDXL inpainting (via diffusers, model ID: SDXL_ID): adding or replacing an object with better style matching
LaMa (simple-lama-inpainting): removal
Segment Anything (facebook/sam-vit-base, via transformers): object masks for the dropped recolour tests
Pillow, NumPy, matplotlib: masks, paste-back, font rendering, plots
Undecided: how the Gradio app will be hosted (Kaggle share link or local)

**Running or inspecting the work**
Inspect without running: open notebooks/01-inpainting.ipynb. Results are saved in the outputs, and images are in evidence/
To rerun: Kaggle notebook with the accelerator set to GPU T4 and Internet on. Upload testImage.png as a dataset and update image_path in the notebook
LaMa install (pip prints dependency warnings, which can be ignored):

pip install -q --no-deps simple-lama-inpainting
pip install -q opencv-python-headless fire
- Run cells top to bottom. The SDXL cells need the cell that loads pipe_xl to run first

**Development record**
Kaggle accelerator: the first run failed because the accelerator was off. Switched to GPU T4 and added a CPU fallback
Garbled text: Stable Diffusion 1.5 repeated the poster word and misspelled both copies. Kept as a failure example, and later text edits use a font renderer
Leakage: the raw output changed areas outside the mask (6.086 against 0.0 after paste-back). Paste-back became a fixed step in every edit
Removal: SD 1.5 invented objects in the mug area every time. Switched removal to LaMa, which fills from the surroundings, and used a mask with a margin
Adding objects: moved to SDXL, which matched style and lighting better. Replacing the mug in one step looked cleaner than removing then adding
Recolouring dropped: three methods tried (prompt, hue range, SAM masks plus hue shift). Kept the lesson that masks can cover too much, so the interface needs a preview and a way to correct them
Text: tried font only, font plus SDXL pass, then printed-ink style (cream colour, transparency, blur, grain) on the moon poster. Several of my boxes landed on the wrong surface, which I only found by drawing the box on the image

Key commits: COMMIT_LINKS

**AI assistance**
Test image: generated with ChatGPT image generation (version and date: IMAGE_VERSION_DATE). Prompt, as written: "Generate a cosy, aesthetically pleasing study space with a desk, laptop, books, a mug, plants and decorative wall posters. Use warm sunset lighting, soft pastel colours and a detailed illustration style." I resized it to 512 by 512 for the inpainting model and did not edit or retouch it.

Test image

Code and notes: I used Claude to help write the notebook code and notes. Examples of what I checked and changed:

The hue-range recolour selected the whole box (7475 pixels, a solid rectangle), so I switched to SAM masks
SAM's highest-scoring mask was the wrong object. I chose candidate 2 by looking at the overlays
A suggested text box landed on the wrong poster. I checked it on the image and moved it
I dropped recolouring after judging it did not support the main interaction

**Additional files**
notebooks/01-inpainting.ipynb: the experiments
evidence/: saved comparison images used in the report

**API key note**
No Hugging Face token is needed at the moment. If one is needed later, store it as an environment variable and never commit it.
