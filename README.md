# AI-Automation
This document the learning process of AI automation. this mark the first workflow I executed. The workflow consist of a schedule trigger(google sheet) which is connected to a Basic LLM engine and Gemini and Gmail (send message).
# 📧 AI-Powered Welcome Email Automation (n8n + Gemini + Gmail)
An automation that sends a **personalized, AI-written welcome email** to everyone who submits a Google Form, with no manual work after setup.
When someone fills the form, their response lands in Google Sheets. n8n detects the new row, Google Gemini writes a welcome message based on what the person is interested in, and Gmail sends it to them automatically.
<img width="1196" height="550" alt="Screenshot 2026-11-06 093009" src="https://github.com/user-attachments/assets/79f6cf9d-a401-49d0-b3dc-4c3daeae849d" />
# 🎯 Problem It Solves
Manually welcoming every new subscriber or form respondent is slow and easy to forget. Generic auto-replies feel cold. This workflow:
- Responds to every new sign-up automatically
- Personalizes each message using the respondent's own answers
- Saves time while keeping the tone warm and human
---
# 🔄 How It Works
```mermaid
flowchart LR
    A[Google Form] --> B[Google Sheet]
    B --> C[Google Sheets Trigger]
    C --> D[Basic LLM Chain]
    E[Google Gemini Chat Model] -.-> D
    D --> F[Gmail: Send a message]
    F --> G[Respondent's inbox]
```
**Google Form**: a person submits their name, email, and interest.
<img width="1337" height="641" alt="Screenshot 2026-11-06 094810" src="https://github.com/user-attachments/assets/0b0e0d5b-de93-4b00-9d8a-83eba11ab563" />
**Google Sheets Trigger**: n8n watches the response sheet and fires when a new row is added.
<img width="1325" height="574" alt="Screenshot 2026-11-06 094902" src="https://github.com/user-attachments/assets/613246be-7e68-42e4-8c86-e9722400a9f4" />
**Basic LLM Chain**: builds a prompt using the respondent's details.
<img width="1346" height="581" alt="Screenshot 2026-11-06 094631" src="https://github.com/user-attachments/assets/4077207b-fb37-48a6-a49e-c7dbdd92d425" /> 
**Google Gemini Chat Model**: the AI model connected to the chain that generates the email text.
 <img width="1342" height="603" alt="Screenshot 2026-11-06 095052" src="https://github.com/user-attachments/assets/13ec80eb-e5c6-498d-b485-77cddd3820be" />
**Gmail (Send a message)**: sends the generated message to the respondent's email address.
<img width="884" height="541" alt="Screenshot 2026-11-06 092713" src="https://github.com/user-attachments/assets/d6e31a95-c66b-4c2e-80b5-502daff49134" />
<img width="1335" height="592" alt="Screenshot 2026-11-06 095326" src="https://github.com/user-attachments/assets/8b7a9070-21d5-40b0-b898-537d0770420f" />
---
## 🛠️ Tech Stack
| Tool | Purpose |
|------|---------|
| [n8n](https://n8n.io) | Workflow automation platform |
| Google Forms | Collects respondent details |
| Google Sheets | Stores form responses |
| Google Gemini | Generates the personalized welcome message |
| Gmail | Sends the email |
---
## ⚙️ Node Configuration
### 1. Google Sheets Trigger
- **Trigger On:** Row Added
- **Document:** the Google Sheet linked to the form
- **Sheet:** Form responses 1
- Run **Fetch Test Event** to load a sample row while building
### 2. Basic LLM Chain
- **Source for Prompt:** Define below
- **Prompt example:**
You are a friendly and professional welcome assistant. Generate a personalised welcome message using the information below.
Name: {{ $json['Full name'] }}
Email: {{ $json['Email Account'] }}
Interest: {{ $json['What are you interested in'] }}
Welcome the person warmly. Mention their area of interest. Keep the message engaging, natural and professional. Do not invent personal information. Return only the welcome message.
```
```
### 3. Google Gemini Chat Model (sub-node)
- Attached to the **Chat Model** connector of the Basic LLM Chain
- **Credential:** Gemini API key from [Google AI Studio](https://aistudio.google.com)
- **Model:** a Gemini Flash model
### 4. Gmail: Send a message
- **Resource:** Message
- **Operation:** Send
- **To:** `{{ $('Google Sheets Trigger').item.json['Email Account'] }}`
- **Subject:** `Welcome Message`
- **Email Type:** Text
- **Message:** `{{ $json.text }}`
> The recipient is read from the trigger node because the LLM chain's output only contains the generated text, not the original sheet columns.
---
## 🚀 Setup Guide
1. **Create a Google Form** with at least an email field and an interest question, and link it to a Google Sheet.
2. **Open n8n** and create a new workflow.
3. **Add the nodes** in the order above and connect the Gemini Chat Model under the Basic LLM Chain.
4. **Add credentials:** Google Sheets (OAuth), Gemini (API key), and Gmail (OAuth).
5. **Test** by submitting the form, then run the workflow and check the inbox.
6. **Activate** the workflow so it runs automatically.
---
## 🐞 Troubleshooting
| Error | Cause | Fix |
|-------|-------|-----|
| `Bad request - Invalid To header` | The email in the sheet is incomplete (e.g. `name@gmail` with no `.com`) | Correct the address in the sheet and re-run. Prevent it by enabling **Response validation → Text → Email** on the Google Form. |
| Empty recipient | Column name in the expression doesn't match the sheet | Match the header exactly, including spaces and capitalization |
| Line breaks collapse | Gmail sends HTML by default | Set Email Type to Text, or ask the model to return simple HTML |
| Same email sent twice | Old rows re-triggered during testing | Keep the workflow active once and avoid re-running on old rows |
---
## 📚 What I Learned
- How **triggers** start a workflow automatically from a data event
- How to connect an **AI model as a sub-node** to a Basic LLM Chain
- Using **expressions** to pass data between nodes
- Why the data from earlier nodes must be referenced explicitly (`$('Node Name')`)
- Reading n8n error messages and tracing them back to the source data
---
## 🔮 Future Improvements
- Add an **IF node** to validate email addresses before sending
- Log sent emails in a Google Sheet column (e.g. "Email Sent ✅")
- Send different messages depending on the respondent's interest
- Add a **Wait** and a follow-up email a few days later
---
## 👤 Author
**Aleshinloye Yusuf Olatunji** (Alesh)
Data analyst in training, educator, and Data analyst**.
---
⭐ If you found this useful, feel free to star the repo.
