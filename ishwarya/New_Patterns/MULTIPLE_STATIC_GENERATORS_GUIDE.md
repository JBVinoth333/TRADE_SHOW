# Multiple STATIC Generators Pattern Guide

**Date**: February 12, 2026  
**Status**: ✅ DOCUMENTED  
**Category**: Best Practices & Patterns  

---

## Overview

While single STATIC generators are useful for individual fixed values, many real-world APIs require **multiple STATIC generators** to generate complete test data. This guide documents the patterns, rules, and best practices for using multiple STATIC generators in a single configuration.

---

## Rule: Multiple STATIC Generators Are Valid and Recommended

### Core Rule

✅ **Multiple STATIC generators can and SHOULD be used in the same generator configuration file** when:

1. **Different fields have different fixed value sets**
   - Example: `customer_name`, `customer_email`, `customer_phone` all use STATIC but with different values
   
2. **Related data needs multiple fixed enumerations**
   - Example: `expense_category`, `payment_method`, `expense_date` in expense tracker
   
3. **Foundation layers need multiple configuration values**
   - Example: `locale`, `timezone`, `country_code` for localization setup

4. **Error testing requires multiple invalid values**
   - Example: `nonexistent_customer_id`, `invalid_expense_id` for 404 testing

---

## Pattern Examples from Production Code

### Example 1: Customer Generator (3 STATIC Generators)

```json
{
  "generators": {
    "customer_name": [{
      "type": "static",
      "value": ["John Doe", "Jane Smith", "Robert Johnson", "Maria Garcia", "Michael Chen"]
    }],
    
    "customer_email": [{
      "type": "static",
      "value": ["john.doe@example.com", "jane.smith@example.com", "robert.johnson@example.com", 
                "maria.garcia@example.com", "michael.chen@example.com"]
    }],
    
    "customer_phone": [{
      "type": "static",
      "value": ["+91-9876543210", "+91-9876543211", "+91-9876543212", "+91-9876543213", "+91-9876543214"]
    }]
  }
}
```

**Usage**: Create customer with name, email, and phone from separate STATIC generators
**Benefit**: Each field independently configured, easy to add/modify values

---

### Example 2: Expense Tracker (8 STATIC Generators)

```json
{
  "generators": {
    "expense_description": [{
      "type": "static",
      "value": ["Grocery shopping", "Gas station fill-up", "Restaurant dinner", "Movie tickets", ...]
    }],
    
    "expense_amount": [{
      "type": "static",
      "value": [50.00, 100.00, 150.50, 75.25, 200.00, 125.75, 300.00, 55.99, 45.50, 85.00]
    }],
    
    "expense_category": [{
      "type": "static",
      "value": ["Food", "Transportation", "Entertainment", "Health", "Shopping", "Utilities", "Education", "Miscellaneous"]
    }],
    
    "expense_date": [{
      "type": "static",
      "value": ["2026-02-01", "2026-02-05", "2026-02-10", "2026-02-12", "2026-02-15"]
    }],
    
    "payment_method": [{
      "type": "static",
      "value": ["Credit Card", "Debit Card", "Cash", "Digital Wallet", "Bank Transfer"]
    }],
    
    "start_date": [{
      "type": "static",
      "value": ["2026-02-01"]
    }],
    
    "end_date": [{
      "type": "static",
      "value": ["2026-02-28"]
    }],
    
    "summary_period": [{
      "type": "static",
      "value": ["daily", "weekly", "monthly", "yearly"]
    }]
  }
}
```

**Usage**: Complete expense creation with 8 different fixed-value fields  
**Benefit**: Comprehensive test data covering all required fields

---

### Example 3: Todo List (4 STATIC Generators)

```json
{
  "generators": {
    "todo_id": [{
      "type": "static",
      "value": ["TD-1001", "TD-1002", "TD-1003", "TD-1004"]
    }],
    
    "todo_title": [{
      "type": "static",
      "value": ["Buy groceries", "Schedule meeting", "Pay bills", "Write report", "Call client"]
    }],
    
    "todo_description": [{
      "type": "static",
      "value": ["Remember to pick up milk and eggs", "Discuss Q1 roadmap", "Electricity and internet bills", 
                "Draft the monthly analysis", "Follow up about the proposal"]
    }],
    
    "todo_status": [{
      "type": "static",
      "value": ["PENDING", "COMPLETED"]
    }]
  }
}
```

