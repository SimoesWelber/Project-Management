# Frontend Components

This directory contains reusable React components.

## Structure

Components are organized by feature or type:

```
components/
├── common/           # Shared components
│   ├── Button.jsx
│   ├── Input.jsx
│   ├── Modal.jsx
│   └── Table.jsx
├── layout/           # Layout components
│   ├── Header.jsx
│   ├── Sidebar.jsx
│   └── Footer.jsx
├── document/         # Document-related components
│   ├── DocumentCard.jsx
│   ├── DocumentForm.jsx
│   └── DocumentList.jsx
└── review/           # Review-related components
    ├── ReviewPanel.jsx
    └── ReviewComments.jsx
```

## Component Guidelines

### Functional Components
Use functional components with hooks:

```jsx
import React, { useState, useEffect } from 'react';

const DocumentCard = ({ document }) => {
  const [expanded, setExpanded] = useState(false);
  
  return (
    <div className="document-card">
      <h3>{document.title}</h3>
      <button onClick={() => setExpanded(!expanded)}>
        {expanded ? 'Collapse' : 'Expand'}
      </button>
      {expanded && <p>{document.description}</p>}
    </div>
  );
};

export default DocumentCard;
```

### Component Structure
1. Imports
2. Component definition
3. State and hooks
4. Helper functions
5. JSX return
6. Export

### Best Practices
- Keep components small and focused
- Use meaningful prop names
- Implement PropTypes or TypeScript
- Extract reusable logic to custom hooks
- Use composition over inheritance
- Memoize expensive computations

### Props Validation

```jsx
import PropTypes from 'prop-types';

DocumentCard.propTypes = {
  document: PropTypes.shape({
    id: PropTypes.string.isRequired,
    title: PropTypes.string.isRequired,
    description: PropTypes.string,
    status: PropTypes.string.isRequired,
  }).isRequired,
  onEdit: PropTypes.func,
  onDelete: PropTypes.func,
};
```

### Styling
- Use CSS modules or styled-components
- Follow consistent naming conventions
- Keep styles co-located with components
- Use design tokens for consistency
