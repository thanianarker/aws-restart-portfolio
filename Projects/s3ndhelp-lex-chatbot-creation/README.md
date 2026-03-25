# S3ndHelp 🤖☁️  
AWS Lex Chatbot Project

## 📌 Project Overview

**S3ndHelp** is an interactive chatbot built using **Amazon Lex**.  
The chatbot was designed to simulate a conversational AI assistant that helps users learn about **Amazon S3** through both informational responses and an interactive quiz.

This project demonstrates the fundamentals of **chatbot development**, including intents, utterances, and conversational flow using AWS services.

---

## 🎯 Project Objectives

- Build a functional chatbot using Amazon Lex  
- Create intent-based responses for user interaction  
- Design a quiz-based chatbot experience  
- Implement branching logic for different user answers  
- Demonstrate chatbot functionality through a live interaction  

---

## 🧠 How the Chatbot Works

### 🔹 Part 1: Basic Chatbot

The chatbot was first designed with a simple intent to answer questions about Amazon S3.

- **Intent:** S3Info  
- **Example Utterances:**
  - "What is S3?"
  - "Tell me about Amazon S3"  

- **Response:**
  - Amazon S3 is a cloud storage service that allows users to store and retrieve data from anywhere.

---

### 🔹 Part 2: Interactive Quiz (S3ndHelp Feature)

The chatbot was extended into a **quiz-based learning assistant**.

#### 📝 Quiz Flow

1. User starts the quiz:
   - "Start quiz"
   - "Quiz me on S3"

2. Chatbot asks:
   - *"What does S3 stand for?"*

3. Multiple-choice answers:
   - A) Simple Storage Service  
   - B) Secure Server Storage  
   - C) Smart Storage System  

4. Chatbot responses:
   - ✅ Correct → Confirms answer and moves forward  
   - ❌ Incorrect → Provides correct answer and continues  

5. Follow-up question:
   - *"What is Amazon S3 mainly used for?"*
     - A) Cloud storage  
     - B) Web hosting  
     - C) Cloud computing  

6. The chatbot provides feedback and guides the user through the quiz.

---

## 🛠️ AWS Services Used

- **Amazon Lex** – Used to build and manage the chatbot  
- Natural Language Processing (NLP) for understanding user input  

---

## ⚙️ Technical Implementation

- Created intents for both **information retrieval** and **quiz interaction**
- Defined **utterances** to simulate real user input  
- Designed **responses** for both correct and incorrect answers  
- Implemented **branching logic** to guide conversation flow  
- Tested chatbot using the AWS Lex test interface  

---

## 💡 Key Learnings

- How chatbots use **intents and utterances** to understand user input  
- How to design **conversational flows**  
- Implementing **branching logic** for interactive experiences  
- Importance of **user-friendly responses**  
- How AI can be used in **education and knowledge testing**  

---

## 🎥 Demo & Screenshots

Screenshots of the chatbot and quiz interaction are included in the `screenshots` folder.

Example:
- Bot creation in Amazon Lex  
- Intent and utterance configuration  
- Quiz interaction flow  
- Correct vs incorrect responses  

---

## ✅ Conclusion

The **S3ndHelp chatbot** demonstrates how AWS Lex can be used to build interactive, educational AI tools.  
It highlights my ability to design conversational systems and apply cloud-based AI services to real-world use cases.
