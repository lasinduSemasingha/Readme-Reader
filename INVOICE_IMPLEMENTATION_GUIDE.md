# Invoice Generation System - Implementation Guide

## Overview
This document outlines the **actual implementation** used in Admin and Staff dashboards for invoice generation at ACP Legal Services.

---

## Database Structure

### Booking Object Schema

All bookings are stored in the KV store with key pattern: `booking:{timestamp}:{random}`

**Invoice-Related Fields:**
```typescript
{
  // Core booking fields
  id: string;                           // "booking:1705234567890:abc123"
  bookingNumber: string;                // "BK-LEI-20260114-001"
  customerName: string;
  customerEmail: string;
  customerPhone: string;
  serviceType: string;
  consultationType: string;             // "in-person" | "video" | "phone"
  officeLocation: string;               // "leicester" | "newcastle-under-lyme"
  appointmentDate: string;              // "2026-01-20"
  appointmentTime: string;              // "10:00"
  price: string;                        // "50.00"
  status: string;                       // "pending" | "confirmed" | "completed" | "cancelled"
  
  // Payment fields
  paymentStatus: string;                // "pending" | "paid" | "failed"
  transactionReference?: string;        // Transaction ref from payment
  paymentUpdatedAt?: string;
  paymentUpdatedBy?: string;
  
  // Invoice fields
  invoiceNumber?: string;               // "INV-20260114-1234"
  invoiceGeneratedAt?: string;          // ISO timestamp
  invoiceGeneratedBy?: string;          // User ID who generated
  
  // Customer details (optional)
  customerNationality?: string;
  customerStreetAddress?: string;
  customerCity?: string;
  customerCounty?: string;
  customerPostcode?: string;
  customerCountry?: string;
  
  // Timestamps
  createdAt: string;
  updatedAt: string;
  createdBy?: string;
  updatedBy?: string;
}
```

---

## Backend API

### Base URL
```
https://{projectId}.supabase.co/functions/v1/make-server-d8c62521
```

### 1. Save Invoice Number API

**Endpoint:** `PUT /bookings/:id/invoice`

**Authentication:** Required (Bearer token)

**Request:**
```typescript
{
  invoiceNumber: string;          // Required: e.g., "INV-20260114-1234"
  invoiceGeneratedAt?: string;    // Optional: ISO timestamp, defaults to now
}
```

**Response (Success - 200):**
```json
{
  "message": "Invoice number saved successfully",
  "booking": {
    "id": "booking:1705234567890:abc123",
    "invoiceNumber": "INV-20260114-1234",
    "invoiceGeneratedAt": "2026-01-14T10:30:00.000Z",
    "invoiceGeneratedBy": "user-abc-123",
    // ... rest of booking object
  }
}
```

**Error Responses:**
```json
// 401 - No authentication
{ "error": "Unauthorized - No access token" }

// 400 - Missing invoice number
{ "error": "Invoice number is required" }

// 400 - Booking not paid
{ "error": "Invoice can only be generated for paid bookings" }

// 404 - Booking not found
{ "error": "Booking not found" }
```

**Validation Rules:**
- ✅ Must be authenticated
- ✅ Invoice number is required
- ✅ Booking must exist
- ✅ Booking `paymentStatus` MUST be `"paid"`

