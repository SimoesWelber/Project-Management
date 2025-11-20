# Backend Controllers

This directory contains API route controllers that handle incoming HTTP requests.

## Structure

Each controller should:
- Handle a specific resource or feature
- Validate request data
- Call appropriate services
- Return appropriate HTTP responses
- Handle errors gracefully

## Example Controllers

- `authController.js` - Authentication endpoints
- `documentController.js` - Document CRUD operations
- `userController.js` - User management
- `reviewController.js` - Review operations

## Controller Pattern

```javascript
// Example: documentController.js

const documentService = require('../services/documentService');

exports.getAllDocuments = async (req, res) => {
  try {
    const documents = await documentService.findAll(req.query);
    res.json({ success: true, data: documents });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

exports.getDocumentById = async (req, res) => {
  try {
    const document = await documentService.findById(req.params.id);
    if (!document) {
      return res.status(404).json({ success: false, error: 'Document not found' });
    }
    res.json({ success: true, data: document });
  } catch (error) {
    res.status(500).json({ success: false, error: error.message });
  }
};

// Additional CRUD operations...
```
