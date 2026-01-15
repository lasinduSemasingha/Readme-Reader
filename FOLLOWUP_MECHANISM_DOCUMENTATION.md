# FOLLOW-UP MECHANISM DOCUMENTATION
# ACP Legal Services - Inquiry Follow-Up System

================================================================================
## OVERVIEW
================================================================================

This document explains the complete follow-up date mechanism used in the 
inquiry/call handling system. Staff can set follow-up dates when logging calls,
and the system automatically tracks and displays inquiries that need follow-up
today.

================================================================================
## SYSTEM ARCHITECTURE
================================================================================

The follow-up system consists of:

1. **Database Structure** - KV Store with inquiry objects containing followUpDate
2. **Backend API Endpoints** - CRUD operations for inquiries
3. **Frontend Integration** - Call Handling UI with follow-up date picker
4. **Dashboard Widget** - Today's Follow-Ups display in Staff Dashboard

================================================================================
## DATABASE STRUCTURE
================================================================================

### Storage Type: Supabase KV Store (Key-Value Store)

### Key Format:
inquiry:{timestamp}:{random_string}

**Example Key:**
inquiry:1705234567890:abc123def

### Value Structure (Inquiry Object):

{
  // === CORE IDENTIFICATION ===
  "id": "inquiry:1705234567890:abc123def",
  
  // === CALLER INFORMATION ===
  "callerName": "John Smith",
  "callerPhone": "+44 7700 900123",
  "callerEmail": "john.smith@example.com",
  
  // === INQUIRY DETAILS ===
  "inquiryType": "visa-application",
  "inquiryDescription": "Questions about Skilled Worker visa eligibility",
  "priority": "high",                    // Values: "urgent", "high", "normal", "low"
  "status": "pending",                   // Values: "pending", "in-progress", "resolved", "closed"
  
  // === FOLLOW-UP INFORMATION ⭐ ===
  "followUpDate": "2026-01-20",          // Format: YYYY-MM-DD
  "followUpRequired": true,
  "followUpNotes": "Need to call back with visa fee information",
  
  // === METADATA ===
  "createdAt": "2026-01-15T10:30:00.000Z",
  "createdBy": "staff_user_id_12345",   // Staff user ID who logged the inquiry
  "updatedAt": "2026-01-15T10:30:00.000Z",
  "updatedBy": "staff_user_id_12345",
  
  // === ADDITIONAL FIELDS ===
  "officeLocation": "Leicester Office",
  "handledBy": "Jane Doe",              // Staff member name
  "callDuration": "15",                 // In minutes
  "callNotes": "Customer needs consultation appointment"
}

### Index Structure:

**Key:** inquiry_index
**Value:** Array of inquiry IDs
[
  "inquiry:1705234567890:abc123def",
  "inquiry:1705234567891:xyz789ghi",
  "inquiry:1705234567892:lmn456opq"
]

This index allows fast retrieval of all inquiries without scanning the entire
database.

================================================================================
## BACKEND API ENDPOINTS
================================================================================

Base URL: https://{projectId}.supabase.co/functions/v1/make-server-d8c62521

All endpoints require authentication via Bearer token in Authorization header.

---

### 1. CREATE NEW INQUIRY (WITH FOLLOW-UP DATE)

**Endpoint:** POST /inquiries
**Authentication:** Required (Bearer token)
**Location in Code:** /supabase/functions/server/index.tsx (Line 699)