**Usage**: Todo CRUD operations with different static field sets  
**Benefit**: Separates different field concerns (IDs, titles, descriptions, statuses)

---

## Best Practices for Multiple STATIC Generators

### 1. Logical Grouping
Group related STATIC generators together in the file for readability:

```json
{
  "generators": {
    // Foundation/Configuration
    "locale": [{"type": "static", "value": [...]}],
    "timezone": [{"type": "static", "value": [...]}],
    
    // Entity Fields
    "customer_name": [{"type": "static", "value": [...]}],
    "customer_email": [{"type": "static", "value": [...]}],
    "customer_phone": [{"type": "static", "value": [...]}],
    
    // Error Testing
    "nonexistent_customer_id": [{"type": "static", "value": [...]}],
    "invalid_email": [{"type": "static", "value": [...]}]
  }
}
```

### 2. Consistent Value Counts
Try to use same number of values across related generators for test data variety:

```json
{
  // Good: All have 5 values for balanced combinations
  "customer_name": [{"type": "static", "value": [5 names]}],
  "customer_email": [{"type": "static", "value": [5 emails]}],
  "customer_phone": [{"type": "static", "value": [5 phones]}],
  
  // Bad: Inconsistent counts reduce test combinations
  "customer_name": [{"type": "static", "value": [5 names]}],
  "customer_email": [{"type": "static", "value": [2 emails]}],
  "customer_phone": [{"type": "static", "value": [10 phones]}]
}
```

### 3. Value Alignment
Ensure related values logically align:

```json
{
  // Good: Names and emails match
  "customer_name": [{"type": "static", "value": [
    "John Doe",
    "Jane Smith",
    "Robert Johnson"
  ]}],
  "customer_email": [{"type": "static", "value": [
    "john.doe@example.com",
    "jane.smith@example.com",
    "robert.johnson@example.com"
  ]}]
}
```

### 4. Naming Convention
Use descriptive, snake_case names for multiple STATIC generators:

✅ **Good**:
- `customer_name`, `customer_email`, `customer_phone`
- `expense_category`, `expense_date`, `payment_method`
- `error_codes`, `invalid_ids`, `nonexistent_values`

❌ **Bad**:
- `name`, `email`, `phone` (not descriptive enough)
- `static1`, `static2`, `static3` (meaningless names)
- `customerName`, `paymentMethod` (not snake_case)

---

## Multiple STATIC with DYNAMIC Pattern

### Common Scenario: Static Foundation + Dynamic Entity Creation

```json
{
  "generators": {
    // Level 1: Multiple STATIC generators for configuration
    "customer_name": [{"type": "static", "value": [...]}],
    "customer_email": [{"type": "static", "value": [...]}],
    "customer_phone": [{"type": "static", "value": [...]}],
    
    // Level 2: DYNAMIC that uses multiple STATIC generators
    "customer_id": [{
      "type": "dynamic",
      "generatorOperationId": "createCustomer",
      "dataPath": "$.response.body:$.id",
      "name": "created_customer",
      "params": {
        "name": "$generators:#/generators/customer_name",
        "email": "$generators:#/generators/customer_email",
        "phone": "$generators:#/generators/customer_phone"
      }
    }],
    
    // Error testing: Multiple STATIC for invalid scenarios
    "nonexistent_customer_id": [{"type": "static", "value": ["cust_invalid_123", "cust_notfound_456"]}]
  }
}
```

**Pattern**: Static → Dynamic (consuming multiple STATIC) → Static (error cases)

---

## API Mapping with Multiple STATIC Generators

### Real-World Example: Expense API with 8 STATIC Generators

