# Arize Holidays - Backend API & Database Documentation
## Phase 1 - Essential Features for Business Launch

---

## 🗄️ DATABASE STRUCTURE (Supabase KV Store)

The system uses a key-value store with the following key patterns:

### **1. User Management**

#### Admin Users
```
Key Pattern: admin:{userId}
Value: {
  id: string,
  email: string,
  name: string,
  role: "admin" | "super_admin",
  createdAt: string (ISO date),
  lastLogin: string (ISO date),
  status: "active" | "inactive",
  permissions: string[] // e.g., ["users", "tours", "bookings", "itineraries"]
}
```

#### Client Users
```
Key Pattern: client:{userId}
Value: {
  id: string,
  email: string,
  name: string,
  phone: string,
  country: string,
  createdAt: string (ISO date),
  lastLogin: string (ISO date),
  status: "active" | "inactive",
  totalBookings: number,
  preferences: {
    interests: string[],
    budgetRange: string,
    travelStyle: string
  }
}
```

#### User Index (for lookups)
```
Key Pattern: user:email:{email}
Value: {
  userId: string,
  userType: "admin" | "client"
}
```

---

### **2. Tour Package Management**

#### Tour Packages
```
Key Pattern: tour:{tourId}
Value: {
  id: string,
  title: string,
  description: string,
  duration: string, // e.g., "7 Days 6 Nights"
  price: number, // in GBP
  currency: "GBP",
  category: string, // e.g., "Adventure", "Cultural", "Beach"
  destinations: string[], // e.g., ["Colombo", "Kandy", "Ella"]
  highlights: string[],
  inclusions: string[],
  exclusions: string[],
  images: string[], // URLs
  availability: "available" | "unavailable",
  maxGroupSize: number,
  minGroupSize: number,
  createdAt: string,
  updatedAt: string,
  createdBy: string, // admin userId
  featured: boolean,
  rating: number, // 0-5
  reviewCount: number
}
```

#### Tour Categories Index
```
Key Pattern: tours:category:{category}
Value: string[] // Array of tourIds
```

---

### **3. Itinerary Management**

#### Itineraries (Proposals)
```
Key Pattern: itinerary:{itineraryId}
Value: {
  id: string,
  clientId: string,
  clientName: string,
  clientEmail: string,
  title: string,
  tourPackageId: string | null, // null if custom itinerary
  status: "draft" | "sent" | "viewed" | "accepted" | "rejected" | "expired",
  createdAt: string,
  updatedAt: string,
  sentAt: string | null,
  viewedAt: string | null,
  respondedAt: string | null,
  expiresAt: string,
  
  // Trip Details
  startDate: string,
  endDate: string,
  duration: string,
  numberOfTravelers: number,
  
  // Itinerary Days
  days: [
    {
      day: number,
      date: string,
      title: string,
      description: string,
      accommodation: {
        name: string,
        type: string, // "Hotel", "Resort", "Villa"
        rating: number,
        location: string,
        checkIn: string,
        checkOut: string
      },
      activities: [
        {
          time: string,
          title: string,
          description: string,
          duration: string,
          location: string,
          included: boolean
        }
      ],
      meals: {
        breakfast: boolean,
        lunch: boolean,
        dinner: boolean
      },
      transport: {
        type: string, // "Private Car", "Train", "Flight"
        details: string
      }
    }
  ],
  
  // Pricing
  pricing: {
    basePrice: number,
    accommodationCost: number,
    transportCost: number,
    activitiesCost: number,
    guideCost: number,
    otherCosts: number,
    subtotal: number,
    tax: number,
    discount: number,
    totalPrice: number,
    currency: "GBP",
    pricePerPerson: number
  },
  
  // Inclusions & Exclusions
  inclusions: string[],
  exclusions: string[],
  
  // Additional Info
  specialRequests: string,
  notes: string, // Admin internal notes
  termsAndConditions: string,
  
  // Admin Details
  createdBy: string, // admin userId
  updatedBy: string // admin userId
}
```

#### Client Itineraries Index
```
Key Pattern: client:itineraries:{clientId}
Value: string[] // Array of itineraryIds
```

---

### **4. Bookings Management**

