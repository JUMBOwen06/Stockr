# Stockr
A Store Item Tracker

# stockr
### A Store Item Tracker

Stockr is a personal inventory management system designed for small stores. 
It allows management to track items, monitor sales, and manage orders, while 
giving customers a read only view of current inventory.

This project is being built as a full stack learning exercise, progressing 
through three deliberate phases from a terminal application to a complete 
web interface.


## Phases

**Phase 1 — Terminal Application** *(In Progress)*
A fully functional terminal based app built with Python and SQLite. 
Focuses on sound data logic and clean database structure before 
any interface work begins.

**Phase 2 — REST API**
A FastAPI backend exposing the core logic as API endpoints, tested 
in isolation using Postman before any frontend is connected.

**Phase 3 — Web Interface**
A locally hosted web interface built with React and Tailwind CSS, 
connected to the Phase 2 API.


## Tech Stack

| Phase | Tools |
|---|---|
| Phase 1 | Python, SQLite, SQLAlchemy, Rich, Typer |
| Phase 2 | FastAPI, Postman |
| Phase 3 | React + Vite, Tailwind CSS |

---

## Project Structure
stockr/
    >-- app/
    |   >-- main.py
    |   >-- models.py
    |   >-- database.py
    |   >-- crud.py
    |   |-- display.py
    |
    >-- data/
    |   |-- store.db
    |
    >-- docs/
    |   |-- Conception_and_Planning.pdf
    |
    >-- tests/
    |   |-- test_crud.py
    |
    |-- requirements.txt
    |-- .gitignore

## Documentation

A full conception and planning document covering the project scope, 
target users, features, requirements, and tech stack decisions is 
available in the `/docs` folder.

## Author

Owen Fugger
owenfugger@icloud.com