```json
{
  "apis": {
    "createExpense": {
      "201": {
        "description": "#/generators/expense_description",
        "amount": "#/generators/expense_amount",
        "category": "#/generators/expense_category",
        "date": "#/generators/expense_date",
        "paymentMethod": "#/generators/payment_method"
      },
      "400": {
        "description": "Invalid expense data"
      }
    },
    
    "updateExpense": {
      "200": {
        "expenseId": "#/generators/expense_id",
        "description": "#/generators/expense_description",
        "amount": "#/generators/expense_amount",
        "category": "#/generators/expense_category",
        "date": "#/generators/expense_date",
        "paymentMethod": "#/generators/payment_method"
      },
      "404": {
        "expenseId": "#/generators/nonexistent_expense_id"
      }
    },
    
    "getAllExpenses": {
      "200": {
        "category": "#/generators/expense_category",
        "startDate": "#/generators/start_date",
        "endDate": "#/generators/end_date"
      }
    }
  }
}
```

**Count**: 8 different STATIC generators used across 3 API operations

---

## When NOT to Use Multiple STATIC Generators

❌ **Use DYNAMIC instead if**:
- Values change based on business logic
- Need real IDs from API responses
- Values depend on database state

❌ **Use REMOTE instead if**:
- Values are organization-specific
- Need computed values (dates, hashes)
- Need custom Java function logic

❌ **Use REFERENCE instead if**:
- Generator already exists in another module
- Want to avoid duplication across modules

---

## Decision Tree: Single vs Multiple STATIC

```
Is the field value fixed and predefined?
├─ YES → Use STATIC
│   ├─ Is there only ONE field?
│   │   └─ Use single generator
│   └─ Are there MULTIPLE related fields?
│       └─ Use multiple STATIC generators (THIS PATTERN)
│
└─ NO → Don't use STATIC
    ├─ Need API call? → Use DYNAMIC
    ├─ Need custom function? → Use REMOTE
    ├─ Already exists elsewhere? → Use REFERENCE
    └─ Conditional logic? → Use CONDITIONAL
```

---

## Summary

### Key Rules ✅

1. **Multiple STATIC generators are VALID and RECOMMENDED** for different fixed-value fields
2. **Each generator handles ONE field's fixed values** (separation of concerns)
3. **Multiple STATIC generators can be consumed by a SINGLE DYNAMIC generator** (composition)
4. **Use multiple STATIC for**: Different fields, foundation layers, error testing, configuration
5. **Combine with DYNAMIC**: STATIC values as parameters to API-calling DYNAMIC generators

### Pattern Examples
- **Customer**: 3 STATIC (name, email, phone) + 1 DYNAMIC (customer_id) + 1 STATIC (error IDs)
- **Expense**: 8 STATIC (description, amount, category, date, payment_method, start_date, end_date, period)
- **Todo**: 4 STATIC (id, title, description, status)

### Common Mistake ❌

**WRONG**: "I should use DYNAMIC for all these values"
```json
{
  "customer_name": {
    "type": "dynamic",
    "generatorOperationId": "getNames"  // ❌ Unnecessary API call
  }
}
```

**CORRECT**: Use STATIC for fixed values
```json
{
  "customer_name": {
    "type": "static",
    "value": ["John", "Jane", ...]  // ✅ Simple and fast
  }
}
```

---

## Next Steps

1. ✅ Review examples in this guide
2. ✅ Check your API's fixed fields
3. ✅ Identify which fields should use STATIC
4. ✅ Create multiple STATIC generators for different fields
5. ✅ Reference them in your "apis" section
6. ✅ Test with your API to ensure correct mapping

---

## Related Documentation

- [TEMPLATES_AND_EXAMPLES.md](TEMPLATES_AND_EXAMPLES.md) - Generator templates
- [GENERATOR_TYPES_COMPARISON.md](GENERATOR_TYPES_COMPARISON.md) - Type selection guide
- [DATA_GENERATION_PATTERNS.md](DATA_GENERATION_PATTERNS.md) - Dependency patterns
- [COMMON_INSTRUCTIONS.md](COMMON_INSTRUCTIONS.md) - Common generator configurations
