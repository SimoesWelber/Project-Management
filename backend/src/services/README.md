# Backend Services

This directory contains business logic and data access services.

## Purpose

Services encapsulate business logic and interact with data models. Controllers call services, not models directly.

## Structure

Each service should:
- Implement business rules
- Handle data transformations
- Interact with database models
- Be testable in isolation
- Throw meaningful errors

## Example Services

- `authService.js` - Authentication logic
- `documentService.js` - Document business logic
- `userService.js` - User management logic
- `reviewService.js` - Review workflow logic
- `emailService.js` - Email notifications
- `fileService.js` - File handling

## Service Pattern

```javascript
// Example: documentService.js

const Document = require('../models/Document');

class DocumentService {
  async findAll(filters = {}) {
    const { status, submitterId, page = 1, limit = 20 } = filters;
    const query = {};
    
    if (status) query.status = status;
    if (submitterId) query.submitter_id = submitterId;
    
    const documents = await Document.find(query)
      .skip((page - 1) * limit)
      .limit(limit)
      .sort({ created_at: -1 });
    
    return documents;
  }
  
  async findById(id) {
    return await Document.findById(id);
  }
  
  async create(data, userId) {
    const document = new Document({
      ...data,
      submitter_id: userId,
      status: 'DRAFT'
    });
    return await document.save();
  }
  
  async update(id, data, userId) {
    // Implement business rules
    const document = await this.findById(id);
    if (!document) throw new Error('Document not found');
    
    // Check permissions
    if (document.submitter_id !== userId) {
      throw new Error('Unauthorized');
    }
    
    Object.assign(document, data);
    return await document.save();
  }
  
  // Additional methods...
}

module.exports = new DocumentService();
```
