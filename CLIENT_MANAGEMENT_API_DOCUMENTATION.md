# Client Management - Backend API & Database Documentation

## Overview
This document provides the complete backend API endpoints and database structure for the Client Management system in ACP Legal Services.

---

## 🔗 Backend API Endpoints

### Base URL
```
Production: https://acp-legal-backend-production.up.railway.app
```

### Authentication
All Client Management endpoints require authentication via Bearer token in the Authorization header:
```
Authorization: Bearer {token}
```

Token is stored in `localStorage.getItem("authToken")`

---

## 📡 API Endpoints

### 1. Get All Clients
**Endpoint:** `GET /Clients`

**Description:** Retrieves all clients in the system

**Request:**
```javascript
GET https://acp-legal-backend-production.up.railway.app/Clients
Headers:
  Authorization: Bearer {token}
```

**Response:** `200 OK`
```json
[
  {
    "id": "uuid-string",
    "full_name": "John Doe",
    "email": "john.doe@example.com",
    "phone": "+44 7700 900000",
    "address": "123 Main Street",
    "city": "London",
    "postcode": "SW1A 1AA",
    "nationality": "British",
    "date_of_birth": "1990-01-15",
    "passport_number": "AB123456",
    "status": "active",
    "created_at": "2024-01-15T10:30:00Z",
    "last_updated": "2024-01-20T14:45:00Z",
    "notes": "VIP client, prefers email contact",
    "total_bookings": 5,
    "total_spent": 850.00
  }
]
```

---

### 2. Get Client by ID
**Endpoint:** `GET /Clients/{id}`

**Description:** Retrieves a specific client by their ID

**Request:**
```javascript
GET https://acp-legal-backend-production.up.railway.app/Clients/{id}
Headers:
  Authorization: Bearer {token}
```

**Response:** `200 OK`
```json
{
  "id": "uuid-string",
  "full_name": "John Doe",
  "email": "john.doe@example.com",
  "phone": "+44 7700 900000",
  "address": "123 Main Street",
  "city": "London",
  "postcode": "SW1A 1AA",
  "nationality": "British",
  "date_of_birth": "1990-01-15",
  "passport_number": "AB123456",
  "status": "active",
  "created_at": "2024-01-15T10:30:00Z",
  "last_updated": "2024-01-20T14:45:00Z",
  "notes": "VIP client, prefers email contact",
  "total_bookings": 5,
  "total_spent": 850.00
}
```

---

### 3. Create New Client
**Endpoint:** `POST /Clients`

**Description:** Creates a new client record

**Request:**
```javascript
POST https://acp-legal-backend-production.up.railway.app/Clients
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json

Body:
{
  "full_name": "Jane Smith",
  "email": "jane.smith@example.com",
  "phone": "+44 7700 900001",
  "address": "456 Oak Avenue",
  "city": "Manchester",
  "postcode": "M1 1AA",
  "nationality": "British",
  "date_of_birth": "1985-05-20",
  "passport_number": "CD789012",
  "status": "active",
  "notes": "Referred by existing client"
}
```

**Response:** `201 Created`
```json
{
  "id": "new-uuid-string",
  "full_name": "Jane Smith",
  "email": "jane.smith@example.com",
  "phone": "+44 7700 900001",
  "address": "456 Oak Avenue",
  "city": "Manchester",
  "postcode": "M1 1AA",
  "nationality": "British",
  "date_of_birth": "1985-05-20",
  "passport_number": "CD789012",
  "status": "active",
  "created_at": "2024-01-22T09:15:00Z",
  "last_updated": "2024-01-22T09:15:00Z",
  "notes": "Referred by existing client",
  "total_bookings": 0,
  "total_spent": 0.00
}
```

---

### 4. Update Client
**Endpoint:** `PUT /Clients/{id}`

**Description:** Updates an existing client record

**Request:**
```javascript
PUT https://acp-legal-backend-production.up.railway.app/Clients/{id}
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json

Body:
{
  "full_name": "Jane Smith-Johnson",
  "email": "jane.smith@example.com",
  "phone": "+44 7700 900001",
  "address": "789 New Street",
  "city": "Manchester",
  "postcode": "M2 2BB",
  "nationality": "British",
  "date_of_birth": "1985-05-20",
  "passport_number": "CD789012",
  "status": "active",
  "notes": "Updated address after relocation"
}
```

**Response:** `200 OK`
```json
{
  "id": "uuid-string",
  "full_name": "Jane Smith-Johnson",
  "email": "jane.smith@example.com",
  "phone": "+44 7700 900001",
  "address": "789 New Street",
  "city": "Manchester",
  "postcode": "M2 2BB",
  "nationality": "British",
  "date_of_birth": "1985-05-20",
  "passport_number": "CD789012",
  "status": "active",
  "created_at": "2024-01-22T09:15:00Z",
  "last_updated": "2024-01-22T11:30:00Z",
  "notes": "Updated address after relocation",
  "total_bookings": 0,
  "total_spent": 0.00
}
```

