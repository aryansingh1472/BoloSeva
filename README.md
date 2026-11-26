# BoloSeva 2.0

**Voice-first access to simulated hospital appointments** · बोलो सेवा

BoloSeva is a working hackathon prototype for seniors, daily-wage workers, rural communities, and people who find conventional booking forms difficult. The patient can speak Hindi, English, or Hinglish, review available times, give consent, enter a name and age, and receive a simulated OPD token.

> **Simulation only.** Jila Chikitsalaya, doctors, rooms, slots, tokens, and reminders in this project are synthetic demo data. This does not contact a hospital or create a real appointment. It does not diagnose or prescribe.

## Start the prototype

Install Docker Desktop with Docker Compose, then from this folder run:

**docker compose up --build**

Open **http://localhost:8080**. Stop with Ctrl+C; use **docker compose down** to stop and preserve the SQLite volume.

There are no mandatory API keys, third-party AI calls, SMS services, or external hospital connections. The first build needs an internet connection to download the pinned container dependencies. The SQLite database is stored in a named Docker volume so demo bookings persist across restarts.

## Try a complete demo booking

1. Open **Book a visit** and select one of the sample phrases, or tap the large microphone and speak. The “eye checkup tomorrow” sample demonstrates the fully booked ophthalmology day and next available date.
2. Choose a time card, or say “first”, “second”, or “morning”.
3. Confirm the time. Review the collection notice and give consent.
4. Say a patient name and age (for example, “My name is Neha, age 62”), then check the summary and confirm.
5. Read or replay the token, doctor, room, reporting time, and clearly labelled simulated reminder.
6. Use the booking card’s controls, or say “cancel my appointment” / “reschedule my appointment” and provide its token.

The sample phrase buttons and voice transcript use the same intent and booking path. **Simulate noisy speech** to show the retry and accessible card fallback. The urgent symptom sample stops routine booking and displays emergency guidance.

## Demo hospital inventory

Dates are computed from the current calendar date in India (Asia/Kolkata). Inventory is seeded in SQLite when the backend starts; an existing database keeps issued tokens and bookings.

| Department | Doctor / room | Demo availability | Initial token |
|---|---|---|---|
| General Medicine | Dr. Sharma · Room 2 | Tomorrow 9:00–11:00 (12); 11:30–13:00 (8) | 14; 32 |
| Orthopedics | Dr. Verma · Room 4 | Tomorrow 10:00–12:00 (5) | 09 |
| Ophthalmology | Dr. Patil · Room 7 | Tomorrow full; day after tomorrow 9:30–11:30 (6) | 04 |

The counts decrease on active bookings and increase on cancellation. The backend serializes writes in SQLite transactions, limits active appointments to a slot’s capacity, blocks duplicate active bookings for the same patient and department on a date, and allocates non-reused tokens.

## Dashboards

- **Hospital demo:** shared live SQLite counts, department availability, queue tokens and statuses. Queue progression moves the oldest booked token to waiting and then completed. The API intentionally omits patient names and ages from this public demo view.
- **Impact targets:** shows the six Round 1 proposal targets as **proposed, not achieved**. A separate activity panel shows database counts from this simulation; those numbers are not pilot impact evidence.

No real patient data should be entered. The demo stores only the name and age supplied for a booking. Do not add Aadhaar, phone numbers, medical records, free-text notes, or real patient data.

## Local development and tests

The Compose command is the simplest launch path. For direct development, use Python 3.12+ and Node 22+ with pnpm.

**Backend**, from the project folder:

    python -m venv .venv

Activate the environment, then:

    python -m pip install -r backend/requirements.txt
    python -m uvicorn app.main:app --app-dir backend --reload --port 8000

**Frontend**, in a second terminal:

    cd frontend
    pnpm install
    pnpm dev

Open **http://localhost:5173**. Vite proxies /api to http://localhost:8000.

Backend automated tests:

    cd backend
    python -m pytest -q

Frontend automated tests and production build:

    cd frontend
    pnpm test
    pnpm build

Inside Compose, run the backend suite with:

    docker compose run --rm backend pytest -q

### Speech settings

- VITE_SPEECH_CONFIDENCE_THRESHOLD controls the minimum browser result confidence; default 0.65.
- VITE_SILENCE_BUFFER_MS controls the post-result pause buffer; default 1500.
- For Compose builds, put either value in a root .env file (copy .env.example first), then rebuild the frontend.

The app uses the browser Web Speech API when available, with Hindi (hi-IN) and Indian English (en-IN) recognition settings. Speech recognition depends on browser support and may require internet connectivity. **There is no offline speech recognizer.** When recognition is unavailable, blocked, unclear, or repeatedly fails, the app offers typed requests and large accessible selection cards. Speech output uses browser SpeechSynthesis when available and can be replayed. BoloSeva does not store raw audio; browser speech services may process speech under their own policies.

## Accessibility and safety

- Mobile-first layout, large high-contrast microphone control, enlarged touch targets, high-contrast toggle, semantic labels, keyboard focus rings, and reduced-motion support.
- English / Hindi controls and bilingual message transcripts. Hinglish and common Hindi terms are included in rule-based intent matching.
- Confidence threshold starts at 0.65; the browser's confidence field is used when supplied. A configurable 1.5-second pause buffer is applied to final recognition results.
- Emergency phrase matching interrupts routine booking for possible severe chest pain, breathing difficulty, unconsciousness, and stroke warning signs. It advises immediate emergency help (112 in India); it is not diagnosis or treatment.
- Symptoms such as joint or eye concerns only suggest a department. They are not diagnoses.
- No phone number is collected and no SMS is sent. The reminder shown after booking is explicitly a simulated reminder.

## Project map

See docs/ARCHITECTURE.md, docs/DEMO.md, docs/DEPENDENCIES_AND_LIMITATIONS.md, and docs/GITHUB_SUBMISSION.md for workflow, dependencies, limitations, and a submission checklist.

## Known limitations

- Speech recognition and synthesis vary by browser, device, language model, and connectivity. The browser may use a remote speech service; this prototype does not provide offline recognition.
- Rule-based date, accent, and phonetic matching is intentionally small and can misunderstand a request. Context is retained and the user can clarify, type, or select a card.
- The administrator/demo API is unauthenticated for local hackathon use. Do not expose it to a public network or use it with real data without authentication, authorization, operational security, and a privacy review.
- Seeded appointment windows are a small synthetic inventory, not live hospital capacity.
- Container execution was not available in the build environment used to prepare this project; run the documented Compose command on a machine with Docker Desktop/Compose to verify the container startup there.

## Public submission notes

No API keys, credentials, production endpoints, real patient data, or third-party contact details are included. Before publishing, choose a repository license and confirm hackathon name/branding requirements. Follow docs/GITHUB_SUBMISSION.md.