**Request Headers:**
```
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**Request Body:**
```json
{
  "callerName": "John Smith",
  "callerPhone": "+44 7700 900123",
  "callerEmail": "john.smith@example.com",
  "inquiryType": "visa-application",
  "inquiryDescription": "Questions about Skilled Worker visa",
  "priority": "high",
  "status": "pending",
  "followUpDate": "2026-01-20",         ⭐ FOLLOW-UP DATE
  "followUpRequired": true,
  "followUpNotes": "Call back with visa fee information",
  "officeLocation": "Leicester Office",
  "handledBy": "Jane Doe",
  "callDuration": "15",
  "callNotes": "Customer needs consultation"
}
```

**Response (Success - 200 OK):**
```json
{
  "message": "Inquiry saved successfully",
  "inquiry": {
    "id": "inquiry:1705234567890:abc123def",
    "callerName": "John Smith",
    "callerPhone": "+44 7700 900123",
    "callerEmail": "john.smith@example.com",
    "inquiryType": "visa-application",
    "inquiryDescription": "Questions about Skilled Worker visa",
    "priority": "high",
    "status": "pending",
    "followUpDate": "2026-01-20",       ⭐ STORED IN DATABASE
    "followUpRequired": true,
    "followUpNotes": "Call back with visa fee information",
    "officeLocation": "Leicester Office",
    "handledBy": "Jane Doe",
    "callDuration": "15",
    "callNotes": "Customer needs consultation",
    "createdAt": "2026-01-15T10:30:00.000Z",
    "createdBy": "user_id_12345",
    "updatedAt": "2026-01-15T10:30:00.000Z",
    "updatedBy": "user_id_12345"
  }
}
```

**Backend Logic:**
```javascript
app.post("/make-server-d8c62521/inquiries", async (c) => {
  try {
    // 1. Verify authentication
    const accessToken = c.req.header('Authorization')?.split(' ')[1];
    const { data: { user }, error } = await supabase.auth.getUser(accessToken);
    
    // 2. Get inquiry data from request body
    const inquiryData = await c.req.json();
    
    // 3. Generate unique ID
    const inquiryId = `inquiry:${Date.now()}:${Math.random().toString(36).substring(7)}`;
    
    // 4. Create inquiry object with metadata
    const inquiry = {
      ...inquiryData,              // Includes followUpDate ⭐
      id: inquiryId,
      createdAt: new Date().toISOString(),
      createdBy: user.id,
      updatedAt: new Date().toISOString(),
      updatedBy: user.id
    };
    
    // 5. Save to KV store
    await kv.set(inquiryId, inquiry);
    
    // 6. Add to index for fast retrieval
    const existingInquiries = await kv.get('inquiry_index') || [];
    existingInquiries.push(inquiryId);
    await kv.set('inquiry_index', existingInquiries);
    
    // 7. Return success response
    return c.json({ message: 'Inquiry saved successfully', inquiry });
  } catch (error) {
    return c.json({ error: 'Internal server error' }, 500);
  }
});
```

---

### 2. UPDATE INQUIRY (UPDATE FOLLOW-UP DATE)

**Endpoint:** PUT /inquiries/{inquiryId}
**Authentication:** Required (Bearer token)
**Location in Code:** /supabase/functions/server/index.tsx (Line 808)

**Request Headers:**
```
Authorization: Bearer {accessToken}
Content-Type: application/json
```

**URL Parameters:**
- `inquiryId` - The inquiry ID (e.g., inquiry:1705234567890:abc123def)

**Request Body (Partial Update):**
```json
{
  "followUpDate": "2026-01-22",         ⭐ NEW FOLLOW-UP DATE
  "followUpNotes": "Updated notes - need to discuss visa fees",
  "status": "in-progress"
}
```

**Response (Success - 200 OK):**
```json
{
  "message": "Inquiry updated successfully",
  "inquiry": {
    "id": "inquiry:1705234567890:abc123def",
    "callerName": "John Smith",
    "followUpDate": "2026-01-22",       ⭐ UPDATED
    "followUpNotes": "Updated notes - need to discuss visa fees",
    "status": "in-progress",
    "updatedAt": "2026-01-18T14:20:00.000Z",
    "updatedBy": "user_id_12345",
    ...
  }
}
```

**Backend Logic:**
```javascript
app.put("/make-server-d8c62521/inquiries/:id", async (c) => {
  try {
    // 1. Verify authentication
    const accessToken = c.req.header('Authorization')?.split(' ')[1];
    const { data: { user }, error } = await supabase.auth.getUser(accessToken);
    
    // 2. Get inquiry ID from URL
    const id = c.req.param('id');
    
    // 3. Get existing inquiry
    const existingInquiry = await kv.get(id);
    if (!existingInquiry) {
      return c.json({ error: 'Inquiry not found' }, 404);
    }
    
    // 4. Get updates from request body
    const updates = await c.req.json();  // Can include new followUpDate ⭐
    
    // 5. Merge updates with existing data
    const updatedInquiry = {
      ...existingInquiry,
      ...updates,                      // New followUpDate overwrites old one
      id,                              // Preserve original ID
      createdAt: existingInquiry.createdAt,
      createdBy: existingInquiry.createdBy,
      updatedAt: new Date().toISOString(),
      updatedBy: user.id
    };
    
    // 6. Save updated inquiry
    await kv.set(id, updatedInquiry);
    
    // 7. Return success response
    return c.json({ 
      message: 'Inquiry updated successfully',
      inquiry: updatedInquiry 
    });
  } catch (error) {
    return c.json({ error: 'Internal server error' }, 500);
  }
});
```

---

### 3. GET TODAY'S FOLLOW-UPS ⭐⭐⭐

**Endpoint:** GET /inquiries/follow-ups/today
**Authentication:** Required (Bearer token)
**Location in Code:** /supabase/functions/server/index.tsx (Line 939)

This is the KEY endpoint for the follow-up system!

**Request Headers:**
```
Authorization: Bearer {accessToken}
```

**Response (Success - 200 OK):**
```json
{
  "followUps": [
    {
      "id": "inquiry:1705234567890:abc123def",
      "callerName": "John Smith",
      "callerPhone": "+44 7700 900123",
      "inquiryType": "visa-application",
      "priority": "urgent",
      "followUpDate": "2026-01-20",     ⭐ MATCHES TODAY
      "followUpNotes": "Call back with visa fee information",
      "status": "pending",
      "createdAt": "2026-01-15T10:30:00.000Z"
    },
    {
      "id": "inquiry:1705234567891:xyz789ghi",
      "callerName": "Sarah Johnson",
      "callerPhone": "+44 7700 900456",
      "inquiryType": "spouse-visa",
      "priority": "high",
      "followUpDate": "2026-01-20",     ⭐ MATCHES TODAY
      "followUpNotes": "Send document checklist",
      "status": "in-progress",
      "createdAt": "2026-01-16T11:00:00.000Z"
    }
  ],
  "count": 2,
  "todayDate": "2026-01-20"
}
```

**Backend Logic (DETAILED):**
```javascript
app.get("/make-server-d8c62521/inquiries/follow-ups/today", async (c) => {
  try {
    // 1. Verify authentication
    const accessToken = c.req.header('Authorization')?.split(' ')[1];
    
    if (!accessToken) {
      return c.json({ error: 'Unauthorized - No access token' }, 401);
    }
    
    const { data: { user }, error: authError } = await supabase.auth.getUser(accessToken);
    
    if (authError || !user) {
      return c.json({ error: 'Unauthorized - Invalid session' }, 401);
    }
    
    // 2. Get all inquiries from database
    const inquiries = await kv.getByPrefix('inquiry:');
    
    // 3. Filter out the index and get only valid inquiries
    const validInquiries = inquiries.filter(inq => inq.id && inq.callerName);
    
    // 4. Get today's date in UK timezone (YYYY-MM-DD format) ⭐
    const ukNow = new Date().toLocaleString('en-GB', { 
      timeZone: 'Europe/London',
      year: 'numeric',
      month: '2-digit',
      day: '2-digit'
    });
    
    // Convert from DD/MM/YYYY to YYYY-MM-DD
    const [day, month, year] = ukNow.split(',')[0].split('/');
    const todayUK = `${year}-${month}-${day}`;  // e.g., "2026-01-20"
    
    // 5. Filter inquiries with follow-up date matching today ⭐⭐⭐
    const todaysFollowUps = validInquiries.filter(inq => {
      if (!inq.followUpDate) return false;         // Skip if no follow-up date
      return inq.followUpDate === todayUK;         // Match today's date
    });
    
    // 6. Sort by priority (urgent first, then high, normal, low)
    const priorityOrder = { 'urgent': 0, 'high': 1, 'normal': 2, 'low': 3 };
    todaysFollowUps.sort((a, b) => {
      const aPriority = priorityOrder[a.priority?.toLowerCase()] ?? 2;
      const bPriority = priorityOrder[b.priority?.toLowerCase()] ?? 2;
      return aPriority - bPriority;
    });
    
    // 7. Return filtered and sorted follow-ups
    return c.json({ 
      followUps: todaysFollowUps,
      count: todaysFollowUps.length,
      todayDate: todayUK
    });
  } catch (error) {
    console.log(`Error fetching today's follow-ups: ${error}`);
    return c.json({ error: 'Internal server error while fetching follow-ups' }, 500);
  }
});
```

**Key Logic Explanation:**

1. **Date Normalization:**
   - Server converts current time to UK timezone
   - Formats as YYYY-MM-DD (e.g., "2026-01-20")
   
2. **Filtering:**
   - Checks each inquiry's `followUpDate` field
   - Only returns inquiries where `followUpDate === todayUK`
   
3. **Sorting:**
   - Urgent inquiries appear first
   - Then High → Normal → Low priority

---

### 4. GET ALL INQUIRIES

**Endpoint:** GET /inquiries
**Authentication:** Required (Bearer token)
**Location in Code:** /supabase/functions/server/index.tsx (Line 746)

**Request Headers:**
```
Authorization: Bearer {accessToken}
```

**Response (Success - 200 OK):**
```json
{
  "inquiries": [
    {
      "id": "inquiry:1705234567890:abc123def",
      "callerName": "John Smith",
      "followUpDate": "2026-01-20",
      "status": "pending",
      ...
    },
    {
      "id": "inquiry:1705234567891:xyz789ghi",
      "callerName": "Sarah Johnson",
      "followUpDate": "2026-01-22",
      "status": "in-progress",
      ...
    }
  ]
}
```

---

### 5. GET SINGLE INQUIRY BY ID

**Endpoint:** GET /inquiries/{inquiryId}
**Authentication:** Required (Bearer token)
**Location in Code:** /supabase/functions/server/index.tsx (Line 779)

**Request Headers:**
```
Authorization: Bearer {accessToken}
```

**URL Parameters:**
- `inquiryId` - The inquiry ID

**Response (Success - 200 OK):**
```json
{
  "inquiry": {
    "id": "inquiry:1705234567890:abc123def",
    "callerName": "John Smith",
    "followUpDate": "2026-01-20",
    "followUpNotes": "Call back with visa fee information",
    ...
  }
}
```

---

### 6. DELETE INQUIRY

**Endpoint:** DELETE /inquiries/{inquiryId}
**Authentication:** Required (Bearer token)
**Location in Code:** /supabase/functions/server/index.tsx (Line 857)

**Request Headers:**
```
Authorization: Bearer {accessToken}
```

**URL Parameters:**
- `inquiryId` - The inquiry ID to delete

**Response (Success - 200 OK):**
```json
{
  "message": "Inquiry deleted successfully"
}
```

================================================================================
## FRONTEND INTEGRATION - CALL HANDLING PAGE
================================================================================

Location: /src/app/pages/CallHandling.tsx

### Setting Follow-Up Date When Creating Inquiry:

```javascript
// State for form data
const [formData, setFormData] = useState({
  callerName: '',
  callerPhone: '',
  callerEmail: '',
  inquiryType: '',
  inquiryDescription: '',
  priority: 'normal',
  status: 'pending',
  followUpDate: '',              ⭐ Follow-up date field
  followUpRequired: false,
  followUpNotes: '',
  officeLocation: '',
  handledBy: '',
  callDuration: '',
  callNotes: ''
});

