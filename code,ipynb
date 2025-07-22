!pip install flask google-genai
!npm install -g localtunnel
from flask import Flask, render_template, request, jsonify
import google.generativeai as genai
import os
import threading

# Set your Google API key here
GOOGLE_API_KEY = ''
genai.configure(api_key=GOOGLE_API_KEY)
model = genai.GenerativeModel("gemini-2.5-flash")
chat = model.start_chat(history=[])

app = Flask(__name__)

# Make sure 'templates' folder exists and save the HTML file
os.makedirs('templates', exist_ok=True)

html_code = """
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>GenAI Chat</title>
  <style>
    /* Reset and base */
    * {
      box-sizing: border-box;
    }
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #667eea, #764ba2);
      margin: 0;
      height: 100vh;
      display: flex;
      justify-content: center;
      align-items: center;
      color: #333;
    }

    .chat-container {
      width: 100%;
      max-width: 600px;
      height: 90vh;
      background: #ffffffee;
      border-radius: 15px;
      box-shadow: 0 8px 24px rgba(0,0,0,0.15);
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    h1 {
      margin: 0;
      padding: 20px;
      background: #6c63ff;
      color: white;
      font-weight: 700;
      text-align: center;
      letter-spacing: 1.2px;
      user-select: none;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }

    #chatbox {
      flex: 1;
      padding: 20px;
      overflow-y: auto;
      background: #f9f9fb;
      font-size: 15px;
      line-height: 1.5;
      display: flex;
      flex-direction: column;
      gap: 12px;
      scroll-behavior: smooth;
    }

    .message {
      max-width: 75%;
      padding: 12px 18px;
      border-radius: 20px;
      position: relative;
      word-wrap: break-word;
      box-shadow: 0 2px 6px rgba(0,0,0,0.1);
      animation: fadeInUp 0.3s ease forwards;
    }

    @keyframes fadeInUp {
      from {
        opacity: 0;
        transform: translateY(10px);
      }
      to {
        opacity: 1;
        transform: translateY(0);
      }
    }

    .user {
      background-color: #6c63ff;
      color: white;
      align-self: flex-end;
      border-bottom-right-radius: 0;
    }

    .bot {
      background-color: #e0e0ff;
      color: #3a3a72;
      align-self: flex-start;
      border-bottom-left-radius: 0;
    }

    .message span {
      font-weight: 700;
      margin-right: 8px;
      user-select: none;
    }

    .input-area {
      display: flex;
      padding: 15px 20px;
      border-top: 1px solid #ddd;
      background: white;
    }

    #message {
      flex-grow: 1;
      border: 1px solid #ccc;
      border-radius: 25px;
      padding: 12px 20px;
      font-size: 16px;
      outline: none;
      transition: border-color 0.2s ease;
    }

    #message:focus {
      border-color: #6c63ff;
      box-shadow: 0 0 8px rgba(108, 99, 255, 0.5);
    }

    button {
      margin-left: 15px;
      background-color: #6c63ff;
      border: none;
      color: white;
      padding: 0 25px;
      border-radius: 25px;
      font-weight: 600;
      font-size: 16px;
      cursor: pointer;
      box-shadow: 0 5px 15px rgba(108, 99, 255, 0.4);
      transition: background-color 0.3s ease, box-shadow 0.3s ease;
    }

    button:hover:not(:disabled) {
      background-color: #584fc6;
      box-shadow: 0 8px 20px rgba(88, 79, 198, 0.6);
    }

    button:disabled {
      background-color: #aaa;
      cursor: not-allowed;
      box-shadow: none;
    }
  </style>

  <script>
    async function sendMessage() {
      const input = document.getElementById('message');
      const message = input.value.trim();
      if (!message) return;

      const chatbox = document.getElementById('chatbox');

      // Show user message
      const userMsgDiv = document.createElement('div');
      userMsgDiv.classList.add('message', 'user');
      userMsgDiv.innerHTML = '<span>You:</span> ' + escapeHtml(message);
      chatbox.appendChild(userMsgDiv);
      chatbox.scrollTop = chatbox.scrollHeight;

      // Clear input and disable button to avoid spam
      input.value = '';
      const sendBtn = document.querySelector('button');
      sendBtn.disabled = true;

      try {
        const response = await fetch('/chat', {
          method: 'POST',
          headers: {'Content-Type': 'application/json'},
          body: JSON.stringify({message})
        });
        const data = await response.json();

        // Show bot response or error
        const botMsgDiv = document.createElement('div');
        botMsgDiv.classList.add('message', 'bot');
        if (data.response) {
          botMsgDiv.innerHTML = '<span>Bot:</span> ' + escapeHtml(data.response);
        } else {
          botMsgDiv.innerHTML = '<span>Error:</span> ' + escapeHtml(data.error || 'Unknown error');
        }
        chatbox.appendChild(botMsgDiv);
        chatbox.scrollTop = chatbox.scrollHeight;

      } catch(err) {
        const botMsgDiv = document.createElement('div');
        botMsgDiv.classList.add('message', 'bot');
        botMsgDiv.innerHTML = '<span>Error:</span> Network error';
        chatbox.appendChild(botMsgDiv);
        chatbox.scrollTop = chatbox.scrollHeight;
      }

      sendBtn.disabled = false;
      input.focus();
    }

    // Escape HTML to prevent injection
    function escapeHtml(text) {
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }

    // Allow sending message by pressing Enter
    document.addEventListener('DOMContentLoaded', () => {
      const input = document.getElementById('message');
      input.addEventListener('keydown', e => {
        if (e.key === 'Enter' && !e.shiftKey) {
          e.preventDefault();
          document.querySelector('button').click();
        }
      });
    });
  </script>
</head>
<body>
  <div class="chat-container" role="main" aria-label="Chat container">
    <h1>GenAI Chat</h1>
    <div id="chatbox" aria-live="polite" aria-relevant="additions"></div>
    <div class="input-area">
      <textarea id="message" rows="1" placeholder="Type your message..." aria-label="Message input"></textarea>
      <button aria-label="Send message" onclick="sendMessage()">Send</button>
    </div>
  </div>
</body>
</html>
"""

with open('templates/index.html', 'w') as f:
    f.write(html_code)

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/chat', methods=['POST'])
def chat_response():
    user_input = request.json.get('message')
    if not user_input:
        return jsonify({"error": "No message provided"}), 400
    try:
        response = chat.send_message(user_input).text
        return jsonify({"response": response})
    except Exception as e:
        print(f"Error: {e}")
        return jsonify({"error": str(e)}), 500

def run_app():
    app.run(host='0.0.0.0', port=5000)

# Run Flask app in a background thread so Colab stays interactive
threading.Thread(target=run_app).start()
lt --port 5000
!curl https://loca.lt/mytunnelpassword
