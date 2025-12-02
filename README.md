# Express Project

This is a basic project using Node.js and Express. The following describes the files and their functionality.

## Project Structure

```
.
├── src/
│   ├── app.js                # Main application file
│   ├── controllers/          # Request handlers
│   ├── db/                   # Database connection and setup
│   ├── middlewares/          # Express middlewares
│   ├── models/               # Data models
│   ├── queries/              # Database queries
│   ├── routes/               # Application routes
│   ├── services/             # Business logic
|   ├── utils/                # Utility functions
│   └── templates/            # Email templates
├── logs/                     # Log files
├── tests/                    # Test files
├── .dockerignore             # Docker ignore file
├── .gitignore                # Git ignore file
├── Dockerfile                # Docker configuration file
├── README.md                 # This file
├── Requirements.md           # Requirements file
├── docker-compose.yml        # Docker compose file
├── env.example               # Example environment variables
├── jest.config.js            # Jest configuration
├── package-lock.json         # NPM lock file
├── package.json              # Project dependencies and scripts
├── setup_server.sh           # Server setup script
├── swagger-autogen.js        # Swagger autogen configuration
├── swagger.js                # Swagger setup
└── swagger.json              # Swagger documentation file
```

## Application Architecture Diagram

```
+-------------------+      +-------------------+      +-------------------+
|                   |      |                   |      |                   |
|      Routes       +----->+   Controllers     +----->+      Services     |
|                   |      |                   |      |                   |
+-------------------+      +-------------------+      +-------------------+
        |                          |                          |
        |                          |                          |
        v                          v                          v
+-------------------+      +-------------------+      +-------------------+
|                   |      |                   |      |                   |
|    Middlewares    +----->+      Models       +----->+      Database     |
|                   |      |                   |      |                   |
+-------------------+      +-------------------+      +-------------------+
```

## Database Schema

```
+-----------------------+       +-----------------------+       +-----------------------+
|      ms_compania      |       |      user_roles       |       |     user_usuarios     |
+-----------------------+       +-----------------------+       +-----------------------+
| id_cia (PK)           |       | id_rol (PK)           |       | id_usuario (PK)       |
| nombre_abreviado      |       | nombre                |       | usuario               |
| nombre_completo       |       | descripcion           |       | contrasena            |
| ...                   |       |                       |       | id_rol (FK)           |
+-----------------------+       +-----------------------+       | id_cia (FK)           |
                                                                +-----------------------+
            |                                      |                      |
            |                                      |                      |
            v                                      v                      v
+-----------------------+       +-----------------------+       +-----------------------+
|     polizas_sigat     |       |      polizas_gp       |       |    user_tmovements    |
+-----------------------+       +-----------------------+       +-----------------------+
| id (PK)               |       | id (PK)               |       | id_tmov (PK)          |
| cedula_rif (PK)       |       | cedula_rif (PK)       |       | id_usuario (FK)       |
| producto (PK)         |       | producto (PK)         |       | action_type           |
| id_cia (FK)           |       | id_cia (FK)           |       | ...                   |
| estado_id (FK)        |       | estado_id (FK)        |       +-----------------------+
| ...                   |       | ...                   |
+-----------------------+       +-----------------------+
```

## Installation

1.  Clone the repository or download the project.
2.  Navigate to the project directory.
3.  Install the dependencies:
    ```bash
    npm install
    ```

## Environment Variables

This project uses environment variables to manage configuration settings. To get started, you'll need to create a `.env` file in the root of the project. You can do this by copying the example file:

```bash
cp env.example .env
```

Next, you'll need to fill in the values for the following variables in the `.env` file:

### Server Configuration

-   `HOST`: The hostname the server will run on (e.g., `localhost`).
-   `PORT`: The port the server will listen on (e.g., `3000`).

### Database Configuration

-   `DB_USER`: The username for your PostgreSQL database.
-   `DB_HOST`: The host of your PostgreSQL database (e.g., `localhost`).
-   `DB_NAME`: The name of your PostgreSQL database.
-   `DB_PASSWORD`: The password for your PostgreSQL database.
-   `DB_PORT`: The port your PostgreSQL database is running on (e.g., `5432`).

### JWT Configuration

-   `JWT_SECRET`: A secret key for signing JSON Web Tokens.
-   `JWT_EXPIRES_IN`: The expiration time for JSON Web Tokens (e.g., `1h`, `1d`).

### Email Configuration

-   `EMAIL_HOST`: The hostname of your email server (e.g., `smtp.example.com`).
-   `EMAIL_PORT`: The port of your email server (e.g., `587`).
-   `EMAIL_SECURE`: Whether to use a secure connection (e.g., `true` or `false`).
-   `EMAIL_USER`: The username for your email account.
-   `EMAIL_PASS`: The password for your email account.

## Usage

### Running the application

To start the application, run:

```bash
npm start
```

The application will be available at `http://localhost:3000` (or the configured port).

### Running in development mode

For development, you can use `nodemon` to automatically restart the server on file changes:

```bash
npm run dev
```

### Running tests

To run the test suite, use:

```bash
npm test
```

### Generating Swagger documentation

To generate the Swagger documentation, run the following command:

```bash
npm run swagger-autogen
```

This will create/update the `swagger.json` file. The documentation will be available at `/api-docs`.

## Running with Docker

To run the application using Docker, you'll need to have Docker and Docker Compose installed.

1.  **Configure your environment:**

    Make sure you have a `.env` file with all the required variables. The Docker Compose setup uses this file to configure both the application and the database containers.

2.  **Database Connection:**

    The database host is configured via the `DB_HOST` variable in your `.env` file.

    *   **To connect to the included PostgreSQL container:**
        Set `DB_HOST=db` in your `.env` file. This is the default value in `env.example`.

    *   **To connect to an external database (e.g., running on your host machine):**
        Set `DB_HOST=host.docker.internal` in your `.env` file. This special DNS name resolves to the internal IP address of the host from within a Docker container.

3.  **Build and run the containers:**

    ```bash
    docker-compose up --build
    ```

    This will build the Docker image for the application and start the application and database containers. The application will be available at `http://localhost:3000` (or the configured port).

4.  **Running in detached mode:**

    To run the containers in the background, use the `-d` flag:

    ```bash
    docker-compose up --build -d
    ```

5.  **Stopping the containers:**

    To stop the containers, run:

    ```bash
    docker-compose down
    ```
