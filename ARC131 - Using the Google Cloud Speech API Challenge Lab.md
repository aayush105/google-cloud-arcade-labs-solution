# ARC131 - Using the Google Cloud Speech API: Challenge Lab

[![Start Lab](https://img.shields.io/badge/Start_Lab-4285F4?style=for-the-badge&logo=googlecloud&logoColor=white)](https://www.skills.google/games/7496/labs/45854)

Google Cloud manual solution for the **ARC131: Using the Google Cloud Speech API: Challenge Lab** --- generating an API key, and using the Speech-to-Text API to transcribe English and Spanish audio files from Cloud Storage using a VM instance.

## Author

**Aayush Shrestha (Shinux)** - Computer Engineer

**Portfolio:** [aayushshrestha7.com.np](https://aayushshrestha7.com.np/) | **GitHub:** [aayush105](https://github.com/aayush105) | **LinkedIn:** [aayushrestha](https://www.linkedin.com/in/aayushrestha/)

[![Watch on YouTube](https://img.shields.io/badge/Watch_on_YouTube-FF0000?style=for-the-badge&logo=youtube&logoColor=white)](https://www.youtube.com/@Er.Shinux)

---

## Lab Overview

This challenge lab has three tasks:

1. **Task 1:** Create an API key.
2. **Task 2:** Transcribe speech to English text.
3. **Task 3:** Transcribe speech to Spanish text.

> **Important:** This is a **manual solution** - you do not need to download or execute any GitHub script. Use the lab-provided values for your dataset name and events table name if any.

---

# TASK 1 - Create an API key

## Step 1 - Generate API key from Credentials

1. Go to **APIs & Services > Credentials** in the Google Cloud console.
2. Click **Create Credentials** and select **API key**.
3. **Save the API key**, you will need it in Task 2 and 3.

---

# TASK 2 - Transcribe speech to English text

## Step 1 - Connect to VM and Create API Request

1. Go to **Compute Engine > VM instances** and click **SSH** to connect to your instance.
2. Create the request JSON file:
   ```bash
   nano speech_request.json
   ```
3. Paste the following configuration:
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
4. Save and exit (Press `Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Call the Speech-to-Text API

1. Export your API key:
   ```bash
   export API_KEY=<YOUR_API_KEY>
   ```
2. Call the Speech-to-Text API and save the response:
   ```bash
   curl -s -X POST \
     -H "Content-Type: application/json" \
     -d @speech_request.json \
     "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" \
     -o response.json
   ```
3. Check the output:
   ```bash
   cat response.json
   ```

---

# TASK 3 - Transcribe speech to Spanish text

## Step 1 - Create API Request for Spanish

1. Create a new request JSON file:
   ```bash
   nano speech_request_sp.json
   ```
2. Paste the following configuration:
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
3. Save and exit (Press `Ctrl+O` → `Enter` → `Ctrl+X`).

## Step 2 - Call the Speech-to-Text API

1. Call the Speech-to-Text API and save the response:
   ```bash
   curl -s -X POST \
     -H "Content-Type: application/json" \
     -d @speech_request_sp.json \
     "https://speech.googleapis.com/v1/speech:recognize?key=${API_KEY}" \
     -o response_sp.json
   ```
2. Check the output:
   ```bash
   cat response_sp.json
   ```

---

### ⚠️ Disclaimer

- **This guide and instructions are provided strictly for educational purposes to help you learn Google Cloud services and advance your engineering skills. Please review all commands thoroughly to understand the underlying infrastructure before execution. Always comply with Qwiklabs/Google Cloud Skills Boost Terms of Service and YouTube Community Guidelines. This material is designed to enhance hands-on learning, not to bypass lab challenges.**

### © Credit & Attribution

- **All educational content, lab scenarios, and original resources belong to [Google Cloud Skills Boost](https://www.cloudskillsboost.google/). No copyright infringement is intended. If you are a copyright owner and have concerns, please reach out via direct message for proper attribution or immediate content removal.** 🙏
