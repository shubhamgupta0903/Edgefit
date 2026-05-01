# EdgeFit — AI Fitness Assistant

> A full-stack AI-powered fitness assistant with a dual-model architecture — a fine-tuned LLaMA3 model (QLoRA) for personalized diet recommendations, and LLaMA 3.3 70B via Groq for general fitness chat — backed by real Fitbit activity data and a secure JWT-authenticated API.

---

## Tech Stack

**Frontend:** Next.js 15, React 19, TypeScript, Tailwind CSS, shadcn/ui  
**Backend:** FastAPI, Python, Groq API (LLaMA 3.3 70B)  
**Fine-tuned Model:** LLaMA 3.2 3B Instruct · QLoRA (4-bit) · Unsloth · deployed on Hugging Face  
**Database:** MongoDB (Motor async driver)  
**Auth:** JWT (OAuth2 + bcrypt)  
**ML/Training:** Unsloth, PEFT, TRL (SFTTrainer), Hugging Face Transformers  
**Data:** Fitbit Daily Activity Dataset  
**Tunneling (dev):** ngrok (Colab → FastAPI)

---

## Architecture — Dual Model Design

```
User Query
    │
    ├─── Diet / Nutrition related ──► Fine-tuned LLaMA 3.2 3B (shubhamtr/llama3-diet-model)
    │                                  Hosted on HF · served via ngrok FastAPI
    │
    └─── General Fitness Chat ──────► LLaMA 3.3 70B via Groq API
                                       Activity dataset injected as context
```

---

## Features

- **Dual AI Model System** — Fine-tuned LLaMA3 for diet recommendations + Groq-powered LLaMA 3.3 70B for general fitness Q&A
- **Custom Fine-tuned Model** — LLaMA 3.2 3B fine-tuned with QLoRA (4-bit, LoRA rank=16) using Unsloth on a custom diet planning dataset (`shubhamtr/llama2_diet_planner`)
- **HF Model Deployment** — Fine-tuned model published to `shubhamtr/llama3-diet-model` on Hugging Face, served via FastAPI + ngrok from Colab
- **JWT Authentication** — Secure register/login with bcrypt hashing and OAuth2 token flow
- **Chat History** — All conversations persisted to MongoDB per user with timestamps
- **Activity-aware Responses** — Fitbit dataset statistics injected into LLM context for data-grounded answers
- **Responsive UI** — Next.js frontend with real-time chat and mobile support

---

## Project Structure

```
Edgefit-main/
├── backend/
│   ├── main.py             # FastAPI app, CORS, router registration
│   ├── auth.py             # JWT auth — register, login, token endpoints
│   ├── chatbot.py          # /bot/chat — Groq LLM + activity dataset context
│   ├── hf_model.py         # Fine-tuned model integration (diet recommendations)
│   ├── database.py         # MongoDB connection via Motor
│   ├── models.py           # Pydantic models (User, Token)
│   ├── utils.py            # bcrypt password hashing helpers
│   └── requirements.txt
├── Edgefit/                # Next.js frontend
│   ├── app/
│   │   ├── page.tsx            # Landing page
│   │   ├── chat/page.tsx       # Chat interface
│   │   ├── login/page.tsx      # Login page
│   │   └── signup/page.tsx     # Signup page
│   └── components/ui/          # shadcn/ui components
├── llm_training.ipynb      # QLoRA fine-tuning pipeline (Unsloth + SFTTrainer)
├── llm_model.ipynb         # Model loading, inference API + ngrok tunnel
├── data_cleaning.ipynb     # Fitbit dataset preprocessing
└── dailyActivity_merged.csv
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js 18+
- MongoDB instance (local or Atlas)
- Groq API key → [console.groq.com](https://console.groq.com)
- Hugging Face account (to access `shubhamtr/llama3-diet-model`)

---

### 1. Clone the Repository

```bash
git clone https://github.com/shubhamgupta0903/EdgeFit.git
cd EdgeFit
```

---

### 2. Backend Setup

```bash
cd backend
pip install -r requirements.txt
```

Create a `.env` file in `backend/`:

```env
JWT_SECRET_KEY=your_secret_key_here
GROQ_API_KEY=your_groq_api_key_here
MONGODB_URI=mongodb://localhost:27017
HF_TOKEN=your_huggingface_token_here
```

Start the backend:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

Swagger docs → `http://localhost:8000/docs`

---

### 3. Fine-tuned Diet Model Server

The diet model runs as a separate FastAPI service from Google Colab using `llm_model.ipynb`:

```python
# Loads fine-tuned model from Hugging Face
base_model = AutoModelForCausalLM.from_pretrained("unsloth/llama-3.2-3b-instruct-unsloth-bnb-4bit")
model = PeftModel.from_pretrained(base_model, "shubhamtr/llama3-diet-model")
```

Exposed via ngrok tunnel on port 8000 with a `/generate` POST endpoint.

---

### 4. Frontend Setup

```bash
cd Edgefit
npm install
npm run dev
```

Frontend → `http://localhost:3000`

---

## API Endpoints

| Method | Endpoint | Description | Auth |
|--------|----------|-------------|------|
| POST | `/auth/register/` | Register new user | No |
| POST | `/auth/login/` | Login, get JWT token | No |
| POST | `/auth/token` | OAuth2 token endpoint | No |
| POST | `/bot/chat/` | General fitness chat (Groq) | Yes |
| POST | `/bot/save` | Save LLM interaction | No |
| POST | `/generate` | Diet recommendation (fine-tuned model) | No |

---

## ML Model — Fine-tuning Details

| Property | Value |
|----------|-------|
| Base model | `unsloth/Llama-3.2-3B-Instruct` |
| Technique | QLoRA (4-bit quantization) |
| LoRA rank | 16, alpha=32, dropout=0.05 |
| Trainer | SFTTrainer (TRL) |
| Epochs | 3 |
| Batch size | 2 |
| Precision | fp16 |
| Dataset | `shubhamtr/llama2_diet_planner` (Hugging Face) |
| Published model | `shubhamtr/llama3-diet-model` |
| Inference speed | 40% faster vs baseline |

**Training pipeline (`llm_training.ipynb`):**
1. Load base model with Unsloth (4-bit)
2. Apply PEFT/LoRA adapters (rank=16)
3. Fine-tune on diet planning dataset with SFTTrainer
4. Save and push to Hugging Face Hub

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `JWT_SECRET_KEY` | Secret key for JWT signing |
| `GROQ_API_KEY` | Groq console API key |
| `MONGODB_URI` | MongoDB connection string |
| `HF_TOKEN` | Hugging Face token (for private model access) |

---

## Future Improvements

- [ ] Unify diet + general chat into a single routing backend
- [ ] Replace ngrok tunnel with persistent HF Inference Endpoint
- [ ] Calorie and macro tracking dashboard with Recharts
- [ ] Wearable / fitness band data ingestion
- [ ] Docker Compose for one-command local deployment

---

