# ACP Legal Services - Booking & Invoice API Documentation

## Table of Contents
1. [Overview](#overview)
2. [Database Schema](#database-schema)
3. [API Endpoints](#api-endpoints)
4. [Booking Status Management](#booking-status-management)
5. [Invoice Generation](#invoice-generation)
6. [Code Examples](#code-examples)
7. [Error Handling](#error-handling)

---

## Overview

This document provides comprehensive documentation for the ACP Legal Services booking management and invoice generation system. The system uses Supabase backend with a key-value store for data persistence.

**Base URL**: `https://{projectId}.supabase.co/functions/v1/make-server-d8c62521`

**Authentication**: All endpoints (except POST `/bookings` for customer submissions) require Bearer token authentication:
```
Authorization: Bearer {access_token}
```

---

## Database Schema

### Booking Object Structure

Bookings are stored in the KV store with the key pattern: `booking:{timestamp}:{random}`

```json
{
  "id": "booking:1705234567890:abc123",
  "bookingNumber": "BK-LEI-20260114-001",
  "customerName": "John Smith",
  "customerEmail": "john.smith@example.com",
  "customerPhone": "+44 7700 900123",
  "serviceType": "Skilled Worker Visa",
  "consultationType": "in-person",
  "officeLocation": "leicester",
  "preferredDate": "2026-01-20",
  "preferredTime": "10:00",
  "price": "50.00",
  "status": "pending",
  "submissionType": "customer",
  "createdBy": "Customer Portal",
  "createdAt": "2026-01-14T10:30:00.000Z",
  "updatedAt": "2026-01-14T10:30:00.000Z",
  "notes": "First-time consultation",
  "caseDetails": null,
  "invoiceNumber": null,
  "paymentStatus": null,
  "transactionReference": null
}
```

### Booking Status Values

| Status | Description | Can be set by |
|--------|-------------|---------------|
| `pending` | Initial status for new bookings | System (auto) |
| `confirmed` | Booking confirmed by staff | Admin, Staff |
| `completed` | Consultation finished | Admin, Staff |
| `cancelled` | Booking cancelled | Admin, Staff, Customer |
| `no-show` | Customer didn't attend | Admin, Staff |
| `rescheduled` | Booking rescheduled | Admin, Staff |
| `paid` | Payment received | Reception, Admin |

### Submission Types

| Type | Description |
|------|-------------|
| `customer` | Direct customer booking via website |
| `staff` | Created by staff member |

### Office Locations

| Code | Name |
|------|------|
| `leicester` | Leicester Office |
| `newcastle-under-lyme` | Newcastle-under-Lyme Office |
| `london` | London Office (if applicable) |

### Booking Number Format

Format: `BK-{BRANCH_CODE}-{YYYYMMDD}-{SEQUENCE}`

Examples:
- `BK-LEI-20260114-001` (Leicester)
- `BK-NEW-20260114-002` (Newcastle-under-Lyme)

---

## API Endpoints

### 1. Create Booking

**POST** `/bookings`

Creates a new booking. This endpoint is PUBLIC for customer bookings.

**Request Body:**
```json
{
  "customerName": "John Smith",
  "customerEmail": "john.smith@example.com",
  "customerPhone": "+44 7700 900123",
  "serviceType": "Skilled Worker Visa",
  "consultationType": "in-person",
  "officeLocation": "leicester",
  "preferredDate": "2026-01-20",
  "preferredTime": "10:00",
  "price": "50.00",
  "notes": "First-time consultation",
  "bookingSource": "customer"
}
```

**Response (201 Created):**
```json
{
  "message": "Booking created successfully",
  "booking": {
    "id": "booking:1705234567890:abc123",
    "bookingNumber": "BK-LEI-20260114-001",
    "customerName": "John Smith",
    "customerEmail": "john.smith@example.com",
    "customerPhone": "+44 7700 900123",
    "serviceType": "Skilled Worker Visa",
    "consultationType": "in-person",
    "officeLocation": "leicester",
    "preferredDate": "2026-01-20",
    "preferredTime": "10:00",
    "price": "50.00",
    "status": "pending",
    "submissionType": "customer",
    "createdBy": "Customer Portal",
    "createdAt": "2026-01-14T10:30:00.000Z",
    "updatedAt": "2026-01-14T10:30:00.000Z",
    "notes": "First-time consultation"
  }
}
```

**Frontend Example:**
```javascript
const response = await fetch(
  `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${publicAnonKey}`, // Public key for customer bookings
    },
    body: JSON.stringify(bookingData),
  }
);

const result = await response.json();
console.log('Booking created:', result.booking.bookingNumber);
```

---

### 2. Get All Bookings

**GET** `/bookings`

Retrieves all bookings (requires authentication).

**Headers:**
```
Authorization: Bearer {staff_access_token}
```

**Response (200 OK):**
```json
{
  "bookings": [
    {
      "id": "booking:1705234567890:abc123",
      "bookingNumber": "BK-LEI-20260114-001",
      "customerName": "John Smith",
      "status": "pending",
      "createdAt": "2026-01-14T10:30:00.000Z",
      // ... full booking object
    },
    {
      "id": "booking:1705234567891:def456",
      "bookingNumber": "BK-LEI-20260114-002",
      "customerName": "Jane Doe",
      "status": "confirmed",
      "createdAt": "2026-01-14T11:00:00.000Z",
      // ... full booking object
    }
  ]
}
```

**Frontend Example:**
```javascript
const fetchBookings = async () => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings`,
    {
      headers: {
        'Authorization': `Bearer ${accessToken}`,
      },
    }
  );

  const data = await response.json();
  return data.bookings;
};
```

---

### 3. Get Single Booking

**GET** `/bookings/:id`

Retrieves a single booking by ID or booking number.

**Parameters:**
- `id`: Booking ID or booking number

**Headers:**
```
Authorization: Bearer {staff_access_token}
```

**Response (200 OK):**
```json
{
  "booking": {
    "id": "booking:1705234567890:abc123",
    "bookingNumber": "BK-LEI-20260114-001",
    "customerName": "John Smith",
    "customerEmail": "john.smith@example.com",
    "status": "confirmed",
    // ... full booking details
  }
}
```

**Error Response (404 Not Found):**
```json
{
  "error": "Booking not found"
}
```

---

### 4. Update Booking (Full Update)

**PUT** `/bookings/:id`

Updates a booking (requires authentication).

**Request Body:**
```json
{
  "status": "confirmed",
  "notes": "Updated consultation notes",
  "preferredDate": "2026-01-21",
  "preferredTime": "14:00"
}
```

**Response (200 OK):**
```json
{
  "message": "Booking updated successfully",
  "booking": {
    "id": "booking:1705234567890:abc123",
    "bookingNumber": "BK-LEI-20260114-001",
    "status": "confirmed",
    "updatedAt": "2026-01-14T12:00:00.000Z",
    "updatedBy": "user-id-123",
    // ... updated booking object
  }
}
```

**Frontend Example:**
```javascript
const updateBooking = async (bookingId, updates) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
      },
      body: JSON.stringify(updates),
    }
  );

  return await response.json();
};
```

---

### 5. Update Booking Status (Confirm)

**POST** `/bookings/:id/confirm`

Confirms a booking and sets status to 'confirmed'.

**Request Body:**
```json
{
  "confirmedDate": "2026-01-20",
  "confirmedTime": "10:00",
  "notes": "Booking confirmed via phone"
}
```

**Response (200 OK):**
```json
{
  "message": "Booking confirmed successfully",
  "booking": {
    "id": "booking:1705234567890:abc123",
    "status": "confirmed",
    "confirmedDate": "2026-01-20",
    "confirmedTime": "10:00",
    "updatedAt": "2026-01-14T12:00:00.000Z"
  }
}
```

---

### 6. Update Case Details

**PUT** `/bookings/:id/case-details`

Updates case-specific details for a booking.

**Request Body:**
```json
{
  "caseDetails": {
    "caseType": "Skilled Worker Visa",
    "applicantName": "John Smith",
    "nationality": "Indian",
    "currentVisaStatus": "Student Visa",
    "employerDetails": "Tech Corp Ltd",
    "salaryOffered": "35000",
    "jobTitle": "Software Developer",
    "additionalNotes": "Applicant has 3 years experience"
  }
}
```

**Response (200 OK):**
```json
{
  "message": "Case details updated successfully",
  "booking": {
    "id": "booking:1705234567890:abc123",
    "caseDetails": {
      "caseType": "Skilled Worker Visa",
      "applicantName": "John Smith",
      // ... full case details
    },
    "updatedAt": "2026-01-14T12:00:00.000Z"
  }
}
```

---

### 7. Generate Invoice

**PUT** `/bookings/:id/invoice`

Generates and saves an invoice number for a booking.

**Request Body:**
```json
{
  "invoiceNumber": "INV-LEI-20260114-001"
}
```

**Response (200 OK):**
```json
{
  "message": "Invoice number saved successfully",
  "booking": {
    "id": "booking:1705234567890:abc123",
    "invoiceNumber": "INV-LEI-20260114-001",
    "updatedAt": "2026-01-14T12:00:00.000Z"
  }
}
```

**Frontend Example (with PDF generation):**
```javascript
const generateInvoice = async (booking) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  // 1. Generate invoice number
  const today = new Date().toISOString().split('T')[0].replace(/-/g, '');
  const randomNum = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
  const branchCode = booking.officeLocation === 'leicester' ? 'LEI' : 'NEW';
  const invoiceNumber = `INV-${branchCode}-${today}-${randomNum}`;
  
  // 2. Create PDF (using jsPDF)
  const doc = new jsPDF();
  
  // Add company header
  doc.setFontSize(20);
  doc.text('ACP Legal Services', 20, 20);
  doc.setFontSize(10);
  doc.text('Immigration Law Specialists', 20, 28);
  
  // Add invoice details
  doc.setFontSize(16);
  doc.text(`Invoice: ${invoiceNumber}`, 20, 50);
  doc.setFontSize(10);
  doc.text(`Date: ${new Date().toLocaleDateString('en-GB')}`, 20, 60);
  doc.text(`Booking Number: ${booking.bookingNumber}`, 20, 68);
  
  // Add customer details
  doc.text('Bill To:', 20, 85);
  doc.text(booking.customerName, 20, 93);
  doc.text(booking.customerEmail, 20, 101);
  doc.text(booking.customerPhone, 20, 109);
  
  // Add service details (table)
  doc.text('Service Details:', 20, 130);
  doc.rect(20, 135, 170, 30);
  doc.text('Service', 25, 143);
  doc.text('Amount', 150, 143);
  doc.line(20, 145, 190, 145);
  doc.text(booking.serviceType, 25, 155);
  doc.text(`£${parseFloat(booking.price).toFixed(2)}`, 150, 155);
  
  // Add total
  doc.setFontSize(12);
  doc.text('Total:', 130, 175);
  doc.text(`£${parseFloat(booking.price).toFixed(2)}`, 150, 175);
  
  // Add footer
  doc.setFontSize(8);
  doc.text('Thank you for choosing ACP Legal Services', 20, 280);
  
  // 3. Download PDF
  doc.save(`Invoice-${invoiceNumber}.pdf`);
  
  // 4. Save invoice number to database
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${booking.id}/invoice`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
      },
      body: JSON.stringify({ invoiceNumber }),
    }
  );
  
  return await response.json();
};
```

---

### 8. Delete Booking

**DELETE** `/bookings/:id`

Deletes a booking (requires authentication).

**Response (200 OK):**
```json
{
  "message": "Booking deleted successfully"
}
```

**Frontend Example:**
```javascript
const deleteBooking = async (bookingId) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
    {
      method: 'DELETE',
      headers: {
        'Authorization': `Bearer ${accessToken}`,
      },
    }
  );

  return await response.json();
};
```

---

## Booking Status Management

### Status Workflow

```
                    ┌─────────────┐
                    │   PENDING   │ (Initial)
                    └──────┬──────┘
                           │
                ┌──────────┼──────────┐
                │          │          │
         ┌──────▼─────┐   │   ┌──────▼──────┐
         │ CONFIRMED  │   │   │  CANCELLED  │
         └──────┬─────┘   │   └─────────────┘
                │          │
         ┌──────┼─────┐   │
         │      │     │    │
    ┌────▼───┐ │  ┌──▼────▼───┐
    │ PAID   │ │  │ RESCHEDULED│
    └────┬───┘ │  └────────────┘
         │     │
    ┌────▼─────▼─────┐
    │   COMPLETED    │
    └────────┬───────┘
             │
      ┌──────▼──────┐
      │   NO-SHOW   │
      └─────────────┘
```

### Status Change Examples

#### Example 1: Confirm a Booking
```javascript
const confirmBooking = async (bookingId) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
      },
      body: JSON.stringify({
        status: 'confirmed',
        confirmedAt: new Date().toISOString(),
        notes: 'Booking confirmed via email'
      }),
    }
  );

  const result = await response.json();
  console.log('Booking confirmed:', result.booking.bookingNumber);
};
```

#### Example 2: Mark as Paid (Reception)
```javascript
const markAsPaid = async (bookingId, transactionRef) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
      },
      body: JSON.stringify({
        status: 'paid',
        paymentStatus: 'paid',
        transactionReference: transactionRef,
        paidAt: new Date().toISOString()
      }),
    }
  );

  return await response.json();
};
```

#### Example 3: Cancel a Booking
```javascript
const cancelBooking = async (bookingId, reason) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
      },
      body: JSON.stringify({
        status: 'cancelled',
        cancellationReason: reason,
        cancelledAt: new Date().toISOString()
      }),
    }
  );

  return await response.json();
};
```

#### Example 4: Mark as Completed
```javascript
const completeBooking = async (bookingId, completionNotes) => {
  const accessToken = localStorage.getItem('staff_access_token');
  
  const response = await fetch(
    `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
    {
      method: 'PUT',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': `Bearer ${accessToken}`,
      },
      body: JSON.stringify({
        status: 'completed',
        completionNotes: completionNotes,
        completedAt: new Date().toISOString()
      }),
    }
  );

  return await response.json();
};
```

---

## Invoice Generation

### Invoice Number Format

Format: `INV-{BRANCH_CODE}-{YYYYMMDD}-{SEQUENCE}`

Examples:
- `INV-LEI-20260114-001` (Leicester)
- `INV-NEW-20260114-002` (Newcastle-under-Lyme)

### Invoice Generation Process

#### Step 1: Generate Invoice Number
```javascript
const generateInvoiceNumber = (officeLocation) => {
  const today = new Date().toISOString().split('T')[0].replace(/-/g, '');
  const randomNum = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
  const branchCode = officeLocation === 'leicester' ? 'LEI' : 
                     officeLocation === 'newcastle-under-lyme' ? 'NEW' : 'UNK';
  return `INV-${branchCode}-${today}-${randomNum}`;
};
```

#### Step 2: Create PDF Invoice (using jsPDF)
```javascript
import jsPDF from 'jspdf';

