# AI Museum Guide

A small Flask app that lets a visitor upload an exhibit picture, ask a question about it, and receive a text answer from a vision-capable model deployed in Microsoft Foundry.

The guide can describe details visible in a picture. A picture by itself may not prove an object's date, maker, origin, or history, so check the museum label or another trusted source for those facts.

## Requirements

- Python 3.10 or newer
- A Microsoft Foundry model deployment that accepts images and returns text
- The endpoint and API key for that deployment

## Setup on Windows

Open PowerShell in this folder and run:

```powershell
py -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install -r requirements.txt
if (-not (Test-Path .env)) { Copy-Item .env.example .env } else { Write-Host ".env already exists; leaving it unchanged." }
```

If PowerShell does not allow activating the virtual environment, run the app with its Python executable directly:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe app.py
```

Edit `.env` and set:

- `AZURE_OPENAI_ENDPOINT` to your Azure OpenAI-compatible resource endpoint, for example `https://YOUR-RESOURCE.openai.azure.com`
- `AZURE_OPENAI_API_KEY` to the key for that resource
- `VISION_MODEL_DEPLOYMENT` to the exact name of your deployed image-capable model
- `SPEECH_API_KEY` to the key for your Azure AI Speech resource in Foundry
- `SPEECH_REGION` to that Speech resource's Azure region identifier, such as `eastus` or `centralindia`
- `SPEECH_ENDPOINT` to the resource endpoint URL if you also use the speech-to-text demo.

Do not share or commit `.env`. The `.env.example` file contains placeholders only.
The text-and-image model key and Speech resource key are separate credentials. Spoken answers require an Azure AI Speech resource; standard Text-to-Speech requests use its region-specific REST URL (`https://<region>.tts.speech.microsoft.com/cognitiveservices/v1`). `SPEECH_ENDPOINT` is not the standard neural Text-to-Speech URL.

## Run

With the virtual environment activated:

```powershell
python app.py
```

Open <http://127.0.0.1:5000> and select **Museum Guide**, or go directly to <http://127.0.0.1:5000/guide>.

## How the picture question works

1. The browser sends the selected image and the visitor's question to Flask.
2. Flask sends both to the configured vision-capable Foundry deployment using the API key kept on the server.
3. The model returns text, which the page displays as the guide's answer.
4. The app sends that answer to the Azure Speech Text-to-Speech API and plays the returned audio. The answer remains visible, and the audio controls let visitors replay it.

This feature analyzes pictures and generates text; it does not generate new images. The deployment must support image input. A text-only model deployment will not work for the guide.

## Deploy on Render

Before deploying, rotate any API keys that were shared or committed, then update them only in your local `.env` file. Never put real credentials in GitHub, `render.yaml`, or browser code.

1. In the project folder, make sure `.env` is not tracked by Git:

   ```powershell
   git rm --cached -- .env
   ```

   This stops Git from uploading `.env` but leaves your local file in place. `.gitignore` already excludes it from future additions.

2. Review the files you're about to upload:

   ```powershell
   git status
   ```

   Confirm `.env` is listed as deleted from Git's index and **not** as a new or staged file. Do not stage or commit `.env`.

3. Commit and push the project to a GitHub repository. If `.env` was committed previously, rotating its keys is essential: removing the file in a new commit does not erase old copies from Git history.

4. In Render, choose **New + → Web Service**, connect your GitHub repository, and set:
   - **Runtime:** Python
   - **Build Command:** `pip install -r requirements.txt`
   - **Start Command:** `gunicorn app:app`

5. In the Render service's **Environment** settings, add the needed values as separate environment variables. For image questions, set `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY`, and `VISION_MODEL_DEPLOYMENT`. To enable spoken answers, also set `SPEECH_API_KEY` and `SPEECH_REGION`. Set `SPEECH_ENDPOINT` only if using the separate speech-to-text demo.

6. Deploy, then open the Render-provided `onrender.com` URL. The service's environment variables are private to the server; do not expose the keys in HTML or JavaScript.

The app accepts uploaded image bytes for the current request and doesn't need to save those uploads to persistent disk. Render's local filesystem should not be used to store uploads that need to survive restarts.
