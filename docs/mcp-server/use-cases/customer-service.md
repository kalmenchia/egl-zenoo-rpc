---
title: "Customer Service Automation"
version: "1.0"
last_updated: "2025-11-15"
status: "Active"
---

# Customer Service Automation

## Business Scenario

Your customer service team needs to quickly access customer information, order history, and resolve issues. Instead of navigating Odoo's interface, they can use Claude Desktop to get instant answers.

---

## Example Queries

### Query 1: Customer Lookup

**User Question**:
```
Find all information about customer "Azure Interior"
```

**Claude's Actions**:
1. Uses `search_records` to find customer
2. Uses `get_record` to get full details
3. Uses `complex_search` to find related orders

**Result**:
```
Azure Interior (Customer #15)

Contact Information:
- Email: azure@example.com
- Phone: +1-555-0123
- Address: 123 Main St, Los Angeles, CA

Customer Since: January 2024
Total Orders: 23
Lifetime Value: $45,600

Recent Orders:
- SO00234: $2,300 (Nov 10) - Delivered
- SO00198: $1,890 (Nov 3) - In Progress
- SO00156: $3,200 (Oct 28) - Delivered
```

---

### Query 2: Order Status

**User Question**:
```
What's the status of order SO00234?
```

**Claude's Actions**:
1. Uses `search_records` with domain filter
2. Gets order details
3. Checks delivery status

**Result**:
```
Order SO00234:
- Customer: Azure Interior
- Date: November 10, 2025
- Amount: $2,300.00
- Status: Delivered ✅
- Delivery Date: November 12, 2025
- Tracking: PKG1234567890

Items:
1. Office Chair Pro x5 - $1,500
2. Desk Lamp LED x10 - $800
```

---

### Query 3: Customer Issue Resolution

**User Question**:
```
Customer Azure Interior says they haven't received invoice for order SO00234.
Can you check and resend?
```

**Claude's Actions**:
1. Finds order SO00234
2. Checks invoice status
3. Provides invoice details

**Result**:
```
Order SO00234 Invoice Status:

Invoice #INV/2025/001234
- Date: November 10, 2025
- Amount: $2,300.00
- Status: Sent ✅
- Sent to: azure@example.com
- Sent on: November 10, 2025 2:30 PM

The invoice was sent to azure@example.com.
Would you like me to:
1. Resend the invoice
2. Send to a different email
3. Generate a new invoice
```

---

## MCP Tools Used

| Tool | Purpose |
|------|---------|
| `search_records` | Find customers, orders, invoices |
| `get_record` | Get detailed customer/order info |
| `complex_search` | Find related records (orders, invoices) |
| `analytics_query` | Customer statistics and metrics |

---

## Benefits

✅ **Faster Response Times**: Instant access to information
✅ **Reduced Errors**: No manual navigation mistakes
✅ **Better Customer Experience**: Quick, accurate answers
✅ **Empowered Team**: Anyone can query data naturally

---

## Best Practices

1. **Be Specific**: Include customer name or ID
2. **Ask Follow-ups**: Claude remembers context
3. **Verify Before Acting**: Always confirm critical information
4. **Use Natural Language**: No need to learn Odoo terminology

---

**Last Updated**: 2025-11-15
**Maintained By**: Zenoo RPC Development Team