// Handle form submission
const handleSubmit = async (e) => {
  e.preventDefault();
  
  try {
    const response = await fetch(
      `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/inquiries`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'Authorization': `Bearer ${sessionToken}`
        },
        body: JSON.stringify(formData)  // Includes followUpDate ⭐
      }
    );
    
    const data = await response.json();
    
    if (response.ok) {
      toast.success('Inquiry saved successfully!');
      // The followUpDate is now stored in the database
    }
  } catch (error) {
    toast.error('Failed to save inquiry');
  }
};
```

### Follow-Up Date Input Field:

```jsx
{/* Follow-Up Date Input */}
<div>
  <label className="block text-sm font-medium mb-2">
    Follow-Up Date
  </label>
  <input
    type="date"
    value={formData.followUpDate}
    onChange={(e) => setFormData({
      ...formData,
      followUpDate: e.target.value  ⭐ Format: YYYY-MM-DD
    })}
    className="w-full px-4 py-2 border rounded-lg"
    min={new Date().toISOString().split('T')[0]}  // Minimum: Today
  />
</div>

{/* Follow-Up Notes */}
<div>
  <label className="block text-sm font-medium mb-2">
    Follow-Up Notes
  </label>
  <textarea
    value={formData.followUpNotes}
    onChange={(e) => setFormData({
      ...formData,
      followUpNotes: e.target.value
    })}
    className="w-full px-4 py-2 border rounded-lg"
    rows={3}
    placeholder="Notes for follow-up call..."
  />
