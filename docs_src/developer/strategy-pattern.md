# Strategy Design Pattern

All AI Services implement a **strategy design pattern** combining abstract base classes with factory logic to produce consistent, modular, extensible service components.

## Three Layers

### Layer 1: Abstract Base Class

The contract / skeleton. Defines *what* every service must implement:

- Pre-defined abstract methods for ingestion, preprocessing, and target operations
- Acts as a backbone — any service must implement these methods
- Example: `DF_PRP_Abstract_Class.py`

```python
class AbstractAIService:
    @abstractmethod
    def ingest(self, payload): ...

    @abstractmethod
    def preprocess(self, data): ...

    @abstractmethod
    def execute(self, data): ...

    @abstractmethod
    def on_finish(self, result): ...

    @abstractmethod
    def on_error(self, error): ...
```

### Layer 2: Concrete Implementation

Fills in the AI-specific logic for each method. Each subclass only implements the template methods for its specific task. May include extra helper functions (e.g., secret-vault connectors, storage clients).

```python
class SuspiciousActivityDetectionService(AbstractAIService):
    def ingest(self, payload):
        # Download person tracks JSON from Azure Blob
        ...
    def execute(self, data):
        # Analyze keypoints and classify occlusion levels
        ...
```

### Layer 3: Context/Strategy Class ("Predefined Flow")

Orchestrates the lifecycle. Acts as a factory that:

1. Reads configuration to determine which concrete class to instantiate
2. Creates the object and invokes methods in order: `execute → on_finish` (or `on_error`)
3. Defines the HTTP route and registers the endpoint
4. Wraps every function in try/except blocks to properly log and raise errors

```python
class AIServiceContext:
    def __init__(self, config):
        self.service = ConcreteServiceFactory.create(config)

    def run(self, payload):
        try:
            data = self.service.ingest(payload)
            result = self.service.execute(data)
            return self.service.on_finish(result)
        except Exception as e:
            return self.service.on_error(e)

# HTTP endpoint registration
@app.route("/datafactory/preprocessing/suspicious_activity", methods=["POST"])
def handle():
    return AIServiceContext(config).run(request.json)
```

## Extending the Pattern

Adding a new AI Service capability:

1. Create a new concrete class extending the abstract base
2. Register it in the context class factory
3. No changes needed to existing logic or other services
