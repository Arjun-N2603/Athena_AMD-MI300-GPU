# AI-Powered Agentic Scheduling Assistant using AMD MI300 GPU

## Overview
It is an innovative AI-powered scheduling assistant designed to autonomously coordinate meetings, resolve conflicts, and optimize calendar schedules. Built specifically for the AMD AI Hackathon, this solution leverages advanced Agentic AI and the power of the AMD Instinct MI300 GPU to eliminate the inefficiencies of manual scheduling and streamline workplace productivity.

It aims to go beyond traditional rule-based tools by reasoning like a human assistant, acting independently, and learning from user preferences.

## The Problem
Modern workplaces are often bogged down by inefficient meeting coordination:

- Meetings consume significant employee time, with many being unproductive.
- The back-and-forth of finding suitable meeting slots leads to wasted effort and delays.
- Manually resolving scheduling conflicts across multiple calendars is a complex and time-consuming task.

## The Solution

- **Autonomous Scheduling:** End-to-end meeting coordination without constant manual intervention.
- **Intelligent Conflict Resolution:** Automatically identifies and proposes optimal alternative time slots for all attendees.
- **Natural Language Interaction:** Users can schedule meetings using conversational inputs via an advanced Large Language Model (LLM).
- **Seamless Calendar Integration:** Synchronizes directly with Google Calendar for real-time availability and event management.
- **Time Optimization:** Designed to significantly reduce time spent on scheduling tasks, boosting overall productivity.

## Features

### Core Capabilities
-  **Autonomous Meeting Coordination:** Processes incoming meeting requests (e.g., from emails) and orchestrates the entire scheduling process, from parsing intent to creating calendar events.
-  **Natural Language Processing (NLP):** Utilizes an AI Agent powered by DeepSeek LLM to extract crucial meeting details (participants, duration, time constraints, preferred times) directly from email content.
-  **Smart Conflict Resolution & Slot Finding:** Leverages Google Calendar's Free/Busy API to identify available time slots across all attendees and proposes the best options, considering working hours and preferred times.
-  **Real-time Google Calendar Sync:** Integrates directly with Google Calendar to create, update, and fetch events, ensuring up-to-date schedule management.
-  **Preference Learning:** Incorporates user time preferences (e.g., 'morning', 'afternoon', specific hours) when finding optimal slots, prioritizing these preferences if possible.
-  **Detailed Metadata:** Provides comprehensive metadata in the output, including status messages, slot selection reasoning, number of API calls, LLM processing time, and conflicts resolved.

### Technical Features
- **vLLM Server Integration:** Utilizes vLLM for high-throughput and low-latency inference with Large Language Models, specifically the DeepSeek LLM 7B Chat Model, running on the AMD MI300 GPU.
- **Google Calendar API:** Extensive use of Google Calendar API v3 for calendar interactions.
- **Flask REST API:** A lightweight Flask application provides a robust HTTP interface (/receive) for receiving and processing meeting requests as JSON.
- **Modular Codebase:** Organized Python modules for clear separation of concerns, including calendar services, AI agents, and the main processing logic.
- **Robust Error Handling:** Mechanisms in place for gracefully handling invalid inputs or failures during LLM parsing or API calls.

##  Architecture

```
graph TD
    A[User Input (e.g., Email Content)] --> B{Flask API (Port 5000)}
    B --> C[AI Agent (SchedulingAIAgent)]
    C -- "Parses Email Content" --> D[vLLM Server (DeepSeek LLM, Port 3000)]
    D --> C
    C -- "Generates Slot Reasoning" --> D
    D --> C
    C -- "Requests Calendar Data" --> E[Google Calendar API]
    E -- "Provides Free/Busy, Events" --> C
    C -- "Creates/Updates Events" --> E
    E --> C
    C -- "Returns Processed Output" --> B
    B --> F[Final JSON Output]
```

### Flow Description:
1. **User Input:** An incoming meeting request, typically an email, is received.
2. **Flask API:** The request is captured by a Flask endpoint (/receive on port 5000).
3. **AI Agent (SchedulingAIAgent):** The core orchestrator (process_meeting_request) leverages the SchedulingAIAgent to parse the EmailContent and extract key meeting details.
4. **vLLM Server:** The SchedulingAIAgent communicates with a vLLM server (configured to run DeepSeek LLM) to perform Natural Language Processing tasks.
5. **Google Calendar API:** After parsing, the system interacts with Google Calendar to retrieve credentials, fetch events, query free/busy information, and create or update the meeting event.
6. **Final Output:** A comprehensive JSON response is returned, detailing the scheduled event and useful metadata.


### Prerequisites
- Python 3.8+
- Google Calendar API Credentials: client_secret.json files and generated token files (e.g., user.token) placed in a Keys/ directory.
- vLLM Server with DeepSeek LLM: Deployed on an AMD MI300 GPU instance.
- Required Python packages are listed in requirements.txt.

### Installation & Setup
1. **Clone the Repository:**
```bash
git clone https://github.com/Arjun-N2603/Athena_AMD-MI300-GPU.git
cd Athena_AMD-MI300-GPU
```

2. **Set Up Google Calendar Credentials:**
   - Follow Google's instructions to enable the Google Calendar API and download your client_secret.json.
   - Run a one-time script to generate *.token files for each user and place them in the Keys/ directory.

