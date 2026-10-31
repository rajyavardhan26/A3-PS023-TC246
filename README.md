# PS-023: Minimal Two-Node Cloud Hypervisor VM Controller

**Demo Video:** [Watch the PS-023 Live Project Demonstration](https://youtu.be/iMdf5uf8Ftw)

**Project Video:** [Watch the PS-023 project explanation](https://youtu.be/2IhxcSRknVc)

**Team:** The Mavericks  
**Team Code:** `A3-PS023-TC246`  
**Project:** A minimal centralized virtual machine controller for two KVM-capable Linux hosts.

> **Project status:** Proposed system design and implementation roadmap. Update the implementation status, installation commands, API endpoints, test results, and security controls to match the actual repository before presenting them as completed.

---

## Table of Contents

- [Demo Video](#demo-video)
- [Project Video](#project-video)
- [1. Project Overview](#1-project-overview)
- [2. Problem Statement](#2-problem-statement)
- [3. Objectives](#3-objectives)
- [4. Key Features](#4-key-features)
- [5. System Architecture](#5-system-architecture)
- [6. Technology Stack](#6-technology-stack)
- [7. System Workflow](#7-system-workflow)
- [8. Data Flow Diagram](#8-data-flow-diagram)
- [9. Memory-Aware Scheduling](#9-memory-aware-scheduling)
- [10. Virtual Machine Lifecycle](#10-virtual-machine-lifecycle)
- [11. Security Architecture](#11-security-architecture)
- [12. Failure Handling and Recovery](#12-failure-handling-and-recovery)
- [13. Database and State Management](#13-database-and-state-management)
- [14. API Design](#14-api-design)
- [15. Deployment Architecture](#15-deployment-architecture)
- [16. Installation and Setup](#16-installation-and-setup)
- [17. Testing and Validation](#17-testing-and-validation)
- [18. Performance Evaluation](#18-performance-evaluation)
- [19. Project Roadmap](#19-project-roadmap)
- [20. Limitations and Non-Goals](#20-limitations-and-non-goals)
- [21. Troubleshooting](#21-troubleshooting)
- [22. Contributing](#22-contributing)
- [23. References](#23-references)

---

## Demo Video

Watch the complete project demonstration:

[https://youtu.be/iMdf5uf8Ftw](https://youtu.be/iMdf5uf8Ftw)

## Project Video

Watch the project explanation and walkthrough:

[https://youtu.be/2IhxcSRknVc](https://youtu.be/2IhxcSRknVc)

---

## 1. Project Overview

PS-023 proposes a lightweight, centralized Virtual Machine (VM) controller that manages virtual machines across two KVM-capable Linux hosts using Cloud Hypervisor.

Instead of managing virtual machines independently on each host, the system introduces a central controller that accepts requests, selects a suitable host, coordinates VM operations, and tracks the resulting state. A worker agent on each host performs the actual VM operations through the local Cloud Hypervisor API.

The system focuses on safe resource allocation, memory-aware scheduling, authenticated controller-worker communication, health monitoring, state reconciliation, and reliable cleanup.

### The core idea

A user submits a VM request. The controller validates the request, reserves the required resources, chooses a suitable host, and dispatches the operation to its worker. The worker executes the operation and returns its status. The controller reconciles the result with the observed VM state and updates its records.

### Project goals

- Simplify VM lifecycle management across two physical or virtualized compute hosts.
- Prevent memory overcommitment caused by concurrent scheduling requests.
- Separate centralized orchestration from host-level execution.
- Reduce inconsistent VM records and duplicate operations after failures.
- Establish a modular foundation for future multi-host virtualization management.

---

## 2. Problem Statement

Managing virtual machines across multiple compute hosts requires reliable coordination, efficient resource allocation, and secure communication. Without centralized management, operations such as creating, starting, stopping, listing, and deleting VMs can become difficult to coordinate when multiple requests compete for limited resources.

The proposed solution is a centralized controller supported by two worker agents. The controller performs host selection and resource reservation, while workers execute VM operations locally. Health monitoring, state reconciliation, and safe resource cleanup help maintain consistency when requests time out or workers become unavailable.

---

## 3. Objectives

1. **Centralized orchestration:** Provide one management interface for VM lifecycle operations across both hosts.
2. **Safe scheduling:** Select a host using available memory and maintain consistent resource reservations.
3. **Secure communication:** Authenticate controller-worker requests and restrict access to privileged VM operations.
4. **Reliable execution:** Handle timeouts, worker failures, and uncertain VM states through reconciliation.
5. **Resource consistency:** Keep VM records and reservations aligned with observed host state.
6. **Testable implementation:** Validate concurrency, failure scenarios, access control, and resource cleanup.

---

## 4. Key Features

| Feature | Purpose |
|---|---|
| Centralized controller | Coordinates requests and host selection |
| Two worker agents | Execute VM operations on individual hosts |
| Cloud Hypervisor integration | Provides host-local VM lifecycle control |
| Memory-aware scheduling | Chooses a host with sufficient allocatable memory |
| Atomic reservations | Prevents concurrent requests from claiming the same capacity |
| VM state tracking | Maintains the controller's view of each VM |
| Authenticated communication | Restricts controller-worker interactions |
| Health monitoring | Detects worker unavailability |
| State reconciliation | Resolves discrepancies between recorded and actual VM states |
| Resource cleanup | Releases reservations when their associated resource usage ends |
| Structured logging | Supports debugging, auditing, and incident investigation |

The table describes the intended feature set. Each item should be marked implemented, tested, or planned based on the actual repository.

---

## 5. System Architecture

The architecture separates request handling, orchestration, state management, and host-level execution.

```mermaid
flowchart TB
    U["User / Client"]
    C["Central Controller<br/>API + Validation + Scheduler"]
    DB[("Controller Database")]
    W1["Worker Agent 1"]
    W2["Worker Agent 2"]
    H1["KVM Host 1<br/>Cloud Hypervisor"]
    H2["KVM Host 2<br/>Cloud Hypervisor"]

    U -->|"VM requests"| C
    C <-->|"VM records and reservations"| DB
    C <-->|"Authenticated commands and status"| W1
    C <-->|"Authenticated commands and status"| W2
    W1 -->|"Local API"| H1
    W2 -->|"Local API"| H2
    H1 -.->|"Observed VM state"| W1
    H2 -.->|"Observed VM state"| W2