#### Bookings
```
Key Pattern: booking:{bookingId}
Value: {
  id: string,
  bookingReference: string, // e.g., "ARZ-2026-001"
  clientId: string,
  clientName: string,
  clientEmail: string,
  clientPhone: string,
  
  // Tour Details
  tourPackageId: string | null,
  itineraryId: string | null, // If booked from custom itinerary
  tourTitle: string,
  startDate: string,
  endDate: string,
  duration: string,
  
  // Travelers
  numberOfAdults: number,
  numberOfChildren: number,
  totalTravelers: number,
  travelerDetails: [
    {
      title: string, // "Mr", "Mrs", "Ms"
      firstName: string,
      lastName: string,
      dateOfBirth: string,
      passportNumber: string,
      passportExpiry: string,
      nationality: string,
      dietaryRequirements: string,
      medicalConditions: string
    }
  ],
  
  // Pricing
  pricing: {
    basePrice: number,
    addonsCost: number,
    subtotal: number,
    tax: number,
    discount: number,
    totalPrice: number,
    currency: "GBP",
    pricePerPerson: number
  },
  
  // Payment
  paymentStatus: "pending" | "deposit_paid" | "partially_paid" | "fully_paid" | "refunded",
  paymentMethod: string,
  depositAmount: number,
  paidAmount: number,
  remainingAmount: number,
  paymentDueDate: string,
  
  // Status
  status: "pending" | "confirmed" | "cancelled" | "completed",
  bookingDate: string,
  confirmedAt: string | null,
  cancelledAt: string | null,
  cancellationReason: string | null,
  
  // Special Requests
  specialRequests: string,
  flightDetails: {
    arrivalFlight: string,
    arrivalDate: string,
    arrivalTime: string,
    departureFlight: string,
    departureDate: string,
    departureTime: string
  },
  
  // Admin Details
  assignedTo: string, // admin userId
  internalNotes: string,
  
  createdAt: string,
  updatedAt: string
}
```

#### Client Bookings Index
```
Key Pattern: client:bookings:{clientId}
Value: string[] // Array of bookingIds
```

---

### **5. Invoices Management**

#### Invoices
```
Key Pattern: invoice:{invoiceId}
Value: {
  id: string,
  invoiceNumber: string, // e.g., "INV-2026-001"
  bookingId: string,
  clientId: string,
  clientName: string,
  clientEmail: string,
  
  // Client Address
  billingAddress: {
    addressLine1: string,
    addressLine2: string,
    city: string,
    postcode: string,
    country: string
  },
  
  // Invoice Details
  issueDate: string,
  dueDate: string,
  status: "draft" | "sent" | "paid" | "overdue" | "cancelled",
  
  // Line Items
  items: [
    {
      description: string,
      quantity: number,
      unitPrice: number,
      total: number
    }
  ],
  
  // Pricing
  subtotal: number,
  tax: number,
  taxRate: number, // percentage
  discount: number,
  discountReason: string,
  totalAmount: number,
  currency: "GBP",
  
  // Payment Tracking
  paidAmount: number,
  remainingAmount: number,
  paymentDueDate: string,
  
  // Payment History
  payments: [
    {
      date: string,
      amount: number,
      method: string, // "Bank Transfer", "Card", "PayPal"
      transactionId: string,
      notes: string
    }
  ],
  
  // Additional Info
  notes: string,
  termsAndConditions: string,
  
  // Timestamps
  createdAt: string,
  updatedAt: string,
  sentAt: string | null,
  paidAt: string | null,
  
  // Admin
  createdBy: string, // admin userId
  updatedBy: string
}
```

#### Booking Invoices Index
```
Key Pattern: booking:invoices:{bookingId}
Value: string[] // Array of invoiceIds
```

#### Client Invoices Index
```
Key Pattern: client:invoices:{clientId}
Value: string[] // Array of invoiceIds
```

---

### **6. Notifications**

#### Notifications
```
Key Pattern: notification:{notificationId}
Value: {
  id: string,
  userId: string,
  userType: "admin" | "client",
  type: "itinerary" | "booking" | "payment" | "general",
  title: string,
  message: string,
  status: "unread" | "read",
  priority: "low" | "medium" | "high",
  relatedId: string | null, // itineraryId, bookingId, invoiceId
  relatedType: string | null, // "itinerary", "booking", "invoice"
  createdAt: string,
  readAt: string | null
}
```

#### User Notifications Index
```
Key Pattern: notifications:{userId}
Value: string[] // Array of notificationIds (sorted by date, newest first)
```

---

## 🚀 API ENDPOINTS

Base URL: `https://${projectId}.supabase.co/functions/v1/make-server-c84c8071`

All requests require Authorization header:
```
Authorization: Bearer ${publicAnonKey}
```

Protected routes (admin-only) require user access token:
```
Authorization: Bearer ${accessToken}
```

---

## 👥 USER MANAGEMENT ENDPOINTS

### **1. Admin Authentication**

#### `POST /auth/admin/login`
Admin login with email and password.

**Request Body:**
```json
{
  "email": "admin@arizeholidays.com",
  "password": "password123"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "user": {
      "id": "uuid",
      "email": "admin@arizeholidays.com",
      "name": "Admin Name",
      "role": "admin"
    },
    "accessToken": "jwt-token",
    "refreshToken": "refresh-token"
  }
}
```

