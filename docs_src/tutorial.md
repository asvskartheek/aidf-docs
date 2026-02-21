# Build an AI Service — Full Tutorial

A complete, code-first walkthrough building a **Support Ticket Classifier** AI Service on AIDF from scratch. The service uses Azure OpenAI GPT-4o to classify support tickets by category, urgency, sentiment, and routes them to the correct support queue.

## Prerequisites

| Requirement | Details |
|------------|---------|
| AIDF Access | AI Engineer role in at least one Workspace + Project with Dev environment set up |
| Python 3.11 | Exact version required — matches the Dockerfile base image |
| Docker Desktop | For local build and test before deploying via Infra Hub |
| Azure CLI | `az login` — needed to pull from the private Azure Artifacts feed |
| Azure OpenAI access | A GPT-4o deployment endpoint + API key (provided via Azure App Configuration by the platform admin) |
| Reference Template | Clone or copy the `reference_template` project from the Code directory |
| VS Code | Recommended IDE; Python and Docker extensions helpful |

!!! tip
    You don't need to set up Azure OpenAI yourself. The platform admin configures all credentials in Azure App Configuration. Your service fetches them automatically via the `ConfigLoader` — you just reference the key name.

---

## Architecture Overview

Every AI Service on AIDF follows the **Strategy Design Pattern**. There are three layers:

```
Abstract_Class.py  →  aiservice.py (you write this)  →  Infra Hub / AKS
(Platform)            (Your logic)                      (Platform)
```

