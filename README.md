The AI Business Advisor
This project is a hybrid desktop application that combines a standard calculator with an AI-powered business tip generator. The application is built in two parts: a Java-based desktop frontend and a Node.js backend that communicates with the Google Gemini API.

Features
Standard Calculator Functionality: Perform basic arithmetic operations.

AI-Powered Tips: Receive a concise, one-sentence business tip by typing a topic and clicking a button.

Hybrid Architecture: A solid example of a Java client communicating with a Node.js server.

Technologies Used
Frontend: Java, Swing

Backend: Node.js, Express.js

API: Google Gemini API

Project Files
index.js (The Backend Server)
This is the Node.js server that connects to the AI.

const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
const port = 3000;

app.use(cors());
app.use(bodyParser.json());

// This is the API key for the Gemini API. Do not modify this line.
const apiKey = "";
const API_URL = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-05-20:generateContent?key=${apiKey}`;

// The API endpoint that the Java application will call.
app.post('/get-tip', async (req, res) => {
    try {
        const userPurpose = req.body.user_purpose_text;

        // Create the payload for the Gemini API call.
        const payload = {
            contents: [{
                parts: [{
                    text: `Act as a creative and helpful AI assistant. Provide a concise, actionable, and encouraging "smart tip" based on the following user purpose: "${userPurpose}". The tip should be a single sentence and not directly mention the purpose.`
                }]
            }],
        };

        let response;
        let retries = 0;
        const maxRetries = 3;

        // Implement exponential backoff for API calls.
        while (retries < maxRetries) {
            try {
                response = await fetch(API_URL, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                if (response.status === 429) { // Too many requests
                    const delay = Math.pow(2, retries) * 1000;
                    console.log(`Rate limit exceeded. Retrying in ${delay / 1000} seconds...`);
                    await new Promise(resolve => setTimeout(resolve, delay));
                    retries++;
                    continue;
                }
                break;
            } catch (error) {
                console.error('API call failed:', error);
                const delay = Math.pow(2, retries) * 1000;
                console.log(`API call failed. Retrying in ${delay / 1000} seconds...`);
                await new Promise(resolve => setTimeout(resolve, delay));
                retries++;
            }
        }

        if (!response || !response.ok) {
            console.error(`API returned an error: ${response.statusText}`);
            return res.status(500).json({
                error: 'Failed to get a tip from the AI.'
            });
        }

        const result = await response.json();
        const tip = result.candidates[0].content.parts[0].text;

        // Send the AI tip back to the Java application.
        res.json({
            tip: tip
        });

    } catch (error) {
        console.error('Error processing request:', error);
        res.status(500).json({
            error: 'An internal server error occurred.'
        });
    }
});

app.listen(port, () => {
    console.log(`Smart Assistant API listening at http://localhost:${port}`);
});

BasicAssistantCalculator.java (The Java Frontend)
This is the code for the graphical user interface.

import javax.swing.*;
import java.awt.*;
import java.awt.event.ActionEvent;
import java.awt.event.ActionListener;
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.net.HttpURLConnection;
import java.net.URL;

public class BasicAssistantCalculator extends JFrame implements ActionListener {
    // UI Components
    private JTextField displayField;
    private JTextField purposeField;
    private JTextArea tipArea;
    private JButton getTipButton;

    // Calculator state
    private String currentInput = "";
    private double firstOperand = 0;
    private String operator = "";
    private boolean isOperatorClicked = false;

    public BasicAssistantCalculator() {
        // Frame setup
        setTitle("Basic Assistant Calculator");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setLayout(new BorderLayout(10, 10));

        // Top panel for calculator display and input
        JPanel topPanel = new JPanel(new BorderLayout());
        displayField = new JTextField("0");
        displayField.setEditable(false);
        displayField.setFont(new Font("Arial", Font.BOLD, 24));
        displayField.setHorizontalAlignment(JTextField.RIGHT);
        topPanel.add(displayField, BorderLayout.CENTER);

        // Center panel for calculator buttons
        JPanel centerPanel = new JPanel(new GridLayout(4, 4, 5, 5));
        String[] buttonLabels = {
            "7", "8", "9", "/",
            "4", "5", "6", "*",
            "1", "2", "3", "-",
            "0", ".", "=", "+"
        };

        for (String label : buttonLabels) {
            JButton button = new JButton(label);
            button.setFont(new Font("Arial", Font.PLAIN, 18));
            button.addActionListener(this);
            centerPanel.add(button);
        }

        // AI Assistant panel at the bottom
        JPanel bottomPanel = new JPanel(new BorderLayout(5, 5));
        
        // Input for tip purpose
        JPanel tipInputPanel = new JPanel(new BorderLayout());
        purposeField = new JTextField("Enter Purpose (e.g., 'startup marketing')");
        purposeField.setFont(new Font("Arial", Font.ITALIC, 14));
        purposeField.setForeground(Color.GRAY);
        tipInputPanel.add(purposeField, BorderLayout.CENTER);

        // Get Tip button
        getTipButton = new JButton("Get Smart Tip");
        getTipButton.addActionListener(this);
        tipInputPanel.add(getTipButton, BorderLayout.EAST);
        
        bottomPanel.add(tipInputPanel, BorderLayout.NORTH);

        // Display area for the tip
        tipArea = new JTextArea("Your tip will appear here.");
        tipArea.setEditable(false);
        tipArea.setLineWrap(true);
        tipArea.setWrapStyleWord(true);
        tipArea.setFont(new Font("Arial", Font.PLAIN, 14));
        tipArea.setBorder(BorderFactory.createTitledBorder("AI Assistant Tip"));
        JScrollPane scrollPane = new JScrollPane(tipArea);
        scrollPane.setPreferredSize(new Dimension(300, 80));
        bottomPanel.add(scrollPane, BorderLayout.CENTER);

        // Add all panels to the frame
        add(topPanel, BorderLayout.NORTH);
        add(centerPanel, BorderLayout.CENTER);
        add(bottomPanel, BorderLayout.SOUTH);

        // Display the frame
        pack();
        setLocationRelativeTo(null); // Center the frame on the screen
        setVisible(true);
    }