</div>
```

================================================================================
## FRONTEND INTEGRATION - STAFF DASHBOARD
================================================================================

Location: /src/app/pages/StaffDashboard.tsx

### Fetching Today's Follow-Ups:

```javascript
const [followUps, setFollowUps] = useState([]);
const [followUpCount, setFollowUpCount] = useState(0);

// Fetch today's follow-ups when dashboard loads
useEffect(() => {
  const fetchTodaysFollowUps = async () => {
    try {
      const response = await fetch(
        `https://${projectId}.supabase.co/functions/v1/make-server-d8c62521/inquiries/follow-ups/today`,
        {
          headers: {
            'Authorization': `Bearer ${sessionToken}`
          }
        }
      );
      
      const data = await response.json();
      
      if (response.ok) {
        setFollowUps(data.followUps);       ⭐ Array of follow-ups for today
        setFollowUpCount(data.count);       ⭐ Number of follow-ups
      }
    } catch (error) {
      console.error('Error fetching follow-ups:', error);
    }
  };
  
  fetchTodaysFollowUps();
}, []);
```

### Displaying Follow-Ups Widget:

```jsx
{/* Today's Follow-Ups Widget */}
<Card className="p-6">
  <div className="flex items-center justify-between mb-4">
    <h3 className="text-lg font-semibold">Today's Follow-Ups</h3>
    <span className="bg-orange-100 text-orange-600 px-3 py-1 rounded-full text-sm font-semibold">
      {followUpCount} pending
    </span>
  </div>
  
  {followUps.length === 0 ? (
    <p className="text-gray-500">No follow-ups scheduled for today</p>
  ) : (
    <div className="space-y-3">
      {followUps.map((followUp) => (
        <div 
          key={followUp.id}
          className="border-l-4 border-orange-500 pl-4 py-2"
        >
          <div className="flex items-center justify-between">
            <div>
              <p className="font-semibold">{followUp.callerName}</p>
              <p className="text-sm text-gray-600">{followUp.callerPhone}</p>
              <p className="text-sm text-gray-500 mt-1">
                {followUp.followUpNotes}
              </p>
            </div>
            <span className={`px-2 py-1 rounded text-xs font-semibold ${
              followUp.priority === 'urgent' ? 'bg-red-100 text-red-700' :
              followUp.priority === 'high' ? 'bg-orange-100 text-orange-700' :
              'bg-blue-100 text-blue-700'
            }`}>
              {followUp.priority}
            </span>
          </div>
        </div>
      ))}
    </div>
  )}
