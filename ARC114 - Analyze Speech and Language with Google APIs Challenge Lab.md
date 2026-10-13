# ARC114 - Analyze Speech and Language with Google APIs: Challenge Lab

[![Start Lab](https://img.shields.io/badge/Start_Lab-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://www.skills.google/course_templates/634/labs/600971)

Google Cloud manual solution for the **ARC114: Analyze Speech and Language with Google APIs: Challenge Lab** - create an API key, call the Cloud Natural Language API for entity analysis, call the Cloud Speech-to-Text API to transcribe audio, and analyze sentiment in Python.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has four tasks:

1. **Task 1:** Create an API key.
2. **Task 2:** Make an entity analysis request and call the Natural Language API.
3. **Task 3:** Create a speech analysis request and call the Speech API.
4. **Task 4:** Analyze sentiment with the Natural Language API using Python.

> **Important:** This is a manual solution. Use the VM instance provided by the lab and the sample audio/review files specified in the lab. Do not share your API key publicly.

---

# TASK 1 - Create an API key

## Step 1 - Generate an API key

1. Open **APIs & Services > Credentials** in the Google Cloud console.
2. Click **Create Credentials** and select **API key**.
3. Copy and save the API key for the following tasks.

Save the API key for use after connecting to the lab-provided VM in Task 2. Do not paste a real API key into a public solution file or commit it to a public repository.

---

# TASK 2 - Make an entity analysis request and call the Natural Language API

## Step 1 - Connect to the VM and create the request file

1. Open **Compute Engine > VM instances** in the Google Cloud console.
2. Find the VM instance provided for the lab and click **SSH**.
3. Set the API key in this SSH session:

   ```bash
   export API_KEY="YOUR_API_KEY"
   ```

   Replace `YOUR_API_KEY` with the key you created in Task 1.

4. Create the request JSON file:

   ```bash
   nano nl_request.json
   ```

5. Paste the following JSON:

   ```json
   {
     "document": {
       "type": "PLAIN_TEXT",
       "content": "With approximately 8.2 million people residing in Boston, the capital city of Massachusetts is one of the largest in the United States."
     },
     "encodingType": "UTF8"
   }
   ```

6. Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Call the Natural Language API

Run the following command to send the request and save the response to `nl_response.json`:

```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  "https://language.googleapis.com/v1/documents:analyzeEntities?key=${API_KEY}" \
  -d @nl_request.json \
  -o nl_response.json
```

View the response:

```bash
cat nl_response.json
```

---

# TASK 3 - Create a speech analysis request and call the Speech API

## Step 1 - Listen to the sample audio and create the request file

The lab uses the sample audio file `gs://cloud-samples-tests/speech/brooklyn.flac`. Listen to the audio before sending it to the Speech API, as requested by the lab.

Create the request file:

```bash
nano speech_request.json
```

Paste the following JSON:

```json
{
  "config": {
    "encoding": "FLAC",
    "languageCode": "en-US"
  },
  "audio": {
    "uri": "gs://cloud-samples-tests/speech/brooklyn.flac"
  }
}
```

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Call the Speech-to-Text API

Run the request and save the response to `speech_response.json`:

```bash
curl -s -X POST \
  -H "Content-Type: application/json" \
  "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" \
  -d @speech_request.json \
  -o speech_response.json
```

View the response:

```bash
cat speech_response.json
```

---

# TASK 4 - Analyze sentiment with the Natural Language API

## Step 1 - Edit `sentiment_analysis.py`

The lab-provided VM already contains `sentiment_analysis.py`. Open it in the terminal:

```bash
nano sentiment_analysis.py
```

Find the method:

```python
def analyze(movie_review_filename):
```

Complete the method with the following code:

```python
def analyze(movie_review_filename):
    client = language.LanguageServiceClient()

    with open(movie_review_filename, "r") as f:
        text = f.read()

    document = language.Document(
        content=text,
        type_=language.Document.Type.PLAIN_TEXT
    )

    response = client.analyze_sentiment(request={"document": document})

    print_result(response)
```

Make sure the file includes the required import near the top:

```python
from google.cloud import language_v1 as language
```

If the import is already present, do not add it a second time. Preserve the rest of the lab-provided code, including the `print_result` function and command-line handling.

Save and exit (`Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Download and extract the sample reviews

Download the archive from Cloud Storage:

```bash
gsutil cp gs://cloud-samples-tests/natural-language/sentiment-samples.tgz .
```

Extract the `.tgz` archive using the following commands:

```bash
gunzip sentiment-samples.tgz
```

```bash
tar -xvf sentiment-samples.tar
```

Find the positive Blade Runner review file:

```bash
find . -name "bladerunner-pos.txt"
```

## Step 3 - Run sentiment analysis

If the file is located at `reviews/bladerunner-pos.txt`, run:

```bash
python3 sentiment_analysis.py reviews/bladerunner-pos.txt
```

If `find` reports a different path, use that path in the command instead.

---

### ⚠️ Disclaimer

- **This guide is provided for educational purposes to support learning and understanding of Google Cloud services. Review the commands and code before running them. Always follow Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is intended to support hands-on learning, not to bypass lab challenges.**
- **Never publish your actual API key.** Use a placeholder in public documentation and keep credentials private.

### © Credit & Attribution

- **The lab scenario and Google Cloud resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out for proper attribution or removal.** 🙏
