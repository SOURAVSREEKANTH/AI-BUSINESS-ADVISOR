# AI-BUSINESS-ADVISOR



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

Getting Started
To run this application, you need to set up both the backend server and the frontend client.

Prerequisites
Node.js: Ensure you have Node.js installed on your machine.

Java Development Kit (JDK): You need the JDK to compile and run the Java code.

Step 1: Set Up the Backend Server
Navigate to the backend directory in your terminal.

Install the required Node.js packages by running the following command:

npm install express cors body-parser node-fetch

Start the server. The server will listen on port 3000.

node index.js

You should see a message in your terminal confirming that the server is running. Leave this terminal window open.

Step 2: Run the Java Frontend
Open a new terminal window.

Navigate to the frontend directory where your Java file (BasicAssistantCalculator.java) is located.

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

Contributing
Feel free to fork this project and add new features. Some ideas for expansion include:

Saving a history of tips.

Adding user authentication.

Integrating with a database to store user tips.

Providing tips on a wider range of topics.

License
This project is open-source and available under the MIT License.
