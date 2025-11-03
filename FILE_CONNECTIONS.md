# File Connections Diagram - Simple Visual Guide

## Complete File Dependency Map

```
┌─────────────────────────────────────────────────────────────────┐
│                        ENTRY POINTS                              │
└─────────────────────────────────────────────────────────────────┘

examples/server_main.cpp
    │
    ├─► #include "bitcoin_config.h"
    └─► #include "http_server.h"
        │
        └─► Compiles with http_server.cpp

examples/main.cpp
    │
    ├─► #include "bitcoin_config.h"
    ├─► #include "wallet.h"
    ├─► #include "token_transfer.h"
    ├─► #include "network_client.h"
    └─► #include "utils.h"


┌─────────────────────────────────────────────────────────────────┐
│                        SOURCE FILES (.cpp)                       │
└─────────────────────────────────────────────────────────────────┘

src/http_server.cpp
    ├─► #include "http_server.h"          (its own header)
    ├─► #include "wallet.h"
    ├─► #include "token_transfer.h"
    ├─► #include "network_client.h"
    └─► #include "utils.h"
    
src/network_client.cpp
    ├─► #include "network_client.h"       (its own header)
    ├─► #include "utils.h"
    ├─► #include "bitcoin_config.h"
    └─► #include "transaction.h"
    
src/token_transfer.cpp
    ├─► #include "token_transfer.h"       (its own header)
    ├─► #include "transaction.h"
    ├─► #include "utils.h"
    └─► #include "wallet.h"
    
src/wallet.cpp
    ├─► #include "wallet.h"               (its own header)
    │   └─► (wallet.h includes key_manager.h and bitcoin_config.h)
    └─► (uses KeyManager and BitcoinConfig via header)
    
src/transaction.cpp
    ├─► #include "transaction.h"          (its own header)
    └─► #include "utils.h"
    
src/key_manager.cpp
    ├─► #include "key_manager.h"          (its own header)
    ├─► #include "utils.h"
    └─► #include "bitcoin_config.h"
    
src/bitcoin_config.cpp
    └─► #include "bitcoin_config.h"       (its own header)
    
src/utils.cpp
    └─► #include "utils.h"                (its own header)


┌─────────────────────────────────────────────────────────────────┐
│                        HEADER FILES (.h)                         │
└─────────────────────────────────────────────────────────────────┘

include/http_server.h
    ├─► Uses: BitcoinConfig, Wallet, NetworkClient (forward references)

include/network_client.h
    ├─► #include "bitcoin_types.h"
    ├─► #include "transaction.h"
    └─► #include "bitcoin_config.h"

include/token_transfer.h
    ├─► #include "bitcoin_types.h"
    └─► Uses: Wallet, Transaction (forward references)

include/wallet.h
    ├─► #include "bitcoin_types.h"
    ├─► #include "key_manager.h"
    └─► #include "bitcoin_config.h"

include/transaction.h
    └─► #include "bitcoin_types.h"

include/key_manager.h
    └─► #include "bitcoin_types.h"

include/bitcoin_config.h
    └─► #include "bitcoin_types.h"

include/utils.h
    └─► #include "bitcoin_types.h"

include/bitcoin_types.h
    └─► (BASE - no dependencies, only includes standard library)


┌─────────────────────────────────────────────────────────────────┐
│                    CONNECTION FLOW DIAGRAM                        │
└─────────────────────────────────────────────────────────────────┘

                    bitcoin_types.h
                    (Foundation)
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
bitcoin_config.h    utils.h         (all others use these)
        │                │
        │                └───────────┐
        │                            │
        ▼                            ▼
bitcoin_config.cpp              utils.cpp
        │                            │
        │                            │
        └──────────────┬──────────────┘
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
  wallet.h    key_manager.h    network_client.h
        │              │              │
        │              │              │
        ▼              ▼              ▼
  wallet.cpp   key_manager.cpp  network_client.cpp
        │              │              │
        │              │              │
        └──────────────┼──────────────┘
                       │
                       ▼
              token_transfer.h
                       │
                       ▼
              token_transfer.cpp
                       │
                       ▼
              transaction.h
                       │
                       ▼
              transaction.cpp
                       │
                       ▼
              http_server.h
                       │
                       ▼
              http_server.cpp


┌─────────────────────────────────────────────────────────────────┐
│                    DEPENDENCY TREE (Top to Bottom)                │
└─────────────────────────────────────────────────────────────────┘

Level 1 (Top - Entry Points):
    server_main.cpp
    main.cpp

Level 2 (Interface Layer):
    http_server.h/cpp

Level 3 (Business Logic):
    token_transfer.h/cpp
    wallet.h/cpp
    transaction.h/cpp

Level 4 (Network & Crypto):
    network_client.h/cpp
    key_manager.h/cpp

Level 5 (Configuration & Utilities):
    bitcoin_config.h/cpp
    utils.h/cpp

Level 6 (Foundation):
    bitcoin_types.h (header only)


┌─────────────────────────────────────────────────────────────────┐
│              CROSS-FILE COMMUNICATION PATTERNS                    │
└─────────────────────────────────────────────────────────────────┘

1. HTTP REQUEST HANDLING:
   http_server.cpp
   └──► wallet.cpp (for wallet operations)
   └──► network_client.cpp (for Bitcoin Core RPC)
   └──► token_transfer.cpp (for sending Bitcoin)

2. WALLET OPERATIONS:
   wallet.cpp
   └──► key_manager.cpp (for key/address generation)
   └──► bitcoin_config.cpp (for network settings)

3. TRANSACTION BUILDING:
   token_transfer.cpp
   └──► transaction.cpp (to build transaction structure)
   └──► wallet.cpp (to get UTXOs and private keys)
   └──► network_client.cpp (to broadcast transaction)

4. BITCOIN CORE COMMUNICATION:
   network_client.cpp
   └──► utils.cpp (for Base64 encoding auth)
   └──► bitcoin_config.cpp (for RPC connection info)
   └──► transaction.cpp (for transaction hex encoding)

5. KEY MANAGEMENT:
   key_manager.cpp
   └──► utils.cpp (for encoding/decoding: hex, base58)
   └──► bitcoin_config.cpp (for network-specific parameters)

6. UTILITY FUNCTIONS:
   utils.cpp
   └──► Used by: key_manager, network_client, 
                transaction, token_transfer, http_server
   └──► Provides: encoding, hashing, formatting


┌─────────────────────────────────────────────────────────────────┐
│                    INCLUDE CHAIN EXAMPLES                         │
└─────────────────────────────────────────────────────────────────┘

Example 1: Creating a Wallet (via HTTP Server)
    http_server.cpp includes:
        ├─► wallet.h
        │   └─► includes key_manager.h
        │       └─► includes bitcoin_types.h
        └─► network_client.h
            ├─► includes bitcoin_config.h
            │   └─► includes bitcoin_types.h
            └─► includes transaction.h
                └─► includes bitcoin_types.h

Example 2: Sending Bitcoin
    http_server.cpp includes:
        ├─► token_transfer.h
        │   ├─► includes bitcoin_types.h
        │   └─► uses Wallet (via forward ref)
        └─► network_client.h
            └─► (as above)

Example 3: Key Generation
    wallet.cpp includes:
        ├─► wallet.h
        │   └─► includes key_manager.h
        │       └─► includes bitcoin_types.h
        └─► (key_manager.h also includes utils.h internally)
            └─► utils.h includes bitcoin_types.h


┌─────────────────────────────────────────────────────────────────┐
│                    COMPILATION ORDER                             │
└─────────────────────────────────────────────────────────────────┘

When compiling, files must be built in dependency order:

1. bitcoin_types.h (header only - no compilation needed)
2. utils.cpp → utils.h
3. bitcoin_config.cpp → bitcoin_config.h
4. transaction.cpp → transaction.h
5. key_manager.cpp → key_manager.h
6. wallet.cpp → wallet.h
7. network_client.cpp → network_client.h
8. token_transfer.cpp → token_transfer.h
9. http_server.cpp → http_server.h
10. server_main.cpp (links everything together)


┌─────────────────────────────────────────────────────────────────┐
│                    KEY CONNECTIONS SUMMARY                        │
└─────────────────────────────────────────────────────────────────┘

Strongest Connections (Most Dependencies):
    http_server.cpp → uses 5 other modules
    token_transfer.cpp → uses 3 other modules
    wallet.cpp → uses 2 other modules

Most Used Module:
    utils.cpp → used by 6+ modules
    
Most Central Module:
    bitcoin_types.h → used by ALL modules
    
Critical Path:
    http_server → network_client → Bitcoin Core RPC
    
Data Flow:
    User → http_server → token_transfer → wallet → key_manager
    User → http_server → network_client → Bitcoin Core