</Card>
```

================================================================================
## COMPLETE DATA FLOW DIAGRAM
================================================================================

┌────────────────────────────────────────────────────────────────────┐
│ 1. STAFF LOGS CALL WITH FOLLOW-UP                                 │
│    - Call Handling Page                                           │
│    - Staff sets followUpDate: "2026-01-20"                        │
│    - Clicks "Save Inquiry"                                        │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 2. FRONTEND SENDS POST REQUEST                                     │
│    POST /inquiries                                                 │
│    Body: { followUpDate: "2026-01-20", ... }                      │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 3. BACKEND CREATES INQUIRY                                         │
│    - Generates ID: inquiry:1705234567890:abc123def                 │
│    - Stores in KV Store with followUpDate field ⭐                 │
│    - Adds to inquiry_index array                                   │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 4. DATABASE STORAGE                                                │
│    Key: inquiry:1705234567890:abc123def                            │
│    Value: {                                                        │
│      followUpDate: "2026-01-20", ⭐                                │
│      callerName: "John Smith",                                     │
│      ...                                                           │
│    }                                                               │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 5. TIME PASSES... DATE IS NOW 2026-01-20                          │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 6. STAFF OPENS DASHBOARD                                           │
│    - Dashboard loads                                               │
│    - useEffect() triggers                                          │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 7. FRONTEND FETCHES TODAY'S FOLLOW-UPS                             │
│    GET /inquiries/follow-ups/today                                 │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 8. BACKEND PROCESSES REQUEST                                       │
│    - Gets today's date: "2026-01-20"                              │
│    - Fetches ALL inquiries from database                           │
│    - Filters: inq.followUpDate === "2026-01-20" ⭐                │
│    - Sorts by priority (urgent first)                              │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 9. BACKEND RETURNS MATCHING INQUIRIES                              │
│    {                                                               │
│      followUps: [                                                  │
│        { id: "...", callerName: "John Smith", ... }               │
│      ],                                                            │
│      count: 1,                                                     │
│      todayDate: "2026-01-20"                                       │
│    }                                                               │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 10. DASHBOARD DISPLAYS FOLLOW-UPS ⭐                               │
│     Today's Follow-Ups          [1 pending]                        │
│     ┌────────────────────────────────────────┐                    │
│     │ ▌John Smith                    URGENT  │                    │
│     │ ▌+44 7700 900123                       │                    │
│     │ ▌Call back with visa fee information   │                    │
│     └────────────────────────────────────────┘                    │
│                                                                    │
│     Staff sees inquiry needs follow-up TODAY!                     │
└────────────────────────────────────────────────────────────────────┘

================================================================================
## UPDATING FOLLOW-UP DATE
================================================================================

### Scenario: Staff needs to reschedule a follow-up

┌────────────────────────────────────────────────────────────────────┐
│ 1. STAFF VIEWS INQUIRY DETAILS                                     │
│    - Clicks on inquiry from list                                   │
│    - Sees current followUpDate: "2026-01-20"                       │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 2. STAFF UPDATES FOLLOW-UP DATE                                    │
│    - Changes date picker to "2026-01-25"                           │
│    - Updates follow-up notes                                       │
│    - Clicks "Update Inquiry"                                       │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 3. FRONTEND SENDS PUT REQUEST                                      │
│    PUT /inquiries/{inquiryId}                                      │
│    Body: {                                                         │
│      followUpDate: "2026-01-25",  ⭐ NEW DATE                      │
│      followUpNotes: "Updated notes..."                             │
│    }                                                               │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 4. BACKEND UPDATES INQUIRY                                         │
│    - Fetches existing inquiry                                      │
│    - Merges with new data                                          │
│    - Saves updated inquiry with new followUpDate ⭐                │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 5. DATABASE NOW STORES UPDATED DATE                                │
│    {                                                               │
│      followUpDate: "2026-01-25",  ⭐ UPDATED                       │
│      updatedAt: "2026-01-20T15:30:00.000Z",                       │
│      ...                                                           │
│    }                                                               │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                          ▼
┌────────────────────────────────────────────────────────────────────┐
│ 6. RESULT                                                          │
│    - Inquiry will NOW appear on 2026-01-25 (not 2026-01-20)      │
│    - When GET /follow-ups/today is called on Jan 25, it will show │
└────────────────────────────────────────────────────────────────────┘

================================================================================
## KEY TECHNICAL DETAILS
================================================================================

### Date Format: YYYY-MM-DD

**Why this format?**
- Consistent with HTML date input (`<input type="date">`)
- Easy to compare strings ("2026-01-20" === "2026-01-20")
- ISO 8601 standard (sortable)
- No timezone confusion

### UK Timezone Handling:

```javascript
// Backend converts server time to UK timezone
const ukNow = new Date().toLocaleString('en-GB', { 
  timeZone: 'Europe/London',
  year: 'numeric',
  month: '2-digit',
  day: '2-digit'
});

