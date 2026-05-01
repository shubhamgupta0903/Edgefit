# EdgeFit — AI Fitness Assistant

> A full-stack AI-powered fitness assistant that delivers personalized diet and workout recommendations through a conversational interface, backed by a fine-tuned LLaMA3 model and real activity data.

---

## Tech Stack

**Frontend:** Next.js 15, React 19, TypeScript, Tailwind CSS, shadcn/ui  
**Backend:** FastAPI, Python, Groq API (LLaMA 3.3 70B)  
**Database:** MongoDB (Motor async driver)  
**Auth:** JWT (OAuth2 + bcrypt)  
**ML:** LLaMA3 7B, QLoRA 4-bit fine-tuning, Hugging Face  
**Data:** Fitbit Daily Activity Dataset

---

## Features

- **AI Chat Interface** — Conversational fitness assistant powered by LLaMA 3.3 70B via Groq, with context-aware responses based on real activity datasets
- **Personalized Recommendations** — Workout plans and diet advice tailored to user prompts and fitness data
- **JWT Authentication** — Secure user registration, login, and protected chat routes using OAuth2 + bcrypt
- **Chat History** — All conversations persisted to MongoDB per user with timestamps
- **Fine-tuned LLM** — 7B LLaMA3 model fine-tuned with QLoRA (4-bit quantization) on custom diet datasets for 40% faster inference
- **Responsive UI** — Clean Next.js frontend with real-time message streaming and mobile support

---

## Project Structure

```
Edgefit-main/
├── backend/
│   ├── main.py          # FastAPI app entry point
│   ├── auth.py          # JWT auth, register/login routes
│   ├── chatbot.py       # Chat endpoint, Groq LLM integration
│   ├── database.py      # MongoDB connection (Motor)
│   ├── models.py        # Pydantic models
│   ├── utils.py         # Helper functions
│   └── requirements.txt
├── Edgefit/             # Next.js frontend
│   ├── app/
│   │   ├── page.tsx         # Landing page
│   │   ├── chat/page.tsx    # Chat interface
│   │   ├── login/page.tsx   # Login page
│   │   └── signup/page.tsx  # Signup page
│   └── components/ui/       # shadcn/ui components
├── llm_training.ipynb   # QLoRA fine-tuning notebook
├── llm_model.ipynb      # Model inference notebook
├── data_cleaning.ipynb  # Dataset preprocessing
└── dailyActivity_merged.csv  # Fitbit activity dataset
```

---

## Getting Started

### Prerequisites

- Python 3.10+
- Node.js 18+
- MongoDB instance (local or Atlas)
- Groq API key → [console.groq.com](https://console.groq.com)

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

Create a `.env` file in the `backend/` directory:

```env
JWT_SECRET_KEY=your_secret_key_here
GROQ_API_KEY=your_groq_api_key_here
MONGODB_URI=mongodb://localhost:27017
```

Update `chatbot.py` to load the Groq API key from environment:

```python
import os
client = Groq(api_key=os.getenv("GROQ_API_KEY"))
```

Start the backend server:

```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
```

API will be available at `http://localhost:8000`  
Swagger docs at `http://localhost:8000/docs`

---

### 3. Frontend Setup

```bash
cd Edgefit
npm install
npm run dev
```

Frontend will be available at `http://localhost:3000`

---

## API Endpoints

| Method | Endpoint | Description | Auth Required |
|--------|----------|-------------|---------------|
| POST | `/auth/register/` | Register a new user | No |
| POST | `/auth/login/` | Login and get JWT token | No |
| POST | `/auth/token` | OAuth2 token endpoint | No |
| POST | `/bot/chat/` | Send message to AI assistant | Yes |
| POST | `/bot/save` | Save LLM interaction to DB | No |

---

## ML Model — Fine-tuning Details

The LLaMA3 7B model was fine-tuned using QLoRA (4-bit quantization) on a custom diet and fitness dataset derived from the Fitbit Daily Activity dataset.

- **Base Model:** LLaMA3 7B
- **Technique:** QLoRA (4-bit) via Hugging Face PEFT
- **Dataset:** `dailyActivity_merged.csv` — preprocessed Fitbit activity logs
- **Outcome:** 40% faster inference with live deployment on Hugging Face
- **Notebooks:**
  - `data_cleaning.ipynb` — dataset preprocessing and feature engineering
  - `llm_training.ipynb` — QLoRA fine-tuning pipeline
  - `llm_model.ipynb` — model loading and inference testing

---

## Environment Variables

| Variable | Description |
|----------|-------------|
| `JWT_SECRET_KEY` | Secret key for signing JWT tokens |
| `GROQ_API_KEY` | API key from Groq console |
| `MONGODB_URI` | MongoDB connection string |

---

## Future Improvements

- [ ] Integrate fine-tuned model directly (replace Groq with local HF model)
- [ ] Add fitness band / wearable data ingestion via API
- [ ] Calorie and macro tracking dashboard
- [ ] Workout history and progress visualization
- [ ] Docker Compose setup for one-command deployment

---


