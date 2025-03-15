
## Using the Gemini API with Python and HTML

### 1. Gemini API Key

**Get an API Key:**
- You need a Google Cloud project and the Gemini API enabled.
- Go to the [Google AI Studio](https://makersuite.google.com/app/apikey) or the Google Cloud Console.
- Follow the instructions to create a project, enable the Gemini API, and generate an API key.
  
**Important:** Keep this key secure! Don't hardcode it directly into your HTML, as it's publicly visible.

### 2. Python (Backend - Flask/FastAPI)

#### Install Libraries
You'll need the `google-generativeai` library and a web framework like Flask or FastAPI to create an API endpoint.

```bash
pip install google-generativeai flask
```
*(or `pip install google-generativeai fastapi uvicorn` for FastAPI)*

#### Flask Example (`app.py`)
```python
from flask import Flask, request, jsonify
import google.generativeai as genai
import os

# Import the 'os' module
app = Flask(__name__)

# Load API key from environment variable
GOOGLE_API_KEY = os.environ.get("GOOGLE_API_KEY")

# Retrieve key from environment
if not GOOGLE_API_KEY:
    raise ValueError("No GOOGLE_API_KEY environment variable set!")

genai.configure(api_key=GOOGLE_API_KEY)
model = genai.GenerativeModel('gemini-pro')

@app.route('/generate', methods=['POST'])
def generate_text():
    data = request.get_json()
    prompt = data.get('prompt')
    
    if not prompt:
        return jsonify({'error': 'No prompt provided'}), 400
    
    try:
        response = model.generate_content(prompt)
        return jsonify({'text': response.text})
    except Exception as e:
        return jsonify({'error': str(e)}), 500

if __name__ == '__main__':
    app.run(debug=True)
```

#### FastAPI Example (`main.py`)
```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import google.generativeai as genai
import os

app = FastAPI()

# Load API key from environment variable
GOOGLE_API_KEY = os.environ.get("GOOGLE_API_KEY")

if not GOOGLE_API_KEY:
    raise ValueError("No GOOGLE_API_KEY environment variable set!")

genai.configure(api_key=GOOGLE_API_KEY)
model = genai.GenerativeModel('gemini-pro')

class Prompt(BaseModel):
    prompt: str

@app.post("/generate")
def generate_text(prompt: Prompt):
    try:
        response = model.generate_content(prompt.prompt)
        return {'text': response.text}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### 3. HTML (User Interface)

You can create a simple HTML form to interact with the API.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Gemini API Interface</title>
</head>
<body>
    <h1>Generate Text with Gemini API</h1>
    <form id="textForm">
        <label for="prompt">Enter your prompt:</label>
        <input type="text" id="prompt" name="prompt" required>
        <button type="submit">Generate</button>
    </form>
    <h2>Generated Text:</h2>
    <pre id="result"></pre>
    
    <script>
        document.getElementById('textForm').addEventListener('submit', async function(event) {
            event.preventDefault();
            const prompt = document.getElementById('prompt').value;

            const response = await fetch('/generate', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json'
                },
                body: JSON.stringify({ prompt })
            });

            const data = await response.json();
            document.getElementById('result').textContent = data.text || data.error;
        });
    </script>
</body>
</html>
```

### Summary
- **API Key**: Securely manage your Gemini API key.
- **Backend**: Use Flask or FastAPI to create an API endpoint that interacts with the Gemini API.
- **Frontend**: Build a simple HTML interface for user interaction.