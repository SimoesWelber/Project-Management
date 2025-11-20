# Frontend Pages

This directory contains page-level components that represent routes in the application.

## Structure

```
pages/
├── Home/
│   ├── Home.jsx
│   ├── Home.module.css
│   └── index.js
├── Dashboard/
│   ├── Dashboard.jsx
│   └── index.js
├── Documents/
│   ├── DocumentList.jsx
│   ├── DocumentDetail.jsx
│   ├── DocumentCreate.jsx
│   └── index.js
├── Reviews/
│   ├── ReviewList.jsx
│   ├── ReviewDetail.jsx
│   └── index.js
├── Users/
│   ├── UserList.jsx
│   ├── UserProfile.jsx
│   └── index.js
└── Auth/
    ├── Login.jsx
    ├── Register.jsx
    └── index.js
```

## Page Guidelines

### Page Components
Pages are container components that:
- Correspond to application routes
- Fetch data from APIs
- Manage page-level state
- Compose smaller components
- Handle page-specific logic

### Example Page

```jsx
// pages/Documents/DocumentList.jsx
import React, { useState, useEffect } from 'react';
import { useNavigate } from 'react-router-dom';
import DocumentCard from '../../components/document/DocumentCard';
import documentService from '../../services/documentService';
import './DocumentList.module.css';

const DocumentList = () => {
  const [documents, setDocuments] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  const navigate = useNavigate();

  useEffect(() => {
    fetchDocuments();
  }, []);

  const fetchDocuments = async () => {
    try {
      const data = await documentService.getAll();
      setDocuments(data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const handleDocumentClick = (id) => {
    navigate(`/documents/${id}`);
  };

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="document-list-page">
      <h1>Documents</h1>
      <button onClick={() => navigate('/documents/new')}>
        New Document
      </button>
      <div className="document-grid">
        {documents.map(doc => (
          <DocumentCard
            key={doc.id}
            document={doc}
            onClick={() => handleDocumentClick(doc.id)}
          />
        ))}
      </div>
    </div>
  );
};

export default DocumentList;
```

### Routing Setup

Pages are connected to routes in the main App component:

```jsx
import { BrowserRouter, Routes, Route } from 'react-router-dom';
import Home from './pages/Home';
import DocumentList from './pages/Documents/DocumentList';
import DocumentDetail from './pages/Documents/DocumentDetail';

function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/documents" element={<DocumentList />} />
        <Route path="/documents/:id" element={<DocumentDetail />} />
        {/* More routes... */}
      </Routes>
    </BrowserRouter>
  );
}
```

## Best Practices

- One page per route
- Keep pages focused on layout and data fetching
- Delegate rendering to smaller components
- Handle loading and error states
- Implement proper SEO metadata
- Use lazy loading for code splitting
