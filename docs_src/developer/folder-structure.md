# AI Service Folder Structure

Every AI Service follows this standardized folder structure:

```
my_ai_service/
├── auto_qa/           # Unit test code and test data
│   └── service_unit_test.py
├── config/            # Configuration files and sample payloads
│   ├── input.json     # Input schema + UI configuration
│   ├── output.json    # Expected output sample
│   ├── feedback.json  # HIL review metadata
│   └── env.config     # Environment-specific settings
├── doc/               # Human-friendly documentation
│   └── note.txt       # Service overview, design rationale
├── lib/               # Python dependencies
│   └── requirements.txt
├── scripts/           # Deployment manifests
│   └── manifest.xml   # Deployment blueprint
├── src/               # Core source code
│   ├── Abstract_Class.py      # Service contract / skeleton
│   ├── Concrete_Class.py      # Service-specific logic
│   ├── aiservice.py           # Context class / HTTP route
│   ├── environment.py         # Config management
│   └── constants.py
├── unit_testing/      # Future integration/e2e tests
├── Dockerfile
└── .gitignore
```

## Folder Details

### `auto_qa/`

Contains all code and data needed for automated unit testing. Allows developers to run checks locally to catch regressions before code leaves the workstation. Validates inputs, outputs, and error handling as part of every build.

### `config/`

Contains configuration files and sample payloads. Key files:

- **`input.json`** — Defines the structure, default values, and UI behavior of all input parameters. Contains `input_payload` (key-value pairs) and `payload_setup` (UI configuration: dropdowns, textboxes, validations). Used to auto-generate parameter forms in GenAI Studio and enable low-code pipeline setup.
- **`output.json`** — Sample of the AI service's response after successful execution. Includes status, message, KPIs (execution time, processed records), and output metadata.
- **`feedback.json`** — Supports human-in-the-loop review by capturing reviewer comments, required adjustments, and approval metadata.

### `doc/`

High-level, human-friendly documentation about the service. Onboarding notes, design rationale, troubleshooting tips. Versioned alongside source code for seamless knowledge transfer.

### `lib/`

All external dependencies as `requirements.txt`. Running `pip install -r lib/requirements.txt` instantly sets up the exact environment. Avoids version conflicts and ensures reproducible builds.

### `scripts/`

Deployment assets and orchestration definitions. The `manifest.xml` specifies how the service is packaged, where it runs, and which endpoints it exposes. Evolves alongside the code in version control.

### `src/`

Core of the service: abstract classes, concrete implementations, and context/strategy classes. Also defines the HTTP route and integrates with config, secret stores, and logging.

### `unit_testing/`

Currently a placeholder. Reserved for broader integration tests, smoke tests, performance benchmarks, and API integration scenarios.
