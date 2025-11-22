Below is the **final, complete, professional README.md**
custom-written for **your exact workflow**, from the **first node to the last node**
including **setup instructions for anyone who wants to run this on their own laptop**.

It is clean, well structured, and production-ready for GitHub.

---

# ⭐ **README.md — FINAL VERSION**

````markdown
# 🧠 Automated Transcript Rubric Evaluation System (n8n Workflow)

This project is a complete **Transcript Evaluation Engine** built using **n8n**.  
It accepts a student introduction transcript, analyzes it using NLP and rule-based scoring, and generates a **full rubric evaluation**, including:

- Content & Structure Score  
- Speech Rate Score  
- Grammar Score  
- Vocabulary Score  
- Clarity Score  
- Engagement / Sentiment Score  
- Final HTML Report (Email or Browser Output)

This README explains **every node from start to finish**, **installation**, **API usage**, and how **others can run it on their own laptop**.

---

# 🔧 Technology Stack

| Component              | Technology |
|------------------------|------------|
|Input Method               POST Webhook  
  Automation Engine      | n8n        |
| Grammar Checking       | LanguageTool API (Free) |
| Sentiment Analysis     | External Free API |
| Vocabulary Metrics     | TTR (Computed via Code Node) |
| Reporting              | HTML + Email Node |
| Email Delivery         | Gmail or Outlook SMTP |


---

# 📂 Workflow Architecture (Node by Node)

Here is the complete sequence of nodes used in the workflow.

---

# 1️⃣ **Webhook Node (POST Input)**  
**Purpose:** Accepts transcript text input.

- **Node Name:** `Webhook`  
- **Method:** `POST`  
- **Path:** `/rubric-evaluator`  
- **Output Format:**

```json
[
  {
    "transcript": "Hello everyone, myself Muskan...",
    "submittedAt": "2025-11-22T18:53:59.017+05:30",
    "formMode": "test"
  }
]
```

---

# 2️⃣ **Fluency Metrics Node (Code Node)**

**Purpose:** Extracts:

* Word Count
* Sentence Count
* Filler Word Count
```

No dependency on the “Webhook” node name.

---

# 3️⃣ **Grammar Node (HTTP Request → LanguageTool API)**

**Purpose:** Detect grammar errors.

### Endpoint:

```
https://api.languagetool.org/v2/check
```

### Body:

```json
{
  "text": "{{$input.first().json.transcript}}",
  "language": "en-US"
}
```

### Outputs:

* Number of grammar errors
* Errors per 100 words
* Grammar Score (2–10)

---

# 4️⃣ **Sentiment Analysis Node (HTTP Request)**

**Purpose:** Retrieve sentiment probability.

You can use **any free API**, including the one you added that returns:

```json
{
  "sentimentLabel": "pos",
  "probability": { "pos": 0.88, "neg": 0.10, "neutral": 0.02 }
}
```

---

# 5️⃣ **Sentiment Parse Node (Code Node)**

**Purpose:** Convert sentiment output to rubric score.

```javascript
const transcript = $input.first().json.transcript || "";
const inp = $json;

return [
  {
    json: {
      transcript,
      sentimentLabel: inp.sentimentLabel || "neutral",
      positiveProb: inp.probability?.pos ?? 0
    }
  }
];
```

---

# 6️⃣ **Rubric Generator Node (Code Node)**

This node applies the entire rubric:

✔ Content & Structure
✔ Salutation
✔ Mandatory Details
✔ Good-to-have Details
✔ Order Flow
✔ Speech Rate
✔ Grammar Score
✔ Vocabulary TTR Score
✔ Clarity (Filler Word Rate)
✔ Engagement (Sentiment Score)
✔ Final Total Score
✔ Comments

It collects values from all previous nodes using:

```
$node["Fluency Metrics"]
$node["Grammar"]
$node["Sentiment Parse"]
...
```

And generates a clean JSON output.

---

# 7️⃣ **HTML Output Node**

Generates a styled HTML report:

* Final Score
* Category-wise breakdown
* Comments
* Timestamp

Used in:

✔ Email Node
✔ Return to Browser Node

---

# 8️⃣ **Return to Browser (Respond to Webhook)**

**Purpose:** Returns the final HTML report when testing via browser/Postman.

Set Response With → **"Using Respond to Webhook Node"**

---

# 9️⃣ **Email Send Node (Optional)**

**Purpose:** Sends report to teachers/parents.
---

# 🔐 What Other Users MUST Change to Run This Project

Anyone installing this project must update:

### ✔ 1. Email Credentials

Gmail / Outlook / SMTP login. using your personal login email with password to get report into your email

### ✔ 3. Webhook Public URL (its free for only 10-15 triggers)

If using n8n Desktop or Self-hosted.

Example:

Replace:

```
https://charan29.app.n8n.cloud/webhook/rubric-evaluator
```

With:

```
http://localhost:5678/webhook/rubric-evaluator

but i suggest to use cloud n8n
```

### ✔ 4. (Optional) LanguageTool Local Server

If they want offline grammar check.

---

# 💻 Running This Project on Another User’s Laptop

Here are exact steps for ANY new user:

---

## 1️⃣ **Install n8n**


### option A - cloud n8n online browser

create your profile and login with credentials


### OPTION B — n8n Desktop (Simple)

Download from:
[https://n8n.io/desktop](https://n8n.io/desktop)

### OPTION C — Docker

```bash
docker run -it --rm \
  -p 5678:5678 \
  -v ~/.n8n:/root/.n8n \
  n8nio/n8n
```


---

## 2️⃣ **Import the Workflow**

* Go to n8n → Workflows → left corner 3 dots -> Import
* Select `rubric-evaluator.json` from git hub 

---

## 3️⃣ **Update Email Credentials**

(Open → Email Node → Credentials → Add New)

---

## 4️⃣ **Start Testing**

start your workflow first

then run postman command 

Use Postman:
method : POST 
URL : https://charan29.app.n8n.cloud/webhook-test/rubric-evaluator
BODY : JSON
{
  "transcript": "Hello everyone, myself Muskan, studying in class 8th B section from Christ Public School. I am 13 years old. I live with my family. There are 3 people in my family, me, my mother and my father.One special thing about my family is that they are very kind hearted to everyone and soft spoken. One thing I really enjoy is play, playing cricket and taking wickets.A fun fact about me is that I see in mirror and talk by myself. One thing people don't know about me is that I once stole a toy from one of my cousin.My favorite subject is science because it is very interesting. Through science I can explore the whole world and make the discoveries and improve the lives of others. Thank you for listening"
}

 FINALLY YOU WILL GET EMAIL OF REPORT:

Second option :
```bash
POST http://localhost:5678/webhook/rubric-evaluator
Content-Type: application/json

{
  "transcript": "Hello everyone…"
}
```

---

# 📝 Example Output (JSON)

```json
{
  "totalScore": 82,
  "contentScore": 33,
  "grammarScore": 6,
  "vocabularyScore": 6,
  "clarityScore": 15,
  "sentimentScore": 12,
  "comments": "Very good clarity and engagement."
}
```

---

# 🖨 HTML Output Example

The HTML Output Node produces a complete styled scorecard, ready for printing or emailing.



# 📜 License

MIT License

---

# 👨‍💻 Author

Charan B

---

# 🎉 Thank You

This project is now fully shareable, installable, and reusable for anyone using n8n.

```

---

If you want, I can also generate:

✅ `rubric-logic.md`  
✅ `api-schema.json`  
✅ Workflow export banner image  
✅ Project logo  

Just tell me: **"Generate rubric-logic.md"**
```
