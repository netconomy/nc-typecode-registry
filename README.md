# Typecode Registry

## Overview
The Typecode Registry is a web application developed for NETCONOMY to support SAP Commerce Cloud projects, ensuring unique Typecode assignments across projects and extensions. It streamlines developers' workflows by providing conflict-free Typecodes, crucial for SAP Commerce's internal type management.

The application is developed by Students of the FH Burgenland `Software Engineering und vernetzte Systeme` program.

Not all features could be fully implemented during the project's duration. The project is published for educational purposes.

SAP, SAP Commerce Cloud are the trademarks or registered trademarks of SAP SE or its affiliates in Germany and in several other countries.

## Features
- **Web Service**: Accessible as a web service, ensuring broad compatibility and easy integration.
- **OAuth 2.0 Authentication**: Secures user sessions and data access through Microsoft Entra ID.
- **Automated User and Project Management**: Supports programmatic user assignments to projects with distinct admin roles.
- **Type Management**: Enables both manual and automated (via `items.xml`) Type registrations.
- **Scope-Based Typecode Generation**: Generates Typecodes within specified number ranges based on the Type's scope.
- **Advanced Filtering and Sorting**: Offers robust search capabilities within the Type registry.
- **RESTful API Backend**: Facilitates automation and frontend interaction, developed in Golang.
- **SPA Frontend**: Delivers a seamless user experience through a Single-Page Application developed in Angular.
- **PostgreSQL Database**: Leverages PostgreSQL for data storage, ensuring reliability and performance.

## Getting Started

Regardless if you want to setup a developer environment build on Docker or locally, you will need to clone the repository first.

 ```bash
 git clone https://github.com/HardCodedCoder/typecode-registry.git
```

Please execute this step as a first step before you continue.

### DOCKER COMPOSE SETUP ###

For a fully containerized environment, you can use Docker to manage both the backend and frontend along with PostgreSQL.

Here's how you can set it up:

```yml
services:
   postgres:
      container_name: PostgreSQL_database
      image: postgres:latest
      environment:
         POSTGRES_DB: backend
         POSTGRES_PASSWORD: mysecretpassword
         POSTGRES_HOST_AUTH_METHOD: trust
         PGDATA: /var/lib/postgresql/data/pgdata
      ports:
         - "5432:5432"
      volumes:
         - ./postgres-data:/var/lib/postgresql/data
         - ./backend/database/001_initial_schema.sql:/docker-entrypoint-initdb.d/001_initial_schema.sql
         - ./backend/database/testscripts/001_initial_test_data.sql:/docker-entrypoint-initdb.d/001_initial_test_data.sql

   backend:
      container_name: tcr_backend
      build:
         context: ./backend
         dockerfile: Dockerfile
      environment:
         TYPECODEREGISTRY_DB_DSN: "postgresql://postgres:mysecretpassword@postgres:5432/backend?sslmode=disable"
      ports:
         - 8080:8080
      volumes:
         - ./backend:/app

   frontend:
      container_name: tcr_frontend
      build:
         context: ./frontend
         dockerfile: Dockerfile
      ports:
         - "4200:80"
      volumes:
         - ./frontend:/app
         - /app/node_modules
      depends_on:
         - backend
```

The file can be found in the root directory of the project as `docker-compose.yml`.

Simply run the following command to start the services:

```bash
docker-compose up --build
```

This will start all services, and you can access the frontend application on `http://localhost:4200`.


### LOCAL DEVELOPMENT SETUP ###

#### Prerequisites
- **Golang**: Ensure you have Golang installed for backend development. Visit [Golang's official site](https://golang.org/dl/) for download and installation instructions.
- **Node.js and npm**: Required for Angular frontend development. Download and install from [Node.js official website](https://nodejs.org/).
- **Angular CLI**: Install Angular CLI via npm with `npm install -g @angular/cli`.
- **PostgreSQL**: Ensure PostgreSQL is installed and running for your database needs. Installation guides can be found on the [PostgreSQL official website](https://www.postgresql.org/download/).

#### Database Setup ####

1. **Change into backend Directory**: Navigate to the backend directory to access the database scripts.

```bash
cd ./backend/database/
```
2. **Install the PostgreSQL database**: Install the database locally or via a docker container.

If you want to use docker for the database standlone, you could use the following `docker-compose.yml` as a base for your container configuration: 

```yml
version: '3'
services:
  db:
    container_name: PostgreSQL-database
    image: postgres:latest
    environment:
      POSTGRES_USER: <username>
      POSTGRES_PASSWORD: <password>
      POSTGRES_DB: <dbname>
    ports:
      - "5432:5432"
    volumes:
      - ~/postgres-data:/var/lib/postgresql/data
  pgadmin:
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: <pg_admin_username>
      PGADMIN_DEFAULT_PASSWORD: <pg_admin_password>
    ports:
      - "5050:80"
    volumes:
      - ~/pgadmin-data:/var/lib/pgadmin
```

**NOTE**: Please replace placeholders like `<username>` with actual values. 

3. **Setup data model**: In the repository directory, please navigate to:

```bash
cd ./backend/database/
```

and execute the script named `001_initial_schema.sql` in a database console to setup the data model. 

#### Backend Setup (Golang)


1. **Configure the Environment**: Configure your local environment variable for database connection. This involves exporting an environment variable directly. Define a variable called `TYPECODEREGISTRY_DB_DSN` which holds the connection string to the database.

2. **Install Dependencies**: Navigate to the backend directory and install the required Golang modules to ensure all dependencies are up to date.

   
```bash
 go mod tidy
```

3. **Run the Backend**: To start the backend server, run the main application file. First, ensure you're in the backend directory `./backend/cmd/app`. Then execute the `go run .` command. This action initiates the server, making it listen for requests on the default port 8080.

The following parameters can be passed as arguments to the application: 

```bash
  -db-dns string
        PostgreSQL DSN (default os.Getenv("TYPECODEREGISTRY_DB_DSN"))
  -loglevel string
        Log level (debug, info, warn, error, fatal, panic) (default "info")
  -port int
        API server port (default 8080)
```

4. **Validation**: Watch for console output indicating that the server is running, printing `API server is up and running`. This confirms that your backend service is up and operational.

#### Frontend Setup (Angular)

1. **Navigate to the Frontend Directory**: Change to the directory where your Angular project is located. This is where you will run commands related to Angular CLI and manage your frontend application.

   `cd path/to/typecode-registry/frontend`

2. **Install Dependencies**: The next step is to install the dependencies required by your Angular project. This step ensures all necessary libraries and frameworks specified in your `package.json` are available for development and build processes.

   `npm install`

3. **Environment Configuration**: Set up your Angular application to communicate with the backend by configuring the `backend.service.ts` file. This file is located under the `src/app/services` directory. You will need to specify the backend API URL.

In `backend.service.ts`, you might have something like:

```typescript
private apiUrl = 'http://localhost:8080';
```

4. **Run the Frontend Development Server**: Fire up your Angular application by starting the Angular development server. The ng serve command compiles the application and launches it in a browser.

```bash
ng serve
```

5. **Access the Application**: With the server running, open your web browser to 'http://localhost:4200' to view the frontend application. It should now be communicating with your backend, allowing you to interact with the full stack of your project.
