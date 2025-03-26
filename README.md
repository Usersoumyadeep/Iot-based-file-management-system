# Iot-based-file-management-system
The IoT-Based  secure File Management System is a web application that enables IoT devices to upload, store, manage, and retrieve files over the internet. This system is designed to facilitate secure file storage and retrieval for IoT devices, allowing users to interact with files via a web interface.
from flask import Flask, request, jsonify, send_from_directory, render_template
import os
import mysql.connector
from werkzeug.utils import secure_filename
import hashlib

app = Flask(__name__)
UPLOAD_FOLDER = 'uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)

db = mysql.connector.connect(
    host="localhost",
    user="root",
    password="password",
    database="file_management"
)
cursor = db.cursor()

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({"error": "No file part"})
    file = request.files['file']
    if file.filename == '':
        return jsonify({"error": "No selected file"})
    filename = secure_filename(file.filename)
    file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
    return jsonify({"message": "File uploaded successfully", "filename": filename})

@app.route('/files/<filename>', methods=['GET'])
def get_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)

@app.route('/register', methods=['POST'])
def register_user():
    data = request.get_json()
    username = data['username']
    password = hashlib.sha256(data['password'].encode()).hexdigest()
    cursor.execute("INSERT INTO users (username, password) VALUES (%s, %s)", (username, password))
    db.commit()
    return jsonify({"message": "User registered successfully"})

@app.route('/login', methods=['POST'])
def login_user():
    data = request.get_json()
    username = data['username']
    password = hashlib.sha256(data['password'].encode()).hexdigest()
    cursor.execute("SELECT * FROM users WHERE username=%s AND password=%s", (username, password))
    user = cursor.fetchone()
    if user:
        return jsonify({"message": "Login successful"})
    return jsonify({"error": "Invalid credentials"})

if __name__ == '__main__':
    app.run(debug=True)



    //ARDUINO CODE
    #include <SPI.h>
#include <MFRC522.h>
#include <ESP8266WiFi.h>
#include <ESP8266HTTPClient.h>

#define SS_PIN  D4  // SDA pin for RFID
#define RST_PIN D3  // Reset pin for RFID
MFRC522 rfid(SS_PIN, RST_PIN);

const char* ssid = "Your_WiFi_SSID";     // Replace with your WiFi SSID
const char* password = "Your_WiFi_Password"; // Replace with your WiFi password
const char* server = "http://your-server-ip:5000/rfid_auth"; // Flask API endpoint

WiFiClient client;
HTTPClient http;

void setup() {
    Serial.begin(115200);
    SPI.begin();
    rfid.PCD_Init();

    WiFi.begin(ssid, password);
    Serial.print("Connecting to WiFi");
    while (WiFi.status() != WL_CONNECTED) {
        delay(1000);
        Serial.print(".");
    }
    Serial.println("\nConnected to WiFi");
}

void loop() {
    if (!rfid.PICC_IsNewCardPresent() || !rfid.PICC_ReadCardSerial()) {
        return;
    }

    String uid = "";
    for (byte i = 0; i < rfid.uid.size; i++) {
        uid += String(rfid.uid.uidByte[i], HEX);
    }
    uid.toUpperCase();
    
    Serial.println("RFID UID: " + uid);

    // Send UID to Flask Server
    if (WiFi.status() == WL_CONNECTED) {
        http.begin(client, server);
        http.addHeader("Content-Type", "application/json");

        String postData = "{\"rfid\":\"" + uid + "\"}";
        int httpCode = http.POST(postData);
        String payload = http.getString();

        Serial.println("Server Response: " + payload);
        http.end();
    }

    delay(3000);
}

