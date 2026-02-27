# Invoice Management API - Backend Railway Implementation

## Overview
This document provides the complete backend Railway API code for creating and managing invoices in the ACP Legal Services system.

---

## 1. API Endpoint: Create Invoice

### Route Definition
```javascript
// POST /api/invoices
// Create a new invoice (draft or sent)
```

### Complete Backend Code (Railway/Node.js/Express)

```javascript
/**
 * ============================================
 * INVOICE MANAGEMENT API ENDPOINTS
 * ============================================
 * Add this to your Railway backend server
 * File: server.js or routes/invoices.js
 */

const express = require('express');
const router = express.Router();

// Middleware for authentication (assumed to be set up)
const { authenticateToken, requireRole } = require('../middleware/auth');

// Database connection (PostgreSQL)
const { pool } = require('../config/database');

/**
 * ============================================
 * POST /api/invoices
 * Create a new invoice
 * ============================================
 */
router.post('/api/invoices', authenticateToken, requireRole(['staff', 'admin', 'superadmin']), async (req, res) => {
  const client = await pool.connect();
  
  try {
    const {
      clientId,
      issueDate,
      dueDate,
      status,
      lineItems,
      subtotal,
      taxRate,
      taxAmount,
      discount,
      discountType,
      totalAmount,
      paidAmount,
      balanceAmount,
      notes,
      termsAndConditions
    } = req.body;

    // Validation
    if (!clientId) {
      return res.status(400).json({ 
        error: 'Client ID is required' 
      });
    }

    if (!issueDate || !dueDate) {
      return res.status(400).json({ 
        error: 'Issue date and due date are required' 
      });
    }

    if (!lineItems || lineItems.length === 0) {
      return res.status(400).json({ 
        error: 'At least one line item is required' 
      });
    }

    if (!status || !['draft', 'sent'].includes(status)) {
      return res.status(400).json({ 
        error: 'Invalid status. Must be draft or sent' 
      });
    }

    // Validate client exists
    const clientCheck = await client.query(
      'SELECT id, full_name, email, phone, address_line_1, city, postal_code FROM clients WHERE id = $1',
      [clientId]
    );

    if (clientCheck.rows.length === 0) {
      return res.status(404).json({ 
        error: 'Client not found' 
      });
    }

    const clientData = clientCheck.rows[0];

    // Start transaction
    await client.query('BEGIN');

    // Generate invoice number
    const lastInvoiceQuery = await client.query(
      `SELECT invoice_number FROM invoices 
       WHERE invoice_number LIKE $1 
       ORDER BY created_at DESC 
       LIMIT 1`,
      [`INV-${new Date().getFullYear()}${String(new Date().getMonth() + 1).padStart(2, '0')}-%`]
    );

    let invoiceNumber;
    if (lastInvoiceQuery.rows.length > 0) {
      const lastNumber = lastInvoiceQuery.rows[0].invoice_number;
      const match = lastNumber.match(/(\d+)$/);
      if (match) {
        const nextNumber = parseInt(match[1], 10) + 1;
        invoiceNumber = `INV-${new Date().getFullYear()}${String(new Date().getMonth() + 1).padStart(2, '0')}-${String(nextNumber).padStart(3, '0')}`;
      } else {
        invoiceNumber = `INV-${new Date().getFullYear()}${String(new Date().getMonth() + 1).padStart(2, '0')}-001`;
      }
    } else {
      invoiceNumber = `INV-${new Date().getFullYear()}${String(new Date().getMonth() + 1).padStart(2, '0')}-001`;
    }

    // Build client address
    const addressParts = [
      clientData.address_line_1,
      clientData.city,
      clientData.postal_code
    ].filter(Boolean);
    const clientAddress = addressParts.length > 0 ? addressParts.join(', ') : null;

    // Insert invoice
    const insertInvoiceQuery = `
      INSERT INTO invoices (
        invoice_number,
        client_id,
        client_name,
        client_email,
        client_phone,
        client_address,
        issue_date,
        due_date,
        status,
        subtotal,
        tax_rate,
        tax_amount,
        discount,
        discount_type,
        total_amount,
        paid_amount,
        balance_amount,
        notes,
        terms_and_conditions,
        created_by,
        created_at,
        updated_at
      ) VALUES (
        $1, $2, $3, $4, $5, $6, $7, $8, $9, $10, 
        $11, $12, $13, $14, $15, $16, $17, $18, $19, 
        $20, NOW(), NOW()
      ) RETURNING *
    `;

    const invoiceResult = await client.query(insertInvoiceQuery, [
      invoiceNumber,
      clientId,
      clientData.full_name || clientData.name,
      clientData.email,
      clientData.phone,
      clientAddress,
      issueDate,
      dueDate,
      status,
      subtotal || 0,
      taxRate || 0,
      taxAmount || 0,
      discount || 0,
      discountType || 'amount',
      totalAmount || 0,
      paidAmount || 0,
      balanceAmount || totalAmount || 0,
      notes || null,
      termsAndConditions || null,
      req.user.id // User ID from authentication middleware
    ]);

    const invoiceId = invoiceResult.rows[0].id;

    // Insert line items
    if (lineItems && lineItems.length > 0) {
      const lineItemInsertQuery = `
        INSERT INTO invoice_line_items (
          invoice_id,
          description,
          quantity,
          unit_price,
          amount,
          service_type
        ) VALUES ($1, $2, $3, $4, $5, $6)
      `;

      for (const item of lineItems) {
        await client.query(lineItemInsertQuery, [
          invoiceId,
          item.description,
          item.quantity,
          item.unitPrice,
          item.amount,
          item.serviceType || null
        ]);
      }
    }

    // Commit transaction
    await client.query('COMMIT');

    // Fetch the complete invoice with line items
    const completeInvoice = await client.query(
      `SELECT 
        i.*,
        json_agg(
          json_build_object(
            'id', li.id,
            'description', li.description,
            'quantity', li.quantity,
            'unitPrice', li.unit_price,
            'amount', li.amount,
            'serviceType', li.service_type
          )
        ) as line_items
       FROM invoices i
       LEFT JOIN invoice_line_items li ON i.id = li.invoice_id
       WHERE i.id = $1
       GROUP BY i.id`,
      [invoiceId]
    );

    // Send email if status is 'sent'
    if (status === 'sent') {
      // TODO: Implement email sending logic
      // await sendInvoiceEmail(completeInvoice.rows[0]);
      console.log('Invoice email would be sent here');
    }

    res.status(201).json({
      message: status === 'draft' 
        ? 'Invoice saved as draft successfully' 
        : 'Invoice created and sent successfully',
      invoice: completeInvoice.rows[0]
    });

  } catch (error) {
    await client.query('ROLLBACK');
    console.error('Create invoice error:', error);
    res.status(500).json({ 
      error: 'Failed to create invoice',
      message: error.message 
    });
  } finally {
    client.release();
  }
});

/**
 * ============================================
 * GET /api/invoices
 * Get all invoices
 * ============================================
 */
router.get('/api/invoices', authenticateToken, requireRole(['staff', 'admin', 'superadmin']), async (req, res) => {
  try {
    const { status, clientId, limit = 100, offset = 0 } = req.query;

    let query = `
      SELECT 
        i.*,
        json_agg(
          json_build_object(
            'id', li.id,
            'description', li.description,
            'quantity', li.quantity,
            'unitPrice', li.unit_price,
            'amount', li.amount,
            'serviceType', li.service_type
          )
        ) as line_items,
        c.full_name as client_full_name,
        c.email as client_email_current
      FROM invoices i
      LEFT JOIN invoice_line_items li ON i.id = li.invoice_id
      LEFT JOIN clients c ON i.client_id = c.id
      WHERE 1=1
    `;
    
    const params = [];
    let paramCount = 1;

    if (status && status !== 'all') {
      query += ` AND i.status = $${paramCount}`;
      params.push(status);
      paramCount++;
    }

    if (clientId) {
      query += ` AND i.client_id = $${paramCount}`;
      params.push(clientId);
      paramCount++;
    }

    query += ` GROUP BY i.id, c.full_name, c.email
               ORDER BY i.created_at DESC
               LIMIT $${paramCount} OFFSET $${paramCount + 1}`;
    
    params.push(limit, offset);

    const result = await pool.query(query, params);

    res.json({
      invoices: result.rows,
      total: result.rowCount
    });

  } catch (error) {
    console.error('Get invoices error:', error);
    res.status(500).json({ 
      error: 'Failed to fetch invoices',
      message: error.message 
    });
  }
});

/**
 * ============================================
 * GET /api/invoices/:id
 * Get single invoice by ID
 * ============================================
 */
router.get('/api/invoices/:id', authenticateToken, requireRole(['staff', 'admin', 'superadmin']), async (req, res) => {
  try {
    const { id } = req.params;

    const result = await pool.query(
      `SELECT 
        i.*,
        json_agg(
          json_build_object(
            'id', li.id,
            'description', li.description,
            'quantity', li.quantity,
            'unitPrice', li.unit_price,
            'amount', li.amount,
            'serviceType', li.service_type
          )
        ) as line_items,
        json_agg(
          DISTINCT jsonb_build_object(
            'id', p.id,
            'amount', p.amount,
            'paymentDate', p.payment_date,
            'paymentMethod', p.payment_method,
            'reference', p.reference,
            'notes', p.notes
          )
        ) FILTER (WHERE p.id IS NOT NULL) as payments
       FROM invoices i
       LEFT JOIN invoice_line_items li ON i.id = li.invoice_id
       LEFT JOIN invoice_payments p ON i.id = p.invoice_id
       WHERE i.id = $1
       GROUP BY i.id`,
      [id]
    );

    if (result.rows.length === 0) {
      return res.status(404).json({ 
        error: 'Invoice not found' 
      });
    }

    res.json(result.rows[0]);

  } catch (error) {
    console.error('Get invoice error:', error);
    res.status(500).json({ 
      error: 'Failed to fetch invoice',
      message: error.message 
    });
  }
});

/**
 * ============================================
 * PUT /api/invoices/:id
 * Update an existing invoice
 * ============================================
 */
router.put('/api/invoices/:id', authenticateToken, requireRole(['staff', 'admin', 'superadmin']), async (req, res) => {
  const client = await pool.connect();
  
  try {
    const { id } = req.params;
    const {
      issueDate,
      dueDate,
      status,
      lineItems,
      subtotal,
      taxRate,
      taxAmount,
      discount,
      discountType,
      totalAmount,
      notes,
      termsAndConditions
    } = req.body;

    // Check if invoice exists
    const invoiceCheck = await client.query(
      'SELECT * FROM invoices WHERE id = $1',
      [id]
    );

    if (invoiceCheck.rows.length === 0) {
      return res.status(404).json({ 
        error: 'Invoice not found' 
      });
    }

    // Start transaction
    await client.query('BEGIN');

    // Update invoice
    const updateQuery = `
      UPDATE invoices SET
        issue_date = COALESCE($1, issue_date),
        due_date = COALESCE($2, due_date),
        status = COALESCE($3, status),
        subtotal = COALESCE($4, subtotal),
        tax_rate = COALESCE($5, tax_rate),
        tax_amount = COALESCE($6, tax_amount),
        discount = COALESCE($7, discount),
        discount_type = COALESCE($8, discount_type),
        total_amount = COALESCE($9, total_amount),
        balance_amount = total_amount - paid_amount,
        notes = COALESCE($10, notes),
        terms_and_conditions = COALESCE($11, terms_and_conditions),
        updated_at = NOW()
      WHERE id = $12
      RETURNING *
    `;

    const updateResult = await client.query(updateQuery, [
      issueDate,
      dueDate,
      status,
      subtotal,
      taxRate,
      taxAmount,
      discount,
      discountType,
      totalAmount,
      notes,
      termsAndConditions,
      id
    ]);

    // Update line items if provided
    if (lineItems && lineItems.length > 0) {
      // Delete existing line items
      await client.query(
        'DELETE FROM invoice_line_items WHERE invoice_id = $1',
        [id]
      );

      // Insert new line items
      const lineItemInsertQuery = `
        INSERT INTO invoice_line_items (
          invoice_id,
          description,
          quantity,
          unit_price,
          amount,
          service_type
        ) VALUES ($1, $2, $3, $4, $5, $6)
      `;

      for (const item of lineItems) {
        await client.query(lineItemInsertQuery, [
          id,
          item.description,
          item.quantity,
          item.unitPrice,
          item.amount,
          item.serviceType || null
        ]);
      }
    }

    // Commit transaction
    await client.query('COMMIT');

    // Fetch updated invoice with line items
    const completeInvoice = await client.query(
      `SELECT 
        i.*,
        json_agg(
          json_build_object(
            'id', li.id,
            'description', li.description,
            'quantity', li.quantity,
            'unitPrice', li.unit_price,
            'amount', li.amount,
            'serviceType', li.service_type
          )
        ) as line_items
       FROM invoices i
       LEFT JOIN invoice_line_items li ON i.id = li.invoice_id
       WHERE i.id = $1
       GROUP BY i.id`,
      [id]
    );

    res.json({
      message: 'Invoice updated successfully',
      invoice: completeInvoice.rows[0]
    });

  } catch (error) {
    await client.query('ROLLBACK');
    console.error('Update invoice error:', error);
    res.status(500).json({ 
      error: 'Failed to update invoice',
      message: error.message 
    });
  } finally {
    client.release();
  }
});

/**
 * ============================================
 * DELETE /api/invoices/:id
 * Delete an invoice
 * ============================================
 */
router.delete('/api/invoices/:id', authenticateToken, requireRole(['admin', 'superadmin']), async (req, res) => {
  const client = await pool.connect();
  
  try {
    const { id } = req.params;

    // Check if invoice exists
    const invoiceCheck = await client.query(
      'SELECT * FROM invoices WHERE id = $1',
      [id]
    );

    if (invoiceCheck.rows.length === 0) {
      return res.status(404).json({ 
        error: 'Invoice not found' 
      });
    }

    // Start transaction
    await client.query('BEGIN');

    // Delete line items first (due to foreign key constraint)
    await client.query(
      'DELETE FROM invoice_line_items WHERE invoice_id = $1',
      [id]
    );

    // Delete invoice
    await client.query(
      'DELETE FROM invoices WHERE id = $1',
      [id]
    );

    // Commit transaction
    await client.query('COMMIT');

    res.json({
      message: 'Invoice deleted successfully'
    });

  } catch (error) {
    await client.query('ROLLBACK');
    console.error('Delete invoice error:', error);
    res.status(500).json({ 
      error: 'Failed to delete invoice',
      message: error.message 
    });
  } finally {
    client.release();
  }
});

/**
 * ============================================
 * POST /api/invoices/:id/send
 * Send invoice to client via email
 * ============================================
 */
router.post('/api/invoices/:id/send', authenticateToken, requireRole(['staff', 'admin', 'superadmin']), async (req, res) => {
  try {
    const { id } = req.params;

    // Get invoice
    const invoiceResult = await pool.query(
      `SELECT 
        i.*,
        json_agg(
          json_build_object(
            'id', li.id,
            'description', li.description,
            'quantity', li.quantity,
            'unitPrice', li.unit_price,
            'amount', li.amount
          )
        ) as line_items
       FROM invoices i
       LEFT JOIN invoice_line_items li ON i.id = li.invoice_id
       WHERE i.id = $1
       GROUP BY i.id`,
      [id]
    );

    if (invoiceResult.rows.length === 0) {
      return res.status(404).json({ 
        error: 'Invoice not found' 
      });
    }

    const invoice = invoiceResult.rows[0];

    // Update status to 'sent'
    await pool.query(
      `UPDATE invoices SET 
        status = 'sent', 
        sent_at = NOW(),
        updated_at = NOW()
       WHERE id = $1`,
      [id]
    );

    // TODO: Send email to client
    // await sendInvoiceEmail(invoice);
    console.log(`Invoice ${invoice.invoice_number} would be sent to ${invoice.client_email}`);

    res.json({
      message: 'Invoice sent successfully',
      invoice: {
        ...invoice,
        status: 'sent',
        sent_at: new Date().toISOString()
      }
    });

  } catch (error) {
    console.error('Send invoice error:', error);
    res.status(500).json({ 
      error: 'Failed to send invoice',
      message: error.message 
    });
  }
});

/**
 * ============================================
 * GET /api/invoices/stats
 * Get invoice statistics
 * ============================================
 */
router.get('/api/invoices/stats', authenticateToken, requireRole(['staff', 'admin', 'superadmin']), async (req, res) => {
  try {
    const statsQuery = `
      SELECT 
        COUNT(*) as total_invoices,
        SUM(total_amount) as total_amount,
        SUM(paid_amount) as paid_amount,
        SUM(balance_amount) as pending_amount,
        SUM(CASE WHEN status = 'overdue' THEN balance_amount ELSE 0 END) as overdue_amount,
        COUNT(CASE WHEN status = 'draft' THEN 1 END) as draft_count,
        COUNT(CASE WHEN status = 'sent' THEN 1 END) as sent_count,
        COUNT(CASE WHEN status = 'paid' THEN 1 END) as paid_count,
        COUNT(CASE WHEN status = 'partial' THEN 1 END) as partial_count,
        COUNT(CASE WHEN status = 'overdue' THEN 1 END) as overdue_count
      FROM invoices
      WHERE status != 'cancelled'
    `;

    const result = await pool.query(statsQuery);

    res.json(result.rows[0]);

  } catch (error) {
    console.error('Get invoice stats error:', error);
    res.status(500).json({ 
      error: 'Failed to fetch invoice statistics',
      message: error.message 
    });
  }
});

module.exports = router;
```