3. **Start the vLLM Server:**
```bash
HIP_VISIBLE_DEVICES=0 vllm serve /home/user/Models/deepseek-ai/deepseek-llm-7b-chat \
  --gpu-memory-utilization 0.9 \
  --swap-space 16 \
  --disable-log-requests \
  --dtype float16 \
  --max-model-len 2048 \
  --tensor-parallel-size 1 \
  --host 0.0.0.0 \
  --port 3000 \
  --num-scheduler-steps 10 \
  --max-num-seqs 128 \
  --max-num-batched-tokens 2048 \
  --distributed-executor-backend "mp"
```

4. **Run the Flask Server:**
```bash
python your__submission_file.py
```
The Flask server will run on http://0.0.0.0:5000.

## Usage Example

### Send a POST request to the /receive endpoint:
```bash
curl -X POST -H "Content-Type: application/json" -d '{
    "Request_id": "6118b54f-907b-4451-8d48-dd13d76033a5",
    "Datetime": "19-07-2025T12:34:55",
    "Location": "IISc Bangalore",
    "From": "userone.amd@gmail.com",
    "Attendees": [
        {
            "email": "usertwo.amd@gmail.com"
        },
        {
            "email": "userthree.amd@gmail.com"
        }
    ],
    "Subject": "Agentic AI Project Status Update",
    "EmailContent": "Hi team, let's meet on Thursday for 30 minutes to discuss the status of Agentic AI Project."
}' http://localhost:5000/receive
```

### Validation
You can send input JSON from your local laptop to the MI300 GPU instance and validate the response:

```python
import requests
import json

SERVER_URL = "<YOUR IP ADDRESSS>"
INPUT_JSON_FILE = "JSON_Samples/Input_Request.json"

with open(INPUT_JSON_FILE) as f:
    input_json = json.load(f)

response = requests.post(SERVER_URL+":5000/receive", json=input_json, timeout=10)
print(response.json())
```

### Expected Output Structure
```json
{
    "Request_id": "6118b54f-907b-4451-8d48-dd13d76033a5",
    "Datetime": "19-07-2025T12:34:55",
    "Location": "IISc Bangalore",
    "From": "userone.amd@gmail.com",
    "Attendees": [
        {
            "email": "userone.amd@gmail.com",
            "events": [
                {
                    "StartTime": "2025-07-24T10:00:00+05:30",
                    "EndTime": "2025-07-24T10:30:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Team Meet"
                },
                {
                    "StartTime": "2025-07-24T10:30:00+05:30",
                    "EndTime": "2025-07-24T11:00:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Agentic AI Project Status Update"
                }
            ]
        },
        {
            "email": "usertwo.amd@gmail.com",
            "events": [
                {
                    "StartTime": "2025-07-24T10:00:00+05:30",
                    "EndTime": "2025-07-24T10:30:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Team Meet"
                },
                {
                    "StartTime": "2025-07-24T10:30:00+05:30",
                    "EndTime": "2025-07-24T11:00:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Agentic AI Project Status Update"
                }
            ]
        },
        {
            "email": "userthree.amd@gmail.com",
            "events": [
                {
                    "StartTime": "2025-07-24T10:00:00+05:30",
                    "EndTime": "2025-07-24T10:30:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Team Meet"
                },
                {
                    "StartTime": "2025-07-24T13:00:00+05:30",
                    "EndTime": "2025-07-24T14:00:00+05:30",
                    "NumAttendees": 1,
                    "Attendees": [
                        "SELF"
                    ],
                    "Summary": "Lunch with Customers"
                },
                {
                    "StartTime": "2025-07-24T10:30:00+05:30",
                    "EndTime": "2025-07-24T11:00:00+05:30",
                    "NumAttendees": 3,
                    "Attendees": [
                        "userone.amd@gmail.com",
                        "usertwo.amd@gmail.com",
                        "userthree.amd@gmail.com"
                    ],
                    "Summary": "Agentic AI Project Status Update"
                }
            ]
        }
    ],
    "Subject": "Agentic AI Project Status Update",
    "EmailContent": "Hi team, let's meet on Thursday for 30 minutes to discuss the status of Agentic AI Project.",
    "EventStart": "2025-07-24T10:30:00+05:30",
    "EventEnd": "2025-07-24T11:00:00+05:30",
    "Duration_mins": "30",
    "MetaData": {}
}
```

## Performance Metrics

- **status:** Indicates the outcome (e.g., "success", "warning", "error").
- **llm_processing_time:** Measures the time for the AI Agent to parse the email content.
- **api_calls:** Counts the number of Google Calendar API calls per request.
- **conflicts_resolved:** A binary indicator of success in finding a conflict-free slot.
- **preferred_time_alloted:** Boolean indicating if the user's preferred time was honored.
- **attempted_days:** The number of days searched to find an optimal slot.

## Impact & Benefits

- **Boosting Productivity:** Significantly reduces time spent on scheduling.
- **Minimizing Friction:** Eliminates back-and-forth scheduling emails.
- **Enhancing User Experience:** Provides a natural, conversational scheduling interface.
- **Ensuring Accuracy:** Reduces human error in managing conflicts.

##  Acknowledgments
- **AMD AI Hackathon:** For the platform and the inspiring challenge.
- **DeepSeek AI:** For the powerful language model used in our AI Agent.
- **Google Calendar API:** For enabling seamless calendar integration.
- **vLLM Team:** For their efficient inference server, crucial for our LLM performance.