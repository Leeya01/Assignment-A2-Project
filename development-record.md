# Development record

Written up on 23 Nov 2026 from my notebook history and screenshots.

**How this record was made:** The work was completed in Kaggle notebooks on the dates listed below. The files were uploaded to GitHub in one go on 23 Nov 2026 through the web interface, so the repository had a single commit rather than one per day. The dates below came from my Kaggle notebook version history and the creation dates of my screenshots, which were captured with Snipping Tool at each completed version of the app. Submission commit: [hash].

**15 Nov 2026 (completed):** Set up the repo, README stub, test image and inpainting model. Built the paste-back step and precision check (the raw output changed about 5 to 6 outside the mask, compared with 0.0 after paste-back). Compared two settings. Built the first Gradio interface in a new Kaggle notebook, with image upload, brush, instruction box and an apply button connected to SDXL.

**15 Nov 2026 (completed):** Added before-and-after comparisons, the red highlight, seed box, guidance slider and Keep editing this result option. Added undo, back to original, a status line and simulated suggestions with a tick list. Decided to keep the suggestions simulated and list an object detector as planned work. Found that full-detail 1024 processing made edits blend away, so the editing process used 512 followed by 1024. Took screenshots of every state.

**17 Nov 2026 (completed):** Compared brush and box masks, ran the underwater tests that failed and kept them as evidence, and created the system diagram and storyboards.

**17 Nov 2026 (completed, deadline extended):** Added three variants with a picker, edit history with thumbnails, the mask check and the settings log with export. Tested each feature and checked that undo and the precision check still worked. Updated the diagram and storyboards. The gradio.live link returned 504 errors for a while, but restarting the session fixed the issue.

**23 Nov 2026 (completed):** Wrote the rationale, abstract and README. Tagged the submission as `a2-submission`.
