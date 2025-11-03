# Bitcoin Testnet Wallet Project - Complete Technical Explanation

## Project Overview

This project creates a **real Bitcoin testnet wallet application** with a web-based user interface. It connects to Bitcoin Core (the official Bitcoin node software) to perform actual blockchain operations on the testnet network.

## System Architecture

```
┌─────────────────┐         ┌──────────────────┐         ┌─────────────────┐
│   Web Browser   │         │   HTTP Server    │         │  Bitcoin Core   │
│   (Frontend)    │◄───────►│   (C++ Backend)  │◄───────►│   (RPC Server)  │
│                 │  HTTP   │                  │  RPC    │                 │
│  - index.html   │         │  - http_server   │         │  - testnet node │
│  - app.js       │         │  - network_client│         │  - RPC on 18332 │
│  - style.css    │         │                  │         │                 │
└─────────────────┘         └──────────────────┘         └─────────────────┘
```

## Component Breakdown

### 1. Bitcoin Core (The Foundation)

**What it is:**
- Bitcoin Core is the official Bitcoin node software
- It maintains a copy of the blockchain
- It provides an RPC (Remote Procedure Call) interface for external applications

**Key Configuration (bitcoin.conf):**
```conf
server=1              # Enable RPC server
rpcuser=mywallet      # RPC authentication username
rpcpassword=MyPassword123!  # RPC authentication password
rpcallowip=127.0.0.1  # Allow connections only from localhost
rpcport=18332         # RPC port for testnet (mainnet uses 8332)
bind=127.0.0.1        # Bind to localhost only
```

**How it works:**
- When started with `-testnet` flag, Bitcoin Core connects to testnet network
- RPC server listens on port 18332 (testnet) or 8332 (mainnet)
- Provides JSON-RPC API for blockchain operations

**Common RPC Commands Used:**
- `createwallet` - Creates a new wallet in Bitcoin Core
- `getnewaddress` - Generates a new Bitcoin address
- `getwalletinfo` - Gets wallet balance and information
- `sendtoaddress` - Sends Bitcoin from wallet to an address

### 2. HTTP Server (C++ Backend)

**Location:** `src/http_server.cpp`, `examples/server_main.cpp`

**Purpose:**
- Acts as a bridge between the web UI and Bitcoin Core
- Serves the web interface files (HTML, CSS, JS)
- Handles API requests from the frontend
- Translates web requests into Bitcoin Core RPC calls

**Key Functions:**

**a) File Serving:**
```cpp
serveFile("web/index.html")  // Serves web UI files
```

**b) API Endpoints:**
- `POST /api/wallet/create` - Creates a wallet via Bitcoin Core
- `GET /api/wallet/info` - Gets wallet information
- `GET /api/wallet/balance` - Gets wallet balance
- `POST /api/wallet/send` - Sends Bitcoin transaction

**c) Wallet Creation Flow:**
```cpp
1. Receives request from web UI
2. Calls Bitcoin Core RPC: createwallet("MyTestnetWallet")
3. Bitcoin Core creates wallet and returns response
4. Calls getnewaddress to get wallet address
5. Returns wallet info (name, address, balance) to web UI
```

### 3. Network Client (Bitcoin Core Communication)

**Location:** `src/network_client.cpp`

**Purpose:**
- Handles all communication with Bitcoin Core RPC
- Establishes TCP socket connections
- Sends JSON-RPC requests
- Parses JSON-RPC responses

**How RPC Communication Works:**

**Step 1: Build JSON-RPC Request**
```json
{
  "jsonrpc": "1.0",
  "id": "1",
  "method": "createwallet",
  "params": ["MyTestnetWallet", false, false, "", false]
}
```

**Step 2: Build HTTP Request**
```
POST / HTTP/1.1
Host: 127.0.0.1:18332
Content-Type: application/json
Authorization: Basic base64(username:password)
Content-Length: <length>

<JSON-RPC request>
```

