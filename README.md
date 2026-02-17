# ApacheAI - Intelligent Aviation Weather Briefings

**ApacheAI** is a next-generation aviation weather briefing system designed to provide pilots with clear, actionable, and AI-enhanced weather insights. By aggregating data from official sources (METARs, TAFs), integrating community reports (PIREPs), and leveraging Google's Gemini AI, ApacheAI transforms raw data into comprehensive flight briefings.

## Key Features

*   **AI-Powered Analysis**: Generates natural language summaries of weather conditions using Google Gemini 2.5, adhering to specific pilot profiles (e.g., VFR, IFR).
*   **Interactive Route Mapping**: 
    *   Visualizes great-circle routes on a dynamic Leaflet map.
    *   Markers for Departure, Destination, and Waypoints.
    *   Color-coded markers based on flight category (VFR, MVFR, IFR, LIFR).
    *   Antimeridian-safe rendering for long-haul flights.
*   **Dynamic Weather Visualization**:
    *   Real-time charts for Temperature, Wind Speed/Gusts, Visibility, Altimeter, and Dew Point along the route.
    *   Responsive Chart.js implementation.
*   **Voice Assistant**: 
    *   Hands-free PIREP creation using voice commands (e.g., "Problem is...").
    *   Triggered via Right-Alt key with visual transcript feedback.
*   **PIREP Management**:
    *   **Submission**: Convert natural language (text or voice) into standardized PIREP format.
    *   **Retrieval**: Fetches relevant PIREPs for the route from the last 6 hours.
*   **NOTAM Integration**: Fetches and displays Notices to Air Missions relevant to the selected route.
*   **PDF Briefing Export**: Generates professional, offline-ready PDF briefings containing all route details, weather summaries, and charts.
*   **Performance Calculations**: Estimates fuel burn, time enroute, and distance based on aircraft profiles (C172, B737, A320).
*   **Alternative Route Suggestions**: (MVP Placeholder) Logic to suggest safer routes based on adverse weather.

---

## Team & Contributions

| Name | Role | Contribution |
| :--- | :--- | :--- |
| **Advait Balachandar** | Backend Lead | Designed the Flask API architecture and implemented AI integration with Google Gemini. |
| **Prayatshu Misra** | Frontend Engineer | Built the responsive UI with TailwindCSS and implemented the Leaflet map optimizations. |
| **Rohan Mathur** | DevOps Engineer | Managed Supabase database schema, deployment pipelines, and environment configuration. |
| **Shreshth Kabra** | ML Engineer | Designed, trained, and deployed machine learning models; performed data preprocessing and feature engineering; validated model performance; and documented pipelines and experiments. |
---

## System Architecture

ApacheAI follows a modern full-stack architecture with a Flask backend serving a robust Vanilla JS frontend.

```mermaid
graph TD
    subgraph "Frontend (Browser)"
        UI["User Interface"]
        Map["Leaflet Map"]
        Charts["Chart.js Graphs"]
        Voice["Voice Assistant Module"]
    end

    subgraph "Backend (Flask)"
        Server["Flask Server"]
        Routes["API Routes"]
        PDF["PDF Generator (wkhtmltopdf)"]
        AI_Logic["AI Logic (engtopirep.py)"]
    end

    subgraph "External Services"
        Gemini["Google Gemini AI"]
        Supabase["Supabase DB (PostgreSQL)"]
        NOAA["AviationWeather.gov API"]
    end

    UI --> Routes
    Voice --> UI
    Routes -->|Fetch METAR/TAF| NOAA
    Routes -->|Fetch/Store PIREPs & NOTAMs| Supabase
    Routes --> AI_Logic
    AI_Logic -->|Generate Summary/PIREP| Gemini
    Routes -->|Generate Briefing PDF| PDF

```

### User Journey Flow

```mermaid
graph LR
    A[Landing Page] -->|Click Start| B(Flight Planning)
    B -->|Input Route & Aircraft| C{Generate Briefing?}
    C -->|Yes| D[Fetching Data...]
    D -->|Success| E[Briefing Dashboard]
    E -->|Interact| F[Map & Charts]
    E -->|Action| G[Download PDF]
    E -->|Action| H[Submit PIREP]
```

### PIREP Submission Process

```mermaid
sequenceDiagram
    participant Pilot
    participant VoiceModule
    participant Backend
    participant GeminiAI
    participant DB

    Pilot->>VoiceModule: "Turbulence moderate at FL350"
    VoiceModule->>Backend: POST /api/convert-to-pirep (Audio/Text)
    Backend->>GeminiAI: Prompt: Convert to Standard Format
    GeminiAI-->>Backend: "UA /OV ... /TB MOD /FL350"
    Backend->>DB: Insert PIREP Record
    Backend-->>Pilot: Confirmation & Display
```

