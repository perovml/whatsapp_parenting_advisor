<p align="center">
        <img alt="logo" src="img/ava_final_design.gif" width=1000 />
    <h1 align="center">📱 Sarah 📱</h1>
    <h3 align="center">Your helpful parenting advisor</h3>
</p>

<p align="center">
    <img alt="logo" src="img/whatsapp_logo.png" width=100 />
</p>

# Sarah WhatsApp Nanny Agent

This repository contains **Sarah**, a WhatsApp‑integrated nanny and parenting‑advisor assistant. Sarah helps you plan your child’s day, suggest activities, track routines, and support both emotional and physical health. Follow the steps below to get Sarah running locally and—optionally—deploy her on Google Cloud Run.

The repository is based on the course from Neural Maze (https://github.com/neural-maze/ava-whatsapp-agent-course)

## 1. Clone the repository

```bash
git clone https://github.com/neural-maze/sarah-whatsapp-agent-course.git
cd sarah-whatsapp-agent-course
```

## 2. Install **uv**

Instead of `pip` or `poetry`, we use **uv** as our Python package manager.
Follow the official [uv installation instructions](https://docs.astral.sh/uv/getting-started/installation/) to get started.

## 3. Install project dependencies

1. **Create** and **activate** a virtual environment:

   ```bash
   uv venv .venv
   # macOS / Linux
   . .venv/bin/activate
   # Windows (PowerShell)
   . .\.venv\Scripts\Activate.ps1
   ```
2. **Install** the package in editable mode:

   ```bash
   uv pip install -e .
   ```
3. **Verify** your Python version:

   ```bash
   uv run python --version
   ```

   You should see:

   ```
   Python 3.12.8
   ```

## 4. Environment Variables

Copy the template and open `.env` in your editor:

```bash
cp .env.example .env
```

Populate these values (you can leave WhatsApp settings empty for now):

```dotenv
GROQ_API_KEY=""
ELEVENLABS_API_KEY=""
ELEVENLABS_VOICE_ID=""
TOGETHER_API_KEY=""
QDRANT_URL=""
QDRANT_API_KEY=""
WHATSAPP_PHONE_NUMBER_ID=""
WHATSAPP_TOKEN=""
WHATSAPP_VERIFY_TOKEN=""
```

### Generating Your API Keys

* **Groq**
  Follow the [Groq quickstart](https://console.groq.com/docs/quickstart) to generate `GROQ_API_KEY`.

* **ElevenLabs**
  Sign up at [ElevenLabs](https://elevenlabs.io/), then create an API key and choose your `ELEVENLABS_VOICE_ID`.

* **Together AI**
  Log in at [Together AI](https://www.together.ai/) and generate `TOGETHER_API_KEY`.

* **Qdrant**

  * **Local**: no setup needed.
  * **Cloud**: create an account at [Qdrant Cloud](https://login.cloud.qdrant.io/), then copy your `QDRANT_URL` and `QDRANT_API_KEY`.

Paste each key into your `.env` file, matching the names in `.env.example`.

## 5. First run

Start Sarah locally via the Makefile:

```bash
make sarah-run
```

This brings up a Docker Compose stack with three services:

* **Qdrant Database**: [http://localhost:6333/dashboard](http://localhost:6333/dashboard)
* **Chainlit UI**: [http://localhost:8000](http://localhost:8000)
* **FastAPI app**: [http://localhost:8080/docs](http://localhost:8080/docs) (used later for WhatsApp integration)

Click the Chainlit link to begin chatting with Sarah, your nanny‑advisor agent.

To tear everything down and clean up volumes:

```bash
make sarah-delete
```

## 6. Google Cloud Run Deployment

You can also deploy Sarah to **Google Cloud Run**. After creating a GCP project and enabling billing, follow these steps:

1. **Authenticate**

   ```bash
   gcloud auth login
   ```
2. **Set your project**

   ```bash
   gcloud config set project <PROJECT_ID>
   ```
3. **Enable required APIs**

   ```bash
   gcloud services enable \
     cloudbuild.googleapis.com \
     run.googleapis.com \
     artifactregistry.googleapis.com \
     cloudresourcemanager.googleapis.com \
     secretmanager.googleapis.com
   ```
4. **Configure Docker auth**

   ```bash
   gcloud config set compute/region <LOCATION>
   gcloud auth configure-docker <LOCATION>-docker.pkg.dev -q
   ```
5. **Create a Docker repository**

   ```bash
   gcloud artifacts repositories create sarah-app \
     --repository-format=docker \
     --location=<LOCATION> \
     --description="Docker repo for Sarah, the WhatsApp Nanny Agent"
   ```
6. **Create secrets in Secret Manager**

   ```bash
   echo -n "<GROQ_API_KEY>"            | gcloud secrets create GROQ_API_KEY            --replication-policy="automatic" --data-file=-
   echo -n "<ELEVENLABS_API_KEY>"      | gcloud secrets create ELEVENLABS_API_KEY      --replication-policy="automatic" --data-file=-
   echo -n "<ELEVENLABS_VOICE_ID>"     | gcloud secrets create ELEVENLABS_VOICE_ID     --replication-policy="automatic" --data-file=-
   echo -n "<TOGETHER_API_KEY>"        | gcloud secrets create TOGETHER_API_KEY        --replication-policy="automatic" --data-file=-
   echo -n "<QDRANT_URL>"              | gcloud secrets create QDRANT_URL              --replication-policy="automatic" --data-file=-
   echo -n "<QDRANT_API_KEY>"          | gcloud secrets create QDRANT_API_KEY          --replication-policy="automatic" --data-file=-
   echo -n "<WHATSAPP_PHONE_NUMBER_ID>"| gcloud secrets create WHATSAPP_PHONE_NUMBER_ID --replication-policy="automatic" --data-file=-
   echo -n "<WHATSAPP_TOKEN>"          | gcloud secrets create WHATSAPP_TOKEN          --replication-policy="automatic" --data-file=-
   echo -n "<WHATSAPP_VERIFY_TOKEN>"   | gcloud secrets create WHATSAPP_VERIFY_TOKEN   --replication-policy="automatic" --data-file=-
   ```
7. **Grant Cloud Run access to secrets**

   ```bash
   gcloud projects add-iam-policy-binding <PROJECT_ID> \
     --member="serviceAccount:$(gcloud projects describe $(gcloud config get-value project) --format='value(projectNumber)')-compute@developer.gserviceaccount.com" \
     --role="roles/secretmanager.secretAccessor"
   ```
8. **Build and deploy**

   ```bash
   gcloud builds submit --region=<LOCATION>
   ```

You’re all set! Sarah will be live on Cloud Run, ready to assist with your parenting needs.