---

#### `POST /auth/admin/create`
Create new admin user (super admin only).

**Request Body:**
```json
{
  "email": "newadmin@arizeholidays.com",
  "password": "password123",
  "name": "New Admin",
  "role": "admin",
  "permissions": ["tours", "bookings"]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Admin user created successfully",
  "data": {
    "userId": "uuid",
    "email": "newadmin@arizeholidays.com"
  }
}
```

---

### **2. Client Management**

#### `POST /clients/create`
Create new client account.

**Request Body:**
```json
{
  "email": "client@example.com",
  "password": "password123",
  "name": "John Smith",
  "phone": "+44 7700 900000",
  "country": "United Kingdom"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Client account created successfully",
  "data": {
    "userId": "uuid",
    "email": "client@example.com"
  }
}
```

---

#### `GET /clients`
Get all clients (admin only).

**Query Parameters:**
- `limit` (optional): Number of results (default: 50)
- `status` (optional): Filter by status ("active" | "inactive")

**Response:**
```json
{
  "success": true,
  "data": {
    "clients": [
      {
        "id": "uuid",
        "email": "client@example.com",
        "name": "John Smith",
        "phone": "+44 7700 900000",
        "country": "United Kingdom",
        "status": "active",
        "totalBookings": 3,
        "createdAt": "2026-01-20T10:00:00Z"
      }
    ],
    "total": 1
  }
}
```

---

#### `GET /clients/:clientId`
Get specific client details (admin only).

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "email": "client@example.com",
    "name": "John Smith",
    "phone": "+44 7700 900000",
    "country": "United Kingdom",
    "status": "active",
    "totalBookings": 3,
    "preferences": {
      "interests": ["adventure", "culture"],
      "budgetRange": "£2000-£5000",
      "travelStyle": "luxury"
    },
    "createdAt": "2026-01-20T10:00:00Z",
    "lastLogin": "2026-01-24T08:30:00Z"
  }
}
```

---

#### `PUT /clients/:clientId`
Update client information.

**Request Body:**
```json
{
  "name": "John Smith Updated",
  "phone": "+44 7700 900001",
  "preferences": {
    "interests": ["adventure", "wildlife"],
    "budgetRange": "£3000-£6000"
  }
}
```

**Response:**
```json
{
  "success": true,
  "message": "Client updated successfully"
}
```

---

#### `DELETE /clients/:clientId`
Deactivate client account (admin only).

**Response:**
```json
{
  "success": true,
  "message": "Client account deactivated"
}
```

---

## 🗺️ TOUR PACKAGE MANAGEMENT ENDPOINTS

### **3. Tour Package Operations**

#### `POST /tours/create`
Create new tour package (admin only).

**Request Body:**
```json
{
  "title": "Sri Lanka Highlights - 7 Days",
  "description": "Discover the best of Sri Lanka...",
  "duration": "7 Days 6 Nights",
  "price": 1499,
  "category": "Cultural",
  "destinations": ["Colombo", "Kandy", "Sigiriya", "Ella"],
  "highlights": [
    "Visit Temple of the Sacred Tooth",
    "Climb Sigiriya Rock Fortress",
    "Train ride through tea country"
  ],
  "inclusions": [
    "6 nights accommodation",
    "Daily breakfast",
    "Private transportation",
    "English-speaking guide"
  ],
  "exclusions": [
    "International flights",
    "Travel insurance",
    "Personal expenses"
  ],
  "images": ["url1", "url2"],
  "maxGroupSize": 12,
  "minGroupSize": 2,
  "featured": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Tour package created successfully",
  "data": {
    "tourId": "uuid",
    "title": "Sri Lanka Highlights - 7 Days"
  }
}
```

---

#### `GET /tours`
Get all tour packages.

**Query Parameters:**
- `category` (optional): Filter by category
- `featured` (optional): true/false
- `availability` (optional): "available" | "unavailable"
- `limit` (optional): Number of results
- `sort` (optional): "price_asc" | "price_desc" | "rating" | "newest"

**Response:**
```json
{
  "success": true,
  "data": {
    "tours": [
      {
        "id": "uuid",
        "title": "Sri Lanka Highlights - 7 Days",
        "description": "Discover the best of Sri Lanka...",
        "duration": "7 Days 6 Nights",
        "price": 1499,
        "currency": "GBP",
        "category": "Cultural",
        "destinations": ["Colombo", "Kandy", "Sigiriya", "Ella"],
        "images": ["url1", "url2"],
        "featured": true,
        "rating": 4.8,
        "reviewCount": 42
      }
    ],
    "total": 1
  }
}
```

---

#### `GET /tours/:tourId`
Get specific tour package details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "Sri Lanka Highlights - 7 Days",
    "description": "Discover the best of Sri Lanka...",
    "duration": "7 Days 6 Nights",
    "price": 1499,
    "currency": "GBP",
    "category": "Cultural",
    "destinations": ["Colombo", "Kandy", "Sigiriya", "Ella"],
    "highlights": ["..."],
    "inclusions": ["..."],
    "exclusions": ["..."],
    "images": ["url1", "url2"],
    "maxGroupSize": 12,
    "minGroupSize": 2,
    "featured": true,
    "rating": 4.8,
    "reviewCount": 42,
    "createdAt": "2026-01-15T10:00:00Z",
    "updatedAt": "2026-01-20T14:30:00Z"
  }
}
```

