# NAMTrainerColab
A nice, user-friendly NAM Trainer to be used on Google Colab — set up for **batch training**:
put `input.wav`, all of your reamped capture files, and a `captures.json` describing each one in
your Google Drive, then run a single cell to train every model and upload it straight back to
Drive.

[Open on Colab](https://colab.research.google.com/github/sdatkinson/NAMTrainerColab/blob/main/notebook.ipynb)

## Batch workflow

1. **Open the notebook in Colab** (use the badge above, pointed at your fork) and set
   **Runtime → Change runtime type → GPU**.

2. **Get the reamp signal.** Download
   [input.wav](https://drive.google.com/file/d/1KbaS4oXXNEuh2aCPLwKrPdf5KFOjda8G/view?usp=drive_link).

3. **Reamp your gear.** For every piece of gear or tone you want to model, reamp it through
   `input.wav` and save the result as its own file (e.g. `output_be100_lead.wav`,
   `output_jcm800_clean.wav`). Use 48kHz, 24-bit, mono. (For other sample rates, use
   [the CLI trainer](https://github.com/sdatkinson/neural-amp-modeler).)

4. **Create `captures.json`.** Copy [`captures.example.json`](captures.example.json) and fill it
   in — see [Manifest format](#manifest-format) below.

5. **Get everything into Google Drive.** Create a folder named **`NAM_uploads`** at the top level
   of `MyDrive`, and put `input.wav`, every `output_*.wav` file, and `captures.json` in it.

   For a large batch, use the **Google Drive desktop app**: drop the files into the
   locally-synced `NAM_uploads` folder and they'll upload in the background at your full internet
   speed. Colab's browser upload widget is *much* slower for many large files (especially in
   Firefox), so this avoids a long, fragile upload session.

   *Quick one-off test:* you can instead upload these files directly into the Colab file browser
   (folder icon on the left) at the `/content` root — the notebook falls back to that if
   `MyDrive/NAM_uploads/captures.json` isn't found.

6. **Run the single training cell.** The first time, you'll be asked to authorize Google Drive
   access — click through it. The notebook will then train a model for every entry in
   `captures.json`, one after another. This can take a long time for many captures; that's
   expected, and you don't need to do anything else while it runs.

7. **Collect your models from Drive.** Each finished model is uploaded to:

   ```
   MyDrive/NAM/[gear_model]/[name].nam
   ```

   Alongside each `.nam` you'll also find `[name].png` (a plot comparing the model to your
   reamped target — check the ESR shown in the title) and a `_batch_results_*.json` summary of
   the whole run (status + ESR for every capture, and the error message for any that failed).

   If a capture's `.nam` already exists in Drive from a previous run, the new one is saved as
   `[name] (2).nam`, `[name] (3).nam`, etc. — nothing is ever overwritten.

## Manifest format (`captures.json`)

```jsonc
{
  "drive_folder": "NAM",          // top-level folder under MyDrive
  "defaults": {                   // shared settings, used unless a capture overrides them
    "modeled_by": "Your name",
    "gear_make": "GearCo",
    "gear_type": "amp",           // amp | pedal | pedal_amp | amp_cab | amp_pedal_cab | preamp | studio
    "tone_type": "clean",         // clean | overdrive | crunch | hi_gain | fuzz
    "epochs": 100,
    "latency": "auto",            // "auto", or an integer number of samples
    "ignore_checks": false,
    "reamp_send_level": "",       // calibration level in dBu, or "" to skip
    "reamp_return_level": ""
  },
  "captures": [
    {
      "output": "output_be100_lead.wav",  // required: the reamped file you uploaded
      "name": "BE100 Lead",                // required: becomes the .nam filename
      "gear_model": "BE100"                // required: becomes the Drive subfolder
      // any "defaults" key can also be set here to override it for this capture
    }
  ]
}
```

Only `output`, `name`, and `gear_model` are required per capture — everything else falls back to
`defaults`. See
[the calibration docs](https://neural-amp-modeler.readthedocs.io/en/stable/tutorials/calibration.html)
for more on `reamp_send_level` / `reamp_return_level`.

## Troubleshooting

### "No latency provided and cannot automatically analyze the latency"

NAM finds the recording latency by looking for two calibration "blips" in `input.wav` (around
0:10.5 and 0:11.5) and matching them against the corresponding blips in your reamped output. If
your audio interface or reamping chain has an unusual delay, NAM's automatic detection can fail to
find a clear match, and training fails with this error before it even starts.

The notebook includes a **diagnostic cell** (in the Troubleshooting section of the notebook
itself) that plots your `input.wav` and the first capture's output around the calibration blips,
and reports the measured background noise level vs. the detection threshold. Run it (with your
files either in `MyDrive/NAM_uploads` or uploaded to `/content`) to see whether the blips are
visible and to estimate the correct latency in samples.

Once you know the value, set it explicitly in `captures.json`'s `defaults` (or per-capture) to
skip automatic detection:

```jsonc
"defaults": {
  "latency": 240
}
```