---

### 5. Get Clients by User ID
**Endpoint:** `GET /Clients/by-user/{userId}`

**Description:** Retrieves all clients associated with a specific user

**Request:**
```javascript
GET https://acp-legal-backend-production.up.railway.app/Clients/by-user/{userId}
Headers:
  Authorization: Bearer {token}
```

**Response:** `200 OK`
```json
[
  {
    "id": "uuid-string",
    "full_name": "John Doe",
    "email": "john.doe@example.com",
    "phone": "+44 7700 900000",
    "status": "active",
    "created_at": "2024-01-15T10:30:00Z",
    "total_bookings": 5,
    "total_spent": 850.00
  }
]
```

---

## 🗄️ Database Structure

### Table: `clients`

| Column Name         | Data Type      | Constraints                  | Description                                    |
|---------------------|----------------|------------------------------|------------------------------------------------|
| `id`                | UUID           | PRIMARY KEY, NOT NULL        | Unique identifier for the client               |
| `full_name`         | VARCHAR(255)   | NOT NULL                     | Client's full name                             |
| `email`             | VARCHAR(255)   | UNIQUE, NOT NULL             | Client's email address                         |
| `phone`             | VARCHAR(50)    | NOT NULL                     | Client's phone number                          |
| `address`           | TEXT           | NULLABLE                     | Street address                                 |
| `city`              | VARCHAR(100)   | NULLABLE                     | City                                           |
| `postcode`          | VARCHAR(20)    | NULLABLE                     | Postal code                                    |
| `nationality`       | VARCHAR(100)   | NULLABLE                     | Client's nationality                           |
| `date_of_birth`     | DATE           | NULLABLE                     | Client's date of birth                         |
| `passport_number`   | VARCHAR(50)    | NULLABLE                     | Passport number                                |
| `status`            | ENUM           | NOT NULL, DEFAULT 'active'   | Client status: 'active' or 'inactive'          |
| `created_at`        | TIMESTAMP      | NOT NULL, DEFAULT NOW()      | Record creation timestamp                      |
| `last_updated`      | TIMESTAMP      | NULLABLE, AUTO UPDATE        | Last modification timestamp                    |
| `notes`             | TEXT           | NULLABLE                     | Additional notes about the client              |
| `total_bookings`    | INTEGER        | DEFAULT 0                    | Total number of bookings (computed field)      |
| `total_spent`       | DECIMAL(10,2)  | DEFAULT 0.00                 | Total amount spent by client (computed field)  |
| `user_id`           | UUID           | FOREIGN KEY, NULLABLE        | Associated user account ID (if applicable)     |

### Indexes
```sql
CREATE INDEX idx_clients_email ON clients(email);
CREATE INDEX idx_clients_phone ON clients(phone);
CREATE INDEX idx_clients_status ON clients(status);
CREATE INDEX idx_clients_created_at ON clients(created_at);
CREATE INDEX idx_clients_user_id ON clients(user_id);
```

### Relationships
- **User Relationship**: `clients.user_id` → `users.id` (One-to-One, Optional)
- **Bookings Relationship**: `clients.id` ← `bookings.client_id` (One-to-Many)

---

## 📊 SQL Schema (CREATE TABLE)

```sql
CREATE TABLE clients (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  full_name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  phone VARCHAR(50) NOT NULL,
  address TEXT,
  city VARCHAR(100),
  postcode VARCHAR(20),
  nationality VARCHAR(100),
  date_of_birth DATE,
  passport_number VARCHAR(50),
  status VARCHAR(20) NOT NULL DEFAULT 'active' CHECK (status IN ('active', 'inactive')),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  last_updated TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  notes TEXT,
  total_bookings INTEGER DEFAULT 0,
  total_spent DECIMAL(10,2) DEFAULT 0.00,
  user_id UUID REFERENCES users(id) ON DELETE SET NULL,
  
  CONSTRAINT chk_email_format CHECK (email ~* '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}$'),
  CONSTRAINT chk_phone_format CHECK (phone ~* '^\+?[0-9\s\-\(\)]+$')
);

-- Indexes for better query performance
CREATE INDEX idx_clients_email ON clients(email);
CREATE INDEX idx_clients_phone ON clients(phone);
CREATE INDEX idx_clients_status ON clients(status);
CREATE INDEX idx_clients_created_at ON clients(created_at);
CREATE INDEX idx_clients_user_id ON clients(user_id);
```

---

## 🔐 Authorization & Access Control

### Role-Based Access

| Role         | GET All | GET by ID | CREATE | UPDATE | DELETE |
|--------------|---------|-----------|--------|--------|--------|
| Super Admin  | ✅      | ✅        | ✅     | ✅     | ✅     |
| Admin        | ✅      | ✅        | ✅     | ✅     | ✅     |
| Staff        | ✅      | ✅        | ✅     | ✅     | ❌     |
| Client       | ❌      | Own Only  | ❌     | Own    | ❌     |

