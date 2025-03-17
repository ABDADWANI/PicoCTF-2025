# **Write-Up: WebSockFish - PicoCTF 2024**

## **Challenge Overview**

- **Challenge Name:** WebSockFish  
- **Category:** Web Exploitation (Client-Side WebSockets)  
- **Difficulty:** Medium to Hard  
- **Description:**  

The challenge presents a web-based chess game where the player competes against the **Stockfish chess engine**. Given Stockfish's strength as a chess engine, winning fairly is nearly impossible. Our objective is to analyze the game's mechanics and find a way to manipulate the system to force a victory.

---

## **Step 1: Initial Analysis and Reconnaissance**

Upon accessing the challenge URL, we are presented with an **interactive chessboard** and a chat feature where a fish icon provides commentary. The chess engine appears to process the moves and respond accordingly.

### **Key Observations:**
1. The game interface allows us to make moves against Stockfish.
2. The chess engine provides **position evaluations** through an interactive chat.
3. The game uses **WebSockets (`ws://`)** to communicate move data and evaluation scores.

### **Examining the JavaScript Code**

By inspecting the browser’s **Developer Tools (DevTools)** and analyzing the **JavaScript source code**, we identified the following important details:

#### **1. WebSocket Communication**
```javascript
var ws_address = "ws://" + location.hostname + ":" + location.port + "/ws/";
const ws = new WebSocket(ws_address);
```
- The game establishes a **WebSocket connection** to exchange data between the client and server.

#### **2. Receiving Messages from the Server**
```javascript
ws.onmessage = (event) => {
    const message = event.data;
    updateChat(message);
};
```
- The game receives data from the server and updates the chat interface with the received messages.

#### **3. Interaction with the Stockfish Chess Engine**
```javascript
var stockfish = new Worker("js/stockfish.min.js");
stockfish.postMessage("uci");
```
- The game uses **Stockfish** (a strong chess engine) to analyze moves.
- The `postMessage` function sends commands to Stockfish.

#### **4. Evaluating Positions**
```javascript
stockfish.onmessage = function (event) {
    if (event.data.startsWith(`info depth ${DEPTH}`)) {
        var splitString = event.data.split(" ");
        if (event.data.includes("mate")) {
            message = "mate " + parseInt(splitString[9]);
        } else {
            message = "eval " + parseInt(splitString[9]);
        }
        sendMessage(message);
    }
};
```
- Stockfish calculates an **evaluation (`eval`) score** for each position:
  - **Positive values (`+N`)**: Favorable for White.
  - **Negative values (`-N`)**: Favorable for Black.
  - **Extreme values indicate a decisive win/loss.**
- If Stockfish finds a **forced checkmate**, it sends `"mate X"`, indicating checkmate in `X` moves.

---

## **Step 2: Identifying a Potential Exploit**

### **Key Observations from the Code**
- Stockfish **sends position evaluations** as **plain messages** via WebSockets.
- There is **no verification** ensuring that the messages originate from Stockfish.
- If we can **manipulate these messages**, we might be able to force a victory.

---

## **Step 3: Exploiting the WebSocket Communication**

### **Initial Attempts**
1. **Testing a Checkmate Message:**
   ```javascript
   ws.send("mate 1");
   ```
   - No immediate effect on the game.

2. **Sending a High Positive Evaluation:**
   ```javascript
   ws.send("eval 99999");
   ```
   - Response from the game:  
     > *"You're in deep water now!"*  
   - This suggests that **high values trigger a reaction but do not force a win**.

3. **Sending a High Negative Evaluation:**
   ```javascript
   ws.send("eval -5000");
   ```
   - Response from the game:  
     > *"Wow, you're quite the chess shark!"*  
   - This implies that **negative values indicate the engine is losing**.

### **The Breakthrough**
Since **negative evaluation values** represent **Stockfish losing**, we hypothesized that sending an extreme negative value might cause the engine to resign.

```javascript
ws.send("eval -100000");
```

**Game Response:**
> *"Huh???? How can I be losing this badly... I resign... here's your flag: picoCTF{c1i3nt_s1d3_w3b_s0ck3t5_50441bef}"*

This confirmed that **Stockfish resigned due to the extreme negative evaluation, awarding us the victory**.

---

## **Step 4: Understanding Why This Exploit Works**

1. **Stockfish Determines Position Evaluations**  
   - The game listens for `eval` values from Stockfish and updates the game state accordingly.

2. **No Verification of WebSocket Messages**  
   - The server blindly accepts `eval` values sent via WebSocket.
   - This allows an attacker to **send arbitrary evaluation scores**, manipulating Stockfish’s decision-making.

3. **Extreme Negative Values Indicate a Lost Game**  
   - In chess engines, **a very negative score means the engine is completely lost**.
   - By sending `"eval -100000"`, we **tricked the engine into thinking it had lost**, forcing it to resign.

---
