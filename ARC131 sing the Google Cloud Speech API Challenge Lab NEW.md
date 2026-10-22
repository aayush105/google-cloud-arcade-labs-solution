# ARC131 - Using the Google Cloud Speech API: Challenge Lab

[![Start
Lab](https://img.shields.io/badge/Start_Lab-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://www.skills.google/focuses/65993)

Google Cloud manual solution for the **ARC131: Using the Google Cloud
Speech API: Challenge Lab** --- create an API key and use the
Speech-to-Text API to transcribe English and Spanish audio files from
Cloud Storage using Google Cloud Shell.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/)
\| **GitHub:** [aayush105](https://github.com/aayush105) \|
**LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on
YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has three tasks:

1.  **Task 1:** Create an API key.
2.  **Task 2:** Transcribe speech to English text.
3.  **Task 3:** Transcribe speech to Spanish text.

> **Important:** This updated version of the lab uses **Google Cloud
> Shell**, not a Compute Engine VM. It also requires you to upload each
> request and response file to the lab's Cloud Storage bucket before
> checking progress. Do not follow the old VM/SSH instructions for this
> version.

---

# TASK 1 - Create an API key

## Step 1 - Generate an API key

1.  In the Google Cloud console, open **Navigation menu \> APIs &
    Services \> Credentials**.
2.  Click **Create Credentials** and select **API key**.
3.  Copy and save the API key for the following tasks.

## Step 2 - Set the API key in Cloud Shell

Open **Google Cloud Shell** from the Google Cloud console and run:

```bash
export API_KEY="YOUR_API_KEY"
```

Replace `YOUR_API_KEY` with the API key you created. Keep the key
private and do not commit it to a public repository.

**Click Check my progress** to verify the objective.

---

# TASK 2 - Transcribe speech to English text

The lab-provided audio file is:

`gs://spls/arc131/question_en.wav`

## Step 1 - Configure the project and bucket

In Google Cloud Shell, run:

```bash
export GOOGLE_CLOUD_PROJECT=$(gcloud config get-value project)
export BUCKET_NAME="${GOOGLE_CLOUD_PROJECT}-speech"
```

Confirm the values:

```bash
echo "$GOOGLE_CLOUD_PROJECT"
echo "$BUCKET_NAME"
```

## Step 2 - Create the English transcription request

Create the request file:

```bash
nano speech_request_en.json
```

Paste the following JSON:

```json
{
  "config": {
    "encoding": "LINEAR16",
    "languageCode": "en-US",
    "audioChannelCount": 2
  },
  "audio": {
    "uri": "gs://spls/arc131/question_en.wav"
  }
}
```

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 3 - Call the Speech-to-Text API

Run the request and save the result in `response.json`:

```bash
curl "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @speech_request_en.json > response.json
```

Inspect the response:

```bash
cat response.json
```

The transcription is normally returned in the `results` array, under
`alternatives` and `transcript`. If the response contains an `error`
field, read the message before proceeding.

## Step 4 - Upload the request and response files

Upload both files to the bucket created for this lab:

```bash
gcloud storage cp speech_request_en.json gs://${BUCKET_NAME}/speech_request_en.json
gcloud storage cp response.json gs://${BUCKET_NAME}/response.json
```

Verify that the files were uploaded:

```bash
gcloud storage ls "gs://${BUCKET_NAME}/"
```

**Click Check my progress** to verify **Create API request for
transcription in English language**.

---

# TASK 3 - Transcribe speech to Spanish text

The lab-provided audio file is:

`gs://spls/arc131/multi_es.flac`

## Step 1 - Create the Spanish transcription request

Create a separate request file:

```bash
nano request_sp.json
```

Paste the following JSON:

```json
{
  "config": {
    "encoding": "FLAC",
    "languageCode": "es-ES"
  },
  "audio": {
    "uri": "gs://spls/arc131/multi_es.flac"
  }
}
```

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Call the Speech-to-Text API

Run the request and save the result in `speech_response_sp.json`:

```bash
curl "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" \
  -s -X POST -H "Content-Type: application/json" --data-binary @request_sp.json > speech_response_sp.json
```

Inspect the response:

```bash
cat speech_response_sp.json
```

Check that the response contains transcription results and does not
contain an API error.

## Step 3 - Upload the request and response files

Upload both files to the same lab bucket:

```bash
gcloud storage cp request_sp.json gs://${BUCKET_NAME}/request_sp.json
gcloud storage cp speech_response_sp.json gs://${BUCKET_NAME}/speech_response_sp.json
```

Verify the uploaded files:

```bash
gcloud storage ls "gs://${BUCKET_NAME}/"
```

**Click Check my progress** to verify **Create API request for
transcription in Spanish language**.

---

## Lab Complete

Congratulations! You have completed the three tasks in the **Using the
Google Cloud Speech API: Challenge Lab (ARC131)**.

---

### ⚠️ Disclaimer

- **This guide is provided for educational purposes to support
  learning and understanding of Google Cloud services.** Review the
  commands and responses as you work, and follow Google Cloud Skills
  Boost Terms of Service and YouTube Community Guidelines. This
  material is intended to support hands-on learning, not to bypass lab
  challenges.
- **Never publish your actual API key.** Use a placeholder in public
  documentation and keep credentials private.
- The current lab environment and automated scoring requirements may
  vary. Follow the filenames and instructions displayed in your active
  lab session if they differ from the examples in this guide.

### © Credit & Attribution

- **The lab scenario and Google Cloud resources belong to [Google
  Cloud Skills Boost](https://www.skills.google/).** This guide is an
  independent learning aid. If you are a copyright owner and have
  concerns, please reach out for proper attribution or removal. 🙏