// Result: "20/01/2026, 14:30:00"
// Extract date: "20/01/2026"
// Convert to: "2026-01-20"
```

This ensures follow-ups are based on UK business hours, not server timezone.

### Priority Sorting:

```javascript
const priorityOrder = { 
  'urgent': 0,    // Shows first
  'high': 1,      // Shows second
  'normal': 2,    // Shows third
  'low': 3        // Shows last
};

todaysFollowUps.sort((a, b) => {
  const aPriority = priorityOrder[a.priority?.toLowerCase()] ?? 2;
  const bPriority = priorityOrder[b.priority?.toLowerCase()] ?? 2;
  return aPriority - bPriority;
});
```

Urgent follow-ups always appear at the top of the list.

================================================================================
## API USAGE EXAMPLES
================================================================================

### Example 1: Create Inquiry with Follow-Up

```bash
curl -X POST \
  https://abcd1234.supabase.co/functions/v1/make-server-d8c62521/inquiries \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "callerName": "John Smith",
    "callerPhone": "+44 7700 900123",
    "inquiryType": "visa-application",
    "priority": "high",
    "followUpDate": "2026-01-20",
    "followUpNotes": "Call back with visa fee information",
    "status": "pending"
  }'
```

### Example 2: Get Today's Follow-Ups

```bash
curl -X GET \
  https://abcd1234.supabase.co/functions/v1/make-server-d8c62521/inquiries/follow-ups/today \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

