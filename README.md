
# 🧠 NIFTY_50_Company_Information_Extraction
---
---

## 📋 Overview

# Problem

Most LLM models do not have direct internet access. Because of this, they mainly function as text generation models and cannot retrieve real-time or latest information on their own.

As a result:
- The model may provide outdated answers
- It cannot access live web data
- It cannot perform real-time research
- Accuracy decreases for current events or newly published information

# Solution

To solve this problem, external knowledge retrieval systems such as RAG (Retrieval-Augmented Generation) are integrated with LLM models.

Tools like STORM, developed by researchers at Stanford University, help enable this workflow. STORM is an AI-powered research and knowledge synthesis system that connects AI workflows with external web knowledge sources.

Using this approach:

- The AI model can retrieve internet-based information  
- Latest and real-time data can be accessed  
- Responses become more accurate and up-to-date  
- External documents, websites, and search engines can be integrated into the AI pipeline  

This creates a hybrid AI system where:

The LLM model handles reasoning and response generation  
External retrieval systems provide real-time knowledge and web information  

This n8n workflow is designed to automate the extraction of detailed company information from input documents generated using STORM WebDriver Selenium, combined with an integrated LLM (Large Language Model), to produce a fully structured company profile.

---

## 📋 List of Extracted Parameters

The workflow extracts and organizes the following major sections:

- ✅ **Company_Name**

- ✅ **Basic_Overview**
  - Legal Name
  - Type of Entity
  - Traded As (BSE, NSE, ISIN, Ticker Symbol)
  - Industry, Sector, Year Founded
  - Founders
  - Headquarters (Address, Contact Details)
  - Key People (with roles, bios, tenure)
  - Mission & Vision
  - Values, Tags

- ✅ **Business_Segments**
  - Segment Details, Products & Services, Competitors
  - Sub-segments, KPIs, Market Share, Growth Rate

- ✅ **Corporate_Structure**
  - Market Cap, Enterprise Value
  - Shareholding Patterns
  - Subsidiaries, Divisions, Joint Ventures, Affiliates

- ✅ **Financial_Highlights**
  - Latest Financials (Revenue, Net Income, EPS, AUM)
  - Cash Flow, Ratios, Historical Data, Growth Drivers

- ✅ **Company_History**
  - Key Historical Events, Milestones, Impacts

- ✅ **Acquisitions_And_Partnerships**
  - M&A Activity, Partnerships, Outcomes

- ✅ **Operations_And_Strategies**
  - Tech Investments, Expansions, Sustainability, BCPs

- ✅ **SWOT_Analysis**
  - Strengths, Weaknesses, Opportunities, Threats

- ✅ **Controversies_And_Legal_Issues**
  - Regulatory Actions, Outcomes, Reputation Impacts

- ✅ **Awards_And_Recognitions**
  - Awards, Categories, Issuers, Significance

- ✅ **Strategic_And_Market_Position**
  - Investment Philosophy, Distribution, Branding

- ✅ **Recent_Strategic_Focus_And_Outlook**
  - Digitization, Growth, Risks, Management Guidance

- ✅ **Conclusion**
  - Summary of the Full Company Profile

---
---
# ✒️PART 1
---
---

