# Iot-based-file-management-system

HARDWERE CODE(RFAID)

#include <SPI.h>
#include <MFRC522.h>

#define SS_PIN 5
#define RST_PIN 22

MFRC522 rfid(SS_PIN, RST_PIN);
MFRC522::MIFARE_Key key;

void setup() {
    Serial.begin(115200);
    SPI.begin();  
    rfid.PCD_Init();  
    Serial.println("Scan RFID Tag...");
}

void loop() {
    // Look for new cards
    if (!rfid.PICC_IsNewCardPresent()) {
        return;
    }

    if (!rfid.PICC_ReadCardSerial()) {
        return;
    }

    Serial.print("UID Tag: ");
    String tagID = "";

    for (byte i = 0; i < rfid.uid.size; i++) {
        Serial.print(rfid.uid.uidByte[i] < 0x10 ? "0" : "");
        Serial.print(rfid.uid.uidByte[i], HEX);
        tagID += String(rfid.uid.uidByte[i], HEX);
    }
    Serial.println();
    
        rfid.PICC_HaltA();
}


FRONTEND CODE(for the website)

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>IoT File Management System</title>
    <style>
        body, html {
            margin: 0;
            padding: 0;
            width: 100%;
            height: 100%;
            font-family: Arial, sans-serif;
            background: linear-gradient(45deg, #ff5733, #ffbd69, #1e3c72, #2a5298, #6a0572);
            background-size: 400% 400%;
            animation: gradientAnimation 10s infinite alternate ease-in-out;
            color: white;
            overflow: hidden;
        }
        @keyframes gradientAnimation {
            0% { background-position: 0% 50%; }
            50% { background-position: 100% 50%; }
            100% { background-position: 0% 50%; }
        }
        .home-container, .main-container, .records-container {
            position: absolute;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: center;
            align-items: center;
            flex-direction: column;
            transition: opacity 1s ease-in-out;
            text-align: center;
        }
        .enter-button, .records-button, .home-button, .back-button, .navigate-button {
            background: #ff5733;
            border: none;
            padding: 15px 30px;
            border-radius: 30px;
            color: white;
            font-size: 18px;
            cursor: pointer;
            margin: 10px;
        }
        .hidden {
            opacity: 0;
            pointer-events: none;
        }
        .project-details {
            max-width: 600px;
            background: rgba(0, 0, 0, 0.6);
            padding: 20px;
            border-radius: 10px;
            margin-top: 20px;
            position: relative;
        }
        .floating-text {
            position: absolute;
            width: 100%;
            top: 50px;
            left: 50%;
            transform: translateX(-50%);
            font-size: 18px;
            color: white;
            animation: floatText 5s infinite alternate ease-in-out;
        }
        @keyframes floatText {
            0% { transform: translateX(-50%) translateY(0); }
            100% { transform: translateX(-50%) translateY(-20px); }
        }
        .bottom-buttons {
            position: absolute;
            bottom: 20px;
            display: flex;
            gap: 20px;
        }
    </style>
</head>
<body>
    <div class="home-container" id="homePage">
        <h1>Welcome to IoT File Management System</h1>
        <button class="enter-button" onclick="enterMainPage()">Enter to Main Page</button>
    </div>
    
    <div class="main-container hidden" id="mainPage">
        <h1>Main Page</h1>
        <div class="project-details">
            <h2>About the Project</h2>
            <p>This IoT-based file management system enables users to store, manage, and edit files efficiently. 
               It allows seamless data handling, with real-time modifications and record tracking. 
               The system ensures accuracy by notifying users when incorrect input is detected.</p>
        </div>
        <div class="floating-text">
            Imagine a world where your files manage themselves—where your documents, images, and data sync effortlessly across all your IoT-connected devices, accessible anytime, anywhere. That’s exactly what an <b>IoT-based file management system website</b> delivers! 🚀 With <b>real-time synchronization, smart tagging, and AI-powered organization</b>, this platform takes the hassle out of file storage, making it faster and smarter. Whether you're working remotely, running a smart home, or managing industrial data, the system ensures seamless access with <b>cloud integration</b>.
        </div>
        <div class="bottom-buttons">
            <button class="home-button" onclick="goHome()">Home</button>
            <button class="records-button" onclick="goToRecords()">Go to Records</button>
        </div>
    </div>

    <div class="records-container hidden" id="recordsPage">
        <h1>Records</h1>
        <p id="noRecords">No existing records found.</p>
        <div id="recordsList" class="hidden">
            <p>Here are your available records:</p>
            <button class="navigate-button" onclick="navigateRecords()">Navigate Records</button>
        </div>
        <div class="bottom-buttons">
            <button class="back-button" onclick="goBack()">Back</button>
        </div>
    </div>
    
    <script>
        function enterMainPage() {
            document.getElementById("homePage").classList.add("hidden");
            setTimeout(() => {
                document.getElementById("mainPage").classList.remove("hidden");
            }, 500);
        }
        function goHome() {
            document.getElementById("mainPage").classList.add("hidden");
            setTimeout(() => {
                document.getElementById("homePage").classList.remove("hidden");
            }, 500);
        }
        function goToRecords() {
            document.getElementById("mainPage").classList.add("hidden");
            setTimeout(() => {
                document.getElementById("recordsPage").classList.remove("hidden");
                checkRecords();
            }, 500);
        }
        function goBack() {
            document.getElementById("recordsPage").classList.add("hidden");
            setTimeout(() => {
                document.getElementById("mainPage").classList.remove("hidden");
            }, 500);
        }
        function checkRecords() {
            let hasRecords = true; // Change this based on actual data availability
            if (hasRecords) {
                document.getElementById("noRecords").classList.add("hidden");
                document.getElementById("recordsList").classList.remove("hidden");
            } else {
                document.getElementById("noRecords").classList.remove("hidden");
                document.getElementById("recordsList").classList.add("hidden");
            }
        }
        function navigateRecords() {
            alert("Navigating through records...");
        }
    </script>
</body>
</html>




BACKEND CODE


from flask import Flask, request, jsonify
import pandas as pd
import os

app = Flask(__name__)

# Define file path for storing records
FILE_PATH = "records.xlsx"

# Initialize or load existing data
if not os.path.exists(FILE_PATH):
    df = pd.DataFrame(columns=["RFID_Tag", "Timestamp"])
    df.to_excel(FILE_PATH, index=False)

@app.route('/upload', methods=['POST'])
def upload_data():
    try:
        data = request.json
        rfid_tag = data.get("rfid_tag")
        timestamp = data.get("timestamp")

        if not rfid_tag or not timestamp:
            return jsonify({"error": "Invalid input"}), 400

       
        df = pd.read_excel(FILE_PATH)
        new_entry = pd.DataFrame({"RFID_Tag": [rfid_tag], "Timestamp": [timestamp]})
        df = pd.concat([df, new_entry], ignore_index=True)

        # Save back to Excel
        df.to_excel(FILE_PATH, index=False)
        return jsonify({"message": "Data added successfully"}), 200

    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/get_records', methods=['GET'])
def get_records():
    try:
        df = pd.read_excel(FILE_PATH)
        records = df.to_dict(orient="records")
        return jsonify(records), 200

    except Exception as e:
        return jsonify({"error": str(e)}), 500

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=5000, debug=True)

