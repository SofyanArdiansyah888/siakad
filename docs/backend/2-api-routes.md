# REST API Endpoints

## Base URL
```
http://localhost:3001/api/v1
```

## Authentication
All endpoints require authentication via Bearer token in Authorization header:
```
Authorization: Bearer <jwt_token>
```

## Response Format
```json
{
  "success": true,
  "data": {},
  "message": "Success message",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

## Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": {}
  },
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

## Student Endpoints

### GET /students
Get list of students with optional filters
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "nim": "2021001",
      "name": "John Doe",
      "programStudi": "Informatika",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z"
    }
  ],
    "pagination": {
      "page": 1,
      "limit": 10,
      "total": 100,
      "totalPages": 10
    }
  }
}
```

### GET /students/:id
Get student by ID
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "nim": "2021001",
    "name": "John Doe",
    "programStudi": "Informatika",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### POST /students
Create new student
```json
// Request
{
  "nim": "2021001",
  "name": "John Doe",
  "programStudi": "Informatika"
}

// Response
{
  "success": true,
  "data": {
    "id": "uuid",
    "nim": "2021001",
    "name": "John Doe",
    "programStudi": "Informatika",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### PUT /students/:id
Update student
```json
// Request
{
  "name": "John Doe Updated",
  "programStudi": "Sistem Informasi"
}

// Response
{
  "success": true,
  "data": {
    "id": "uuid",
    "nim": "2021001",
    "name": "John Doe Updated",
    "programStudi": "Sistem Informasi",
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z"
  }
}
```

### DELETE /students/:id
Delete student
```json
{
  "success": true,
  "data": {
    "deleted": true
  },
  "message": "Student deleted successfully"
}
```



## Authentication Endpoints

### POST /auth/login
Login user
```json
// Request
{
  "email": "user@example.com",
  "password": "password123"
}

// Response
{
  "success": true,
  "data": {
    "accessToken": "jwt_token",
    "refreshToken": "refresh_token",
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "role": "student"
    }
  }
}
```

### POST /auth/refresh
Refresh access token
```json
// Request
{
  "refreshToken": "refresh_token"
}

// Response
{
  "success": true,
  "data": {
    "accessToken": "new_jwt_token"
  }
}
```

### POST /auth/logout
Logout user
```json
{
  "success": true,
  "message": "Logged out successfully"
}
```

## HTTP Status Codes

- `200` - Success
- `201` - Created
- `400` - Bad Request
- `401` - Unauthorized
- `403` - Forbidden
- `404` - Not Found
- `422` - Validation Error
- `500` - Internal Server Error