---

## 2. Database Schema

### Required Tables

#### invoices table
```sql
CREATE TABLE IF NOT EXISTS invoices (
  id SERIAL PRIMARY KEY,
  invoice_number VARCHAR(50) UNIQUE NOT NULL,
  client_id INTEGER NOT NULL REFERENCES clients(id),
  
  -- Client details (denormalized for invoice record)
  client_name VARCHAR(255) NOT NULL,
  client_email VARCHAR(255) NOT NULL,
  client_phone VARCHAR(50),
  client_address TEXT,
  
  -- Invoice dates
  issue_date DATE NOT NULL,
  due_date DATE NOT NULL,
  sent_at TIMESTAMP,
  
  -- Status
  status VARCHAR(20) NOT NULL DEFAULT 'draft' 
    CHECK (status IN ('draft', 'sent', 'paid', 'partial', 'overdue', 'cancelled')),
  
  -- Amounts
  subtotal DECIMAL(10, 2) NOT NULL DEFAULT 0,
  tax_rate DECIMAL(5, 2) NOT NULL DEFAULT 0,
  tax_amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
  discount DECIMAL(10, 2) DEFAULT 0,
  discount_type VARCHAR(20) DEFAULT 'amount' CHECK (discount_type IN ('amount', 'percentage')),
  total_amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
  paid_amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
  balance_amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
  
  -- Additional info
  notes TEXT,
  terms_and_conditions TEXT,
  
  -- Audit fields
  created_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoices_client_id ON invoices(client_id);
CREATE INDEX idx_invoices_status ON invoices(status);
CREATE INDEX idx_invoices_issue_date ON invoices(issue_date);
CREATE INDEX idx_invoices_due_date ON invoices(due_date);
CREATE INDEX idx_invoices_invoice_number ON invoices(invoice_number);
```

