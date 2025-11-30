FASTAPI, UVICORN, PYDANTIC, ASGI vs WSGI – QUICK NOTES

1. What Is FastAPI

FastAPI is a modern, high-performance Python web framework for building APIs.

It is used for:
- REST APIs
- AI / ML backends
- GenAI APIs
- Microservices
- Model inference services

Key features:
- Very fast (close to Node.js & Go)
- Async by default
- Automatic Swagger / OpenAPI docs
- Built-in request validation
- Easy to use and production-ready

Example:
from fastapi import FastAPI

app = FastAPI()

@app.get("/hello")
def hello():
    return {"message": "Hello World"}

--------------------------------------------------

2. What Is Uvicorn and Why It Is Needed

FastAPI is only a framework. It cannot run by itself.

Uvicorn is the ASGI server that:
- Listens for HTTP requests
- Manages client connections
- Runs the FastAPI app

Run command:
uvicorn main:app --reload

Meaning:
- main = file name
- app = FastAPI instance
- --reload = auto-restart on code changes

Simple meaning:
FastAPI = API logic  
Uvicorn = Engine that runs the API

--------------------------------------------------

3. What Is Pydantic and Why FastAPI Uses It

Pydantic is a data validation and type enforcement library.

It is used to:
- Validate incoming JSON requests
- Enforce correct data types
- Convert JSON into Python objects
- Reject invalid input automatically
- Generate API documentation schemas

Example:

from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

If age is not a number → request is rejected  
If name is missing → request is rejected  

Why FastAPI depends on Pydantic:
- Safe API inputs
- No manual validation code
- Automatic OpenAPI docs
- Production-safe request handling

--------------------------------------------------

4. How FastAPI, Uvicorn, and Pydantic Work Together

Client → Uvicorn → FastAPI → Pydantic → API Logic → Response

Uvicorn handles the HTTP connection  
FastAPI handles routing and logic  
Pydantic validates and parses request data  

--------------------------------------------------

5. What Is WSGI

WSGI (Web Server Gateway Interface) is the old Python web standard.

It is:
- Synchronous
- Blocking
- Request-response only
- One request blocks one worker

Used by:
- Flask (default)
- Old Django
- Pyramid

WSGI servers:
- Gunicorn (WSGI mode)
- uWSGI
- Waitress

--------------------------------------------------

6. What Is ASGI

ASGI (Asynchronous Server Gateway Interface) is the modern replacement for WSGI.

It supports:
- Async and sync code
- WebSockets
- Streaming
- Background tasks
- Long-lived connections
- High concurrency

Used by:
- FastAPI
- Starlette
- Django (ASGI mode)
- Sanic

ASGI servers:
- Uvicorn
- Hypercorn
- Daphne

--------------------------------------------------

7. Core Difference: ASGI vs WSGI

WSGI:
- Blocking
- One request per worker
- No WebSockets
- Poor streaming
- Lower concurrency

ASGI:
- Non-blocking
- Many requests per worker
- WebSockets supported
- Built-in streaming
- High concurrency

--------------------------------------------------

8. Why FastAPI Is Preferred for AI and GenAI APIs

FastAPI is preferred because:

- AI APIs are I/O heavy (API calls, vector DB, streaming)
- FastAPI is async and non-blocking
- Can handle thousands of concurrent AI requests
- Native token streaming support for chat responses
- Pydantic enforces strict request validation
- Automatic API docs for rapid AI iteration
- Very high performance
- Direct integration with Python AI stack:
  - PyTorch
  - TensorFlow
  - HuggingFace
  - LangChain
  - LlamaIndex
  - OpenAI SDKs
- Works perfectly with:
  - Docker
  - Kubernetes
  - GPUs
  - Background task systems

--------------------------------------------------

9. Why Flask or Django Are Less Ideal for GenAI

Flask:
- Blocking by default
- Weak async support
- Poor streaming at scale
- Manual validation required

Django:
- Heavy framework
- Slower startup
- Overkill for AI microservices

FastAPI:
- Lightweight
- Async-first
- Built for modern APIs
- Ideal for AI microservices

--------------------------------------------------

10. One-Line Final Summary

FastAPI builds high-performance AI APIs, Uvicorn runs them using ASGI, and Pydantic ensures all AI request data is validated safely at production level, while ASGI enables massive concurrency and real-time streaming.