---

#### `PUT /tours/:tourId`
Update tour package (admin only).

**Request Body:**
```json
{
  "title": "Sri Lanka Highlights - 7 Days (Updated)",
  "price": 1599,
  "availability": "available"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Tour package updated successfully"
}
```

---

#### `DELETE /tours/:tourId`
Delete tour package (admin only).

**Response:**
```json
{
  "success": true,
  "message": "Tour package deleted successfully"
}
```

---

## 📋 ITINERARY MANAGEMENT ENDPOINTS

### **4. Itinerary (Proposal) Operations**

#### `POST /itineraries/create`
Create new custom itinerary/proposal (admin only).

**Request Body:**
```json
{
  "clientId": "uuid",
  "clientName": "John Smith",
  "clientEmail": "client@example.com",
  "title": "Custom Sri Lanka Adventure",
  "tourPackageId": null,
  "startDate": "2026-03-15",
  "endDate": "2026-03-22",
  "duration": "8 Days 7 Nights",
  "numberOfTravelers": 2,
  "days": [
    {
      "day": 1,
      "date": "2026-03-15",
      "title": "Arrival in Colombo",
      "description": "Welcome to Sri Lanka! Upon arrival...",
      "accommodation": {
        "name": "Galle Face Hotel",
        "type": "Hotel",
        "rating": 5,
        "location": "Colombo",
        "checkIn": "14:00",
        "checkOut": ""
      },
      "activities": [
        {
          "time": "15:00",
          "title": "City Tour",
          "description": "Explore Colombo's colonial heritage...",
          "duration": "3 hours",
          "location": "Colombo",
          "included": true
        }
      ],
      "meals": {
        "breakfast": false,
        "lunch": false,
        "dinner": true
      },
      "transport": {
        "type": "Private Car",
        "details": "Airport pickup with English-speaking driver"
      }
    }
  ],
  "pricing": {
    "basePrice": 1200,
    "accommodationCost": 600,
    "transportCost": 300,
    "activitiesCost": 200,
    "guideCost": 150,
    "otherCosts": 50,
    "subtotal": 2500,
    "tax": 125,
    "discount": 0,
    "totalPrice": 2625,
    "currency": "GBP",
    "pricePerPerson": 1312.50
  },
  "inclusions": [
    "7 nights accommodation",
    "All meals as specified",
    "Private transportation"
  ],
  "exclusions": [
    "International flights",
    "Travel insurance"
  ],
  "specialRequests": "Vegetarian meals preferred",
  "notes": "Client interested in photography tours",
  "termsAndConditions": "Full payment required 30 days before travel..."
}
```

**Response:**
```json
{
  "success": true,
  "message": "Itinerary created successfully",
  "data": {
    "itineraryId": "uuid",
    "title": "Custom Sri Lanka Adventure",
    "status": "draft"
  }
}
```

---

#### `GET /itineraries`
Get all itineraries (admin only).

**Query Parameters:**
- `clientId` (optional): Filter by client
- `status` (optional): Filter by status
- `limit` (optional): Number of results

**Response:**
```json
{
  "success": true,
  "data": {
    "itineraries": [
      {
        "id": "uuid",
        "clientName": "John Smith",
        "clientEmail": "client@example.com",
        "title": "Custom Sri Lanka Adventure",
        "status": "sent",
        "startDate": "2026-03-15",
        "duration": "8 Days 7 Nights",
        "totalPrice": 2625,
        "createdAt": "2026-01-20T10:00:00Z",
        "sentAt": "2026-01-21T09:00:00Z"
      }
    ],
    "total": 1
  }
}
```

---

