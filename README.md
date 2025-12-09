<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SMS Sender</title>
    <style>
        :root {
            --primary-color: #007bff;
            --primary-hover-color: #0056b3;
            --background-color: #f8f9fa;
            --card-background: #ffffff;
            --text-color: #343a40;
            --border-color: #ced4da;
            --success-color: #28a745;
            --error-color: #dc3545;
            --shadow-color: rgba(0, 0, 0, 0.08);
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            margin: 0;
            padding: 20px;
            background-color: var(--background-color);
            color: var(--text-color);
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            box-sizing: border-box;
        }

        .container {
            background-color: var(--card-background);
            border-radius: 10px;
            box-shadow: 0 4px 15px var(--shadow-color);
            padding: 30px;
            width: 100%;
            max-width: 500px;
            box-sizing: border-box;
            text-align: center;
        }

        h1 {
            color: var(--primary-color);
            margin-bottom: 25px;
            font-size: 2.2em;
        }

        .form-group {
            margin-bottom: 20px;
            text-align: left;
        }

        label {
            display: block;
            margin-bottom: 8px;
            font-weight: 600;
            color: var(--text-color);
        }

        input[type="text"],
        input[type="url"],
        textarea {
            width: 100%;
            padding: 12px;
            border: 1px solid var(--border-color);
            border-radius: 5px;
            font-size: 1rem;
            box-sizing: border-box;
            transition: border-color 0.3s ease, box-shadow 0.3s ease;
        }

        input[type="text"]:focus,
        input[type="url"]:focus,
        textarea:focus {
            outline: none;
            border-color: var(--primary-color);
            box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
        }

        textarea {
            resize: vertical;
            min-height: 100px;
        }

        button {
            background-color: var(--primary-color);
            color: white;
            padding: 12px 25px;
            border: none;
            border-radius: 5px;
            font-size: 1.1rem;
            cursor: pointer;
            transition: background-color 0.3s ease, transform 0.2s ease;
            width: 100%;
            font-weight: 600;
        }

        button:hover {
            background-color: var(--primary-hover-color);
            transform: translateY(-2px);
        }

        .message-status {
            margin-top: 25px;
            padding: 12px;
            border-radius: 5px;
            display: none; /* Hidden by default */
            font-weight: 500;
            text-align: center;
        }

        .message-status.success {
            background-color: #d4edda;
            color: var(--success-color);
            border: 1px solid #c3e6cb;
        }

        .message-status.error {
            background-color: #f8d7da;
            color: var(--error-color);
            border: 1px solid #f5c6cb;
        }

        @media screen and (max-width: 600px) {
            .container {
                padding: 20px;
                margin: 10px;
            }

            h1 {
                font-size: 1.8em;
            }

            button {
                padding: 10px 20px;
                font-size: 1rem;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📨 SMS Sender</h1>

        <form id="smsForm">
            <div class="form-group">
                <label for="apiUrl">API Endpoint URL:</label>
                <input type="url" id="apiUrl" placeholder="e.g., https://api.your-sms-provider.com/send" required>
            </div>

            <div class="form-group">
                <label for="apiKey">API Key (or Authorization Header Value):</label>
                <input type="text" id="apiKey" placeholder="e.g., Bearer sk-your_api_key_here" required>
            </div>

            <div class="form-group">
                <label for="to">Recipient (e.g., +1234567890):</label>
                <input type="text" id="to" placeholder="+1XXXXXXXXXX" required>
            </div>

            <div class="form-group">
                <label for="message">Message:</label>
                <textarea id="message" maxlength="160" placeholder="Your SMS message here (max 160 chars)" required></textarea>
                <small style="float: right;">Characters: <span id="charCount">0</span>/160</small>
            </div>

            <button type="submit" id="sendButton">Send SMS</button>
        </form>

        <div id="statusMessage" class="message-status"></div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const smsForm = document.getElementById('smsForm');
            const apiUrlInput = document.getElementById('apiUrl');
            const apiKeyInput = document.getElementById('apiKey');
            const toInput = document.getElementById('to');
            const messageInput = document.getElementById('message');
            const sendButton = document.getElementById('sendButton');
            const statusMessageDiv = document.getElementById('statusMessage');
            const charCountSpan = document.getElementById('charCount');

            // Load saved settings from local storage
            apiUrlInput.value = localStorage.getItem('smsApiUrl') || '';
            apiKeyInput.value = localStorage.getItem('smsApiKey') || '';
            toInput.value = localStorage.getItem('smsRecipient') || '';

            messageInput.addEventListener('input', () => {
                const currentLength = messageInput.value.length;
                charCountSpan.textContent = currentLength;
            });

            smsForm.addEventListener('submit', async (e) => {
                e.preventDefault();

                // Save current settings to local storage
                localStorage.setItem('smsApiUrl', apiUrlInput.value);
                localStorage.setItem('smsApiKey', apiKeyInput.value);
                localStorage.setItem('smsRecipient', toInput.value);

                const apiUrl = apiUrlInput.value.trim();
                const apiKey = apiKeyInput.value.trim();
                const to = toInput.value.trim();
                const message = messageInput.value.trim();

                if (!apiUrl || !apiKey || !to || !message) {
                    showStatus('All fields are required!', 'error');
                    return;
                }

                sendButton.disabled = true;
                sendButton.textContent = 'Sending...';
                statusMessageDiv.style.display = 'none';

                try {
                    const response = await fetch(apiUrl, {
                        method: 'POST',
                        headers: {
                            'Content-Type': 'application/json',
                            // Assuming API Key is sent as an Authorization header.
                            // This might need adjustment based on your specific SMS provider's API.
                            // e.g., 'Authorization': Bearer ${apiKey} or 'X-API-Key': apiKey
                            'Authorization': apiKey.startsWith('Bearer ') ? apiKey : Bearer ${apiKey}
                            // For other providers, it might be:
                            // 'X-API-Key': apiKey,
                            // 'api_key': apiKey,
                        },
                        body: JSON.stringify({
                            to: to,
                            message: message,
                            // Add other necessary parameters for your API, e.g., 'from', 'sender_id', etc.
                            // from: 'YourSenderID',
                            // channel: 'sms'
                        }),
                    });

                    const responseData = await response.json();

                    if (response.ok) {
                        showStatus('SMS sent successfully! Response: ' + JSON.stringify(responseData), 'success');
                        // Optionally clear the message after success
                        messageInput.value = '';
                        charCountSpan.textContent = '0';
                    } else {
                        showStatus(Error sending SMS: ${response.status} - ${responseData.message || JSON.stringify(responseData)}, 'error');
                    }

                } catch (error) {
                    console.error('Network or API Error:', error);
                    showStatus('Failed to connect to API or an unexpected error occurred. Check console for details.', 'error');
                } finally {
                    sendButton.disabled = false;
                    sendButton.textContent = 'Send SMS';
                }
            });

            function showStatus(message, type) {
                statusMessageDiv.textContent = message;
                statusMessageDiv.className = message-status ${type};
                statusMessageDiv.style.display = 'block';
            }
        });
    </script>
</body>
</html>