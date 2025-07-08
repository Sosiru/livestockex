# LiveStockEx <> Dime Credit Scoring Integration API

This document provides comprehensive information about integrating LiveStockEx platform with Dime's credit scoring API to assess farmer creditworthiness and determine appropriate credit limits.

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Credit Scoring Endpoint](#credit-scoring-endpoint)
- [Request Parameters](#request-parameters)
- [Response Format](#response-format)
- [Error Handling](#error-handling)
- [Integration Examples](#integration-examples)
- [Rate Limits](#rate-limits)
- [Support](#support)

## Overview

The Dime Credit Scoring API provides real-time credit assessment for livestock farmers based on various factors including:
- Identity verification status
- Platform reputation and ratings
- Livestock portfolio value and quality
- Trading activity and transaction history
- Platform engagement metrics
- Insurance coverage
- Default history

## Authentication

### Endpoint
```
POST https://api.dimeapp.co.ke/api/partner/token/
```

### Request Body
```json
{
  "consumer_key": "your_consumer_key",
  "consumer_secret": "your_consumer_secret"
}
```

### Sample Request
```json
{
  "consumer_key": "xxxxxxxxx",
  "consumer_secret": "xxxxxxxxxxxxxx"
}
```

### Sample Response
```json
{
  "code": "200.001",
  "data": {
    "token": "xxxxxxxxxxxxxxxxx",
    "expires_at": "1751984041"
  }
}
```

### Authentication Notes
- Tokens are time-limited and expire based on the `expires_at` timestamp
- Store the token securely and implement refresh logic before expiration
- Use the token in the Authorization header for credit scoring requests

## Credit Scoring Endpoint

### Endpoint
```
POST https://api.dimeapp.co.ke/api/partner/score/
```

### Headers
```
Authorization: Bearer {your_token}
Content-Type: application/json
```

## Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `has_nid` | boolean | Yes | Whether farmer has National ID verification |
| `has_selfie` | boolean | Yes | Whether farmer has selfie verification |
| `platform_verified` | boolean | Yes | Whether farmer is verified on platform |
| `star_rating` | float | Yes | Farmer's star rating (0.0 - 5.0) |
| `num_animals` | integer | Yes | Number of livestock animals owned |
| `avg_value_kes` | integer | Yes | Average value of livestock in KES |
| `breed_quality` | string | Yes | Quality of livestock breed ("standard", "premium", "basic") |
| `sales_value_3months` | integer | Yes | Total sales value in last 3 months (KES) |
| `wallet_inflows` | integer | Yes | Total wallet inflows (KES) |
| `wallet_outflows` | integer | Yes | Total wallet outflows (KES) |
| `time_on_platform_months` | integer | Yes | Time farmer has been on platform (months) |
| `num_transactions` | integer | Yes | Total number of transactions |
| `has_life_insurance` | boolean | Yes | Whether farmer has life insurance |
| `has_livestock_insurance` | boolean | Yes | Whether farmer has livestock insurance |
| `has_default_history` | boolean | Yes | Whether farmer has default history |

### Sample Request
```json
{
  "has_nid": true,
  "has_selfie": true,
  "platform_verified": true,
  "star_rating": 4.5,
  "num_animals": 4,
  "avg_value_kes": 57208,
  "breed_quality": "standard",
  "sales_value_3months": 168600,
  "wallet_inflows": 216856,
  "wallet_outflows": 134700,
  "time_on_platform_months": 2,
  "num_transactions": 10,
  "has_life_insurance": true,
  "has_livestock_insurance": true,
  "has_default_history": false
}
```

## Response Format

### Sample Response
```json
{
  "code": "200.001",
  "message": {
    "credit_score": 77.0,
    "risk_tier": "Farmer 2",
    "credit_limit": "KES 50,000–75,000",
    "score_breakdown": {
      "Identity Verification": 10.0,
      "Reputation & Ratings": 13.5,
      "Livestock Portfolio": 11.0,
      "Trading Activity": 20.0,
      "Platform Engagement": 7.5,
      "Risk Cover": 10.0,
      "Default History": 5.0
    },
    "recommendations": "Increase livestock portfolio size and consider higher-value breeds; Increase platform engagement through regular transactions; Good credit profile - consider expanding business operations"
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `code` | string | Response status code |
| `credit_score` | float | Overall credit score (0-100) |
| `risk_tier` | string | Risk classification tier |
| `credit_limit` | string | Recommended credit limit range |
| `score_breakdown` | object | Detailed breakdown of score components |
| `recommendations` | string | Actionable recommendations for improvement |

### Risk Tiers
- **Farmer 1**: High-risk tier (Score: 0-50)
- **Farmer 2**: Medium-risk tier (Score: 51-75)
- **Farmer 3**: Low-risk tier (Score: 76-100)

## Error Handling

### Common Error Codes
- `400.001`: Invalid request parameters
- `401.001`: Authentication failed
- `403.001`: Access denied
- `500.001`: Internal server error

### Error Response Format
```json
{
  "code": "400.001",
  "message": "Invalid request parameters",
  "details": "Missing required field: has_nid"
}
```

## Integration Examples

### Python Example
```python
import requests
import json

# Authenticate
auth_url = "https://api.dimeapp.co.ke/api/partner/token/"
auth_data = {
    "consumer_key": "ck",
    "consumer_secret": "cs"
}

auth_response = requests.post(auth_url, json=auth_data)
token = auth_response.json()["data"]["token"]

# Get credit score
score_url = "https://api.dimeapp.co.ke/api/partner/score/"
headers = {
    "Authorization": f"Bearer {token}",
    "Content-Type": "application/json"
}

farmer_data = {
    "has_nid": True,
    "has_selfie": True,
    "platform_verified": True,
    "star_rating": 4.5,
    "num_animals": 4,
    "avg_value_kes": 57208,
    "breed_quality": "standard",
    "sales_value_3months": 168600,
    "wallet_inflows": 216856,
    "wallet_outflows": 134700,
    "time_on_platform_months": 2,
    "num_transactions": 10,
    "has_life_insurance": True,
    "has_livestock_insurance": True,
    "has_default_history": False
}

score_response = requests.post(score_url, json=farmer_data, headers=headers)
credit_score = score_response.json()

print(f"Credit Score: {credit_score['message']['credit_score']}")
print(f"Risk Tier: {credit_score['message']['risk_tier']}")
print(f"Credit Limit: {credit_score['message']['credit_limit']}")
```

### JavaScript Example
```javascript
// Authenticate
const authenticate = async () => {
  const response = await fetch('https://api.dimeapp.co.ke/api/partner/token/', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      consumer_key: 'your_consumer_key',
      consumer_secret: 'your_consumer_secret'
    })
  });
  
  const data = await response.json();
  return data.data.token;
};

// Get credit score
const getCreditScore = async (token, farmerData) => {
  const response = await fetch('https://api.dimeapp.co.ke/api/partner/score/', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify(farmerData)
  });
  
  return await response.json();
};

// Usage
const token = await authenticate();
const farmerData = {
  has_nid: true,
  has_selfie: true,
  platform_verified: true,
  star_rating: 4.5,
  num_animals: 4,
  avg_value_kes: 57208,
  breed_quality: "standard",
  sales_value_3months: 168600,
  wallet_inflows: 216856,
  wallet_outflows: 134700,
  time_on_platform_months: 2,
  num_transactions: 10,
  has_life_insurance: true,
  has_livestock_insurance: true,
  has_default_history: false
};

const creditScore = await getCreditScore(token, farmerData);
console.log(creditScore);
```

## Rate Limits

- **Authentication**: 100 requests per hour
- **Credit Scoring**: 1000 requests per hour
- **Burst Limit**: 10 requests per minute

## Best Practices

1. **Token Management**: Implement proper token refresh logic
2. **Error Handling**: Always handle API errors gracefully
3. **Data Validation**: Validate all input parameters before sending
4. **Caching**: Cache tokens until expiration to reduce authentication calls
5. **Monitoring**: Monitor API usage and response times
6. **Security**: Keep consumer keys and secrets secure

## Support

For technical support and API access:
- Email: stephenosiru@gmail.com


---

**Version**: 1.0  
**Last Updated**: July 2025  
**API Base URL**: https://api.dimeapp.co.ke
