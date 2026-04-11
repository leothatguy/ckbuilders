## Builder Track Weekly Report — Week 4

**Name:** Divine Oshione Anesi  
**Week Ending:** 04-11-2026  

### Courses Completed

- Completed Intro to Script Programming 1–5 (Script development course)
  - Class 1: Validation Model
  - Class 2: Script Basics
  - Class 3: UDT
  - Class 4: WebAssembly on CKB
  - Class 5: Debugging
  - Class 6: Type ID
- Completed Simple UDT (sUDT) Standard (RFC 0025)
- Completed sUDT Operations Tutorial (CKB CLI)

### Key Learnings

- Developed a solid understanding of CKB’s Validation Model and Script Execution Flow:
    - Transactions are validated through execution of lock scripts (inputs) and type scripts (inputs and outputs)
    - Script execution is grouped and deduplicated, meaning identical scripts run once per transaction
    - Any script failure results in the entire transaction being rejected

- Gained clarity on the distinction between Script and Script Code
- Learned how CKB-VM executes scripts
- Developed understanding of Script Grouping and Transaction Context
- Learned the structure and behavior of Simple UDT (sUDT)
- Understood token operations and control mechanisms

### Practical Progress

- Studied and followed the full script programming flow from validation model to execution
- Explored how scripts are compiled, deployed, and referenced in transactions
- Reviewed and understood the sUDT standard structure and validation rules
- Analyzed how token transfers and minting logic are enforced through type scripts
- Prepared to implement token logic using the sUDT model

### Environment
- Rust (latest stable)
- Cargo (latest)
- CKB CLI
- OffCKB CLI
- CKB Debugger
- Node.js (≥18)
- Git (≥2.40.0)