const createInvoicePDF = (booking, invoiceNumber) => {
  const doc = new jsPDF();
  
  // Company Header
  doc.setFontSize(22);
  doc.setFont('helvetica', 'bold');
  doc.text('ACP LEGAL SERVICES', 105, 25, { align: 'center' });
  
  doc.setFontSize(10);
  doc.setFont('helvetica', 'normal');
  doc.text('Immigration Law Specialists', 105, 32, { align: 'center' });
  doc.text('Regulated by OISC', 105, 38, { align: 'center' });
  
  // Office Address
  doc.setFontSize(9);
  if (booking.officeLocation === 'leicester') {
    doc.text('Leicester Office: 123 High Street, Leicester, LE1 1AA', 105, 45, { align: 'center' });
    doc.text('Tel: 0116 123 4567', 105, 50, { align: 'center' });
  } else {
    doc.text('Newcastle Office: 456 Main Road, Newcastle-under-Lyme, ST5 1XX', 105, 45, { align: 'center' });
    doc.text('Tel: 01782 123 4567', 105, 50, { align: 'center' });
  }
  
  // Horizontal Line
  doc.setLineWidth(0.5);
  doc.line(20, 55, 190, 55);
  
  // Invoice Title
  doc.setFontSize(18);
  doc.setFont('helvetica', 'bold');
  doc.text('INVOICE', 20, 68);
  
  // Invoice Details
  doc.setFontSize(10);
  doc.setFont('helvetica', 'normal');
  doc.text(`Invoice Number: ${invoiceNumber}`, 20, 78);
  doc.text(`Booking Number: ${booking.bookingNumber}`, 20, 85);
  doc.text(`Date: ${new Date().toLocaleDateString('en-GB', { 
    day: '2-digit', 
    month: 'long', 
    year: 'numeric' 
  })}`, 20, 92);
  
  // Client Details Box
  doc.setFontSize(11);
  doc.setFont('helvetica', 'bold');
  doc.text('BILL TO:', 20, 108);
  
  doc.setFont('helvetica', 'normal');
  doc.setFontSize(10);
  doc.text(booking.customerName, 20, 116);
  doc.text(booking.customerEmail, 20, 123);
  doc.text(booking.customerPhone, 20, 130);
  
  // Service Details Table Header
  doc.setFillColor(41, 128, 185);
  doc.rect(20, 145, 170, 10, 'F');
  
  doc.setTextColor(255, 255, 255);
  doc.setFont('helvetica', 'bold');
  doc.text('Description', 25, 151);
  doc.text('Type', 110, 151);
  doc.text('Amount', 165, 151);
  
  // Service Details Row
  doc.setTextColor(0, 0, 0);
  doc.setFont('helvetica', 'normal');
  doc.rect(20, 155, 170, 20);
  doc.text(booking.serviceType, 25, 165);
  doc.text(booking.consultationType === 'in-person' ? 'In-Person' : 
           booking.consultationType === 'video' ? 'Video Call' : 'Phone', 110, 165);
  doc.text(`£${parseFloat(booking.price).toFixed(2)}`, 165, 165);
  
  // Consultation Details
  if (booking.preferredDate && booking.preferredTime) {
    doc.setFontSize(9);
    doc.setTextColor(100, 100, 100);
    doc.text(`Consultation Date: ${new Date(booking.preferredDate).toLocaleDateString('en-GB')}`, 25, 172);
    doc.text(`Time: ${booking.preferredTime}`, 110, 172);
  }
  
  // Subtotal and Total
  doc.setDrawColor(200, 200, 200);
  doc.line(130, 180, 190, 180);
  
  doc.setFontSize(10);
  doc.setTextColor(0, 0, 0);
  doc.text('Subtotal:', 135, 188);
  doc.text(`£${parseFloat(booking.price).toFixed(2)}`, 165, 188);
  
  doc.text('VAT (Exempt):', 135, 195);
  doc.text('£0.00', 165, 195);
  
  doc.setLineWidth(0.5);
  doc.line(130, 200, 190, 200);
  
  doc.setFontSize(12);
  doc.setFont('helvetica', 'bold');
  doc.text('TOTAL:', 135, 210);
  doc.text(`£${parseFloat(booking.price).toFixed(2)}`, 165, 210);
  
  // Payment Status
  doc.setFontSize(10);
  doc.setFont('helvetica', 'normal');
  if (booking.paymentStatus === 'paid') {
    doc.setFillColor(39, 174, 96);
    doc.rect(20, 220, 60, 8, 'F');
    doc.setTextColor(255, 255, 255);
    doc.text('PAID', 35, 226);
    
    if (booking.transactionReference) {
      doc.setTextColor(0, 0, 0);
      doc.setFontSize(9);
      doc.text(`Transaction Ref: ${booking.transactionReference}`, 20, 235);
    }
  } else {
    doc.setFillColor(231, 76, 60);
    doc.rect(20, 220, 60, 8, 'F');
    doc.setTextColor(255, 255, 255);
    doc.text('UNPAID', 30, 226);
  }
  
  // Payment Instructions (if unpaid)
  if (booking.paymentStatus !== 'paid') {
    doc.setTextColor(0, 0, 0);
    doc.setFontSize(9);
    doc.setFont('helvetica', 'bold');
    doc.text('Payment Instructions:', 20, 245);
    
    doc.setFont('helvetica', 'normal');
    doc.text('Payment can be made by:', 20, 252);
    doc.text('• Cash or card at our office', 25, 258);
    doc.text('• Bank transfer to: Sort Code: 12-34-56, Account: 12345678', 25, 264);
    doc.text('• Please quote your booking number as reference', 25, 270);
  }
  
  // Footer
  doc.setFontSize(8);
  doc.setTextColor(100, 100, 100);
  doc.text('Thank you for choosing ACP Legal Services', 105, 285, { align: 'center' });
  doc.text('www.acplegalservices.co.uk | info@acplegalservices.co.uk', 105, 290, { align: 'center' });
  
  return doc;
};
```

#### Step 3: Complete Invoice Generation Flow
```javascript
const generateAndDownloadInvoice = async (booking) => {
  try {
    const accessToken = localStorage.getItem('staff_access_token');
    
    // 1. Generate invoice number
    const invoiceNumber = generateInvoiceNumber(booking.officeLocation);
    
    // 2. Create PDF
    const doc = createInvoicePDF(booking, invoiceNumber);
    
    // 3. Download PDF
    doc.save(`Invoice-${invoiceNumber}.pdf`);
    
    // 4. Save invoice number to database
    const response = await fetch(
      `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${booking.id}/invoice`,
      {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${accessToken}`,
        },
        body: JSON.stringify({ invoiceNumber }),
      }
    );
    
    if (!response.ok) {
      throw new Error('Failed to save invoice number');
    }
    
    const result = await response.json();
    console.log('Invoice generated:', invoiceNumber);
    
    return {
      success: true,
      invoiceNumber,
      booking: result.booking
    };
    
  } catch (error) {
    console.error('Invoice generation error:', error);
    return {
      success: false,
      error: error.message
    };
  }
};
```

---

## Code Examples

### Complete Booking Management Component

```javascript
import { useState, useEffect } from 'react';
import { projectId } from '@/utils/supabase/info';
import { toast } from 'sonner';
import jsPDF from 'jspdf';

export function BookingManagement() {
  const [bookings, setBookings] = useState([]);
  const [loading, setLoading] = useState(true);
  const [selectedBooking, setSelectedBooking] = useState(null);

  // Fetch all bookings
  const fetchBookings = async () => {
    const accessToken = localStorage.getItem('staff_access_token');
    
    try {
      const response = await fetch(
        `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings`,
        {
          headers: {
            'Authorization': `Bearer ${accessToken}`,
          },
        }
      );

      if (response.ok) {
        const data = await response.json();
        setBookings(data.bookings);
      } else {
        toast.error('Failed to load bookings');
      }
    } catch (error) {
      console.error('Error fetching bookings:', error);
      toast.error('Error loading bookings');
    } finally {
      setLoading(false);
    }
  };

  // Update booking status
  const updateBookingStatus = async (bookingId, newStatus) => {
    const accessToken = localStorage.getItem('staff_access_token');
    
    try {
      const response = await fetch(
        `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${bookingId}`,
        {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${accessToken}`,
          },
          body: JSON.stringify({ 
            status: newStatus,
            updatedAt: new Date().toISOString()
          }),
        }
      );

      if (response.ok) {
        toast.success(`Booking ${newStatus}`);
        fetchBookings(); // Refresh list
      } else {
        toast.error('Failed to update booking');
      }
    } catch (error) {
      console.error('Error updating booking:', error);
      toast.error('Error updating booking');
    }
  };

  // Generate invoice
  const handleGenerateInvoice = async (booking) => {
    const accessToken = localStorage.getItem('staff_access_token');
    
    try {
      // Generate invoice number
      const today = new Date().toISOString().split('T')[0].replace(/-/g, '');
      const randomNum = Math.floor(Math.random() * 1000).toString().padStart(3, '0');
      const branchCode = booking.officeLocation === 'leicester' ? 'LEI' : 'NEW';
      const invoiceNumber = `INV-${branchCode}-${today}-${randomNum}`;
      
      // Create PDF
      const doc = new jsPDF();
      // ... (use the createInvoicePDF function from above)
      
      // Download PDF
      doc.save(`Invoice-${invoiceNumber}.pdf`);
      
      // Save to database
      const response = await fetch(
        `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${booking.id}/invoice`,
        {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${accessToken}`,
          },
          body: JSON.stringify({ invoiceNumber }),
        }
      );
      
      if (response.ok) {
        toast.success('Invoice generated successfully');
        fetchBookings();
      }
    } catch (error) {
      console.error('Error generating invoice:', error);
      toast.error('Failed to generate invoice');
    }
  };

  useEffect(() => {
    fetchBookings();
  }, []);

  return (
    <div className="p-6">
      <h1 className="text-2xl font-bold mb-6">Booking Management</h1>
      
      {/* Bookings Table */}
      <table className="w-full">
        <thead>
          <tr className="border-b">
            <th className="text-left p-3">Booking #</th>
            <th className="text-left p-3">Customer</th>
            <th className="text-left p-3">Service</th>
            <th className="text-left p-3">Status</th>
            <th className="text-left p-3">Actions</th>
          </tr>
        </thead>
        <tbody>
          {bookings.map((booking) => (
            <tr key={booking.id} className="border-b hover:bg-gray-50">
              <td className="p-3">{booking.bookingNumber}</td>
              <td className="p-3">{booking.customerName}</td>
              <td className="p-3">{booking.serviceType}</td>
              <td className="p-3">
                <select
                  value={booking.status}
                  onChange={(e) => updateBookingStatus(booking.id, e.target.value)}
                  className="border rounded px-2 py-1"
                >
                  <option value="pending">Pending</option>
                  <option value="confirmed">Confirmed</option>
                  <option value="paid">Paid</option>
                  <option value="completed">Completed</option>
                  <option value="cancelled">Cancelled</option>
                </select>
              </td>
              <td className="p-3">
                <button
                  onClick={() => handleGenerateInvoice(booking)}
                  className="bg-blue-600 text-white px-3 py-1 rounded hover:bg-blue-700"
                >
                  Generate Invoice
                </button>
              </td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

## Error Handling

### Common Error Responses

#### 401 Unauthorized
```json
{
  "error": "Unauthorized - No access token"
}
```

**Solution**: Ensure valid access token is provided in Authorization header.

#### 404 Not Found
```json
{
  "error": "Booking not found"
}
```

**Solution**: Verify the booking ID or booking number is correct.

#### 500 Internal Server Error
```json
{
  "error": "Internal server error while creating booking",
  "details": "Additional error information",
  "type": "TypeError"
}
```

**Solution**: Check server logs for detailed error information.

### Error Handling Best Practices

```javascript
const handleBookingOperation = async () => {
  try {
    const response = await fetch(endpoint, options);
    const data = await response.json();
    
    if (!response.ok) {
      // Handle HTTP errors
      if (response.status === 401) {
        toast.error('Session expired. Please login again.');
        // Redirect to login
      } else if (response.status === 404) {
        toast.error('Booking not found');
      } else {
        toast.error(data.error || 'An error occurred');
      }
      return;
    }
    
    // Success
    toast.success('Operation successful');
    return data;
    
  } catch (error) {
    // Handle network errors
    console.error('Network error:', error);
    toast.error('Network error. Please check your connection.');
  }
};
```

---

## Database Query Examples

### Get All Pending Bookings
```javascript
const bookings = await kv.getByPrefix('booking:');
const pendingBookings = bookings.filter(b => b.status === 'pending');
```

### Get Bookings by Date
```javascript
const bookings = await kv.getByPrefix('booking:');
const todayBookings = bookings.filter(b => 
  b.preferredDate === '2026-01-14'
);
```

### Get Bookings by Office
```javascript
const bookings = await kv.getByPrefix('booking:');
const leicesterBookings = bookings.filter(b => 
  b.officeLocation === 'leicester'
);
```

### Calculate Revenue
```javascript
const bookings = await kv.getByPrefix('booking:');
const paidBookings = bookings.filter(b => b.status === 'paid');
const totalRevenue = paidBookings.reduce((sum, b) => 
  sum + parseFloat(b.price), 0
);
```

### Get Bookings Created Today
```javascript
const bookings = await kv.getByPrefix('booking:');
const today = new Date().toISOString().split('T')[0];
const todayBookings = bookings.filter(b => 
  b.createdAt.startsWith(today)
);
```

---

## Testing

### Test Data Example

```javascript
// Create test booking
const testBooking = {
  customerName: "Test Customer",
  customerEmail: "test@example.com",
  customerPhone: "+44 7700 900000",
  serviceType: "Skilled Worker Visa",
  consultationType: "in-person",
  officeLocation: "leicester",
  preferredDate: "2026-01-20",
  preferredTime: "10:00",
  price: "50.00",
  bookingSource: "customer"
};

// Test creating a booking
fetch(`https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': `Bearer ${publicAnonKey}`,
  },
  body: JSON.stringify(testBooking)
})
.then(res => res.json())
.then(data => console.log('Created:', data))
.catch(err => console.error('Error:', err));
```

---

## Appendix

### Service Types
- Skilled Worker Visa
- Student Visa
- Family Visa
- Visit Visa
- Settlement/ILR
- Citizenship
- Other Immigration Matters

### Consultation Types
- `in-person`: In-Person Consultation (£50)
- `video`: Video Consultation (£50)
- `phone`: Phone Consultation (£60)

### Office Locations
- `leicester`: Leicester Office
- `newcastle-under-lyme`: Newcastle-under-Lyme Office

---

**Last Updated**: January 14, 2026  
**Version**: 1.0  
**Maintained By**: ACP Legal Services Development Team