#### `GET /itineraries/:itineraryId`
Get specific itinerary details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "clientId": "uuid",
    "clientName": "John Smith",
    "clientEmail": "client@example.com",
    "title": "Custom Sri Lanka Adventure",
    "status": "sent",
    "startDate": "2026-03-15",
    "endDate": "2026-03-22",
    "duration": "8 Days 7 Nights",
    "numberOfTravelers": 2,
    "days": [...],
    "pricing": {...},
    "inclusions": [...],
    "exclusions": [...],
    "createdAt": "2026-01-20T10:00:00Z",
    "sentAt": "2026-01-21T09:00:00Z",
    "expiresAt": "2026-02-21T09:00:00Z"
  }
}
```

---

#### `PUT /itineraries/:itineraryId`
Update itinerary (admin only).

**Request Body:**
```json
{
  "title": "Updated Custom Sri Lanka Adventure",
  "days": [...],
  "pricing": {...},
  "notes": "Updated with client feedback"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Itinerary updated successfully"
}
```

---

#### `POST /itineraries/:itineraryId/send`
Send itinerary to client (admin only).

**Request Body:**
```json
{
  "message": "Dear John, Please find your custom itinerary attached...",
  "expiryDays": 30
}
```

**Response:**
```json
{
  "success": true,
  "message": "Itinerary sent to client successfully",
  "data": {
    "sentAt": "2026-01-21T09:00:00Z",
    "expiresAt": "2026-02-21T09:00:00Z",
    "status": "sent"
  }
}
```

**Actions Performed:**
- Updates itinerary status to "sent"
- Records sentAt timestamp
- Sends email to client with itinerary link
- Creates notification for client
- Sets expiry date

---

#### `GET /itineraries/:itineraryId/view`
View itinerary (client side - public link with token).

**Query Parameters:**
- `token`: Secure access token sent via email

**Response:**
```json
{
  "success": true,
  "data": {
    // Full itinerary details (same as GET /itineraries/:itineraryId)
  }
}
```

**Actions Performed:**
- Updates status to "viewed" if first time
- Records viewedAt timestamp
- Creates admin notification that client viewed itinerary

---

#### `POST /itineraries/:itineraryId/respond`
Client responds to itinerary.

**Request Body:**
```json
{
  "response": "accept" | "reject",
  "message": "Thank you, we'd love to book this trip!",
  "requestedChanges": "Can we add an extra day in Ella?"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Response recorded successfully"
}
```

**Actions Performed:**
- Updates status to "accepted" or "rejected"
- Records respondedAt timestamp
- Sends email to admin
- Creates admin notification
- If accepted, can trigger booking creation

---

#### `DELETE /itineraries/:itineraryId`
Delete itinerary (admin only).

**Response:**
```json
{
  "success": true,
  "message": "Itinerary deleted successfully"
}
```

---

## 📦 BOOKING MANAGEMENT ENDPOINTS

### **5. Booking Operations**

#### `POST /bookings/create`
Create new booking.

**Request Body:**
```json
{
  "clientId": "uuid",
  "clientName": "John Smith",
  "clientEmail": "client@example.com",
  "clientPhone": "+44 7700 900000",
  "tourPackageId": "uuid",
  "itineraryId": null,
  "tourTitle": "Sri Lanka Highlights - 7 Days",
  "startDate": "2026-03-15",
  "endDate": "2026-03-22",
  "duration": "8 Days 7 Nights",
  "numberOfAdults": 2,
  "numberOfChildren": 0,
  "totalTravelers": 2,
  "travelerDetails": [
    {
      "title": "Mr",
      "firstName": "John",
      "lastName": "Smith",
      "dateOfBirth": "1985-06-15",
      "passportNumber": "AB123456",
      "passportExpiry": "2028-06-15",
      "nationality": "British",
      "dietaryRequirements": "Vegetarian",
      "medicalConditions": "None"
    },
    {
      "title": "Mrs",
      "firstName": "Jane",
      "lastName": "Smith",
      "dateOfBirth": "1987-08-20",
      "passportNumber": "CD789012",
      "passportExpiry": "2029-08-20",
      "nationality": "British",
      "dietaryRequirements": "None",
      "medicalConditions": "None"
    }
  ],
  "pricing": {
    "basePrice": 2998,
    "addonsCost": 200,
    "subtotal": 3198,
    "tax": 159.90,
    "discount": 0,
    "totalPrice": 3357.90,
    "currency": "GBP",
    "pricePerPerson": 1678.95
  },
  "depositAmount": 1000,
  "paymentDueDate": "2026-02-15",
  "specialRequests": "Window seats on train journeys preferred",
  "flightDetails": {
    "arrivalFlight": "BA123",
    "arrivalDate": "2026-03-15",
    "arrivalTime": "10:30",
    "departureFlight": "BA456",
    "departureDate": "2026-03-22",
    "departureTime": "14:00"
  }
}
```

**Response:**
```json
{
  "success": true,
  "message": "Booking created successfully",
  "data": {
    "bookingId": "uuid",
    "bookingReference": "ARZ-2026-001",
    "status": "pending"
  }
}
```

**Actions Performed:**
- Generates unique booking reference
- Creates booking record
- Sends confirmation email to client
- Creates admin notification
- Can optionally create invoice

---

#### `GET /bookings`
Get all bookings (admin only).

**Query Parameters:**
- `clientId` (optional): Filter by client
- `status` (optional): Filter by status
- `paymentStatus` (optional): Filter by payment status
- `startDate` (optional): Filter bookings starting from date
- `limit` (optional): Number of results

**Response:**
```json
{
  "success": true,
  "data": {
    "bookings": [
      {
        "id": "uuid",
        "bookingReference": "ARZ-2026-001",
        "clientName": "John Smith",
        "clientEmail": "client@example.com",
        "tourTitle": "Sri Lanka Highlights - 7 Days",
        "startDate": "2026-03-15",
        "totalTravelers": 2,
        "totalPrice": 3357.90,
        "paymentStatus": "deposit_paid",
        "status": "confirmed",
        "bookingDate": "2026-01-20T10:00:00Z"
      }
    ],
    "total": 1
  }
}
```

---

#### `GET /bookings/:bookingId`
Get specific booking details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "bookingReference": "ARZ-2026-001",
    "clientId": "uuid",
    "clientName": "John Smith",
    "clientEmail": "client@example.com",
    "tourPackageId": "uuid",
    "tourTitle": "Sri Lanka Highlights - 7 Days",
    "startDate": "2026-03-15",
    "endDate": "2026-03-22",
    "travelerDetails": [...],
    "pricing": {...},
    "paymentStatus": "deposit_paid",
    "paidAmount": 1000,
    "remainingAmount": 2357.90,
    "status": "confirmed",
    "specialRequests": "...",
    "flightDetails": {...},
    "bookingDate": "2026-01-20T10:00:00Z"
  }
}
```