### Example 3: Update Follow-Up Date

```bash
curl -X PUT \
  https://abcd1234.supabase.co/functions/v1/make-server-d8c62521/inquiries/inquiry:1705234567890:abc123def \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "followUpDate": "2026-01-25",
    "followUpNotes": "Rescheduled - customer requested later date"
  }'
```

================================================================================
## COMMON USE CASES
================================================================================

### Use Case 1: Staff Logs Call Needing Follow-Up Tomorrow

```
1. Staff answers phone call
2. Customer asks about visa process
3. Staff needs to gather information and call back tomorrow
4. Staff logs inquiry with followUpDate = tomorrow's date
5. Next day, inquiry appears in "Today's Follow-Ups" widget
6. Staff calls customer back
```

### Use Case 2: Rescheduling a Follow-Up

```
1. Follow-up appears in today's list
2. Staff tries to call but customer is unavailable
3. Staff updates inquiry with new followUpDate = 3 days later
4. Inquiry disappears from today's list
5. In 3 days, inquiry appears again in "Today's Follow-Ups"
```

### Use Case 3: Marking Follow-Up as Complete

```
1. Staff completes follow-up call
2. Staff updates inquiry:
   - status: "resolved"
   - followUpDate: null (or leave as-is)
3. Inquiry no longer needs follow-up
4. Can optionally delete or archive the inquiry
```

================================================================================
## SUMMARY
================================================================================

✅ **Database Structure:** KV Store with inquiry objects containing followUpDate
✅ **Date Format:** YYYY-MM-DD (ISO 8601)
✅ **Creation:** POST /inquiries with followUpDate in body
✅ **Update:** PUT /inquiries/{id} with new followUpDate
✅ **Retrieval:** GET /inquiries/follow-ups/today returns today's matches
✅ **Filtering:** Backend compares followUpDate === todayUK
✅ **Sorting:** Urgent → High → Normal → Low priority
✅ **Display:** Staff Dashboard shows follow-ups in widget
✅ **Timezone:** Uses UK timezone for "today" calculation

**The mechanism is simple but powerful:**
- Staff sets a date when logging an inquiry
- Backend stores it in the database
- Each day, backend checks which inquiries have followUpDate = today
- Dashboard displays those inquiries to staff
- Staff can update the date to reschedule

================================================================================
Document Created: January 15, 2026
Last Updated: January 15, 2026
Version: 1.0
================================================================================
