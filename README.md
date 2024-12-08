# User API - Go with Gin, GORM, PostgreSQL, and Docker
This is a simple API for managing users using the **Gin** web framework, **GORM** ORM, **PostgreSQL** for database storage, and Swagger for API documentation. The project is containerized using Docker for easy deployment and isolated environments.

## Table of Contents
1. [Technologies Used](#technologies-used)
2. [Features](#features)
3. [Setup Instructions](#setup-instructions)
4. [Endpoints](#endpoints)
5. [Code Explanation (main.go)](#code-explanation-maingo)
6. [Handlers Explanation](#handlers-explanation)
6. [Access Swagger UI](#access-swagger-ui)

## Technologies Used
- **Go (Golang)**: A statically typed, compiled programming language used to implement the backend API.
- **Gin**: A fast, minimalist web framework for Go, used to route and handle HTTP requests.
- **GORM**: An ORM (Object-Relational Mapper) for Go that simplifies interacting with the PostgreSQL database. GORM automatically enforces the uniqueness of the `email` field in the `users` table, preventing duplicate email entries.
- PostgreSQL**: A powerful, open-source relational database used to store user data.
- **Swagger**: Used for API documentation generation. The application serves Swagger UI for easy interaction with the API.
- **Docker**: A tool for containerizing the Go application and PostgreSQL database for seamless deployment and execution in isolated environments.
- **Docker Compose**: Tool for defining and running multi-container Docker applications.

## Features
- Create a new user
- Retrieve all users
- Retrieve a single user by ID
- Update an existing user
- Delete a user
- Auto-generating API documentation using Swagger UI

## Setup Instructions
**Step 1: Clone the Repository**

Clone the repository to your local machine:

```bash
git clone https://github.com/Fahad-I-Khan/go-gin-gorm.git
cd go-gin-gorm
```

**Step 2: Install Go Dependencies**

Run the following commands to install the necessary Go dependencies for the project:

- Install Gin (the web framework used in this project):

```bash
go get github.com/gin-gonic/gin
```
- Install PostgreSQL driver for Go:

```bash
go get github.com/lib/pq
```
- Install Swagger dependencies for API documentation generation:
  - Install Swag CLI (this command-line tool generates Swagger docs from Go comments):

    ```bash
    go install github.com/swaggo/swag/cmd/swag@latest
    ```
  - Install Swagger UI for Gin:

    ```bash
    go get github.com/swaggo/gin-swagger
    go get github.com/swaggo/files
    ```
- Install CORS middleware for Gin (to handle cross-origin requests):

```bash
go get github.com/gin-contrib/cors
```
- Optional: Run `go mod tidy` to clean up the `go.mod` file and download any missing dependencies:

```bash
go mod tidy
```
- Generate Swagger documentation:

```bash
swag init
```

This command scans your Go code and generates the Swagger documentation in the `docs` folder. You need to run s`wag init` every time you modify your API routes or comments.

## Stopping the Application
To stop the application, use the following command:

```bash
docker-compose down
```

This stops and removes the containers, but the data in the PostgreSQL container persists due to the pgdata volume.

## Endpoints

The API provides the following routes:

### 1. Get All Users
- **URL**: `/api/v1/users`
- **Method**: `GET`
- **Response**: A list of all users.
- **Success Response**:
  - **Code**: 200
  - **Content**: An array of user objects.

### 2. Get a Single User
- **URL**: `/api/v1/users/{id}`
- **Method**: `GET`
- **URL Params**: `id` (User ID)
- **Response**: A single user object.
- **Success Response**:
  - **Code**: 200
  - **Content**: A user object.

### 3. Create a New User
- **URL**: `/api/v1/users`
- **Method**: `POST`
- **Body**: User object containing `name` and `email`.
- **Response**: The created user object.
- **Success Response**:
  - **Code**: 201
  - **Content**: The created user object.

### 4. Update an Existing User
- **URL**: `/api/v1/users/{id}`
- **Method**: `PUT`
- **URL Params**: `id` (User ID)
- **Body**: User object containing updated `name` and `email`.
- **Response**: The updated user object.
- **Success Response**:
  - **Code**: 200
  - **Content**: The updated user object.

### 5. Delete a User
- **URL**: `/api/v1/users/{id}`
- **Method**: `DELETE`
- **URL Params**: `id` (User ID)
- **Response**: A success message indicating the user was deleted.
- **Success Response**:
  - **Code**: 200
  - **Content**: `{ "message": "User deleted" }`

## Code Explanation (main.go)

### Model Definition
The `User` struct represents the data model for the `users` table. It has the following fields:

- `ID`: The primary key of the user (auto-incremented).
- `Name`: The user's name.
- `Email`: The user's email address, which must be unique.

```go
type User struct {
    ID    int    `json:"id" gorm:"primaryKey;autoIncrement"`
    Name  string `json:"name" gorm:"type:varchar(100);not null"`
    Email string `json:"email" gorm:"type:varchar(100);uniqueIndex;not null"`
}
```

## Handlers Explanation
The handler functions define the logic for each route in the application. They interact with the database using GORM and handle different HTTP requests (GET, POST, PUT, DELETE). Below is a detailed explanation of each handler function:

1. `getUsers` - **Fetch All Users**
- Retrieves all users from the database.
- **Functionality**:
  - It queries the `users` table in the PostgreSQL database using GORM's `Find` method, which returns all user records.
- **Error Handling**:
  - Returns `200 OK` with user data or `500` if an error occurs.

2. `getUser` - **Fetch User by ID**
- Retrieves a user by their unique ID.
- **Functionality**:
  - Functionality: It uses the First method from GORM to retrieve the `first` user that matches the provided `id`.
- **Error Handling**:
  - Returns `200 OK` if found, or `404 Not Found` if the user doesn't exist.

3. `createUser` - **Create a New User**
- Creates a new user from the request body (name and email).
- **Error Handling**:
  - Returns `201 Created` on success or `400/500` if there are errors in input or creation.

4. `updateUser` - **Update User by ID**
- Updates the user's `name` and `email` based on their `ID`.
- **Functionality**:
  - It first fetches the user by id using db.First().
  - Then, it binds the updated user data (name, email) from the request body.
  - After updating the user in the database, it uses db.Save() to apply the changes.

- **Error Handling**:
  - Returns `200 OK` with updated data, `404 Not Found` if the user doesn't exist, or 400/500 for invalid input or errors.

5. `deleteUser` - **Delete User by ID**
- Deletes a user by their unique ID.
- **Functionality**:
  - It first retrieves the user from the database using `db.First()`.
  - If the user is found, it deletes the user using `db.Delete()`.
- **Error Handling**:
  - Returns `200 OK` with a success message, `404 Not Found` if the user is missing, or `500` if deletion fails.

### Error Handling
- **404 Not Found**: If a user is not found.
- **400 Bad Request**: For invalid input.
- **500 Internal Server Error**: If a database operation fails.

### 4. Swagger Documentation
Swagger is integrated into the project to provide an interactive API documentation interface. By visiting `/swagger/*any`, users can view the documentation and test API endpoints directly in the browser.

```go
r.GET("/swagger/*any", ginSwagger.WrapHandler(swaggerFiles.Handler))
```
### 5. CORS Middleware
The application uses the cors middleware to handle cross-origin requests, which is especially useful during development.

```go
r.Use(cors.Default())
```
## Access Swagger UI
Once the server is running, you can access the Swagger UI at:

```bash
http://localhost:8000/swagger/index.html
```