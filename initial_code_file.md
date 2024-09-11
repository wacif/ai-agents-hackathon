To implement the AI-powered document management system with automated legal document generation, the code snippets for each stage of the process will written below. For now i'll break it down into parts for easier integration.

---

### **1. Document Classification using BERT**

We’ll use the Hugging Face `transformers` library to fine-tune a pre-trained BERT model to classify legal documents.

**Install Dependencies:**
```bash
pip install transformers datasets torch
```

**Code for Fine-tuning BERT for Document Classification:**
```python
from transformers import BertTokenizer, BertForSequenceClassification, Trainer, TrainingArguments
from datasets import load_dataset

# Load tokenizer and model
tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertForSequenceClassification.from_pretrained('bert-base-uncased', num_labels=3)  # Example: 3 categories

# Load your dataset
dataset = load_dataset('your_dataset_path', split='train')

# Tokenize the text
def preprocess_function(examples):
    return tokenizer(examples['text'], padding='max_length', truncation=True)

tokenized_dataset = dataset.map(preprocess_function, batched=True)

# Fine-tune the model
training_args = TrainingArguments(
    output_dir='./results',
    evaluation_strategy="epoch",
    learning_rate=2e-5,
    per_device_train_batch_size=16,
    num_train_epochs=3,
)

trainer = Trainer(
    model=model,
    args=training_args,
    train_dataset=tokenized_dataset,
)

trainer.train()
```

This code fine-tunes the BERT model to classify documents into categories like “Contracts,” “Invoices,” and “NDAs.” After training, you can use the model to classify incoming documents.

---

### **2. Automated Data Extraction using OCR and NLP**

For this part, we'll use Tesseract for OCR and SpaCy for Named Entity Recognition (NER).

**Install Dependencies:**
```bash
pip install pytesseract spacy
python -m spacy download en_core_web_sm
```

**OCR and Entity Extraction Code:**
```python
import pytesseract
from PIL import Image
import spacy

# Load SpaCy model for NER
nlp = spacy.load('en_core_web_sm')

# Function to extract text from an image or PDF using OCR
def extract_text_from_image(image_path):
    img = Image.open(image_path)
    return pytesseract.image_to_string(img)

# Function to extract named entities using SpaCy
def extract_entities(text):
    doc = nlp(text)
    entities = {}
    for ent in doc.ents:
        entities[ent.label_] = ent.text
    return entities

# Example usage
image_path = 'path_to_scanned_document.jpg'
text = extract_text_from_image(image_path)
entities = extract_entities(text)
print("Extracted Entities:", entities)
```

This code uses Tesseract to extract text from scanned documents and SpaCy to extract entities like names, dates, and addresses.

---

### **3. Template-Based Document Generation using Jinja2**

We’ll generate legal documents by filling in a DOCX template with case-specific data using Jinja2 and `python-docx`.

**Install Dependencies:**
```bash
pip install python-docx Jinja2
```

**Code for Document Generation:**
```python
from jinja2 import Template
import docx

# Define a template with placeholders
doc_template = """
Dear {{ client_name }},

This is your contract regarding {{ case_type }}. The contract is valid until {{ end_date }}.

Sincerely,
Your Legal Team
"""

# Sample data to fill in the template
data = {
    'client_name': 'John Doe',
    'case_type': 'Property Dispute',
    'end_date': '2024-12-31'
}

# Render the template
template = Template(doc_template)
filled_template = template.render(data)

# Create a Word document and write the filled template
doc = docx.Document()
doc.add_paragraph(filled_template)
doc.save('generated_contract.docx')

print("Document generated and saved as 'generated_contract.docx'")
```

This code fills in a simple contract template with dynamic data like client name and case type, then generates the document in DOCX format.

---

### **4. Automated Workflow for Approvals and E-Signatures**

For e-signatures, we’ll use DocuSign's API. You can follow the official DocuSign documentation to set up the API and integrate it with your system. Below is an example using Python’s `requests` library to send documents for signature.

**Install Dependencies:**
```bash
pip install requests
```

**Code for Sending Document for E-Signature:**
```python
import requests

# Define DocuSign API endpoint and headers
url = "https://demo.docusign.net/restapi/v2.1/accounts/{account_id}/envelopes"
headers = {
    "Authorization": "Bearer YOUR_ACCESS_TOKEN",
    "Content-Type": "application/json"
}

# Define the payload with document and recipient info
payload = {
    "documents": [
        {
            "documentBase64": "BASE64_ENCODED_DOCUMENT",
            "name": "Contract",
            "fileExtension": "pdf",
            "documentId": "1"
        }
    ],
    "recipients": {
        "signers": [
            {
                "email": "client@example.com",
                "name": "John Doe",
                "recipientId": "1",
                "routingOrder": "1"
            }
        ]
    },
    "status": "sent"
}

# Send the request to DocuSign
response = requests.post(url, json=payload, headers=headers)

if response.status_code == 201:
    print("Document sent for signature")
else:
    print(f"Failed to send document: {response.text}")
```

This code sends a document to the DocuSign API for an electronic signature.

---

### **5. Dynamic Clause Insertion using Rule-Based System**

We’ll define a simple rule engine for selecting and inserting clauses based on case type.

**Code for Rule-Based Clause Insertion:**
```python
# Define a set of clauses
clauses = {
    'property_dispute': 'This contract involves property rights and ownership disputes.',
    'intellectual_property': 'This contract involves protection of intellectual property rights.'
}

# Function to insert clause based on case type
def insert_clause(case_type):
    return clauses.get(case_type.lower(), "No specific clauses for this case type.")

# Example usage
case_type = 'property_dispute'
clause = insert_clause(case_type)
print(f"Inserted Clause: {clause}")
```

This simple rule engine selects and inserts the appropriate legal clause based on the case type.

---

### **6. Case Intake and Data Extraction Integration**

**Code for Integrating Case Intake:**
You can use APIs or data from a case management system, then extract the case details for document generation.

```python
import requests

# Function to pull case data from case management API
def get_case_data(case_id):
    url = f"https://your_case_management_api.com/cases/{case_id}"
    response = requests.get(url)
    
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Error fetching case data: {response.text}")
        return None

# Example usage
case_data = get_case_data('12345')
if case_data:
    client_name = case_data.get('client_name', 'Unknown')
    case_type = case_data.get('case_type', 'Unknown')
    print(f"Client: {client_name}, Case Type: {case_type}")
```

This code pulls case data from a case management API, which can then be fed into your document generation workflow.

---

### **7. Legal Document Scheduling using APScheduler**

For scheduling, use APScheduler to automate document generation at intervals (e.g., monthly reports).

**Install Dependencies:**
```bash
pip install apscheduler
```

**Code for Scheduling Document Generation:**
```python
from apscheduler.schedulers.blocking import BlockingScheduler
import datetime

# Define the task to generate documents
def generate_report():
    print(f"Generating report at {datetime.datetime.now()}")

# Schedule the task to run daily
scheduler = BlockingScheduler()
scheduler.add_job(generate_report, 'interval', days=1)

scheduler.start()
```

This code sets up a scheduled task that generates reports every day.

---