**Backend Implementation:**
```typescript
// File: /supabase/functions/server/index.tsx (lines 1493-1546)

app.put("/make-server-d8c62521/bookings/:id/invoice", async (c) => {
  // 1. Validate authentication
  const accessToken = c.req.header('Authorization')?.split(' ')[1];
  if (!accessToken) {
    return c.json({ error: 'Unauthorized - No access token' }, 401);
  }

  // 2. Verify user session
  const { data: { user }, error: authError } = await supabase.auth.getUser(accessToken);
  if (authError || !user) {
    return c.json({ error: 'Unauthorized - Invalid session' }, 401);
  }

  // 3. Get request data
  const id = c.req.param('id');
  const { invoiceNumber, invoiceGeneratedAt } = await c.req.json();

  // 4. Validate invoice number
  if (!invoiceNumber) {
    return c.json({ error: 'Invoice number is required' }, 400);
  }

  // 5. Get existing booking
  const existingBooking = await kv.get(id);
  if (!existingBooking) {
    return c.json({ error: 'Booking not found' }, 404);
  }

  // 6. Validate payment status
  if (existingBooking.paymentStatus !== 'paid') {
    return c.json({ error: 'Invoice can only be generated for paid bookings' }, 400);
  }

  // 7. Update booking with invoice info
  const updatedBooking = {
    ...existingBooking,
    invoiceNumber: invoiceNumber,
    invoiceGeneratedAt: invoiceGeneratedAt || new Date().toISOString(),
    invoiceGeneratedBy: user.id,
    updatedAt: new Date().toISOString(),
  };

  // 8. Save to database
  await kv.set(id, updatedBooking);

  // 9. Return success
  return c.json({ 
    message: 'Invoice number saved successfully',
    booking: updatedBooking
  });
});
```

---

## Frontend Implementation

### Invoice Number Generation Logic

**Format:** `INV-{YYYYMMDD}-{RANDOM_4_DIGIT}`

**Example:** `INV-20260114-1234`

```typescript
// Generate unique invoice number
const generateInvoiceNumber = () => {
  const datePrefix = new Date().toISOString().slice(0, 10).replace(/-/g, '');
  const randomSuffix = Math.floor(1000 + Math.random() * 9000);
  return `INV-${datePrefix}-${randomSuffix}`;
};
```

### Invoice Generation Flow

**File:** `/src/app/pages/AdminDashboard.tsx` (lines 1347-1663)

