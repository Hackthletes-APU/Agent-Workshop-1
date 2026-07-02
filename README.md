# APU Hackthletes - AI Agent Workshop Flow


## Checkpoint 1: Install n8n

Choose **one** of the two methods below.

### Method 1: Using Node.js / npm

**Step 1 — Install Node.js**
n8n requires Node.js to run.
- Go to the [official Node.js website](https://nodejs.org/)

**Step 2 — Verify the installation**
Open your Command Prompt (press `Win + R`, type `cmd`, hit Enter) and run:

```bash
node -v
npm -v
```

Both commands should return a version number.

**Step 3 — Install and run n8n**
In your Command Prompt, run:

```bash
npx n8n start
```

This downloads and starts n8n instantly using `npx`, avoiding global installation clutter.

**Step 4 — Access n8n**
Once the terminal shows `n8n ready`, open your browser and go to:

```
http://localhost:5678
```

> ⚠️ Keep the Command Prompt window open while using n8n. Closing it stops the server.

---

### Method 2: Using Docker (Recommended)

**Step 1 — Create a folder for n8n**
- Open File Explorer / Finder
- Create a new folder anywhere (e.g., `C:\n8n`)

**Step 2 — Create the `docker-compose.yml` file**
- Inside the folder, create a new text file
- Name it exactly `docker-compose.yml` (change the extension from `.txt` to `.yml`)
- Open it with Notepad / Code Editor and paste:

```yaml
version: '3.8'
volumes:
  n8n_storage:
    name: n8n_storage
services:
  n8n:
    image: docker.n8n.io/n8nio/n8n:latest
    container_name: n8n_local
    restart: unless-stopped
    ports:
      - "5678:5678"
    volumes:
      - n8n_storage:/home/node/.n8n
    environment:
      - N8N_SECURE_COOKIE=false
```

**Step 3 — Run Docker Compose**
Open Command Prompt or PowerShell and navigate to your folder:

```
cd C:\n8n
```

Then run:

```
docker compose up -d
```

The `-d` flag runs it in detached (background) mode, so you can safely close your terminal window.

**Step 4 — Access your setup**
Open your browser and go to:

```
http://localhost:5678
```

---

## Ready-To-Use N8N Instance

Use the shared workshop link to join a hosted n8n instance for participants:

🔗 https://apuhackthletes-ai-workshop.vercel.app/

---

## Checkpoint 2: Build Your N8N Flow

- Create your own flow using your **own personal username**
- Do **not** interrupt other participants' sessions

### Ingestion Pipeline (high-level structure)
- Allow upload of past documents, handling different file types
- Pass documents to an LLM model for analysis
- Save extracted information into a table

Parse Uploaded Files into Array

```
// Get the array of uploaded files metadata from the JSON body
const filesMetadata = $input.item.json["Upload Files Here"] || [];
const binaryData = $input.item.binary || {};

// Map each file into its own standalone n8n item
return filesMetadata.map((file, index) => {
  
  // n8n typically names multi-upload binary keys sequentially (e.g., "Upload Files Here_0", "Upload Files Here_1")
  const binaryKey = `Upload Files Here_${index}`;
  
  return {
    json: {
      filename: file.filename,
      mimetype: file.mimetype,
    },
    binary: {
      // Pass the specific file binary data down as 'data' so Extract File nodes can read it automatically
      data: binaryData[binaryKey] || Object.values(binaryData)[index]
    }
  };
});
```

Writing Styles Analysis LLM System Message
```
You are an expert assignment ghostwriter. Your goal is to write the requested paragraph while perfectly replicating a specific student's imperfect human writing fingerprint.

CRITICAL MECHANICAL CONSTRAINTS (Follow strictly to match the user's statistics):
1. SENTENCE BURSTINESS: Your sentences must heavily fluctuate. Write a long complex sentence (25-30 words) matching this structure: "{{ $json.typological_sentence_blueprints.complex_argument_example }}", followed immediately by a short, punchy sentence (under 10 words).
2. TRANSITIONS & VOCABULARY: Only use these transitional words: {{ $json.lexicon_habits.preferred_transition_words.join(', ') }}.
3. STRICTLY FORBIDDEN PHRASES: If you use any of these phrases, the generation fails: {{ $json.lexicon_habits.banned_or_avoided_phrases.join(', ') }}.
4. HUMAN ECHOING: Humans naturally repeat certain core technical verbs. Notice how the user repeats words like "enable" frequently. Do not try to make the vocabulary perfectly diverse or polished. Match the third-person neutral bias defined here: 
CRITICAL ANALYSIS CONSTRAINTS:
1. For 'structural_breakdown', do NOT use standard grammar terms like "Independent + subordinate clause". Instead, describe the literal blueprint mechanism (e.g., "The author list-dumps 4 nouns separated by commas mid-sentence, then loops back to the main corporate subject with a relative pronoun clause").
2. For 'argumentation_flow', explain exactly how they use their context (e.g., "The author routinely couples a technical framework name like Tier III directly with a corporate entity name like Sime Darby to force an immediate business justification").
3. Make sure the 'exact_source_sentence' catches actual human flaws, such as repeating the word 'suitability' or stacking three prepositional phrases ('of the enterprise' + 'at a cost' + 'without being').

Assignment Topic to write about:
{{ $json.data }} {{ $json.text }}
```


Output Parser
```
{
  "sentence_mechanics": {
    "average_word_count_per_sentence": "15-20 words",
    "structural_footprint": "Detailed description of clause ordering (e.g., 'Tends to place subordinate clauses at the beginning of sentences before the main predicate').",
    "punctuation_quirks": "Detail the exact physical placement of symbols (e.g., 'Frequently uses pairs of commas to isolate appositives or mid-sentence real-world examples')."
  },
  "verbatim_punctuation_and_syntax_blueprints": [
    {
      "pattern_type": "The grammatical structure being highlighted (e.g., 'Mid-sentence parenthetical example', 'Semicolon transition', 'Comma-heavy descriptive string')",
      "exact_source_sentence": "Paste an exact, verbatim sentence from the text that perfectly exhibits this punctuation and syntax cadence.",
      "structural_breakdown": "A mechanical token breakdown of why this sentence fits the pattern (e.g., 'Noun + comma + transition phrase + comma + active verb + semicolon + conditional clause')."
    },
    {
      "pattern_type": "Another structural footprint pattern",
      "exact_source_sentence": "Another exact, verbatim sentence showcasing a different signature punctuation habit.",
      "structural_breakdown": "Mechanical token breakdown."
    }
  ],
  "paragraph_cadence": {
    "average_paragraph_length": "5-7 sentences",
    "paragraph_opening_habit": "Specify structural preferences for paragraph openings.",
    "argumentation_flow": "Explain how they layout facts sequentially."
  },
  "tonal_fingerprint": {
    "vocabulary_sophistication": "Describe specific word choices.",
    "assertiveness_scale": "Does the author use authoritative verbs or cautious academic hedging?",
    "emotional_distance": "Identify stylistic traits matching their level of detachment."
  },
  "idiosyncratic_habits": {
    "linguistic_crutches": [
      "enable",
      "support"
    ],
    "cliché_or_filler_avoidance": "Note specific elements the author completely strips out."
  }
}
```

> 

---

## Checkpoint 3: Assignment Agent Flow

Assignment Agent System Message

```
You are an expert academic research assistant and professional writer. Your goal is to rewrite the user's input text to match a specific student's structural metrics while ensuring the output remains grammatically correct, highly professional, and perfectly readable.

CRITICAL ARCHITECTURAL RULES:
1. PARAGRAPH MATCHING: You MUST output the exact same number of paragraphs as the user provided. If the user inputs 2 paragraphs, you must return exactly 2 rewritten paragraphs. 
2. CONDITIONAL RESEARCH: Only use the Tavily tool if the user explicitly asks for external facts/references, or if the student profile metrics require adding missing external citations. If the input already contains sufficient context, focus purely on rewriting and refining.
3. BIBLIOGRAPHY GENERATION: If the Tavily tool is utilized during this execution, you MUST compile a dedicated "References" section at the very bottom of your final response. 

OPERATIONAL EXECUTION ORDER:
1. FIRST: Immediately use the 'Get row(s) in Data table' tool to fetch the student's metrics.
2. SECOND: Analyze the input text. If external research is required, use Tavily with a short keyword query (<50 chars). If not, proceed to drafting.
3. THIRD: Rewrite the content paragraph-by-paragraph using the metrics retrieved from the Data Table tool.

STRICT MECHANICAL FILTERS (Apply per paragraph using the JSON values returned from the Data Table tool):
- CADENCE: Read the retrieved JSON object. Match the target sentence count per paragraph specified in `paragraph_cadence.average_paragraph_length` and hit the sentence length target specified in `sentence_mechanics.average_word_count_per_sentence`.
- OPENING HABIT: Explicitly start the first sentence of the rewritten block using the exact transitional phrase or style found in `paragraph_cadence.paragraph_opening_habit`.
- READABLE COMPLEXITY: Mirror the punctuation habits found in `sentence_mechanics.punctuation_quirks` by using compound sentences smoothly (e.g., proper semicolons or coordinating conjunctions). Do NOT create unreadable, infinite run-on sentences using repetitive commas like "and, moreover, because, firstly". Every sentence must be grammatically sound.
- LINGUISTIC FOCUS: Frame the arguments around how these technical shifts "enable and support business needs" and operational impact.
- CITATIONS & REFERENCE LIST: 
  * Inside the rewritten paragraphs: Format citations strictly as APA inline tags (e.g., Author, Year).
  * At the very end of the output (only if Tavily was called): Create a clear `### References` markdown heading. List the full APA 7th edition reference format for each source used, and append the direct citation URL at the end of each reference entry so it is easy to copy and paste.

Input text to rewrite:
{{ $json.message.text }}
```



1. **Register a Groq account** and get your API key
   🔗 https://console.groq.com/keys

2. **Register a Tavily account** and get your API key
   🔗 https://app.tavily.com/home

3. **Register a Google AI Studio account** and get your API key
🔗 https://aistudio.google.com/api-keys

4. **Enable the Google Docs API**
   🔗 https://console.cloud.google.com/marketplace/product/google/docs.googleapis.com

5. **Create OAuth credentials**
   Create an OAuth Client ID and Secret so you can link your Google Doc to your n8n flow.

6. **Download Community Nodes to Convert Docx to Text in N8N**
   n8n-nodes-docx-converter

---