---

#### `PUT /bookings/:bookingId`
Update booking details.

**Request Body:**
```json
{
  "status": "confirmed",
  "paymentStatus": "deposit_paid",
  "paidAmount": 1000,
  "internalNotes": "Deposit received via bank transfer"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Booking updated successfully"
}
```

---

#### `POST /bookings/:bookingId/confirm`
Confirm booking (admin only).

**Request Body:**
```json
{
  "message": "Your booking has been confirmed...",
  "sendEmail": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Booking confirmed successfully",
  "data": {
    "confirmedAt": "2026-01-21T10:00:00Z"
  }
}
```

**Actions Performed:**
- Updates status to "confirmed"
- Records confirmedAt timestamp
- Sends confirmation email to client
- Creates client notification

---

#### `POST /bookings/:bookingId/cancel`
Cancel booking.

**Request Body:**
```json
{
  "reason": "Client requested cancellation",
  "refundAmount": 500,
  "sendEmail": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Booking cancelled successfully"
}
```

**Actions Performed:**
- Updates status to "cancelled"
- Records cancellation reason and timestamp
- Updates payment status
- Sends cancellation email to client
- Creates admin notification

---

## 💰 INVOICE MANAGEMENT ENDPOINTS

### **6. Invoice Operations**

#### `POST /invoices/create`
Create new invoice (admin only).

**Request Body:**
```json
{
  "bookingId": "uuid",
  "clientId": "uuid",
  "clientName": "John Smith",
  "clientEmail": "client@example.com",
  "billingAddress": {
    "addressLine1": "123 High Street",
    "addressLine2": "Flat 4",
    "city": "London",
    "postcode": "SW1A 1AA",
    "country": "United Kingdom"
  },
  "issueDate": "2026-01-21",
  "dueDate": "2026-02-15",
  "items": [
    {
      "description": "Sri Lanka Highlights - 7 Days (2 persons)",
      "quantity": 1,
      "unitPrice": 2998,
      "total": 2998
    },
    {
      "description": "Additional activities package",
      "quantity": 1,
      "unitPrice": 200,
      "total": 200
    }
  ],
  "subtotal": 3198,
  "tax": 159.90,
  "taxRate": 5,
  "discount": 0,
  "totalAmount": 3357.90,
  "currency": "GBP",
  "notes": "Payment terms: 30% deposit, balance due 30 days before travel",
  "termsAndConditions": "Full terms available at www.arizeholidays.com/terms"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Invoice created successfully",
  "data": {
    "invoiceId": "uuid",
    "invoiceNumber": "INV-2026-001",
    "totalAmount": 3357.90
  }
}
```