| Layer | File | Who Owns It | What It Does |
|-------|------|------------|--------------|
| Abstract Contract | `Abstract_Class.py` | Platform (don't modify) | Defines ~20 abstract methods every AI Service must implement. Enforces the interface. |
| Concrete Implementation | `aiservice.py` | **You** | Your business logic — the actual LLM calls, data processing, ML inference. Implements all abstract methods. |
| Config System | `environment.py` | You (thin wrapper) | Pulls all credentials and settings from Azure App Configuration. Singleton pattern. |
| Deployment Blueprint | `manifest.xml` | **You** | Tells the Manifest Engine how to build, push, and deploy your container. |
| Container | `Dockerfile` | You (template) | Packages your Flask service into a Docker image for AKS deployment. |

**Request Lifecycle:** When a client POSTs a ticket to your service:

1. Flask route receives the JSON body
2. `ModelStrategy()` is instantiated — fresh per request
3. `on_start()` — initializes timing and stores raw input
4. `execute()` — validates, preprocesses, calls GPT-4o, returns raw result
5. `on_finish()` — wraps result in the standard AIDF response envelope
6. `monitor_performance()` and `explainability_monitor()` — emit telemetry to ELK
7. Flask returns `200 + JSON` to the caller

If any step throws an exception, `on_error()` catches it and returns a structured error response.

---

## Project Structure

Start by copying the `reference_template` directory and renaming it:

```
ticket-classifier/
├── src/
│   ├── Abstract_Class.py        # Platform contract — do NOT modify
│   ├── aiservice.py             # ← YOUR main implementation file
│   ├── environment.py           # Config loader (thin wrapper)
│   ├── constants.py             # Enums: categories, urgency, queues
│   └── api_data_access_layer.py # PostgreSQL logging helpers
├── scripts/
│   └── manifest.xml             # Deployment blueprint for Manifest Engine
├── config/
│   ├── input.json               # Input payload schema
│   ├── output.json              # Expected output shape
│   ├── feedback.json            # HIL feedback schema
│   └── env.config               # Environment variables (not committed to git)
├── lib/
│   └── requirements.txt         # Python dependencies
├── auto_qa/
│   └── ticket_classifier_unit_test.py
└── Dockerfile
```

!!! warning "Convention"
    The platform's Manifest Engine expects this exact folder layout. Do not rename `src/`, `scripts/`, `config/`, or `lib/`.

---

## Step 1 — The Abstract Contract

This file comes from the platform. **You never modify it.** It defines the interface your service *must* implement.

```python title="src/Abstract_Class.py"
from abc import ABC, abstractmethod
from typing import Dict, Any, Optional


class GENAI_Abstract_Class(ABC):
    """
    Platform-enforced base class for all AIDF AI Services.
    Every concrete implementation must override all abstract methods.
    """

    # ─── Lifecycle Methods ───────────────────────────────────────────────────

    @abstractmethod
    def on_start(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        """Called at the very beginning of each request.
        Initialize timing, set instance state, return init status."""
        pass

    @abstractmethod
    def validate_input_data(self) -> bool:
        """Validate that all required fields are present and well-formed.
        Raise ValueError with a descriptive message on failure."""
        pass

    @abstractmethod
    def preprocess_input_data(self) -> Dict[str, Any]:
        """Extract, clean, and normalize the raw input payload."""
        pass

    @abstractmethod
    def execute(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        """Core business logic: LLM calls, ML inference, data processing."""
        pass

    @abstractmethod
    def on_finish(self, data: Dict[str, Any]) -> Dict[str, Any]:
        """Wrap result in the standard AIDF response envelope."""
        pass

    @abstractmethod
    def on_error(self, error_message: str, error_code: int) -> Dict[str, Any]:
        """Return a structured error response."""
        pass

    # ─── Monitoring & Observability ──────────────────────────────────────────

    @abstractmethod
    def monitor_performance(self, execution_time: float) -> None: pass

    @abstractmethod
    def explainability_monitor(self, input_data: Dict, output_data: Dict) -> None: pass

    @abstractmethod
    def alert_system(self, message: str) -> None: pass

    @abstractmethod
    def model_drift_check(self) -> None: pass

    @abstractmethod
    def data_quality_check(self, data: Any) -> bool: pass

    # ─── Compute & Caching ──────────────────────────────────────────────────

    @abstractmethod
    def compute_offload(self, payload: Any) -> Any: pass

    @abstractmethod
    def cache_check(self, key: str) -> Optional[Any]: pass

    @abstractmethod
    def cache_store(self, key: str, value: Any) -> None: pass

    # ─── Platform Hooks ─────────────────────────────────────────────────────

    @abstractmethod
    def pre_process_hook(self) -> None: pass

    @abstractmethod
    def post_process_hook(self) -> None: pass

    @abstractmethod
    def get_model_metadata(self) -> Dict[str, Any]: pass

    @abstractmethod
    def health_check(self) -> Dict[str, Any]: pass
```

!!! note "Why so many methods?"
    The platform enforces this interface so every AI Service is observable, auditable, and manageable in a uniform way — regardless of whether it's a computer vision model, an LLM chain, or a classical ML pipeline.

---

## Step 2 — Config & Environment

The platform stores all credentials in **Azure App Configuration**. Your service fetches them at startup via the `ConfigLoader` singleton. You never hardcode secrets.

```python title="src/environment.py"
import os
from dataclasses import dataclass
from ai_service_common_utils.app_configuration import get_config_keys

ENV = os.getenv("ENVIRONMENT", "Development")


class ConfigLoader:
    """Singleton that connects to Azure App Configuration once at startup."""
    _instance = None

    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            cls._instance._initialize_azure_app_config()
        return cls._instance

    def _initialize_azure_app_config(self):
        trim_prefixes = [
            "Common.Be.",
            "PaasInfra.Be.",
            "DataMarketPlace.Be.",
            "Datahub.Be.",
            "Modalfoundry.Be.",
        ]
        self.app_config_client = get_config_keys(trim_prefixes)

    def get(self, key: str, default=None):
        try:
            return self.app_config_client.get(key, default)
        except Exception:
            return os.getenv(key, default)


config_loader = ConfigLoader()


@dataclass
class Config:
    # ── Azure OpenAI ──────────────────────────────────────────────────────────
    AZURE_OPENAI_ENDPOINT:    str = config_loader.get("AZURE_OPENAI_ENDPOINT")
    AZURE_OPENAI_KEY:         str = config_loader.get("AZURE_OPENAI_KEY")
    AZURE_OPENAI_DEPLOYMENT:  str = config_loader.get("AZURE_OPENAI_DEPLOYMENT", "gpt-4o")
    AZURE_OPENAI_API_VERSION: str = config_loader.get("AZURE_OPENAI_API_VERSION", "2024-08-01-preview")

    # ── PostgreSQL ────────────────────────────────────────────────────────────
    DB_CONNECTION_STRING: str = config_loader.get("DB_CONNECTION_STRING")
    DB_SCHEMA:            str = config_loader.get("DB_SCHEMA", "aiservices")

    # ── Service Identity ──────────────────────────────────────────────────────
    AI_SERVICE_ID: str = config_loader.get("AI_SERVICE_ID", "ticket-classifier")
    WORKSPACE_ID:  str = config_loader.get("WORKSPACE_ID")
    PROJECT_ID:    str = config_loader.get("PROJECT_ID")
```

```ini title="config/env.config  ⚠ Add to .gitignore"
AZURE_APP_CONFIG_CONNECTION_STRING=Endpoint=https://your-appconfig.azconfig.io;Id=XXXX;Secret=XXXX
AZURE_OPENAI_ENDPOINT=https://your-oai-resource.openai.azure.com/
AZURE_OPENAI_KEY=your-api-key-here
AZURE_OPENAI_DEPLOYMENT=gpt-4o
AZURE_OPENAI_API_VERSION=2024-08-01-preview
DB_CONNECTION_STRING=postgresql://user:password@host:5432/aiservices_db
AI_SERVICE_ID=ticket-classifier
ENVIRONMENT=Development
FLASK_APP=src/aiservice.py
FLASK_RUN_HOST=0.0.0.0
FLASK_RUN_PORT=8002
```

!!! tip "Local Dev"
    Load env.config before starting: `export $(grep -v '^#' config/env.config | xargs) && flask run`

---

## Step 3 — Constants

Define all magic strings as `Enum` classes to prevent typos and get IDE autocomplete.

```python title="src/constants.py"
from enum import Enum


class TICKET_CLASSIFIER(Enum):
    MODULE       = "ticket_commandpattern"
    SERVICE_NAME = "support_ticket_classifier"
    VERSION      = "1.0.0"


class TICKET_CATEGORY(Enum):
    BILLING   = "Billing"
    TECHNICAL = "Technical"
    ACCOUNT   = "Account"
    SHIPPING  = "Shipping"
    PRODUCT   = "Product"
    GENERAL   = "General"


class TICKET_URGENCY(Enum):
    LOW      = "Low"
    MEDIUM   = "Medium"
    HIGH     = "High"
    CRITICAL = "Critical"


class TICKET_SENTIMENT(Enum):
    POSITIVE   = "Positive"
    NEUTRAL    = "Neutral"
    FRUSTRATED = "Frustrated"
    ANGRY      = "Angry"


class ROUTING_QUEUE(Enum):
    L1_GENERAL     = "L1-General"
    L1_BILLING     = "L1-Billing"
    L2_TECHNICAL   = "L2-Technical"
    L2_ACCOUNT     = "L2-Account"
    L3_ESCALATION  = "L3-Escalation"
    PRIORITY_QUEUE = "Priority-Queue"


class COMMAND(Enum):
    INFERENCE = "INFERENCE"
    BATCH     = "BATCH"
    HEALTH    = "HEALTH"


class ELK_INDEX(Enum):
    ELK_INDEX_NAME    = "datafactory_logs"
    SERVICE_LOG_INDEX = "ticket_classifier_logs"
```

---

## Step 4 — Data Access Layer

Handles all database interactions — logging every classification result to PostgreSQL for HIL review, analytics, and drift detection.

```python title="src/api_data_access_layer.py"
import psycopg2
from environment import Config


def postgres_connection(env: str = "stage"):
    connection = psycopg2.connect(
        Config.DB_CONNECTION_STRING,
        options=f"-c search_path={Config.DB_SCHEMA}"
    )
    return connection


def log_classification_result(
    ticket_id: str, category: str, urgency: str,
    sentiment: str, confidence: float, execution_time: float
) -> None:
    conn = None
    try:
        conn = postgres_connection()
        cursor = conn.cursor()
        cursor.execute("""
            INSERT INTO ticket_classifications
                (ticket_id, category, urgency, sentiment,
                 confidence_score, execution_time_ms, created_at)
            VALUES (%s, %s, %s, %s, %s, %s, NOW())
            ON CONFLICT (ticket_id) DO UPDATE SET
                category         = EXCLUDED.category,
                urgency          = EXCLUDED.urgency,
                sentiment        = EXCLUDED.sentiment,
                confidence_score = EXCLUDED.confidence_score,
                updated_at       = NOW()
        """, (ticket_id, category, urgency, sentiment,
              round(confidence, 4), round(execution_time * 1000, 2)))
        conn.commit()
    except Exception as e:
        if conn:
            conn.rollback()
        raise e
    finally:
        if conn:
            conn.close()
```

!!! note "DB Schema"
    The `ticket_classifications` table lives in your service's PostgreSQL schema, provisioned by the platform when you create the environment. The `DBService` entry in `manifest.xml` (Step 9) tells the platform to provision this schema automatically.

---

## Step 5 — Core Service Logic (`aiservice.py`)

This is the main file you write. It contains your `ModelStrategy` class and Flask route definitions.

### Imports & Setup

```python title="src/aiservice.py — Imports & Setup"
import time, json, hashlib, logging
from typing import Dict, Any, Optional

from flask import Flask, request, jsonify
from flask_cors import CORS
from openai import AzureOpenAI

from Abstract_Class import GENAI_Abstract_Class
from environment import Config
from constants import (
    TICKET_CLASSIFIER, TICKET_CATEGORY, TICKET_URGENCY,
    TICKET_SENTIMENT, ROUTING_QUEUE, COMMAND, ELK_INDEX
)
from api_data_access_layer import log_classification_result

logging.basicConfig(level=logging.INFO, format='%(asctime)s %(levelname)s %(message)s')
logger = logging.getLogger(__name__)

app = Flask(__name__)
CORS(app)  # Required: pipeline canvas calls services cross-origin

openai_client = AzureOpenAI(
    azure_endpoint=Config.AZURE_OPENAI_ENDPOINT,
    api_key=Config.AZURE_OPENAI_KEY,
    api_version=Config.AZURE_OPENAI_API_VERSION,
)

ROUTING_MAP = {
    ("Billing",   "Low"):      ROUTING_QUEUE.L1_BILLING.value,
    ("Billing",   "High"):     ROUTING_QUEUE.L3_ESCALATION.value,
    ("Billing",   "Critical"): ROUTING_QUEUE.PRIORITY_QUEUE.value,
    ("Technical", "Low"):      ROUTING_QUEUE.L2_TECHNICAL.value,
    ("Technical", "High"):     ROUTING_QUEUE.L3_ESCALATION.value,
    ("Technical", "Critical"): ROUTING_QUEUE.PRIORITY_QUEUE.value,
    ("Account",   "Low"):      ROUTING_QUEUE.L2_ACCOUNT.value,
    ("Account",   "Critical"): ROUTING_QUEUE.PRIORITY_QUEUE.value,
    # ... (full map covers all category × urgency combinations)
}
```

### `on_start()` — Initialize Request Context

```python title="src/aiservice.py — on_start()"
class ModelStrategy(GENAI_Abstract_Class):

    def __init__(self):
        self.start_time:      float = 0.0
        self.input_data:      Dict[str, Any] = {}
        self.processed_input: Dict[str, Any] = {}
        self.module_name:     str = TICKET_CLASSIFIER.MODULE.value

    def on_start(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        self.start_time = time.time()
        self.input_data = input_data
        ticket_id = input_data.get("input_payload", {}).get("ticket_id", {}).get("value", "unknown")
        logger.info(f"[on_start] Request received | ticket_id={ticket_id}")
        return {"status": "initialized", "ticket_id": ticket_id}
```

### `validate_input_data()` — Input Validation

```python title="src/aiservice.py — validate_input_data()"
    def validate_input_data(self) -> bool:
        payload = self.input_data.get("input_payload", {})

        command = payload.get("command", {}).get("value", "")
        if not command:
            raise ValueError("Missing required field: input_payload.command.value")
        valid_commands = [c.value for c in COMMAND]
        if command not in valid_commands:
            raise ValueError(f"Invalid command '{command}'. Must be one of: {valid_commands}")

        ticket_text = payload.get("ticket_text", {}).get("value", "")
        if not ticket_text or not ticket_text.strip():
            raise ValueError("Missing required field: input_payload.ticket_text.value cannot be empty")
        if len(ticket_text.strip()) < 10:
            raise ValueError("ticket_text must be at least 10 characters")
        if len(ticket_text) > 8000:
            raise ValueError("ticket_text exceeds maximum length of 8000 characters")

        return True
```

### `execute()` — Core Inference

```python title="src/aiservice.py — execute()"
    def execute(self, input_data: Dict[str, Any]) -> Dict[str, Any]:
        # Step 1: Validate & Preprocess
        self.validate_input_data()
        preprocessed  = self.preprocess_input_data()
        ticket_text   = preprocessed["ticket_text"]
        ticket_id     = preprocessed["ticket_id"]
        customer_tier = preprocessed["customer_tier"]

        # Step 2: Cache Check
        cache_key = hashlib.md5(ticket_text.encode()).hexdigest()
        cached = self.cache_check(cache_key)
        if cached:
            return cached

        # Step 3: Build Prompt
        tier_note = ""
        if customer_tier in ("premium", "enterprise"):
            tier_note = "\nIMPORTANT: Automatically elevate urgency by one step."

        system_prompt = """You are an expert customer support triage specialist.
Analyze the given support ticket and return a JSON object with EXACTLY these fields:
{
  "category":    one of ["Billing", "Technical", "Account", "Shipping", "Product", "General"],
  "subcategory": a brief phrase (max 5 words),
  "urgency":     one of ["Low", "Medium", "High", "Critical"],
  "sentiment":   one of ["Positive", "Neutral", "Frustrated", "Angry"],
  "confidence_score": float between 0.0 and 1.0,
  "reasoning":   one sentence explaining the classification,
  "auto_response_template": a 2-3 sentence empathetic opening response
}
Return ONLY the JSON object. No markdown fences."""

        user_prompt = f"Support Ticket (ID: {ticket_id}):\n---\n{ticket_text}\n---{tier_note}\n\nClassify this ticket now."

        # Step 4: Call Azure OpenAI
        response = openai_client.chat.completions.create(
            model=Config.AZURE_OPENAI_DEPLOYMENT,
            messages=[
                {"role": "system", "content": system_prompt},
                {"role": "user",   "content": user_prompt},
            ],
            temperature=0.1,
            max_tokens=600,
            response_format={"type": "json_object"},
        )

        # Step 5: Parse Response
        classification = json.loads(response.choices[0].message.content)
        cat  = classification.get("category",  "General")
        urg  = classification.get("urgency",   "Low")
        conf = float(classification.get("confidence_score", 0.0))

        # Step 6: Determine Routing Queue
        recommended_queue = ROUTING_MAP.get((cat, urg), ROUTING_QUEUE.L1_GENERAL.value)

        # Step 7: Persist to DB (non-fatal)
        try:
            log_classification_result(
                ticket_id=ticket_id, category=cat, urgency=urg,
                sentiment=classification.get("sentiment", "Neutral"),
                confidence=conf, execution_time=time.time() - self.start_time,
            )
        except Exception as db_err:
            logger.warning(f"[execute] DB log failed (non-fatal): {db_err}")

        result = {
            "ticket_id":              ticket_id,
            "category":               cat,
            "subcategory":            classification.get("subcategory", ""),
            "urgency":                urg,
            "sentiment":              classification.get("sentiment", "Neutral"),
            "confidence_score":       round(conf, 4),
            "reasoning":              classification.get("reasoning", ""),
            "recommended_queue":      recommended_queue,
            "auto_response_template": classification.get("auto_response_template", ""),
            "tokens_used":            response.usage.total_tokens,
            "customer_tier":          customer_tier,
        }

        if conf >= 0.85:
            self.cache_store(cache_key, result)

        return result
```

### `on_finish()` & `on_error()` — Response Envelope

```python title="src/aiservice.py — on_finish() & on_error()"
    def on_finish(self, data: Dict[str, Any]) -> Dict[str, Any]:
        execution_time = round(time.time() - self.start_time, 4)
        return {
            "execution_status": "success",
            "status_code":      200,
            "output_kpis": {
                "execution_time":    execution_time,
                "processed_records": 1,
                "tokens_used":       data.get("tokens_used", 0),
            },
            "service_output": {k: v for k, v in data.items() if k != "tokens_used"},
        }

    def on_error(self, error_message: str, error_code: int = 500) -> Dict[str, Any]:
        execution_time = round(time.time() - self.start_time, 4)
        logger.error(f"[on_error] code={error_code} | {error_message}")
        return {
            "execution_status": "failed",
            "status_code":      error_code,
            "error_message":    str(error_message),
            "output_kpis":      {"execution_time": execution_time},
        }
```

### Flask Routes

```python title="src/aiservice.py — Flask Routes"
@app.route('/api/<ai_service_id>/<deploy_env_id>/Basic_functionality', methods=['POST'])
def process_request(ai_service_id: str, deploy_env_id: str):
    Model = ModelStrategy()
    try:
        input_data = request.get_json(force=True)
        if not input_data:
            return jsonify(Model.on_error("Empty or invalid JSON body", 400)), 400

        Model.on_start(input_data)
        result   = Model.execute(input_data)
        response = Model.on_finish(result)

        Model.monitor_performance(response["output_kpis"]["execution_time"])
        Model.explainability_monitor(input_data, response)

        return jsonify(response), 200

    except ValueError as ve:
        return jsonify(Model.on_error(str(ve), error_code=422)), 422
    except Exception as e:
        logger.exception(f"[process_request] Unhandled exception: {e}")
        return jsonify(Model.on_error(str(e), error_code=500)), 500


@app.route('/health', methods=['GET'])
def health():
    return jsonify(ModelStrategy().health_check()), 200


@app.route('/metadata', methods=['GET'])
def metadata():
    return jsonify(ModelStrategy().get_model_metadata()), 200


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8002, debug=False)
```

---

## Step 6 — Input / Output Schemas

```json title="config/input.json"
{
  "input_payload": {
    "command": {
      "value": "INFERENCE",
      "data_type": "string",
      "is_mandatory": "Y",
      "default_value": "INFERENCE",
      "source_from": "runtime",
      "description": "Processing mode. INFERENCE=single ticket, BATCH=bulk, HEALTH=probe."
    },
    "ticket_text": {
      "value": "",
      "data_type": "string",
      "is_mandatory": "Y",
      "default_value": "",
      "source_from": "runtime",
      "description": "The raw text of the support ticket. Min 10 chars, max 8000 chars."
    },
    "ticket_id": {
      "value": "",
      "data_type": "string",
      "is_mandatory": "N",
      "source_from": "runtime",
      "description": "Optional unique identifier. Auto-generated if not provided."
    },
    "customer_tier": {
      "value": "standard",
      "data_type": "string",
      "is_mandatory": "N",
      "source_from": "runtime",
      "description": "standard | premium | enterprise. Premium/enterprise escalates urgency."
    }
  }
}
```

```json title="config/output.json"
{
  "execution_status": "success",
  "status_code": 200,
  "output_kpis": {
    "execution_time": 1.243,
    "processed_records": 1,
    "tokens_used": 312
  },
  "service_output": {
    "ticket_id": "TK-1738291200",
    "category": "Technical",
    "subcategory": "Login failure 2FA",
    "urgency": "High",
    "sentiment": "Frustrated",
    "confidence_score": 0.94,
    "reasoning": "User cannot access account due to 2FA failure — High urgency.",
    "recommended_queue": "L3-Escalation",
    "auto_response_template": "Thank you for reaching out about this login issue...",
    "customer_tier": "standard"
  }
}
```

---

## Step 7 — Dockerfile

```dockerfile title="Dockerfile"
FROM python:3.11
WORKDIR /code

COPY lib/requirements.txt /code/requirements.txt

ARG ARTIFACTS_PAT
RUN pip install --no-cache-dir --upgrade \
    -r requirements.txt \
    --extra-index-url "https://build:${ARTIFACTS_PAT}@pkgs.dev.azure.com/YourOrg/YourProject/_packaging/YourFeed/pypi/simple/"

COPY ./ /code/

EXPOSE 8002

ENV FLASK_APP=src/aiservice.py \
    FLASK_RUN_HOST=0.0.0.0 \
    FLASK_RUN_PORT=8002 \
    PYTHONPATH=/code/src \
    ENVIRONMENT=Production

CMD ["flask", "run"]
```

!!! warning "Security Note"
    Never bake the `ARTIFACTS_PAT` or any secret directly into the Dockerfile. The platform's CI/CD pipeline injects it from Azure Key Vault.

---

## Step 8 — Python Dependencies

```text title="lib/requirements.txt"
# Platform packages (from private Azure Artifacts Feed)
ai-service-common-utils
loop-common-utils==1.1203.0

# Web Framework
Flask==3.0.3
Flask-Cors==4.0.1

# Azure OpenAI
openai==1.40.0

# Database
psycopg2-binary==2.9.9

# Utilities
python-dotenv==1.0.1
```

| Package | Purpose |
|---------|---------|
| `ai-service-common-utils` | Platform package — provides `GENAI_Abstract_Class`, ELK logger, platform monitoring hooks |
| `loop-common-utils` | Platform package — provides `get_config_keys()` for Azure App Configuration |
| `Flask` | Lightweight WSGI web framework for the REST API |
| `Flask-Cors` | CORS headers — required for the pipeline canvas to call your service from the browser |
| `openai` | Official Azure OpenAI Python SDK |
| `psycopg2-binary` | PostgreSQL database adapter |
| `python-dotenv` | Loads `env.config` for local development |

---

## Step 9 — Deployment Blueprint (manifest.xml)

```xml title="scripts/manifest.xml"
<?xml version="1.0" encoding="UTF-8"?>
<Package>

  <ArtifactInfo>
    <Name>ticket-classifier</Name>
    <Module_Name>LLM</Module_Name>
    <Component>Classification</Component>
    <AIServiceVersion>1</AIServiceVersion>
  </ArtifactInfo>

  <Configuration>

    <Deployment
      type="Algo"
      technology="Azure"
      route="/Basic_functionality"
      cluster="shared">

      <DevSize     DR="yes">small</DevSize>
      <QaSize      DR="yes">medium</QaSize>
      <StagingSize DR="yes">medium</StagingSize>
      <ProdSize    DR="yes">large</ProdSize>
    </Deployment>

    <DBServices>
      <DBService type="SQL" technology="AzureSQL">
        <Schema>aiservices</Schema>
        <Table>ticket_classifications</Table>
      </DBService>
    </DBServices>

    <ContainerRegistry>
      <Registry>yourorg.azurecr.io</Registry>
      <Repository>ticket-classifier</Repository>
      <BuildContext>./</BuildContext>
      <DockerfilePath>./Dockerfile</DockerfilePath>
    </ContainerRegistry>

  </Configuration>

</Package>
```

| Field | What It Controls |
|-------|-----------------|
| `<Name>` | The `ai_service_id` used in the Flask URL path. Must match exactly. |
| `<Module_Name>` | How Infra Hub categorizes your service (LLM, CV, Classical ML, etc.) |
| `route="/Basic_functionality"` | Suffix of your Flask endpoint URL after `/api/{id}/{env}/` |
| `cluster="shared"` | Deploys to the shared AKS cluster. Use `"dedicated"` for high-throughput. |
| `<DBService>` | Tells the engine to provision a PostgreSQL schema + table at deployment time. |

---

## Step 10 — Unit Tests

```python title="auto_qa/ticket_classifier_unit_test.py"
import os, json, unittest, requests

BASE_URL      = os.getenv("SERVICE_BASE_URL", "http://localhost:8002")
AI_SERVICE_ID = os.getenv("AI_SERVICE_ID",    "ticket-classifier")
ENV_ID        = os.getenv("DEPLOY_ENV_ID",    "dev")
ENDPOINT      = f"{BASE_URL}/api/{AI_SERVICE_ID}/{ENV_ID}/Basic_functionality"

CONFIG_DIR = os.path.join(os.path.dirname(__file__), "..", "config")
with open(os.path.join(CONFIG_DIR, "input.json"))  as f: INPUT_SCHEMA  = json.load(f)
with open(os.path.join(CONFIG_DIR, "output.json")) as f: OUTPUT_SCHEMA = json.load(f)


class TestTicketClassifier(unittest.TestCase):

    def _call_service(self, ticket_text, ticket_id="TEST-001", customer_tier="standard"):
        payload = {
            "input_payload": {
                "command":       {"value": "INFERENCE", "data_type": "string", "is_mandatory": "Y"},
                "ticket_text":   {"value": ticket_text, "data_type": "string", "is_mandatory": "Y"},
                "ticket_id":     {"value": ticket_id,   "data_type": "string", "is_mandatory": "N"},
                "customer_tier": {"value": customer_tier, "data_type": "string", "is_mandatory": "N"},
            }
        }
        return requests.post(ENDPOINT, json=payload, timeout=30)

    def test_response_has_required_envelope_fields(self):
        resp = self._call_service("I cannot log into my account. Please help urgently.")
        self.assertEqual(resp.status_code, 200)
        body = resp.json()
        self.assertIn("execution_status", body)
        self.assertIn("service_output",   body)
        self.assertEqual(body["execution_status"], "success")

    def test_billing_ticket_classified_correctly(self):
        text = "I was charged twice for my subscription this month. Please refund."
        output = self._call_service(text, "TEST-BILLING-001").json()["service_output"]
        self.assertEqual(output["category"], "Billing")
        self.assertGreater(output["confidence_score"], 0.7)

    def test_empty_ticket_text_returns_422(self):
        resp = self._call_service("")
        self.assertEqual(resp.status_code, 422)

    def test_health_endpoint_returns_200(self):
        resp = requests.get(f"{BASE_URL}/health", timeout=10)
        self.assertEqual(resp.status_code, 200)
        self.assertEqual(resp.json()["status"], "healthy")


if __name__ == "__main__":
    unittest.main(verbosity=2)
```

---

## Step 11 — Local Testing

=== "Flask (fast iteration)"

    ```bash
    # Load environment variables
    export $(grep -v '^#' config/env.config | xargs)

    # Install dependencies
    python -m venv venv && source venv/bin/activate
    pip install -r lib/requirements.txt

    # Start the service
    flask run
    # → Running on http://0.0.0.0:8002
    ```

=== "Docker (matches production)"

    ```bash
    # Build
    docker build --build-arg ARTIFACTS_PAT="${ARTIFACTS_PAT}" -t ticket-classifier:local .

    # Run
    docker run --rm --env-file config/env.config -p 8002:8002 ticket-classifier:local
    ```

**Send a test request:**

```bash
curl -s -X POST http://localhost:8002/api/ticket-classifier/dev/Basic_functionality \
  -H "Content-Type: application/json" \
  -d '{
    "input_payload": {
      "command":       {"value": "INFERENCE", "data_type": "string", "is_mandatory": "Y"},
      "ticket_text":   {"value": "I was charged twice for my order last week and I need an immediate refund!", "data_type": "string", "is_mandatory": "Y"},
      "ticket_id":     {"value": "TK-20260220-001", "data_type": "string", "is_mandatory": "N"},
      "customer_tier": {"value": "standard", "data_type": "string", "is_mandatory": "N"}
    }
  }' | python -m json.tool
```

**Expected response:**

```json
{
  "execution_status": "success",
  "status_code": 200,
  "output_kpis": { "execution_time": 1.47, "processed_records": 1 },
  "service_output": {
    "ticket_id": "TK-20260220-001",
    "category": "Billing",
    "urgency": "High",
    "sentiment": "Angry",
    "confidence_score": 0.96,
    "recommended_queue": "L3-Escalation",
    "auto_response_template": "Thank you for bringing this billing issue to our attention..."
  }
}
```

**Run unit tests:**

```bash
python -m pytest auto_qa/ticket_classifier_unit_test.py -v
```

---

## Step 12 — Deploy via Infra Hub

1. **Navigate to Infra Hub → Your Project → AI Services**
2. **Click "Register AI Service"** — Name: `ticket-classifier`, Module: LLM, Version: 1
3. **Upload your code** — push to the connected Git repository or use file upload
4. **Trigger the Manifest Engine** — click "Build & Deploy":
    - Reads `manifest.xml` to get build parameters
    - Runs `docker build` with `ARTIFACTS_PAT` from Azure Key Vault
    - Pushes image to ACR (`yourorg.azurecr.io/ticket-classifier:v1-dev`)
    - Applies AKS deployment with `small` pod size (Dev)
    - Provisions `ticket_classifications` table in PostgreSQL
5. **Verify deployment** — wait for all pods to show Running; platform calls `/health`
6. **Run QA Tests from Infra Hub** — Go to QA view → select your service → "Run Auto QA"
7. **Promote:** Dev → QA → Staging → Production (each promotion re-deploys with appropriate pod size)

!!! success "After Deployment"
    Your service is now available to the entire platform. The production URL will be:
    `https://aidf.yourorg.com/api/ticket-classifier/prod/Basic_functionality`

---

## Step 13 — Calling the Deployed Service

=== "cURL"

    ```bash
    curl -s -X POST \
      https://aidf.yourorg.com/api/ticket-classifier/prod/Basic_functionality \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer ${PLATFORM_TOKEN}" \
      -d '{
        "input_payload": {
          "command":       {"value": "INFERENCE", "data_type": "string", "is_mandatory": "Y"},
          "ticket_text":   {"value": "My account was suspended without any warning.", "data_type": "string", "is_mandatory": "Y"},
          "customer_tier": {"value": "enterprise", "data_type": "string", "is_mandatory": "N"}
        }
      }'
    ```

=== "Python"

    ```python
    import requests

    response = requests.post(
        "https://aidf.yourorg.com/api/ticket-classifier/prod/Basic_functionality",
        headers={"Authorization": f"Bearer {platform_token}"},
        json={
            "input_payload": {
                "command":     {"value": "INFERENCE", "data_type": "string", "is_mandatory": "Y"},
                "ticket_text": {"value": "My account was suspended.", "data_type": "string", "is_mandatory": "Y"},
            }
        }
    )
    result = response.json()
    print(result["service_output"]["category"])      # "Account"
    print(result["service_output"]["urgency"])        # "High"
    print(result["service_output"]["recommended_queue"])  # "L3-Escalation"
    ```