#### invoice_line_items table
```sql
CREATE TABLE IF NOT EXISTS invoice_line_items (
  id SERIAL PRIMARY KEY,
  invoice_id INTEGER NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
  description TEXT NOT NULL,
  quantity DECIMAL(10, 2) NOT NULL DEFAULT 1,
  unit_price DECIMAL(10, 2) NOT NULL DEFAULT 0,
  amount DECIMAL(10, 2) NOT NULL DEFAULT 0,
  service_type VARCHAR(100),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoice_line_items_invoice_id ON invoice_line_items(invoice_id);
```

#### invoice_payments table (optional, for payment tracking)
```sql
CREATE TABLE IF NOT EXISTS invoice_payments (
  id SERIAL PRIMARY KEY,
  invoice_id INTEGER NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
  amount DECIMAL(10, 2) NOT NULL,
  payment_date DATE NOT NULL,
  payment_method VARCHAR(50) NOT NULL CHECK (payment_method IN ('cash', 'card', 'bank_transfer', 'cheque', 'other')),
  reference VARCHAR(100),
  notes TEXT,
  recorded_by UUID REFERENCES auth.users(id),
  created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_invoice_payments_invoice_id ON invoice_payments(invoice_id);
CREATE INDEX idx_invoice_payments_payment_date ON invoice_payments(payment_date);
```

