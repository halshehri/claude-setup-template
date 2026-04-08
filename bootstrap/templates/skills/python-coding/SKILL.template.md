---
name: python-coding
description: Apply Python coding standards when writing backend code, APIs, or scripts. Use when implementing features in Python services, FastAPI, Django, or Flask applications.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Python Coding Standards

Apply these standards when writing Python code.

## Project Structure

```
src/
├── app/
│   ├── __init__.py
│   ├── main.py              # Entry point
│   ├── config.py            # Configuration
│   ├── dependencies.py      # Dependency injection
│   ├── routers/
│   │   ├── __init__.py
│   │   └── {resource}.py
│   ├── services/
│   │   ├── __init__.py
│   │   └── {resource}.py
│   ├── repositories/
│   │   ├── __init__.py
│   │   └── {resource}.py
│   ├── models/
│   │   ├── __init__.py
│   │   └── {resource}.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   └── {resource}.py
│   └── utils/
│       └── __init__.py
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   └── test_{module}.py
├── pyproject.toml
└── README.md
```

## Code Conventions

### Naming
- **Files/Modules**: `snake_case.py`
- **Classes**: `PascalCase`
- **Functions/Variables**: `snake_case`
- **Constants**: `SCREAMING_SNAKE_CASE`
- **Private**: `_leading_underscore`

### Type Hints

Always use type hints:

```python
from typing import Optional, List, Dict, Any
from datetime import datetime

def get_user(user_id: str) -> Optional[User]:
    """Retrieve a user by ID."""
    return user_repository.find_by_id(user_id)

def create_users(data: List[CreateUserDto]) -> List[User]:
    """Create multiple users."""
    return [create_user(d) for d in data]

async def fetch_data(url: str, params: Dict[str, Any] | None = None) -> dict:
    """Fetch data from external API."""
    async with httpx.AsyncClient() as client:
        response = await client.get(url, params=params)
        return response.json()
```

### Pydantic Models (FastAPI)

```python
from pydantic import BaseModel, Field, EmailStr
from datetime import datetime
from enum import Enum

class UserRole(str, Enum):
    USER = "user"
    ADMIN = "admin"

class UserBase(BaseModel):
    email: EmailStr
    name: str = Field(..., min_length=1, max_length=100)
    role: UserRole = UserRole.USER

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserResponse(UserBase):
    id: str
    created_at: datetime

    class Config:
        from_attributes = True  # For ORM compatibility

class UserListResponse(BaseModel):
    data: List[UserResponse]
    total: int
    page: int
    limit: int
```

### Error Handling

Custom exceptions:

```python
# app/exceptions.py
from fastapi import HTTPException, status

class AppException(Exception):
    """Base application exception."""
    def __init__(
        self,
        message: str,
        code: str = "INTERNAL_ERROR",
        status_code: int = status.HTTP_500_INTERNAL_SERVER_ERROR,
    ):
        self.message = message
        self.code = code
        self.status_code = status_code
        super().__init__(message)

class NotFoundError(AppException):
    def __init__(self, message: str):
        super().__init__(message, "NOT_FOUND", status.HTTP_404_NOT_FOUND)

class ValidationError(AppException):
    def __init__(self, message: str, details: dict | None = None):
        super().__init__(message, "VALIDATION_ERROR", status.HTTP_400_BAD_REQUEST)
        self.details = details

class UnauthorizedError(AppException):
    def __init__(self, message: str = "Unauthorized"):
        super().__init__(message, "UNAUTHORIZED", status.HTTP_401_UNAUTHORIZED)
```

Exception handler:

```python
# app/main.py
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

app = FastAPI()

@app.exception_handler(AppException)
async def app_exception_handler(request: Request, exc: AppException):
    return JSONResponse(
        status_code=exc.status_code,
        content={
            "error": {
                "code": exc.code,
                "message": exc.message,
            }
        },
    )
```

### Dependency Injection (FastAPI)

```python
# app/dependencies.py
from functools import lru_cache
from typing import Annotated
from fastapi import Depends
from sqlalchemy.orm import Session

from app.config import Settings
from app.database import SessionLocal
from app.repositories.user import UserRepository
from app.services.user import UserService

@lru_cache
def get_settings() -> Settings:
    return Settings()

def get_db() -> Generator[Session, None, None]:
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()

def get_user_repository(
    db: Annotated[Session, Depends(get_db)]
) -> UserRepository:
    return UserRepository(db)

def get_user_service(
    repo: Annotated[UserRepository, Depends(get_user_repository)]
) -> UserService:
    return UserService(repo)

# Usage in router
@router.get("/users/{user_id}")
async def get_user(
    user_id: str,
    user_service: Annotated[UserService, Depends(get_user_service)]
) -> UserResponse:
    return user_service.get_user(user_id)
```

### Async Patterns

