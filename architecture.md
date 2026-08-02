... see more in the design overview [readme.md](https://github.com/vinas1/vibecode/blob/main/README.md)

| Architectural L1 Context Diagram |
| :---: |

```mermaid
flowchart LR
    Developer[Developer] --> Cline[Cline in VS Code]
    Cline --> LMStudio[LM Studio]
    LMStudio --> Gemma[Gemma 4]
    Cline --> Repository[Git Repository]
```

---

| Architectural L2 Container Diagram |
| :---: |

```mermaid
flowchart LR
    DEV[Developer]

    subgraph WORKSTATION[Developer Workstation]
        VSC[Visual Studio Code]
        CLINE[Cline Agent]
        REPO[Local Git Repository]
        LMS[LM Studio API]
    end

    subgraph LAB[DevPlat Lab Box]
        LMLINK[LM Link]
        GEMMA[Gemma 4 12B QAT]
    end

    GITHUB[GitHub]

    DEV --> VSC
    VSC --> CLINE
    CLINE -->|Read and edit files| REPO
    CLINE -->|Inference requests| LMS
    LMS -->|Secure model routing| LMLINK
    LMLINK --> GEMMA
    REPO -->|Push and pull| GITHUB
```

---

| Architectural L3 Component Diagram |
| :---: |

```mermaid
flowchart TD
    A[Developer MacBook] --> B[Visual Studio Code]
    B --> C[Cline Extension]
    C --> D[LM Studio Local API]
    D --> E{Model Location}

    E -->|Loaded locally| F[Fallback Model]
    E -->|Available through LM Link| G[Linux Lab Box]
    G --> H[Gemma 4 12B QAT]
    G --> I[Blackwell]

    C --> J{Cline Mode}

    J -->|Plan| K[Read + analyze workspace]
    J -->|Act| L[Use agent tools]

    L --> M[Read files]
    L --> N[Propose file edits]
    L --> O[Run terminal commands]

    M --> P[Auto-approved read operation]
    N --> Q[Developer reviews diff]
    O --> R[Developer approves command]

    Q -->|Save| S[Apply targeted change]
    Q -->|Reject| T[Discard proposed change]

    S --> U[Run git diff]
    R --> U
    U --> V[Developer validates result]
    V --> W[Commit and open pull request]
```

---

| Architectural L4 Component Diagram |
| :---: |

```mermaid
flowchart TD
    USER[Developer Instruction]

    subgraph CLINE[Cline Extension]
        TASK[Task Orchestrator]
        CONTEXT[Context Collector]
        CLIENT[LM Studio API Client]
        TOOLS[Tool Dispatcher]
        APPROVAL[Diff and Approval Handler]
    end

    subgraph WORKSPACE[VS Code Workspace]
        FILES[Workspace Files]
        TERMINAL[Integrated Terminal]
        GIT[Git Working Tree]
    end

    subgraph INFERENCE[Local Inference]
        API[LM Studio API]
        MODEL[Gemma 4 12B QAT]
    end

    USER --> TASK

    TASK --> CONTEXT
    CONTEXT -->|Read context| FILES
    CONTEXT -->|Return file content| TASK

    TASK -->|Prompt and context| CLIENT
    CLIENT -->|Chat completion request| API
    API --> MODEL
    MODEL -->|Agent response| API
    API --> CLIENT
    CLIENT --> TASK

    TASK --> TOOLS
    TOOLS -->|Read files| FILES
    TOOLS -->|Propose changes| APPROVAL
    TOOLS -->|Run approved commands| TERMINAL

    APPROVAL -->|Developer saves| FILES
    FILES --> GIT
    TERMINAL -->|Command output| TASK
```

Deep dive and code is available in the [design overview](https://github.com/vinas1/vibecode/blob/main/README.md).