#### Step 1: Validation
```typescript
const generateInvoice = async (booking: Booking) => {
  // Check if payment is done
  if (booking.paymentStatus !== 'paid') {
    toast.error('Invoice can only be generated for paid bookings');
    return;
  }
  
  // Use existing invoice number or generate new one
  let invoiceNumber = booking.invoiceNumber;
  
  if (!invoiceNumber) {
    invoiceNumber = generateInvoiceNumber();
    
    // Save invoice number to database
    try {
      const response = await fetch(
        `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/bookings/${booking.id}/invoice`,
        {
          method: 'PUT',
          headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${accessToken}`,
          },
          body: JSON.stringify({
            invoiceNumber: invoiceNumber,
            invoiceGeneratedAt: new Date().toISOString()
          }),
        }
      );

      if (!response.ok) {
        throw new Error('Failed to save invoice number');
      }

      await fetchBookings(); // Refresh list
      toast.success('Invoice number generated and saved');
    } catch (error: any) {
      console.error('Error saving invoice number:', error);
      toast.error('Failed to save invoice number');
      return;
    }
  }
  
  // Continue to Step 2...
};
```

#### Step 2: Generate HTML Invoice
```typescript
const invoiceHTML = `
  <!DOCTYPE html>
  <html>
  <head>
    <meta charset="UTF-8">
    <title>Invoice - ${booking.bookingNumber}</title>
    <style>
      /* Company branding styles */
      body {
        font-family: Arial, sans-serif;
        max-width: 800px;
        margin: 0 auto;
        padding: 40px 20px;
        color: #333;
      }
      .header {
        text-align: center;
        margin-bottom: 40px;
        border-bottom: 3px solid #0B1F3B;
        padding-bottom: 20px;
      }
      .company-name {
        font-size: 28px;
        font-weight: bold;
        color: #0B1F3B;
      }
      .invoice-title {
        font-size: 32px;
        font-weight: bold;
        color: #0B1F3B;
      }
      /* Additional styles... */
    </style>
  </head>
  <body>
    <div class="header">
      <img src="${companyLogo}" alt="ACP Legal Services" class="company-logo" />
      <div class="company-name">ACP LEGAL SERVICES</div>
      <div class="company-tagline">Expert immigration services tailored to your unique journey to the UK</div>
      <div class="invoice-title">INVOICE</div>
    </div>

    <div class="invoice-meta">
      <div class="invoice-meta-section">
        <h3>FROM:</h3>
        <p><strong>ACP Legal Services</strong></p>
        <p>${booking.officeLocation === 'leicester' ? 
          'Leicester Office<br>Business Address<br>Leicester, UK' : 
          'Newcastle-Under-Lyme Office<br>Business Address<br>Newcastle-Under-Lyme, UK'
        }</p>
      </div>
      <div class="invoice-meta-section">
        <h3>TO:</h3>
        <p><strong>${booking.customerName}</strong></p>
        <p>Email: ${booking.customerEmail}</p>
        <p>Phone: ${booking.customerPhone}</p>
      </div>
      <div class="invoice-meta-section">
        <h3>INVOICE DETAILS:</h3>
        <p><strong>Invoice #:</strong> ${invoiceNumber}</p>
        <p><strong>Booking #:</strong> ${booking.bookingNumber}</p>
        <p><strong>Invoice Date:</strong> ${new Date().toLocaleDateString('en-GB')}</p>
      </div>
    </div>

    <table class="details-table">
      <thead>
        <tr>
          <th>Description</th>
          <th>Date</th>
          <th>Location</th>
          <th style="text-align: right;">Amount</th>
        </tr>
      </thead>
      <tbody>
        <tr>
          <td>${booking.serviceType}</td>
          <td>${new Date(booking.appointmentDate).toLocaleDateString('en-GB')}</td>
          <td>${booking.officeLocation === 'leicester' ? 'Leicester' : 'Newcastle-Under-Lyme'}</td>
          <td style="text-align: right;">£${parseFloat(booking.price).toFixed(2)}</td>
        </tr>
      </tbody>
    </table>

    <div class="totals">
      <div class="total-row">
        <span>Subtotal:</span>
        <span>£${parseFloat(booking.price).toFixed(2)}</span>
      </div>
      <div class="total-row grand-total">
        <span>TOTAL:</span>
        <span>£${parseFloat(booking.price).toFixed(2)}</span>
      </div>
    </div>

    ${booking.paymentStatus === 'paid' ? `
      <div class="payment-info paid">
        <h3>✓ PAYMENT RECEIVED</h3>
        <p><strong>Transaction Reference:</strong> ${booking.transactionReference || 'N/A'}</p>
        <p><strong>Payment Date:</strong> ${new Date(booking.paymentUpdatedAt).toLocaleDateString('en-GB')}</p>
        <p>Thank you for your payment!</p>
      </div>
    ` : ''}

    <div class="footer">
      <p><strong>ACP Legal Services</strong></p>
      <p>Email: info@acplegalservices.co.uk | Website: www.acplegalservices.co.uk</p>
      <p>IAA Registration Number: F202000156</p>
    </div>
  </body>
  </html>
`;
```

#### Step 3: Print/Download Invoice
```typescript
// Open in new window and trigger print
const printWindow = window.open('', '_blank');
if (printWindow) {
  printWindow.document.write(invoiceHTML);
  printWindow.document.close();
  
  printWindow.onload = () => {
    printWindow.print();
  };
  
  toast.success('Invoice generated successfully');
} else {
  toast.error('Please allow popups to generate invoice');
}
```

---

## Usage in UI

### Admin Dashboard - Bookings Tab

**Display Invoice Info:**
```tsx
{/* Invoice Number Display */}
{booking.invoiceNumber && (
  <div className="mt-3 pt-3 border-t border-gray-200">
    <div className="p-2 bg-purple-50 border border-purple-200 rounded text-xs text-purple-700">
      <div className="flex items-center justify-center gap-2">
        <FileText className="w-3 h-3" />
        <span className="font-semibold">Invoice: {booking.invoiceNumber}</span>
      </div>
      {booking.invoiceGeneratedAt && (
        <div className="text-center mt-1 text-purple-600">
          Generated {new Date(booking.invoiceGeneratedAt).toLocaleDateString('en-GB')}
        </div>
      )}
    </div>
  </div>
)}
```

**Generate Invoice Button (Only for Paid Bookings):**
```tsx
{/* Invoice Generation - Only for Paid Bookings */}
{booking.paymentStatus === 'paid' && (
  <div className="mt-3 pt-3 border-t border-gray-200 space-y-2">
    <Button
      onClick={() => generateInvoice(booking)}
      variant="outline"
      size="sm"
      className="w-full text-xs"
    >
      <FileText className="w-3 h-3 mr-1" />
      {booking.invoiceNumber ? 'Re-generate Invoice' : 'Generate Invoice'}
    </Button>
    
    <Button
      onClick={() => emailInvoice(booking)}
      variant="outline"
      size="sm"
      className="w-full text-xs"
    >
      <Mail className="w-3 h-3 mr-1" />
      Email Invoice
    </Button>
  </div>
)}
```

---

## Complete Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│ 1. User Creates Booking                                      │
│    - Status: "pending"                                       │
│    - Payment Status: "pending"                               │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 2. Reception/Admin Marks Payment as Paid                     │
│    - Update paymentStatus to "paid"                          │
│    - Add transactionReference                                │
│    - Set paymentUpdatedAt                                    │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 3. Admin/Staff Generates Invoice                             │
│    ├─ Check: paymentStatus === "paid" ✓                     │
│    ├─ Generate invoice number: "INV-20260114-1234"          │
│    ├─ Save to DB via PUT /bookings/:id/invoice              │
│    └─ Store: invoiceNumber, invoiceGeneratedAt              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 4. Generate HTML Invoice                                      │
│    ├─ Create HTML with company branding                      │
│    ├─ Include customer details                               │
│    ├─ Show service details                                   │
│    ├─ Display payment information                            │
│    └─ Add invoice number & date                              │
└────────────────────┬────────────────────────────────────────┘
                     │
                     ▼
┌─────────────────────────────────────────────────────────────┐
│ 5. Print/Download Invoice                                    │
│    ├─ Open in new window                                     │
│    ├─ Trigger browser print dialog                           │
│    └─ Customer can save as PDF                               │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Points

### ✅ Requirements
1. **Payment Must Be Completed First**
   - `paymentStatus` must equal `"paid"`
   - System enforces this at API level
   - Frontend also checks before allowing generation

2. **Invoice Number is Unique**
   - Format: `INV-{YYYYMMDD}-{4-digit-random}`
   - Generated client-side
   - Saved to database immediately
   - Can be regenerated (same booking, new number)

3. **Invoice is Stored in Database**
   - `invoiceNumber`: Saved for record-keeping
   - `invoiceGeneratedAt`: Timestamp of generation
   - `invoiceGeneratedBy`: User ID who generated it

### ⚠️ Important Notes
- Invoice HTML is generated **client-side** (not stored in DB)
- Each generation creates a **new print window**
- Users can save as PDF via browser print dialog
- Invoice can be **regenerated** multiple times
- Old invoice numbers are **overwritten** if regenerated

---

## Testing

### Test Invoice Generation
```typescript
// 1. Create a test booking
const testBooking = {
  id: "booking:1705234567890:test",
  bookingNumber: "BK-LEI-20260114-001",
  customerName: "Test Customer",
  customerEmail: "test@example.com",
  customerPhone: "+44 7700 900000",
  serviceType: "Skilled Worker Visa",
  appointmentDate: "2026-01-20",
  appointmentTime: "10:00",
  price: "50.00",
  paymentStatus: "paid",  // ← Must be "paid"
  transactionReference: "TXN-12345",
  officeLocation: "leicester"
};

// 2. Call generate invoice
await generateInvoice(testBooking);

// 3. Expected outcome:
// - Invoice number saved to DB
// - Print dialog opens
// - HTML invoice displays correctly
```

---

## Error Handling

```typescript
try {
  await generateInvoice(booking);
} catch (error) {
  if (error.message.includes('paid')) {
    toast.error('Invoice can only be generated for paid bookings');
  } else if (error.message.includes('popups')) {
    toast.error('Please allow popups to generate invoice');
  } else {
    toast.error('Failed to generate invoice. Please try again.');
  }
}
```

---

**Document Version:** 1.0  
**Last Updated:** January 14, 2026  
**Implementation Files:**
- Backend: `/supabase/functions/server/index.tsx` (lines 1493-1546)
- Frontend: `/src/app/pages/AdminDashboard.tsx` (lines 1347-1663)