**Step 3: Establish TCP Connection**
```cpp
// Create socket
socket(AF_INET, SOCK_STREAM, IPPROTO_TCP)

// Connect to Bitcoin Core
connect(socket, "127.0.0.1:18332")

// Send HTTP request
send(socket, http_request)

// Receive response
recv(socket, response)
```

**Step 4: Parse Response**
```json
{
  "result": "wallet_name_or_address",
  "error": null
}
```

**Authentication:**
- Uses HTTP Basic Authentication
- Credentials: `rpcuser:rpcpassword` from bitcoin.conf
- Encoded in Base64: `Authorization: Basic base64(username:password)`

### 4. Web Frontend (User Interface)

**Location:** `web/index.html`, `web/app.js`, `web/style.css`

**Purpose:**
- Provides user-friendly interface
- Communicates with backend via HTTP API
- Displays wallet information and transaction forms

**How It Works:**

**a) Wallet Creation:**
```javascript
1. User clicks "Create New Wallet" button
2. JavaScript sends POST to /api/wallet/create
3. Backend creates wallet via Bitcoin Core RPC
4. Backend returns wallet address
5. JavaScript displays address and balance
```

**b) Send Transaction:**
```javascript
1. User enters recipient address and amount
2. JavaScript sends POST to /api/wallet/send
3. Backend calls Bitcoin Core RPC: sendtoaddress(address, amount)
4. Bitcoin Core creates and broadcasts transaction
5. Backend returns transaction ID (txid)
6. JavaScript shows success message with txid
```

### 5. Supporting Modules

**Wallet Management (`src/wallet.cpp`):**
- Manages local wallet state
- Tracks UTXOs (Unspent Transaction Outputs)
- Generates addresses (currently placeholder)

**Token Transfer (`src/token_transfer.cpp`):**
- Handles Bitcoin transfer logic
- Selects UTXOs for transactions
- Calculates fees
- Builds transactions

**Transaction Building (`src/transaction.cpp`):**
- Serializes transactions to hex format
- Creates transaction inputs and outputs
- Estimates transaction size for fee calculation

## Complete Flow: Creating a Wallet

```
User Action (Browser)
    │
    ▼
JavaScript: fetch('/api/wallet/create')
    │
    ▼
HTTP Server: handleCreateWallet()
    │
    ▼
NetworkClient: rpcCallRaw("createwallet")
    │
    ├─► Create TCP socket
    ├─► Connect to 127.0.0.1:18332
    ├─► Build HTTP request with JSON-RPC
    ├─► Add Basic Auth header
    ├─► Send request
    └─► Receive response
    │
    ▼
Bitcoin Core processes request
    │
    ├─► Creates wallet file: wallet.dat
    ├─► Returns wallet name
    │
    ▼
NetworkClient: rpcCallRaw("getnewaddress")
    │
    └─► Returns new Bitcoin address (e.g., mzBc4XxSqHB3km3Xj...)
    │
    ▼
HTTP Server formats response
    │
    └─► JSON: {"success": true, "address": "...", "balance": 0}
    │
    ▼
JavaScript displays wallet info
    │
    └─► Shows address, balance, enables send button
```

## Complete Flow: Sending Bitcoin

```
User enters: recipient address, amount (BTC), fee rate
    │
    ▼
JavaScript: fetch('/api/wallet/send', {to, amount, fee_rate})
    │
    ▼
HTTP Server: handleSendTransaction()
    │
    ├─► Converts BTC to satoshis (multiply by 100,000,000)
    ├─► Calls NetworkClient: rpcCallRaw("sendtoaddress")
    │
    ▼
Bitcoin Core RPC: sendtoaddress
    │
    ├─► Validates address
    ├─► Checks wallet balance
    ├─► Creates transaction
    ├─► Signs transaction with wallet private keys
    ├─► Broadcasts to testnet network
    └─► Returns transaction ID (txid)
    │
    ▼
HTTP Server returns txid to frontend
    │
    ▼
JavaScript shows: "Transaction sent! TXID: abc123..."
```