---

## Codebase Structure & Components

### 1. Backend (`/` root)
The backend is built with **Flask** and handles API requests, data aggregation, and PDF generation.

*   `get_weather.py`: **Main Application Entry Point**.
    *   Initializes Flask app.
    *   Configures Gemini AI and Supabase.
    *   Handles PDF generation using `pdfkit`.
*   `engtopirep.py`: **AI Logic Module**.
    *   Specialized script to convert natural language pilot reports into standardized PIREP strings using Gemini.
*   `notams.sql` / `notams_data.sql`: **Database Schema**.
    *   SQL definitions for the `NOTAMs` table in Supabase.
*   `templates/`: **Jinja2 Templates**.
    *   `briefing_template.html`: The HTML template used to generate the downloadable PDF briefing.

### 2. Frontend (`/aura`)
The frontend is a Single Page Application (SPA) served by Flask.

*   `aura/index.html`: **Main HTML Structure**.
    *   Loads TailwindCSS, Leaflet, Chart.js.
    *   Includes global loader and rain overlay effects.
*   `aura/app.js`: **Core Application Logic**.
    *   **Router**: Simple hash-based interaction handling (Landing -> Plan -> Briefing).
    *   **State Management**: `AppState` object tracks route, weather data, and params.
    *   **Visualizations**: `createWeatherGraphs()` and `initMapIfData()`.
    *   **Voice Assistant**: Singleton module handling microphone input.
*   `aura/styles.css`: **Custom Styling**.
    *   Contains animations (glow effects, shimmer) and map marker styles.

---

## API Reference

| Endpoint | Method | Description | Parameters |
| :--- | :--- | :--- | :--- |
| `/briefing` | `GET` | Generates full weather briefing | `codes` (ICAO list), `include_notams` (bool) |
| `/api/convert-to-pirep` | `POST` | Converts text/voice to PIREP format | JSON body with `text`, `icao`, `aircraftModel` |
| `/download-briefing` | `GET` | Generates PDF download | `codes`, `include_notams` |
| `/` | `GET` | Serves the SPA application | None |

---

## Installation & Setup

### Prerequisites
*   Python 3.8+
*   `wkhtmltopdf` (Required for PDF generation)
*   Supabase Account (for Database)
*   Google Cloud Account (for Gemini API)

### Environment Configuration (.env)

| Variable | Description | Required | example |
| :--- | :--- | :--- | :--- |
| `GEMINI_API_KEY` | Google Gemini API Key for AI generation | **Yes** | `AIzaSy...` |
| `SUPABASE_URL` | Supabase Project URL | **Yes** | `https://xyz.supabase.co` |
| `SUPABASE_SERVICE_KEY` | Service Role Key for Admin Access | **Yes** | `eyJhbGciOiJIUzI1...` |
| `PILOT_PROFILE` | AI Persona for briefing context | No | `General aviation VFR pilot` |

### 1. Clone & Install
```bash
git clone <repository-url>
cd apache-msc
pip install -r requirements.txt
```

### 2. Database Setup
Run the SQL scripts in your Supabase SQL Editor to create the necessary tables:
*   Run content of `notams.sql`

### 3. Run the Application
```bash
python get_weather.py
```
Access the app at `http://localhost:5001`.

---

## User Workflow

1.  **Landing Page**: Click "Start Briefing".
2.  **Flight Planning**:
    *   Enter ICAO codes (e.g., `VIDP VABB`).
    *   Select Aircraft Type (Affects performance calcs).
    *   Toggle "Include NOTAMs" if desired.
    *   Click "Generate Briefing".
3.  **Briefing Dashboard**:
    *   **Read**: View the AI-generated "Primary Route Briefing".
    *   **Visualize**: Check the Map for route and flight categories.
    *   **Analyze**: Review the temperature/wind charts below the map.
    *   **Expand**: Click "Read More" for per-airport raw METAR/TAF data.
4.  **Export/Share**:
    *   Click "Download PDF" to get a comprehensive report.
    *   Click "Share" to copy the briefing URL.
5.  **Submit PIREP** (Optional):
    *   Use the "Convert PIREP" section or Voice Assistant to submit a report.

---

## Contribution

Contributions are welcome! Please follow these steps:
1.  Fork the repository.
2.  Create a feature branch (`git checkout -b feature/NewFeature`).
3.  Commit your changes.
4.  Push to the branch.
5.  Open a Pull Request.

---

## License

[MIT License](LICENSE) © 2025 ApacheAI Team
