# NAMTrainerColab
A nice, user-friendly NAM Trainer to be used on Google Colab — set up for **batch training**:
upload `input.wav`, all of your reamped capture files, and a `captures.json` describing each one,
then run a single cell to train every model and upload it straight to your Google Drive.

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

5. **Upload everything** into the Colab file browser (the folder icon on the left): `input.wav`,
   every `output_*.wav` file, and `captures.json`.

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
