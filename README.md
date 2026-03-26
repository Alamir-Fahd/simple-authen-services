# Simple Auth Service (Keycloak + RabbitMQ + Node API)

This project demonstrates a secure, asynchronous logging architecture. It uses Keycloak as an Identity Provider (IdP) to issue OpenID Connect (OIDC) tokens, a Node.js Express API to validate those tokens, and RabbitMQ to queue the validated messages.

## 🏗️ Architecture
* **Auth:** Keycloak (Running on port `8080`)
* **Queue:** RabbitMQ (Running on ports `5672` and `15672`)
* **API:** Node.js / Express (Running on port `3000`)

---

## 🚀 Quick Start Guide

#Step 1: Boot the Infrastructure
-Make sure you are in the project root folder, then start the containers.

##Step 2: Keycloak Dashboard Setup
-Once the containers are healthy, open http://localhost:8080 in your browser and log in with admin / admin.
-Create the Realm: Name it exactly log (This must match KEYCLOAK_REALM in the docker-compose.yml).
-Create the Client: Name it log-verify.
-Go to the Settings tab.
-Toggle Client authentication to ON.
-Click Save.
-Go to the Credentials tab and copy the Client Secret.
-Create the Test User: Go to Users and create a user named bob.
-Go to Bob's Credentials tab, set the password to bobby12345, and toggle Temporary to OFF.
-Go to Bob's Details tab, toggle Email verified to ON, and hit Save.

Step 3: Run the Automated Integration Test
-Open a new terminal tab (do not close the one running Docker).
-Paste your Client Secret from Step 2 into the client_secret field in the script below. This script will automatically hit Keycloak, extract the pure token (avoiding invisible newline characters), and immediately POST a secure log to your API.

# Replace YOUR_COPIED_SECRET with the secret from the log-verify client
TOKEN=$(curl -s -X POST http://localhost:8080/realms/log/protocol/openid-connect/token \
-d "client_id=log-verify" \
-d "client_secret=YOUR_COPIED_SECRET" \
-d "username=bob" \
-d "password=bobby12345" \
-d "grant_type=password" | python3 -c "import sys, json; print(json.load(sys.stdin)['access_token'])")

# Post the message using the token
curl -X POST http://localhost:3000/logs \
-H "Content-Type: application/json" \
-H "Authorization: Bearer $TOKEN" \
-d '{"level": "info", "message": "Hello RabbitMQ, Bob is officially back!"}'


Expected Output:
If successful, you will receive a 201 Created response looking like this:
{"message":"Log published successfully","log":{"id":"...","userId":"...","level":"info","message":"Hello RabbitMQ, Bob is officially back!","timestamp":"..."}}