---

## 3. Environment Variables

Add these to your Railway environment:

```env
# Database
DATABASE_URL=postgresql://user:password@host:port/database

# API
API_BASE_URL=https://your-railway-app.railway.app
PORT=3000

# Email (for sending invoices)
EMAIL_SERVICE=resend
RESEND_API_KEY=your_resend_api_key
FROM_EMAIL=invoices@acplegalservices.co.uk
```

---

## 4. Testing the API

### Create Invoice (Draft)
```bash
curl -X POST https://your-railway-app.railway.app/api/invoices \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{
    "clientId": 1,
    "issueDate": "2026-02-25",
    "dueDate": "2026-03-11",
    "status": "draft",
    "lineItems": [
      {
        "description": "Spouse Visa Consultation",
        "quantity": 1,
        "unitPrice": 50.00,
        "amount": 50.00,
        "serviceType": "Consultation"
      }
    ],
    "subtotal": 50.00,
    "taxRate": 20,
    "taxAmount": 10.00,
    "discount": 0,
    "discountType": "amount",
    "totalAmount": 60.00,
    "paidAmount": 0,
    "balanceAmount": 60.00,
    "notes": "Initial consultation for spouse visa application",
    "termsAndConditions": "Payment due within 14 days"
  }'
```

### Create Invoice (Sent)
Same as above but with `"status": "sent"` to immediately mark it as sent.

---

## 5. Implementation Checklist

- [ ] Create database tables (invoices, invoice_line_items, invoice_payments)
- [ ] Add invoice routes to your Railway Express server
- [ ] Set up authentication middleware
- [ ] Configure database connection pool
- [ ] Test API endpoints with Postman or curl
- [ ] Update frontend to call the new endpoints
- [ ] Implement email sending for sent invoices (optional)
- [ ] Add invoice PDF generation (optional)
- [ ] Set up error logging and monitoring

---

## 6. Next Steps

1. **Deploy to Railway**: Push the updated backend code to Railway
2. **Run Migrations**: Execute the SQL schema to create the required tables
3. **Test**: Verify the API endpoints work correctly
4. **Update Frontend**: The frontend is already prepared to call these endpoints
5. **Email Integration**: Set up Resend or another email service for sending invoices

---

## Support

For questions or issues:
- Check Railway logs: `railway logs`
- Test database connection: `railway run psql`
- Verify environment variables: `railway variables`
