# Simple Social

A small social media app built with FastAPI, Streamlit, and ImageKit. Users register and log in, upload photos or videos with a caption, view a shared feed, and delete their own posts.

## Features

- Email and password registration and login with JWT bearer tokens
- Upload images and videos, stored on ImageKit
- Shared feed with captions, author emails, and post dates
- Caption overlay on images and videos through ImageKit transformations
- Delete your own posts
- Streamlit frontend that talks to the FastAPI backend

## Tech Stack

- **Backend:** FastAPI, Uvicorn, SQLAlchemy (async), aiosqlite
- **Authentication:** fastapi-users with JWT bearer tokens
- **Media storage:** ImageKit (`imagekitio` 4.x)
- **Frontend:** Streamlit and Requests
- **Database:** SQLite (`test.db`)
- **Package management:** uv, Python 3.14

## Project Structure

```
.
├── main.py            # Starts the FastAPI server with Uvicorn
├── frontend.py        # Streamlit user interface
├── app/
│   ├── app.py         # FastAPI routes: upload, feed, delete
│   ├── db.py          # Database engine, models, and session setup
│   ├── users.py       # User manager, JWT backend, and auth dependency
│   ├── schemas.py     # Pydantic schemas for users
│   └── images.py      # ImageKit client setup
├── pyproject.toml     # Project metadata and dependencies
└── uv.lock            # Locked dependency versions
```

## Prerequisites

- [uv](https://docs.astral.sh/uv/getting-started/installation/)
- Python 3.14 (uv installs it if it is missing)
- An [ImageKit](https://imagekit.io/) account with a private key, a public key, and a URL endpoint

## Setup

1. Clone the repository and open the project folder.

2. Install the dependencies:

   ```bash
   uv sync
   ```

3. Create a `.env` file in the project root with your ImageKit values:

   ```env
   IMAGEKIT_PRIVATE_KEY=your_private_key
   IMAGEKIT_PUBLIC_KEY=your_public_key
   IMAGEKIT_URL=https://ik.imagekit.io/your_imagekit_id
   ```

   The `.env` file is listed in `.gitignore`. Do not commit it.

4. Before deploying, change `SECRET` in `app/users.py` to a long random value. The value is hardcoded in the repository for development only.

## Running the App

The backend and frontend run as two separate processes. Start the backend first.

**Terminal 1: FastAPI backend (port 8000)**

```bash
uv run .\main.py
```

**Terminal 2: Streamlit frontend (port 8501)**

```bash
uv run streamlit run frontend.py
```

Then open:

- Application: http://localhost:8501
- API documentation (Swagger UI): http://localhost:8000/docs

The database tables are created automatically when the backend starts. The database file is `test.db` in the project root.

## API Endpoints

| Method | Path | Auth | Description |
| --- | --- | --- | --- |
| POST | `/auth/register` | No | Create an account with `email` and `password` (JSON) |
| POST | `/auth/jwt/login` | No | Log in with form fields `username` (the email) and `password`. Returns `access_token` |
| GET | `/users/me` | Yes | Return the logged-in user |
| POST | `/upload` | Yes | Upload a file (`file`) with an optional `caption` (form data) |
| GET | `/feed` | Yes | List all posts, newest first |
| DELETE | `/posts/{post_id}` | Yes | Delete a post. Only the owner can delete it |

Send the token on protected routes:

```
Authorization: Bearer <access_token>
```

### Example: Log in and read the feed

```bash
curl -X POST http://localhost:8000/auth/jwt/login -H "Content-Type: application/x-www-form-urlencoded" -d "username=you@example.com&password=yourpassword"

curl http://localhost:8000/feed -H "Authorization: Bearer PASTE_TOKEN_HERE"
```

## Troubleshooting

- **`{"detail": "Unauthorized"}`**: the request has no valid token. Log in and send the `access_token` as a bearer token.
- **Internal Server Error (500) on `/feed` after a model change**: `create_all` does not change existing tables. Stop the server, delete `test.db`, and restart. This deletes all users and posts.
- **Frontend shows "Failed to load feed" or login errors**: the backend is not running on port 8000. Start it first.
- **Upload fails**: check that the three `IMAGEKIT_*` values in `.env` are correct and that the backend was restarted after you changed them.

## Notes

- The app uses SQLite and a local file-based setup. It is meant for learning and local testing, not production.
- Uploaded files are stored on ImageKit. Only the URL and metadata are saved in the database.
