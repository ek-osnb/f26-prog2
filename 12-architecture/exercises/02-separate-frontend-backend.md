# Exercise: Separate Frontend and Backend with nginx reverse proxy

Most of your projects use a single Spring Boot application that serves both the frontend (inside the `src/main/resources/static` folder) and backend. This is fine for small projects, but as your application grows, it can become difficult to manage and scale. In this exercise, you will learn how to separate the frontend and backend into two different applications.


### Exercise 1: Move the frontend to a separate application

In this exercise you will move the frontend code into a separate folder. You will end up with something similar to this:

```
my-project/
├─ .git/
├─ .github/
├─ README.md
├─ frontend/
│  ├─ index.html
│  ├─ styles.css
│  └─ app.js
│  └─ ...
├─ backend/
│  ├─ pom.xml
│  ├─ Dockerfile
│  └─ src/
├─ compose.yml
└─ .env
```
> Notice that the `.git` folder is in the root of the project, so both the frontend and backend are part of the same git repository. This way you can still manage both applications together, but they are separated in terms of code structure.

1. Create a new folder with the same name as your project (e.g., `kinoxp`).
2. Move the the whole project folder into the newly created folder, and rename it to `backend`.
3. Create a new folder called `frontend` in the root of the project.
4. Move the frontend files  from `backend/src/main/resources/static` to the `frontend` folder.
5. Move `.git`, `README.md`, `compose.yml`, `.github` and any other files that are not part of the backend code to the root of the project (if they are not already there). Your sample `.env` file should also be moved to the root folder.
6. After moving the files, run your backend application and make sure everything is working as expected. You might need to update the paths to the frontend files in your backend code (e.g., in your controllers or configuration).

### Exercise 2: Nginx reverse proxy and Docker

Right now, you will have two separate applications runnning on different ports (e.g., backend on `localhost:8080` and frontend on `localhost:5500`). This will give us CORS issues when the frontend tries to make API calls to the backend. To solve this, we can use Nginx as a reverse proxy to route requests to the correct application based on the URL path.

1. Create a `Dockerfile` for the frontend application:

```dockerfile
FROM nginx:alpine
COPY . /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
```
2. Create an `nginx.conf` file in the `/frontend` folder with the following content:

```nginx
server {
    listen 80;

    location / {
        root /usr/share/nginx/html;
        index index.html;
    }

    location /api/ {
        proxy_pass http://app:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```
> This configuration tells Nginx to serve the frontend files for any request to the root path (`/`), and to proxy any requests to `/api/` to the backend application running on `http://app:8080/` (The `app` hostname will be resolved by Docker Compose, and should match the service name defined in the compose file).

3. Update your `compose.yml` file to include the frontend service and the Nginx configuration:

```yaml
services:
  db:
    image: mysql:latest
    # Same as before...
  app: # should match the name in the nginx.conf file
    build: ./backend
    ports:
      - "127.0.0.1:8080:8080" # Only expose the backend to localhost

  frontend:
    build: ./frontend
    ports:
      - "80:80"
```

4. After updating the `compose.yml` file, run `docker compose up --build` to start both the frontend and backend applications. You should now be able to access the frontend at `http://localhost/` and make API calls to the backend without any CORS issues.

## Troubleshooting:
- Make sure that the paths in your backend use `/api` prefix for the API endpoints, so that Nginx can route the requests correctly.
- Make sure that your frontend doesn't ask `http://localhost:8080` directly for API calls, but instead uses relative paths (e.g., `/api/movies`) so that the requests are routed through Nginx. I.e. you can omit the base URL in your frontend API calls, and just use the relative path (e.g., `/api/movies` instead of `http://localhost:8080/api/movies`).