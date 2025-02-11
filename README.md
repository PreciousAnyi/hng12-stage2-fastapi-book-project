# FastAPI Book Management API

## Overview

This project is a RESTful API built with FastAPI for managing a book collection. It provides comprehensive CRUD (Create, Read, Update, Delete) operations for books with proper error handling, input validation, and documentation.

## Features

- 📚 Book management (CRUD operations)
- ✅ Input validation using Pydantic models
- 🔍 Enum-based genre classification
- 🧪 Complete test coverage
- 📝 API documentation (auto-generated by FastAPI)
- 🔒 CORS middleware enabled

## Project Structure

```
fastapi-book-project/
├── api/
│   ├── db/
│   │   ├── __init__.py
│   │   └── schemas.py      # Data models and in-memory database
│   ├── routes/
│   │   ├── __init__.py
│   │   └── books.py        # Book route handlers
│   └── router.py           # API router configuration
├── core/
│   ├── __init__.py
│   └── config.py           # Application settings
├── tests/
│   ├── __init__.py
│   └── test_books.py       # API endpoint tests
├── main.py                 # Application entry point
├── requirements.txt        # Project dependencies
└── README.md
```

## Technologies Used

- Python 3.12
- FastAPI
- Pydantic
- pytest
- uvicorn
- Google Cloud Platform (GCP) (VM Instance for deployment)
- NGINX (Reverse Proxy)
- Systemd (Process management)
- GitHub Actions (CI/CD automation)
- Python Virtual Environment (venv)

## Installation

1. Clone the repository:

```bash
git clone https://github.com/hng12-devbotops/fastapi-book-project.git
cd fastapi-book-project
```

2. Create a virtual environment:

```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

3. Install dependencies:

```bash
pip install -r requirements.txt
```

## Running the Application

1. Start the server:

```bash
uvicorn main:app
```

2. Access the API documentation:

- Swagger UI: http://localhost:8000/docs
- ReDoc: http://localhost:8000/redoc

## API Endpoints

### Books

- `GET /api/v1/books/` - Get all books
- `GET /api/v1/books/{book_id}` - Get a specific book
- `POST /api/v1/books/` - Create a new book
- `PUT /api/v1/books/{book_id}` - Update a book
- `DELETE /api/v1/books/{book_id}` - Delete a book

### Health Check

- `GET /healthcheck` - Check API status

## Book Schema

```json
{
  "id": 1,
  "title": "Book Title",
  "author": "Author Name",
  "publication_year": 2024,
  "genre": "Fantasy"
}
```

Available genres:

- Science Fiction
- Fantasy
- Horror
- Mystery
- Romance
- Thriller

## Running Tests

```bash
pytest
```

## Error Handling

The API includes proper error handling for:

- Non-existent books
- Invalid book IDs
- Invalid genre types
- Malformed requests

## Deployment
This project is deployed on a **Google Cloud Platform (GCP) VM Instance** with **NGINX** as a reverse proxy.

### 1. Provision a Server on GCP
- Create a new **Compute Engine VM instance** on Google Cloud.
- Select an appropriate machine type (e.g., `e2-medium (2 vCPU, 4GB RAM)`).
- Allow HTTP and HTTPS traffic in firewall settings.
- Connect to your instance via SSH.

### 2. Install Virtual Environment & Dependencies
SSH into your server:
```sh
ssh -i ~/.ssh/id_rsa_gcp your-user@your-instance-ip
```

Once inside, set up the environment:
```sh
sudo apt update && sudo apt install -y python3-venv
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### 3. Set Up Systemd for FastAPI
Create a systemd service file:
```sh
sudo nano /etc/systemd/system/fastapi.service
```

Add the following configuration:
```
[Unit]
Description=FastAPI Service
After=network.target

[Service]
User=your-user
Group=your-user
WorkingDirectory=/home/your-user/fastapi-book-project
ExecStart=/home/your-user/fastapi-book-project/venv/bin/uvicorn app.main:app --host 0.0.0.0 --port 8000
Restart=always

[Install]
WantedBy=multi-user.target
```
Save and exit, then enable the service:
```sh
sudo systemctl daemon-reload
sudo systemctl enable fastapi
sudo systemctl start fastapi
sudo systemctl status fastapi
```

### 4. Configure NGINX as a Reverse Proxy
Install NGINX:
```sh
sudo apt install -y nginx
```

Create a new NGINX configuration:
```sh
sudo nano /etc/nginx/sites-available/fastapi
```

Add this:
```
server {
    listen 80;
    server_name your-instance-ip;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
Save and exit.

Enable the configuration:
```sh
sudo ln -s /etc/nginx/sites-available/fastapi /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl restart nginx
```

### 5. Configure SSH Key for Deployment
- Generate an SSH key on your local machine:
```sh
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_gcp
```
- Copy the **public key** to your server:
```sh
cat ~/.ssh/id_rsa_gcp.pub | ssh your-user@your-instance-ip "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

Now, you can securely SSH into your server and automate deployment.

## CI/CD Pipeline
This project includes a **GitHub Actions workflow** for automatic testing and deployment:

### 1. Continuous Integration (CI)
- Runs **pytest** when a pull request is made.
- Ensures all tests pass before merging.

### 2. Continuous Deployment (CD)
- Automatically deploys to the **GCP VM Instance** on merging to `main`.
- Uses **SSH and Git Pull** to update the code and restart the service.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add AmazingFeature'`)
4. Push to branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Support

For support, please open an issue in the GitHub repository.
