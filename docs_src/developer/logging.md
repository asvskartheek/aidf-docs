# Logging

AI Services in AIDF support two complementary log types:

## 1. Verbose Logs (Structured)

Structured, high-level execution entries stored in the database (`AI_SERVICE_EXECUTION_LOGS`) and visible in the pipeline UI. Each entry includes:

- Timestamps for start/end of each step
- Total duration
- Number of input/output records
- Status (success/failure)
- Error messages, if any

Useful for tracking service performance, auditing, and reviewing processed data summaries.

## 2. Console Logs (Real-time)

Real-time messages printed to terminal/log stream for development and debugging:

```python
logger.info("TTS model loaded successfully")        # High-level milestones
logger.debug(f"Intermediate value: {variable}")      # Debug variables
logger.error(f"Exception: {str(e)}")                 # Error traces
```

!!! tip "Rule"
    Every function must be wrapped in a `try/except` block. Log exceptions with `logger.error()` before raising or returning an error response.