```python
# Prefer async for I/O operations
async def fetch_user_with_orders(user_id: str) -> UserWithOrders:
    # Run independent queries concurrently
    user, orders = await asyncio.gather(
        user_repository.find_by_id(user_id),
        order_repository.find_by_user(user_id),
    )

    if not user:
        raise NotFoundError(f"User {user_id} not found")

    return UserWithOrders(user=user, orders=orders)

# Use async context managers
async def make_request(url: str) -> dict:
    async with httpx.AsyncClient() as client:
        response = await client.get(url)
        response.raise_for_status()
        return response.json()
```

### Configuration

```python
# app/config.py
from pydantic_settings import BaseSettings
from functools import lru_cache

class Settings(BaseSettings):
    # Database
    database_url: str

    # API
    api_prefix: str = "/api/v1"
    debug: bool = False

    # Auth
    secret_key: str
    access_token_expire_minutes: int = 30

    # External services
    smtp_host: str | None = None
    smtp_port: int = 587

    class Config:
        env_file = ".env"
        env_file_encoding = "utf-8"

@lru_cache
def get_settings() -> Settings:
    return Settings()
```

### SQLAlchemy Models

```python
# app/models/user.py
from sqlalchemy import Column, String, DateTime, Enum
from sqlalchemy.orm import relationship
from datetime import datetime
import enum

from app.database import Base

class UserRole(enum.Enum):
    USER = "user"
    ADMIN = "admin"

class User(Base):
    __tablename__ = "users"

    id = Column(String, primary_key=True)
    email = Column(String, unique=True, nullable=False, index=True)
    name = Column(String(100), nullable=False)
    hashed_password = Column(String, nullable=False)
    role = Column(Enum(UserRole), default=UserRole.USER)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)

    # Relationships
    orders = relationship("Order", back_populates="user")
```

### Repository Pattern

```python
# app/repositories/user.py
from sqlalchemy.orm import Session
from sqlalchemy import select
from typing import Optional, List

from app.models.user import User
from app.schemas.user import UserCreate

class UserRepository:
    def __init__(self, db: Session):
        self.db = db

    def find_by_id(self, user_id: str) -> Optional[User]:
        return self.db.get(User, user_id)

    def find_by_email(self, email: str) -> Optional[User]:
        stmt = select(User).where(User.email == email)
        return self.db.execute(stmt).scalar_one_or_none()

    def find_all(self, skip: int = 0, limit: int = 100) -> List[User]:
        stmt = select(User).offset(skip).limit(limit)
        return list(self.db.execute(stmt).scalars())

    def create(self, data: UserCreate) -> User:
        user = User(**data.model_dump())
        self.db.add(user)
        self.db.commit()
        self.db.refresh(user)
        return user

    def delete(self, user: User) -> None:
        self.db.delete(user)
        self.db.commit()
```

### Logging

```python
# app/logging.py
import structlog
import logging

def setup_logging(debug: bool = False):
    structlog.configure(
        processors=[
            structlog.contextvars.merge_contextvars,
            structlog.processors.add_log_level,
            structlog.processors.TimeStamper(fmt="iso"),
            structlog.dev.ConsoleRenderer() if debug
            else structlog.processors.JSONRenderer(),
        ],
        wrapper_class=structlog.make_filtering_bound_logger(
            logging.DEBUG if debug else logging.INFO
        ),
    )

logger = structlog.get_logger()

# Usage
logger.info("user_created", user_id=user.id, email=user.email)
logger.error("request_failed", error=str(e), request_id=request_id)
```

### Testing

```python
# tests/conftest.py
import pytest
from fastapi.testclient import TestClient
from sqlalchemy import create_engine
from sqlalchemy.orm import sessionmaker

from app.main import app
from app.database import Base, get_db

SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"
engine = create_engine(SQLALCHEMY_DATABASE_URL)
TestingSessionLocal = sessionmaker(bind=engine)

@pytest.fixture
def db():
    Base.metadata.create_all(bind=engine)
    db = TestingSessionLocal()
    try:
        yield db
    finally:
        db.close()
        Base.metadata.drop_all(bind=engine)

@pytest.fixture
def client(db):
    def override_get_db():
        yield db
    app.dependency_overrides[get_db] = override_get_db
    yield TestClient(app)
    app.dependency_overrides.clear()

# tests/test_users.py
def test_create_user(client):
    response = client.post(
        "/api/v1/users",
        json={"email": "test@example.com", "name": "Test", "password": "password123"}
    )
    assert response.status_code == 201
    data = response.json()
    assert data["email"] == "test@example.com"

def test_get_user_not_found(client):
    response = client.get("/api/v1/users/nonexistent")
    assert response.status_code == 404
    assert response.json()["error"]["code"] == "NOT_FOUND"
```

## Anti-Patterns to Avoid

- **Mutable default arguments**: Use `None` and set inside function
- **Bare except**: Always specify exception type
- **God classes**: Keep classes focused
- **String concatenation for SQL**: Use parameterized queries
- **Ignoring type hints**: Add types everywhere
- **print() for logging**: Use proper logging
- **Hardcoded secrets**: Use environment variables
