# AI Customer Support Automation with n8n

Two n8n workflows that work together to handle a customer support inbox. One drafts replies to new emails using AI. The other sends a daily report on what customers are asking about.
The example shows fake event information and contact details due to GDPR. 

## Workflows

### 1. Support Email Reply Drafter

Runs every minute and checks Gmail for new unread emails.

```
New Support Email (Gmail trigger)
  └─ Is Customer Enquiry? (AI text classifier)
       ├─ Draft reply
       │    Get Support Knowledge (Google Docs)
       │      → Extract Text and Links (Code)
       │      → Write Reply (AI agent + read_web_page tool)
       │      → Save Reply as Draft (Gmail, in the original thread)
       │      → Mark Email as Read
       └─ Log for daily report
            Analyze Topic and Sentiment (AI information extractor)
              → Log Enquiry (n8n Data Table)
```

- **Filtering:** an LLM classifier sorts real customer enquiries from automated mail, newsletters, sales pitches and spam. Everything that isn't an enquiry is discarded.
- **Grounded answers:** the reply agent can only use facts from a Google Doc knowledge base. It can also open pages linked from that doc with the `read_web_page` tool, but it may only fetch URLs from the doc's own link list. This blocks prompt injection through links in incoming emails.
- **Human in the loop:** replies are saved as Gmail **drafts** in the original thread and are never sent automatically.
- **Same language:** the agent replies in whatever language the customer wrote in.
- **Analytics:** each enquiry gets a topic, a sentiment and a one-line summary, and is logged to a Data Table.


<img width="1998" height="1151" alt="Screenshot 2026-10-16 131813" src="https://github.com/user-attachments/assets/e88dbdcc-2eca-43aa-b795-080d9eca30d4" />



### 2. Daily Support Report

Runs every day at 08:00 (Europe/Copenhagen).

```
Every Day at 8:00 → Get Last 24h Enquiries (Data Table)
  → Build Stats (Code) → Write Report (LLM) → Email Report (Gmail)
```

It gathers the last 24 hours of logged enquiries and counts them by sentiment and topic. An LLM then writes a short HTML report covering an **overview**, the **main enquiries** and **issues to watch**, with suggested fixes. The report goes out by email.

## Tech

- **n8n**: Gmail Trigger, Gmail, Google Docs, Data Table, Code, HTTP Request Tool, Schedule Trigger
- **n8n LangChain nodes**: Text Classifier, Information Extractor, AI Agent, Basic LLM Chain
- **OpenAI** `gpt-5.4-mini` as the model


<img width="1711" height="1233" alt="Screenshot 2026-10-16 124238" src="https://github.com/user-attachments/assets/b6b32906-9425-433b-a59e-8290d0fdcf92" />


## Setup

1. In n8n, go to **Workflows → Import from File** and import both JSON files.
2. Create credentials and assign them to the nodes:
   - Gmail OAuth2
   - Google Docs OAuth2
   - OpenAI API
3. Create a Data Table called `Support Enquiry Log` with these columns:
   `received_at` (dateTime), `customer_email`, `subject`, `topic`, `sentiment`, `summary`, `message_id`.
   Then point both Data Table nodes at it.
4. In **Get Support Knowledge**, replace `YOUR_GOOGLE_DOC_ID` with your own knowledge base doc.
5. In **Email Report**, replace `you@example.com` with the address that should get the daily report.
6. Change the classifier and topic descriptions to fit your business. They currently describe a fictional event called "ExampleConf".
7. Activate both workflows.



<img width="1998" height="1151" alt="Screenshot 2026-10-16 131813" src="https://github.com/user-attachments/assets/0489a512-fa7f-4bc2-ad9b-205159796d38" />



## OUTPUT

Email response saved as a draft, ready for human correction; 

<img width="2159" height="323" alt="Screenshot 2026-10-16 141419" src="https://github.com/user-attachments/assets/08f296cb-3877-41b9-bc7f-f705961ef4ed" />

At the start of the day, the email recveives a summary of the last 24 hours of inquiries with semantic analysis and short summaries;

<img width="1361" height="391" alt="Screenshot 2026-10-16 122939" src="https://github.com/user-attachments/assets/b4eea461-3ccd-4e5f-a18f-f47c2122752a" />