---

#### `GET /invoices`
Get all invoices (admin only).

**Query Parameters:**
- `clientId` (optional): Filter by client
- `bookingId` (optional): Filter by booking
- `status` (optional): Filter by status
- `limit` (optional): Number of results

**Response:**
```json
{
  "success": true,
  "data": {
    "invoices": [
      {
        "id": "uuid",
        "invoiceNumber": "INV-2026-001",
        "bookingId": "uuid",
        "clientName": "John Smith",
        "issueDate": "2026-01-21",
        "dueDate": "2026-02-15",
        "totalAmount": 3357.90,
        "paidAmount": 1000,
        "remainingAmount": 2357.90,
        "status": "sent"
      }
    ],
    "total": 1
  }
}
```

---

#### `GET /invoices/:invoiceId`
Get specific invoice details.

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "invoiceNumber": "INV-2026-001",
    "bookingId": "uuid",
    "clientName": "John Smith",
    "clientEmail": "client@example.com",
    "billingAddress": {...},
    "issueDate": "2026-01-21",
    "dueDate": "2026-02-15",
    "status": "sent",
    "items": [...],
    "subtotal": 3198,
    "tax": 159.90,
    "totalAmount": 3357.90,
    "paidAmount": 1000,
    "remainingAmount": 2357.90,
    "payments": [
      {
        "date": "2026-01-22",
        "amount": 1000,
        "method": "Bank Transfer",
        "transactionId": "TXN123456",
        "notes": "Deposit payment"
      }
    ],
    "createdAt": "2026-01-21T10:00:00Z"
  }
}
```

---

#### `PUT /invoices/:invoiceId`
Update invoice (admin only).

**Request Body:**
```json
{
  "status": "paid",
  "items": [...],
  "notes": "Updated payment terms"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Invoice updated successfully"
}
```

---

#### `POST /invoices/:invoiceId/send`
Send invoice to client (admin only).

**Request Body:**
```json
{
  "message": "Please find your invoice attached...",
  "sendEmail": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Invoice sent successfully",
  "data": {
    "sentAt": "2026-01-21T11:00:00Z"
  }
}
```

**Actions Performed:**
- Updates status to "sent"
- Records sentAt timestamp
- Sends email with PDF invoice
- Creates client notification

---

#### `POST /invoices/:invoiceId/record-payment`
Record payment against invoice (admin only).

**Request Body:**
```json
{
  "amount": 1000,
  "date": "2026-01-22",
  "method": "Bank Transfer",
  "transactionId": "TXN123456",
  "notes": "Deposit payment received"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Payment recorded successfully",
  "data": {
    "paidAmount": 1000,
    "remainingAmount": 2357.90,
    "status": "sent"
  }
}
```

**Actions Performed:**
- Adds payment to payments array
- Updates paidAmount and remainingAmount
- Updates status to "paid" if fully paid
- Records paidAt timestamp if fully paid
- Updates linked booking's payment status
- Sends payment confirmation email to client

---

#### `GET /invoices/:invoiceId/pdf`
Generate and download invoice PDF.

**Response:**
PDF file download

---

## 🔔 NOTIFICATION ENDPOINTS

### **7. Notification Operations**

#### `POST /notifications/create`
Create notification (system/admin).

**Request Body:**
```json
{
  "userId": "uuid",
  "userType": "client",
  "type": "itinerary",
  "title": "New Itinerary Received",
  "message": "Your custom Sri Lanka itinerary is ready to view",
  "priority": "high",
  "relatedId": "itinerary-uuid",
  "relatedType": "itinerary"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Notification created successfully",
  "data": {
    "notificationId": "uuid"
  }
}
```

---

#### `GET /notifications/user/:userId`
Get user notifications.

**Query Parameters:**
- `status` (optional): "unread" | "read"
- `limit` (optional): Number of results

**Response:**
```json
{
  "success": true,
  "data": {
    "notifications": [
      {
        "id": "uuid",
        "type": "itinerary",
        "title": "New Itinerary Received",
        "message": "Your custom Sri Lanka itinerary is ready to view",
        "status": "unread",
        "priority": "high",
        "relatedId": "itinerary-uuid",
        "relatedType": "itinerary",
        "createdAt": "2026-01-21T09:00:00Z"
      }
    ],
    "total": 1,
    "unreadCount": 1
  }
}
```

---

#### `PUT /notifications/:notificationId/read`
Mark notification as read.

**Response:**
```json
{
  "success": true,
  "message": "Notification marked as read"
}
```

---

#### `POST /notifications/send-to-client`
Send notification to client (admin only).

**Request Body:**
```json
{
  "clientId": "uuid",
  "title": "Special Offer",
  "message": "We have a special offer on beach packages this month!",
  "priority": "medium",
  "sendEmail": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Notification sent to client successfully"
}
```

---

## 📊 DASHBOARD & ANALYTICS ENDPOINTS

### **8. Admin Dashboard**

#### `GET /dashboard/stats`
Get dashboard statistics (admin only).

**Response:**
```json
{
  "success": true,
  "data": {
    "totalClients": 145,
    "activeBookings": 23,
    "pendingItineraries": 8,
    "revenue": {
      "thisMonth": 45600,
      "lastMonth": 38900,
      "total": 234500
    },
    "recentActivity": [
      {
        "type": "booking",
        "message": "New booking from John Smith",
        "timestamp": "2026-01-24T10:30:00Z"
      }
    ]
  }
}
```

---

## 🔒 AUTHENTICATION & AUTHORIZATION

### **Token-Based Authentication**

All protected endpoints require a valid access token in the Authorization header:

```
Authorization: Bearer {accessToken}
```

### **User Roles & Permissions**

**Super Admin:**
- Full access to all endpoints
- User management (create/edit admins)
- System configuration

**Admin:**
- Tour management
- Itinerary management
- Booking management
- Invoice management
- Client communication

**Client:**
- View own bookings
- View own itineraries
- View own invoices
- Update profile
- Respond to itineraries

---

## 📧 EMAIL NOTIFICATIONS

### **Automated Emails**

The system sends automated emails for:

1. **Client Registration**
   - Welcome email with login credentials

2. **Itinerary Sent**
   - Email with secure link to view itinerary
   - Expires after 30 days

3. **Booking Confirmation**
   - Booking details
   - Payment instructions
   - Travel information

4. **Invoice Sent**
   - Invoice PDF attachment
   - Payment instructions

5. **Payment Received**
   - Payment confirmation
   - Updated invoice

6. **Admin Notifications**
   - New client registration
   - Itinerary viewed by client
   - Client responded to itinerary
   - New booking created
   - Payment received

---

## 🔄 WORKFLOW EXAMPLES

### **Example 1: Custom Itinerary Flow**

1. Admin creates itinerary → `POST /itineraries/create`
2. Admin sends to client → `POST /itineraries/:id/send`
3. Client receives email with link
4. Client views itinerary → `GET /itineraries/:id/view?token=xxx`
5. Client responds → `POST /itineraries/:id/respond`
6. If accepted, admin creates booking → `POST /bookings/create`
7. System creates invoice → `POST /invoices/create`
8. Admin sends invoice → `POST /invoices/:id/send`

### **Example 2: Standard Booking Flow**

1. Client selects tour package from website
2. Client creates booking → `POST /bookings/create`
3. System auto-creates invoice
4. Admin confirms booking → `POST /bookings/:id/confirm`
5. Client pays deposit
6. Admin records payment → `POST /invoices/:id/record-payment`
7. Balance payment before travel
8. Admin records final payment
9. Invoice marked as paid
10. Booking confirmed and finalized

---

## 🛠️ ERROR HANDLING

### **Standard Error Response**

```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": {}
  }
}
```

### **Common Error Codes**

- `UNAUTHORIZED` - Invalid or missing authentication token
- `FORBIDDEN` - User doesn't have permission
- `NOT_FOUND` - Resource not found
- `VALIDATION_ERROR` - Invalid request data
- `DUPLICATE_ENTRY` - Resource already exists
- `PAYMENT_ERROR` - Payment processing failed
- `EMAIL_ERROR` - Email sending failed

---

## 🚀 IMPLEMENTATION PRIORITIES (Phase 1)

### **Week 1: Foundation**
1. User authentication (admin & client)
2. Client management CRUD
3. Database structure setup

### **Week 2: Core Features**
1. Tour package management
2. Basic booking system
3. Email notifications

### **Week 3: Itineraries**
1. Itinerary creation
2. Send to client functionality
3. Client viewing & response

### **Week 4: Invoicing**
1. Invoice creation & management
2. Payment recording
3. PDF generation

### **Week 5: Polish & Testing**
1. Admin dashboard
2. Notifications system
3. Testing & bug fixes

---

## 📝 NOTES

- All dates use ISO 8601 format (YYYY-MM-DDTHH:mm:ssZ)
- All prices in GBP (£)
- All endpoints return JSON
- Files (PDFs, images) handled via Supabase Storage
- Email sending via Resend API
- Secure tokens for client itinerary/invoice viewing
- Rate limiting: 100 requests per minute per IP
- File uploads limited to 10MB

---

**Document Version:** 1.0  
**Last Updated:** January 24, 2026  
**Contact:** support@arizeholidays.com