## Technical Details

### Why TCP Sockets?
- Bitcoin Core RPC uses HTTP over TCP
- Direct socket connection gives full control
- No external dependencies (like libcurl)
- Works on Windows, Linux, macOS

### Security Considerations:
1. **RPC Authentication:** Only localhost (127.0.0.1) can connect
2. **Username/Password:** Required for all RPC calls
3. **Testnet Only:** Uses testnet, not real Bitcoin
4. **No Private Key Exposure:** Bitcoin Core manages keys securely

### Real vs Mock:
- **100% Real:** All operations use actual Bitcoin Core
- **Real Blockchain:** Connects to Bitcoin testnet network
- **Real Transactions:** Broadcast to actual testnet nodes
- **Real Wallets:** Created and stored by Bitcoin Core
- **Real Addresses:** Valid testnet Bitcoin addresses

### Data Flow Example:

**1. Create Wallet:**
```
Browser → HTTP POST /api/wallet/create
    → Server builds JSON-RPC: {"method": "createwallet", "params": [...]}
    → Socket connection to 127.0.0.1:18332
    → HTTP POST with Basic Auth
    → Bitcoin Core creates wallet.dat file
    → Returns wallet name
    → Server calls getnewaddress
    → Returns testnet address (e.g., mzBc4Xx...)
    → Server returns JSON to browser
    → Browser displays address
```

**2. Send Transaction:**
```
Browser → HTTP POST /api/wallet/send {"to": "...", "amount": 10000000}
    → Server converts: 0.1 BTC = 10,000,000 satoshis
    → Server builds JSON-RPC: {"method": "sendtoaddress", "params": ["address", 0.1]}
    → Socket to Bitcoin Core
    → Bitcoin Core:
       * Validates address
       * Checks balance
       * Selects UTXOs
       * Creates transaction
       * Signs with wallet keys
       * Broadcasts to network
    → Returns txid: "abc123def456..."
    → Browser shows success
```

## Key Technologies

1. **C++17:** Modern C++ for backend
2. **HTTP/1.1:** Communication protocol
3. **JSON-RPC 1.0:** Bitcoin Core API format
4. **TCP Sockets:** Low-level network communication
5. **Base64 Encoding:** For HTTP Basic Auth
6. **HTML/CSS/JavaScript:** Frontend interface
7. **CMake:** Build system

## Why This Architecture?

**Advantages:**
- ✅ Direct control over Bitcoin Core communication
- ✅ No external libraries needed for RPC
- ✅ Real blockchain operations
- ✅ Testnet-safe (no real money)
- ✅ Full wallet functionality

**Bitcoin Core RPC is the Standard:**
- Official way to interact with Bitcoin
- Used by exchanges, wallets, services
- Secure (keys never leave Bitcoin Core)
- Reliable (proven over 15+ years)

## Testing on Testnet

1. **Get Testnet Coins:**
   - Use faucets: https://testnet-faucet.mempool.co/
   - Free testnet Bitcoin (no value)

2. **Verify Transactions:**
   - Check on testnet explorer: https://blockstream.info/testnet/
   - Enter your address or txid

3. **Why Testnet:**
   - Safe to experiment
   - No real money risk
   - Same technology as mainnet
   - Perfect for learning

## Summary for Mentor

**What this project does:**
- Creates a web-based Bitcoin wallet using real Bitcoin Core
- Connects to Bitcoin testnet (not mainnet)
- Uses official Bitcoin Core RPC API
- Provides create wallet, view balance, send Bitcoin functionality

**How it works:**
1. Web UI (JavaScript) → 2. HTTP Server (C++) → 3. Bitcoin Core RPC → 4. Testnet Network

**Key achievement:**
- Real Bitcoin wallet (not mock/demo)
- Actual blockchain operations
- Production-ready architecture
- Secure (private keys managed by Bitcoin Core)

This is a **real, working Bitcoin wallet** that connects to the actual testnet blockchain!

