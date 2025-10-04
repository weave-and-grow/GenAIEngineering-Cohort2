# Low Level Design Document for the Time Management App

## Overview

This document outlines the low-level design for the Time Management App, which employs FastAPI for the backend, Streamlit for the frontend, and PostgreSQL for database management. Our primary goal is to ensure that this design fosters a scalable, maintainable, and robust architecture while adhering to best practices throughout.

## 1. Module Organization

### 1.1 Project Structure

```plaintext
time_management_app/
|-- backend/
|   |-- main.py
|   |-- app/
|       |-- api/
|           |-- v1/
|               |-- routes.py
|               |-- schemas.py
|               |-- services.py
|               |-- dependencies.py
|       |-- core/
|           |-- config.py
|           |-- security.py
|       |-- models/
|           |-- user.py
|           |-- task.py
|       |-- db/
|           |-- session.py
|           |-- models.py
|       |-- tests/
|           |-- test_routes.py
|-- frontend/
|   |-- streamlit_app.py
|-- scripts/
|-- README.md
|-- requirements.txt
```

## 2. Class Diagrams

### 2.1 User Class Diagram

```plaintext
+----------------+
|     User       |
+----------------+
| - id: int      |
| - username: str|
| - email: str   |
| - password_hash: str|
+----------------+
| + create_user()|
| + verify_password()|
+----------------+
```

### 2.2 Task Class Diagram

```plaintext
+----------------+
|     Task       |
+----------------+
| - id: int      |
| - title: str   |
| - description: str|
| - user_id: int |
| - due_date: datetime|
| - completed: bool|
+----------------+
| + create_task()|
| + update_task()|
| + delete_task()|
| + mark_completed()|
+----------------+
```

## 3. Database Schema

### 3.1 PostgreSQL Schema

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE tasks (
    id SERIAL PRIMARY KEY,
    title VARCHAR(100) NOT NULL,
    description TEXT,
    user_id INT REFERENCES users(id) ON DELETE CASCADE,
    due_date TIMESTAMP,
    completed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_user_id ON tasks(user_id);
```

### 3.2 Constraints and Indexes

- Unique constraints on `username` and `email` in the `users` table.
- Foreign key constraint on `user_id` in the `tasks` table pointing to `users(id)`.
- An index on `tasks(user_id)` for faster query operations based on user tasks.

## 4. FastAPI Route Implementations

### 4.1 Routes (routes.py)

```python
from fastapi import APIRouter, Depends, HTTPException
from .schemas import UserCreate, TaskCreate, UserOut, TaskOut
from .services import create_user, get_tasks, create_task
from .dependencies import get_current_user

router = APIRouter()

@router.post("/users/", response_model=UserOut)
async def register_user(user: UserCreate):
    return await create_user(user)

@router.get("/tasks/", response_model=List[TaskOut])
async def read_tasks(current_user: User = Depends(get_current_user)):
    return await get_tasks(current_user.id)

@router.post("/tasks/", response_model=TaskOut)
async def add_task(task: TaskCreate, current_user: User = Depends(get_current_user)):
    return await create_task(task, current_user.id)
```

### 4.2 Dependency Injection (dependencies.py)

```python
from fastapi import Depends, HTTPException
from sqlalchemy.orm import Session
from .db.session import get_db
from .models.user import User

async def get_current_user(token: str, db: Session = Depends(get_db)) -> User:
    user = await verify_token(token, db)
    if user is None:
        raise HTTPException(status_code=401, detail="Invalid token")
    return user
```

## 5. Streamlit Component Design

### 5.1 Frontend Structure (streamlit_app.py)

```python
import streamlit as st
import requests

# Title of the app
st.title("Time Management App")

# User Registration Form
with st.form("user_registration"):
    username = st.text_input("Username")
    email = st.text_input("Email")
    password = st.text_input("Password", type="password")
    submitted = st.form_submit_button("Register")
    if submitted:
        response = requests.post("http://localhost:8000/users/", json={
            "username": username,
            "email": email,
            "password": password
        })
        if response.ok:
            st.success("User registered successfully!")
        else:
            st.error("Registration failed!")

# Task Management
if st.button("Get My Tasks"):
    tasks = requests.get("http://localhost:8000/tasks/")
    for task in tasks.json():
        st.write(f"- {task['title']}")
```

## 6. Coding Standards

1. **Python Version**: Ensure compatibility with Python 3.8 or higher.
2. **PEP 8 Compliance**: Follow PEP 8 coding style conventions for Python code.
3. **Documentation**: Code should be thoroughly commented, and docstrings should be used for all public classes and methods.
4. **Error Handling**: Utilize FastAPI's exception handling for a consistent error response structure.
5. **Testing**: Implement unit tests and integration tests using `pytest`.

---

This low-level design document specifies the detailed architecture, database schema, APIs, and frontend interactions for the Time Management App. Compliance with the outlined coding standards will facilitate smooth development and ensure maintainable code.