### Authentication Flow
1. User logs in via `/auth/login` endpoint
2. Backend returns JWT token
3. Token stored in `localStorage.getItem("authToken")`
4. All subsequent requests include: `Authorization: Bearer {token}`

---

## 🧪 Frontend Implementation Example

### Fetching Clients
```javascript
import { buildApiUrl, API_ENDPOINTS } from "@/utils/api/config";

const fetchClients = async () => {
  try {
    const token = localStorage.getItem("authToken");
    const response = await fetch(
      buildApiUrl(API_ENDPOINTS.CLIENTS.BASE),
      {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      }
    );

    if (response.ok) {
      const data = await response.json();
      return data;
    }
  } catch (error) {
    console.error("Error fetching clients:", error);
  }
};
```

### Creating a Client
```javascript
const createClient = async (clientData) => {
  try {
    const token = localStorage.getItem("authToken");
    const response = await fetch(
      buildApiUrl(API_ENDPOINTS.CLIENTS.BASE),
      {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify(clientData),
      }
    );

    if (response.ok) {
      const newClient = await response.json();
      return newClient;
    }
  } catch (error) {
    console.error("Error creating client:", error);
  }
};
```

### Updating a Client
```javascript
const updateClient = async (clientId, updatedData) => {
  try {
    const token = localStorage.getItem("authToken");
    const response = await fetch(
      buildApiUrl(API_ENDPOINTS.CLIENTS.BY_ID(clientId)),
      {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify(updatedData),
      }
    );

    if (response.ok) {
      const updatedClient = await response.json();
      return updatedClient;
    }
  } catch (error) {
    console.error("Error updating client:", error);
  }
};
```

---

## 🚨 Error Responses

### 400 Bad Request
```json
{
  "error": "Validation failed",
  "details": {
    "email": "Invalid email format",
    "phone": "Phone number is required"
  }
}
```

### 401 Unauthorized
```json
{
  "error": "Unauthorized",
  "message": "Invalid or expired token"
}
```

### 403 Forbidden
```json
{
  "error": "Forbidden",
  "message": "You do not have permission to access this resource"
}
```

### 404 Not Found
```json
{
  "error": "Not Found",
  "message": "Client with specified ID not found"
}
```

### 409 Conflict
```json
{
  "error": "Conflict",
  "message": "A client with this email already exists"
}
```

### 500 Internal Server Error
```json
{
  "error": "Internal Server Error",
  "message": "An unexpected error occurred"
}
```

---

## 📝 Computed Fields

### `total_bookings`
Calculated from the count of related bookings:
```sql
SELECT COUNT(*) FROM bookings WHERE client_id = clients.id
```

### `total_spent`
Calculated from the sum of all completed booking payments:
```sql
SELECT SUM(total_amount) FROM bookings 
WHERE client_id = clients.id 
AND status = 'completed'
AND payment_status = 'paid'
```

---

## 🔄 Data Validation Rules

### Email
- Format: Valid email pattern (RFC 5322)
- Unique: Must not exist for another client
- Required: Cannot be null or empty

### Phone
- Format: International format recommended (e.g., +44 7700 900000)
- Required: Cannot be null or empty

### Status
- Values: 'active' or 'inactive'
- Default: 'active'

### Date of Birth
- Format: ISO 8601 date (YYYY-MM-DD)
- Validation: Must be in the past

### Postcode
- Format: Valid UK postcode pattern (if UK client)

---

## 🔗 Related Endpoints

The Client Management system integrates with these related endpoints:

- **Bookings**: `/api/Bookings` - Link clients to their bookings
- **Users**: `/Users` - Link clients to user accounts
- **Invoices**: `/api/invoices` - Track client invoices

---

## 📋 Integration Notes

1. **Client Portal Integration**: When a user registers for the Client Portal, a corresponding entry should be created in the `clients` table with `user_id` linking to their user account.

2. **Booking Integration**: When creating a booking, validate that the `client_id` exists in the clients table.

3. **Data Privacy**: Implement GDPR compliance by allowing clients to request data deletion and providing data export functionality.

4. **Audit Trail**: Consider adding audit logging for all client record modifications (create, update, delete operations).

---

## 📱 Frontend Routes

- **Admin**: `/admin/client-management`
- **Staff**: `/staff/client-management`
- **Component**: `/src/app/pages/ClientManagement.tsx`

---

## 📈 Future Enhancements

1. **Soft Delete**: Add `deleted_at` field for soft deletion instead of hard delete
2. **Tags/Labels**: Add support for client categorization
3. **Custom Fields**: Allow custom fields per client
4. **Document Attachments**: Link client documents and files
5. **Communication History**: Track all communications with client
6. **Appointment History**: Full audit trail of all appointments

---

*Last Updated: January 22, 2025*
*Backend: Railway Production*
*API Version: 1.0*
