# Security Patterns Quick Reference

Common secure coding patterns for SDD implementations.

---

## Input Validation

### ✅ Do: Validate All Input
```python
from pydantic import BaseModel, Field, validator

class PaymentRequest(BaseModel):
    amount: int = Field(gt=0, le=1000000)  # Cents, max $10,000
    currency: str = Field(regex=r'^[A-Z]{3}$')
    idempotency_key: str = Field(min_length=16, max_length=64)
    
    @validator('currency')
    def validate_currency(cls, v):
        allowed = {'USD', 'EUR', 'GBP'}
        if v not in allowed:
            raise ValueError(f'Currency must be one of {allowed}')
        return v
```

### ❌ Don't: Trust User Input
```python
# NEVER do this
amount = request.json['amount']  # No validation
query = f"SELECT * FROM orders WHERE id = {user_input}"  # SQL injection
```

---

## Authentication & Authorization

### ✅ Do: Verify on Every Request
```python
from fastapi import Depends, HTTPException, Header

async def verify_token(authorization: str = Header(...)):
    if not authorization.startswith("Bearer "):
        raise HTTPException(401, "Invalid authorization header")
    token = authorization[7:]
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
        return payload
    except jwt.InvalidTokenError:
        raise HTTPException(401, "Invalid token")

@app.post("/pay")
async def process_payment(
    request: PaymentRequest,
    user: dict = Depends(verify_token)  # Auth on every endpoint
):
    ...
```

### ❌ Don't: Check Auth Once
```python
# NEVER do this - checking only at login
if logged_in:  # Set at login, never checked again
    process_payment()
```

---

## Secrets Management

### ✅ Do: Use Environment Variables
```python
import os

DATABASE_URL = os.environ.get("DATABASE_URL")
API_KEY = os.environ.get("PAYMENT_API_KEY")

if not DATABASE_URL or not API_KEY:
    raise RuntimeError("Required environment variables not set")
```

### ❌ Don't: Hardcode Secrets
```python
# NEVER do this
DATABASE_URL = "postgresql://admin:password123@db.example.com/prod"
API_KEY = "sk_live_abc123..."
```

---

## SQL Injection Prevention

### ✅ Do: Use Parameterized Queries
```python
# SQLAlchemy
result = session.execute(
    text("SELECT * FROM users WHERE id = :id"),
    {"id": user_id}
)

# Raw psycopg2
cursor.execute(
    "SELECT * FROM orders WHERE customer_id = %s",
    (customer_id,)
)
```

### ❌ Don't: String Concatenation
```python
# NEVER do this
query = f"SELECT * FROM users WHERE name = '{user_input}'"
cursor.execute(query)
```

---

## Sensitive Data Logging

### ✅ Do: Mask Sensitive Fields
```python
import logging

def log_payment(payment: PaymentRequest):
    logging.info(
        "Payment processed",
        extra={
            "amount": payment.amount,
            "currency": payment.currency,
            "card_last_four": payment.card_number[-4:],  # Only last 4
            # NEVER log full card_number
        }
    )
```

### ❌ Don't: Log Everything
```python
# NEVER do this
logging.info(f"Payment request: {request.json()}")  # Logs card numbers!
```

---

## Password Handling

### ✅ Do: Use Strong Hashing
```python
import bcrypt

def hash_password(password: str) -> str:
    salt = bcrypt.gensalt(rounds=12)  # Cost factor 12+
    return bcrypt.hashpw(password.encode(), salt).decode()

def verify_password(password: str, hashed: str) -> bool:
    return bcrypt.checkpw(password.encode(), hashed.encode())
```

### ❌ Don't: Use Weak Hashing
```python
# NEVER do this
import hashlib
hashed = hashlib.md5(password.encode()).hexdigest()  # MD5 is broken
hashed = hashlib.sha256(password.encode()).hexdigest()  # No salt!
```

---

## Error Handling

### ✅ Do: Return Safe Error Messages
```python
@app.exception_handler(Exception)
async def generic_exception_handler(request, exc):
    # Log full error internally
    logging.error(f"Error: {exc}", exc_info=True)
    
    # Return safe message to user
    return JSONResponse(
        status_code=500,
        content={"error": "An internal error occurred", "code": "INTERNAL_ERROR"}
    )
```

### ❌ Don't: Expose Internal Details
```python
# NEVER do this
except Exception as e:
    return {"error": str(e)}  # Exposes stack traces, paths, etc.
```

---

## Rate Limiting

### ✅ Do: Implement Rate Limits
```python
from fastapi_limiter.depends import RateLimiter

@app.post("/pay")
@app.depends(RateLimiter(times=10, seconds=60))  # 10 requests/minute
async def process_payment(request: PaymentRequest):
    ...
```

---

## CORS Configuration

### ✅ Do: Restrict Origins
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["https://myapp.com"],  # Specific origin
    allow_methods=["GET", "POST"],
    allow_headers=["Authorization", "Content-Type"],
)
```

### ❌ Don't: Allow Everything
```python
# NEVER in production
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Any origin
    allow_methods=["*"],  # Any method
)
```

---

## Security Scanning Commands

```bash
# Semgrep (SAST)
semgrep --config=auto src/

# Bandit (Python-specific)
bandit -r src/ -ll  # Medium and above

# Dependency check
pip-audit

# All together in CI
semgrep --config=auto --error src/ && \
bandit -r src/ -ll && \
pip-audit
```

---

## Security Spec Requirements

Always include in GOVERNANCE CONSTRAINTS:

```markdown
## GOVERNANCE CONSTRAINTS

### Security
- GC-SEC-001: All endpoints require authentication (Bearer token)
- GC-SEC-002: Input validation on all user-provided data
- GC-SEC-003: Passwords hashed with bcrypt (cost ≥ 12)
- GC-SEC-004: No sensitive data in logs (card numbers, passwords, SSN)
- GC-SEC-005: SQL queries use parameterized statements only
- GC-SEC-006: Rate limiting: 100 requests/minute per user
- GC-SEC-007: HTTPS only in production
```