## N8N WORKFLOW - (WITHOUT RELAYING ON THE STORM API)
![image](https://github.com/user-attachments/assets/f6d11cc8-759d-4d40-9a33-117ee89fd13f)

## 📋 Overview

This workflow is designed to **automatically generate detailed company profiles** using only the **input parameters provided**—without relying on the STORM API or any third-party data enrichment tools.

## 🛠 Workflow Nodes & Descriptions
---

### 1️⃣ Schedule Trigger Node
**⚙ What It Does:**  
Starts the workflow automatically (time-based schedule).

### 2️⃣ Google Sheets Node
**Name of the Node:** `Google Sheets`  
**⚙ What It Does:**  
Reads the sheet where company data is stored.

### 3️⃣ Loop Over Node
**Name of the Node:** `Loop Over Items - 1`  
**⚙ What It Does:**  
Iterates through each row of data (likely each company).

### 4️⃣ Edit Node
**Name of the Node:** `SELECT Company Name, Industry, Symbol, ISIN Code`  
**⚙ What It Does:**  
Move Forward with Only Selective Parameters:
- Company Name
- Industry
- Symbol
- ISIN Code

### 5️⃣ AI Agent Node (Deepseek Model - deepseek/deepseek-r1-distill-qwen-32b):
**Name of the Node:** `Extract Company Name`  
**⚙ What It Does:**  
Takes the selected parameters from the previous node as input and uses a **crafted prompt with the integrated DeepSeek R1 Qwine 70B Free Model** to extract all required company's output parameters,

### 6️⃣ Edit Node
**Name of the Node:** `Company_Info_Output`  
**⚙ What It Does:**  
Collects the AI Agent output.

### 7️⃣ Code Node
**Name of the Node:** `Return Cleaned Output`  
**⚙ What It Does:**  
It **cleans** the AI Agent output.

### 8️⃣ Code Node
**Name of the Node:** `Company_Info_Output TO JSON Format`  
**⚙ What It Does:**  
Converts the result into **strict JSON** (ready for database insertion).

### 9️⃣ Postgres Node (Insert)
**Name of the Node:** `Insert_Result_To_Companies_Details_Table`  
**⚙ What It Does:**  
Inserts data into the table.

## Output

Some parameters in the output are generated accurately, but most of the values are returned as null.

---
---
# ✒️PART 2
---
---

## N8N WORKFLOW  (USING STORM WITH SELENIUM)
![image](https://github.com/user-attachments/assets/2d8ccfb7-39e1-4a9d-a1a8-ab88372c4c6d)

## 📋 Overview

This workflow is designed using STORM with Selenium WebDriver. Selenium WebDriver clicks on the platform’s feature to download article PDFs automatically.

The downloaded PDF is then passed to a custom Python script, which extracts the text content from the document. The extracted text is further provided to the AI Agent as contextual input for processing and structured information generation.

---

## 📥 Part A → Input Ingestion & Preprocessing

**Purpose:**

This section pulls raw company data files (from **Google Drive**), extracts their contents, loops over Excel rows, applies any needed cleaning/patching, and prepares them for summary analysis.

---

### 1️⃣ Schedule Trigger

**⚙ What It Does:**  
Automatically starts the workflow on a defined schedule (e.g., daily, weekly).

### 2️⃣ Google Drive Node

**Name of the Node:** `Google Drive`  
**⚙ What It Does:**  
Connects to Google Drive to pull the target Excel file containing company data.

### 3️⃣ Extract from File Node

**Name of the Node:** `Node Excel File`  
**⚙ What It Does:**  
Reads the Excel file content and parses it into usable JSON rows.

### 4️⃣ Loop Over Node

**Name of the Node:** `Loop Over Items - 1`  
**⚙ What It Does:**  
Iterates over each extracted row from the Excel file to process one company at a time.

### 5️⃣ Edit Node

**Name of the Node:** `Select Company Name and Website`  
**⚙ What It Does:**  
Transforms the data by rearranging or modifying fields.  
Here, it selects only the **Company Name and Website** from all parameters to proceed further.

### 6️⃣ AI Agent – (Gemini Model - google/gemini-2.0-flash-001)

**Name of the Node:** `Title and Response Generate`  
**⚙ What It Does:**  
Integrates with the Gemini LLM model.

- **Model:** `google/gemini-2.0-flash-001`
- **Input:** Company Name and Website
- **Output:** Uses a crafted prompt that generates **50 enriched Titles and Responses** for each company, covering all company information parameters.
  
### 7️⃣ Code Node

**Name of the Node:** `Cleaning Node`  
**⚙ What It Does:**  
Runs additional cleanup to remove noise, special characters, or irrelevant content from the text.

- Cleans up strings
- Removes triple backticks if present
- Fixes common issues (like unnecessary escape characters)
- Converts the string into actual JSON

### 8️⃣ Loop Over Item (Split in Batches) Node

**Name of the Node:** `Loop Over Items - 2`  
**⚙ What It Does:**  
Iterates over each cleaned Title and Response, processing one company at a time and preparing for external API calls.

### 9️⃣ HTTP Request

**Name of the Node:** `STORM API`  
**⚙ What It Does:**  
Sends API requests to **STORM** to get additional responses or enrichments.


---

## 🔄 Part 2 → Dual Path Processing (With & Without Summary)

**Purpose:**

This part intelligently splits the flow:

- ✅ **If the API response exists →** it’s passed forward for merging and profiling.
- ❌ **If the API response does not exist →** it is passed to a second HTTP Request (STORM API) to retrieve the missing response.

### 1️⃣ Edit Node

**Name of the Node:** `Title/Response/Summary - 1`  
**⚙ What It Does:**  
After completing all 50 iterations of API calls, this node reorganizes the data — structuring it into **Title, Response, and API-Response** from the previous HTTP loop.

### 2️⃣ IF Node

**Name of the Node:** `if`  
**⚙ What It Does:**  
Checks if an API-Response exists and splits the flow into two branches:

- 🔵 **True branch:** has API-Response
- 🔴 **False branch:** missing API-Response

## 🔵 True Branch → Merge and Clean → Wait at Merge Node Input 1

- **Code Node**  
  **Name:** `Merging entire text before brand profiling result - 1`  
  **⚙ What It Does:**  
  Merges all existing API-Response texts together, preparing for the next cleaning step.

- **Code Node**  
  **Name:** `Cleaning code node before brand profiling_1`  
  **⚙ What It Does:**  
  Cleans the merged text by:
  - Removing all escaped newline characters (`\n` or `\\n`)
  - Removing curly brackets `{` and `}`
  - Removing all double quotes (`"`)

- **Merge Node (Input 1)**  
  **Name:** `Merge`  
  **⚙ What It Does:**  
  After cleaning, the text proceeds to **Merge Node Input 1** and waits for **Input 2**.

## 🔴 False Branch → Retry Missing Responses → Merge and Clean → Wait at Merge Node Input 2

- **Code Node**  
  **Name:** `PassesFilter`  
  **⚙ What It Does:**  
  Filters out items where API-Response is empty or null.

- **Edit Node**  
  **Name:** `Select Company Name and Website - 2`  
  **⚙ What It Does:**  
  Rearranges the data; selects Company Name and Website again to prepare for another HTTP API call.

- **Loop Over Items (Split in Batches)**  
  **Name:** `Loop Over Items - 3`  
  **⚙ What It Does:**  
  Iterates over each extracted Company Name and Website to process them individually through the next API call.

- **HTTP Request**  
  **Name:** `STORM API - 2`  
  **⚙ What It Does:**  
  Sends API requests to the **STORM system** to retrieve missing responses.

- **Edit Node**  
  **Name:** `Title/Response/Summary - 2`  
  **⚙ What It Does:**  
  After completing all iterations, this node reorganizes the data into **Title, Response, and API-Response** from the second HTTP loop.

- **Filter Node**  
  **Name:** `Filter`  
  **⚙ What It Does:**  
  Checks that only items with now-filled API-Responses proceed (filters out anything still empty or null).

- **Code Node**  
  **Name:** `Merging entire text before brand profiling result - 2`  
  **⚙ What It Does:**  
  Merges all new API-Response texts together, preparing for final cleaning.

- **Code Node**  
  **Name:** `Cleaning code node before brand profiling_2`  
  **⚙ What It Does:**  
  Cleans the merged text by:
  - Removing all escaped newline characters (`\n` or `\\n`)
  - Removing curly brackets `{` and `}`
  - Removing all double quotes (`"`)

- **Merge Node (Input 2)**  
  **Name:** `Merge`  
  **⚙ What It Does:**  
  After cleaning, the text proceeds to **Merge Node Input 2**.

## 🟢 Final Combination

- **Code Node**  
  **Name:** `Merge Both Clean Text`  
  **⚙ What It Does:**  
  Combines the merged text from **Input 1 and Input 2** into one complete, long text block for downstream processing.

---

## 📤 Part 3: Final Combination & Data Output

### 1️⃣ Code Node: 
**Name:** `Merge Both Clean Text`
**⚙ What It Does:**  
Combines the merged text from **Input 1 and Input 2** into one complete, long text block for downstream processing.

### 2️⃣ AI Agent (Deepseek Model - deepseek/deepseek-r1-distill-qwen-32b): 
**Name:** `Brand Profiling`
**⚙ What It Does:**  
Processes the combined text using a **crafted prompt** with the integrated **DeepSeek R1 Qwine 70B Free Model** to extract all required company parameters: Business, Financials, Operations, Strategy, and More

### 3️⃣ Edit Node: 
**Name:** `Company_Info_Output`
**⚙ What It Does:**  
Passes the result from the AI Agent forward **without making changes**, preparing it for JSON formatting.

### 4️⃣ Code Node: 
**Name:** `Company_Info_Output TO JSON Format`
**⚙ What It Does:**  
Converts the AI Agent’s output into a **well-structured JSON format** to ensure it’s clean and ready for **database insertion**.

### 5️⃣ Postgres Node: 
**Name:** `Insert Data To Company_Details Table`
**⚙ What It Does:**  
Inserts the finalized **structured company profile data** into the `Company_Details` table in **PostgreSQL** for permanent storage.

---

## 🌐 STORM API Endpoint Details

| **Route**      | `/get_article`                                  |
|----------------|-------------------------------------------------|
| **Method**     | `GET`                                           |
| **Test URL**   | `http://localhost:8000/get_article/`            |
| **Content-Type** | `application/json`                             |

### 🔍 What It Does

This API is powered by **FastAPI + Selenium automation** and is used to:

- ✅ Launch **Chrome** (with rotating profiles)
- ✅ Visit the **STORM Stanford platform**  
  ➡️ [https://storm.genie.stanford.edu/](https://storm.genie.stanford.edu/)
- ✅ Automatically fill in the **Title & Response** fields for each company
- ✅ Generate and extract an **article** that acts as a **knowledge base** for the AI agent

The article is used downstream in **n8n** to help the AI Agent (Model) fill in all detailed company information parameters:
Business, Finance, Strategy, History, and More

### 🚨 Note

- **Selenium automation** is used because the STORM platform **does not expose a direct API**, so the automation simulates human browsing to fetch content.
- 📄 The **generated article** acts as an **input document** to feed into the AI agent for complete company profiling downstream.

---

## 🖥️ Running the Service

To start the FastAPI server locally, run the following command:

```
uvicorn routes:app --host 0.0.0.0 --port 8000 --reload
```
###
---
---

Inserts the final **structured JSON** into your Postgres database.

