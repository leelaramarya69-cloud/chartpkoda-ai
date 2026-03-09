<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Universal AI Agent (OpenRouter)</title>
    <style>
        :root {
            --bg-color: #050505;
            --sidebar-color: #171717;
            --text-color: #ececec;
            --accent-color: #10a37f; /* ChatGPT Green */
            --border-color: #333;
        }

        .light-theme {
            --bg-color: #ffffff;
            --sidebar-color: #f9f9f9;
            --text-color: #212121;
            --border-color: #e5e5e5;
        }

        body, html {
            margin: 0; padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: var(--bg-color);
            color: var(--text-color);
            height: 100%;
            overflow: hidden;
        }

        /* Layout Structure */
        .container { display: flex; height: 100vh; }

        /* Sidebar */
        .sidebar {
            width: 260px;
            background-color: var(--sidebar-color);
            border-right: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
            padding: 15px;
        }

        .new-chat-btn {
            border: 1px solid var(--border-color);
            padding: 10px;
            border-radius: 5px;
            text-align: center;
            cursor: pointer;
            transition: 0.3s;
        }

        .new-chat-btn:hover { background: #2b2b2b; }

        /* Main Chat Area */
        .main-chat {
            flex-grow: 1;
            display: flex;
            flex-direction: column;
            position: relative;
        }

        header {
            padding: 15px;
            border-bottom: 1px solid var(--border-color);
            display: flex;
            justify-content: space-between;
            align-items: center;
        }

        #chat-window {
            flex-grow: 1;
            overflow-y: auto;
            padding: 20px;
            display: flex;
            flex-direction: column;
            gap: 20px;
        }

        .message { max-width: 80%; padding: 12px; border-radius: 10px; line-height: 1.5; }
        .user-msg { align-self: flex-end; background: var(--accent-color); color: white; }
        .ai-msg { align-self: flex-start; background: #2f2f2f; border: 1px solid var(--border-color); }

        /* Input Area */
        .input-area {
            padding: 20px;
            display: flex;
            gap: 10px;
            border-top: 1px solid var(--border-color);
        }

        input[type="text"] {
            flex-grow: 1;
            background: #212121;
            border: 1px solid var(--border-color);
            color: white;
            padding: 12px;
            border-radius: 8px;
            outline: none;
        }

        /* Settings Modal */
        #settings-overlay {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            background: var(--bg-color);
            display: none;
            padding: 40px;
            z-index: 100;
        }

        .settings-content { max-width: 600px; margin: auto; }
        .setting-item { margin-bottom: 20px; }
        label { display: block; margin-bottom: 8px; font-weight: bold; }
        
        button.primary-btn {
            background: var(--accent-color);
            color: white; border: none;
            padding: 10px 20px; border-radius: 5px; cursor: pointer;
        }

        .back-btn { background: #444; color: white; border: none; padding: 8px 15px; border-radius: 5px; cursor: pointer; margin-bottom: 20px; }
    </style>
</head>
<body>

<div class="container">
    <div class="sidebar">
        <div class="new-chat-btn">+ New Chat</div>
        <div style="margin-top: auto;">
            <div onclick="openSettings()" style="cursor:pointer; padding: 10px;">⚙️ Settings</div>
        </div>
    </div>

    <div class="main-chat">
        <header>
            <div id="current-model-display">Model: Loading...</div>
            <button onclick="toggleTheme()" style="background:none; border:none; cursor:pointer; font-size: 20px;">🌗</button>
        </header>

        <div id="chat-window">
            <div class="message ai-msg">Namaste! Main aapka custom AI Agent hoon. Settings mein jaakar OpenRouter API key set karein.</div>
        </div>

        <div class="input-area">
            <input type="text" id="user-input" placeholder="Ask me anything (DeepSeek, Grok, GPT...)" />
            <button class="primary-btn" onclick="sendMessage()">Send</button>
        </div>

        <div id="settings-overlay">
            <button class="back-btn" onclick="closeSettings()">← Back</button>
            <div class="settings-content">
                <h2>AI Agent Settings</h2>
                <div class="setting-item">
                    <label>OpenRouter API Key</label>
                    <input type="password" id="api-key" style="width: 100%; padding: 10px; border-radius: 5px;" placeholder="sk-or-v1-...">
                </div>
                <div class="setting-item">
                    <label>Model ID (DeepSeek, Grok, or GPT)</label>
                    <input type="text" id="model-id" style="width: 100%; padding: 10px; border-radius: 5px;" placeholder="deepseek/deepseek-chat">
                    <small>Example: `google/gemini-2.0-flash-001` or `x-ai/grok-2`</small>
                </div>
                <button class="primary-btn" onclick="saveSettings()">Save Configuration</button>
            </div>
        </div>
    </div>
</div>

<script>
    // Load Settings
    let apiKey = localStorage.getItem('ai_api_key') || '';
    let modelId = localStorage.getItem('ai_model_id') || 'google/gemini-2.0-flash-001';
    
    document.getElementById('api-key').value = apiKey;
    document.getElementById('model-id').value = modelId;
    document.getElementById('current-model-display').innerText = "Model: " + modelId;

    function openSettings() { document.getElementById('settings-overlay').style.display = 'block'; }
    function closeSettings() { document.getElementById('settings-overlay').style.display = 'none'; }

    function saveSettings() {
        apiKey = document.getElementById('api-key').value;
        modelId = document.getElementById('model-id').value;
        localStorage.setItem('ai_api_key', apiKey);
        localStorage.setItem('ai_model_id', modelId);
        document.getElementById('current-model-display').innerText = "Model: " + modelId;
        alert("Settings Saved!");
        closeSettings();
    }

    function toggleTheme() {
        document.body.classList.toggle('light-theme');
    }

    async function sendMessage() {
        const inputField = document.getElementById('user-input');
        const chatWindow = document.getElementById('chat-window');
        const text = inputField.value.trim();

        if (!text || !apiKey) {
            alert("Please enter text and ensure API Key is set in Settings.");
            return;
        }

        // Add User Message
        chatWindow.innerHTML += `<div class="message user-msg">${text}</div>`;
        inputField.value = '';

        // API Call to OpenRouter
        try {
            const response = await fetch("https://openrouter.ai/api/v1/chat/completions", {
                method: "POST",
                headers: {
                    "Authorization": `Bearer ${apiKey}`,
                    "Content-Type": "application/json"
                },
                body: JSON.stringify({
                    "model": modelId,
                    "messages": [{"role": "user", "content": text}]
                })
            });

            const data = await response.json();
            const aiResponse = data.choices[0].message.content;

            // Add AI Message
            chatWindow.innerHTML += `<div class="message ai-msg">${aiResponse}</div>`;
            chatWindow.scrollTop = chatWindow.scrollHeight;
        } catch (error) {
            chatWindow.innerHTML += `<div class="message ai-msg" style="color:red;">Error: API key galat hai ya network issue hai.</div>`;
        }
    }
</script>

</body>
</html>