    @Override
    public void actionPerformed(ActionEvent e) {
        String command = e.getActionCommand();

        if (Character.isDigit(command.charAt(0)) || command.equals(".")) {
            if (isOperatorClicked) {
                currentInput = command;
                isOperatorClicked = false;
            } else {
                currentInput += command;
            }
            displayField.setText(currentInput);
        } else if (command.matches("[+\\-*/]")) {
            firstOperand = Double.parseDouble(currentInput);
            operator = command;
            isOperatorClicked = true;
        } else if (command.equals("=")) {
            double secondOperand = Double.parseDouble(currentInput);
            double result = 0;
            switch (operator) {
                case "+":
                    result = firstOperand + secondOperand;
                    break;
                case "-":
                    result = firstOperand - secondOperand;
                    break;
                case "*":
                    result = firstOperand * secondOperand;
                    break;
                case "/":
                    if (secondOperand != 0) {
                        result = firstOperand / secondOperand;
                    } else {
                        displayField.setText("Error");
                        return;
                    }
                    break;
            }
            displayField.setText(String.valueOf(result));
            currentInput = String.valueOf(result);
        } else if (e.getSource() == getTipButton) {
            String purpose = purposeField.getText();
            new Thread(() -> {
                try {
                    String tip = getSmartTip(purpose);
                    SwingUtilities.invokeLater(() -> tipArea.setText(tip));
                } catch (Exception ex) {
                    SwingUtilities.invokeLater(() -> tipArea.setText("Error: Could not get a tip. Please check the server."));
                    ex.printStackTrace();
                }
            }).start();
        }
    }

    private String getSmartTip(String purpose) throws Exception {
        URL url = new URL("http://localhost:3000/get-tip");
        HttpURLConnection conn = (HttpURLConnection) url.openConnection();
        conn.setRequestMethod("POST");
        conn.setRequestProperty("Content-Type", "application/json; utf-8");
        conn.setRequestProperty("Accept", "application/json");
        conn.setDoOutput(true);

        String jsonInputString = "{\"user_purpose_text\": \"" + purpose + "\"}";

        try (OutputStream os = conn.getOutputStream()) {
            byte[] input = jsonInputString.getBytes("utf-8");
            os.write(input, 0, input.length);
        }

        try (BufferedReader br = new BufferedReader(new InputStreamReader(conn.getInputStream(), "utf-8"))) {
            StringBuilder response = new StringBuilder();
            String responseLine = null;
            while ((responseLine = br.readLine()) != null) {
                response.append(responseLine.trim());
            }

            // Simple JSON parsing to get the tip
            String jsonResponse = response.toString();
            int tipStartIndex = jsonResponse.indexOf("\"tip\":") + 7;
            int tipEndIndex = jsonResponse.lastIndexOf("\"");
            return jsonResponse.substring(tipStartIndex, tipEndIndex);
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> new BasicAssistantCalculator());
    }
}

Getting Started
To run this application, you need to set up both the backend server and the frontend client.

Prerequisites
Node.js: Ensure you have Node.js installed on your machine.

Java Development Kit (JDK): You need the JDK to compile and run the Java code.

Step 1: Set Up the Backend Server
Navigate to the directory containing index.js in your terminal.

Install the required Node.js packages by running the following command:

npm install express cors body-parser node-fetch

Start the server. The server will listen on port 3000.

node index.js

You should see a message in your terminal confirming that the server is running. Leave this terminal window open.

Step 2: Run the Java Frontend
Open a new terminal window.

Navigate to the directory containing BasicAssistantCalculator.java.

Compile the Java code with the following command:

javac BasicAssistantCalculator.java

Run the compiled application:

java BasicAssistantCalculator

This will launch the calculator GUI.

Step 3: Using the Application
With both the server and the Java application running, you can:

Use the standard calculator functions.

To get a business tip, type a topic (e.g., "startup marketing," "improving sales," or "customer service") into the text field.

Click the "Get Smart Tip" button.

The application will send your request to the backend server, which will query the AI and display a tip in the text area below.


