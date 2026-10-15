# ARC132 - Implement Speech and Language Solutions with Pre-trained APIs: Challenge Lab

[![Start
Lab](https://img.shields.io/badge/Start_Lab-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://www.skills.google/course_templates/700/labs/408075)

Google Cloud manual solution for **ARC132: Implement Speech and Language
Solutions with Pre-trained APIs: Challenge Lab** --- create an API key,
generate synthetic speech with the Text-to-Speech API, transcribe French
audio with the Speech-to-Text API, translate text, and detect a language
with the Cloud Translation API.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/)
\| **GitHub:** [aayush105](https://github.com/aayush105) \|
**LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on
YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has five tasks:

1.  **Task 1:** Create an API key.
2.  **Task 2:** Create synthetic speech from text using the
    Text-to-Speech API.
3.  **Task 3:** Perform speech-to-text transcription in French using the
    Cloud Speech API.
4.  **Task 4:** Translate text into English using the Cloud Translation
    API.
5.  **Task 5:** Detect a language using the Cloud Translation API.

> **Important:** Use the VM instance provided by the lab for Tasks 2--5.
> Keep your API key private. The commands below use a placeholder;
> replace it with the key you created in Task 1.

---

# TASK 1 - Create an API key

## Step 1 - Generate an API key

1.  Open **APIs & Services \> Credentials** in the Google Cloud console.
2.  Click **Create Credentials** and select **API key**.
3.  Copy and save the API key for use in the following tasks.

Do not publish your real API key in a public solution file or commit it
to a public repository.

---

# TASK 2 - Create synthetic speech from text using the Text-to-Speech API

## Step 1 - Connect to the VM

1.  Open **Compute Engine \> VM instances** in the Google Cloud console.

2.  Find the VM instance provided for the lab and click **SSH**.

3.  Set the API key in the SSH session:

    ```bash
    export API_KEY="YOUR_API_KEY"
    ```

    Replace `YOUR_API_KEY` with the key you created in Task 1.

## Step 2 - Create the Text-to-Speech request

Create the JSON request file:

```bash
nano synthesize-text.json
```

Paste the following valid JSON:

```json
{
  "input": {
    "text": "Cloud Text-to-Speech API allows developers to include natural-sounding, synthetic human speech as playable audio in their applications. The Text-to-Speech API converts text or Speech Synthesis Markup Language (SSML) input into audio data like MP3 or LINEAR16 (the encoding used in WAV files)."
  },
  "voice": {
    "languageCode": "en-GB",
    "name": "en-GB-Standard-A",
    "ssmlGender": "FEMALE"
  },
  "audioConfig": {
    "audioEncoding": "MP3"
  }
}
```

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 3 - Call the Text-to-Speech API

Get an access token from the lab VM's authenticated Google Cloud
session:

```bash
TOKEN=$(gcloud auth print-access-token)
```

Send the request and save the API response to `synthesize-text.txt`:

```bash
curl -sS -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @synthesize-text.json \
  "https://texttospeech.googleapis.com/v1/text:synthesize" \
  -o synthesize-text.txt
```

Inspect the response:

```bash
cat synthesize-text.txt
```

The response contains an `audioContent` field encoded in Base64.

## Step 4 - Decode the response into an MP3 file

Create the decoder script:

```bash
nano tts_decode.py
```

Paste the following Python code:

```python
import argparse
from base64 import decodebytes
import json


def decode_tts_output(input_file, output_file):
    """Decode output from Cloud Text-to-Speech."""
    with open(input_file, "r", encoding="utf-8") as input_file_handle:
        response = json.load(input_file_handle)
        audio_data = response["audioContent"]

    with open(output_file, "wb") as output_file_handle:
        output_file_handle.write(decodebytes(audio_data.encode("utf-8")))


if __name__ == "__main__":
    parser = argparse.ArgumentParser(
        description="Decode output from Cloud Text-to-Speech",
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )
    parser.add_argument(
        "--input",
        help="The response from the Text-to-Speech API.",
        required=True,
    )
    parser.add_argument(
        "--output",
        help="The name of the audio file to create",
        required=True,
    )

    args = parser.parse_args()
    decode_tts_output(args.input, args.output)
```

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

Run the script:

```bash
python3 tts_decode.py \
  --input "synthesize-text.txt" \
  --output "synthesize-text-audio.mp3"
```

Confirm that the MP3 file was created:

```bash
ls -lh synthesize-text-audio.mp3
```

Use the **Download File** option in the VM's SSH session to download
`synthesize-text-audio.mp3` and listen to it.

**Click Check my progress** to verify the objective.

---

# TASK 3 - Perform speech-to-text transcription with the Cloud Speech API

## Step 1 - Create the transcription request

The lab-provided audio file is:

`gs://cloud-samples-data/speech/corbeau_renard.flac`

Create the request file:

```bash
nano request.json
```

Paste the following JSON to request French transcription:

```json
{
  "config": {
    "encoding": "FLAC",
    "languageCode": "fr-FR"
  },
  "audio": {
    "uri": "gs://cloud-samples-data/speech/corbeau_renard.flac"
  }
}
```

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Call the Speech-to-Text API

Use the API key created in Task 1:

```bash
curl -sS -X POST \
  -H "Content-Type: application/json; charset=utf-8" \
  -d @request.json \
  "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" \
  -o response.json
```

View the transcription response:

```bash
cat response.json
```

Look for the recognized text in the `results` and `alternatives` fields.

**Click Check my progress** to verify the objective.

---

# TASK 4 - Translate text with the Cloud Translation API

## Step 1 - Request a translation into English

This example translates the Japanese sentence `これは日本語です。` into
English.

Get an access token:

```bash
TOKEN=$(gcloud auth print-access-token)
```

Send the translation request and save the response to
`translation_response.txt`:

```bash
curl -sS -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "q": "これは日本語です。",
    "target": "en"
  }' \
  "https://translation.googleapis.com/language/translate/v2" \
  -o translation_response.txt
```

View the response:

```bash
cat translation_response.txt
```

The translated text is returned in the response's `data.translations`
section.

**Click Check my progress** to verify the objective.

---

# TASK 5 - Detect a language with the Cloud Translation API

## Step 1 - Detect the language

This example detects the language of the Portuguese sentence
`Este é japonês.`. The sentence is intentionally written in Portuguese
so the API can identify its language; it means "This is Japanese."

Get an access token:

```bash
TOKEN=$(gcloud auth print-access-token)
```

Send the language-detection request and save the response to
`detected_response.txt`:

```bash
curl -sS -X POST \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json; charset=utf-8" \
  -d '{
    "q": "Este é japonês."
  }' \
  "https://translation.googleapis.com/language/translate/v2/detect" \
  -o detected_response.txt
```

View the response:

```bash
cat detected_response.txt
```

The response's `data.detections` section contains the detected language
code and confidence information.

**Click Check my progress** to verify the objective.

---

## Lab Complete

Congratulations! You have completed the five tasks in the **Implement
Speech and Language Solutions with Pre-trained APIs: Challenge Lab
(ARC132)**.

---

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Qwiklabs/Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏
