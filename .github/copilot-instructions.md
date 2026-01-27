# Copilot Instructions

This file contains instructions for GitHub Copilot and other LLM assistants when generating code for this project.

## Project Overview

**TCX Reducer** is a client-side web application that reduces TCX (Training Center XML) file size by thinning out GPS trackpoints.

### Technology Stack

- **Frontend**: Pure HTML, CSS, JavaScript (no frameworks)
- **Hosting**: Cloudflare Pages
- **Build**: No build step required (static files only)

### Key Features

- Drag & drop file upload
- Client-side XML parsing with DOMParser
- Trackpoint reduction algorithm
- File download generation
- Responsive design for mobile devices

## Development Guidelines

### Code Style

- Use vanilla JavaScript (no frameworks or libraries)
- Keep all code in a single `index.html` file for simplicity
- Use modern ES6+ syntax
- Maintain responsive design for mobile compatibility

### TCX File Processing

When working with TCX files:

1. **XML Parsing**: Use `DOMParser` for parsing TCX/XML content
2. **Validation**: Always validate TCX structure (check for `TrainingCenterDatabase` root element)
3. **Trackpoints**: Target `Trackpoint` elements for reduction operations
4. **Serialization**: Use `XMLSerializer` to convert back to string

```javascript
// Example: Parse and validate TCX
const parser = new DOMParser();
const xmlDoc = parser.parseFromString(content, 'text/xml');

// Check for parse errors
const parserError = xmlDoc.querySelector('parsererror');
if (parserError) {
    throw new Error('Failed to parse XML');
}

// Validate TCX structure
const root = xmlDoc.documentElement;
if (!root || root.tagName !== 'TrainingCenterDatabase') {
    throw new Error('Not a valid TCX file');
}
```

### File Handling

- Maximum file size: 10MB
- Supported format: `.tcx` files only
- All processing happens client-side (no server uploads)

## Deployment

### Platform: Cloudflare Pages

This application is deployed on **Cloudflare Pages**.

- Static files are served from the `public/` directory
- Configuration is in `wrangler.jsonc`

#### Local Development

```bash
# Using Wrangler CLI
npx wrangler pages dev public

# Or simply open public/index.html in a browser
```

#### Deployment Commands

```bash
# Deploy to Cloudflare Pages
npx wrangler pages deploy public
```

## Security Guidelines

### Security Checklist for File Processing

This application processes user-uploaded files. Follow these security measures:

#### 1. Input Validation

- [ ] **File size limit**: Enforce maximum file size (10MB)
- [ ] **Filename validation**: Check for path traversal attacks (`..`, `/`, `\`)
- [ ] **File extension check**: Only accept `.tcx` files
- [ ] **Content validation**: Verify XML structure before processing

```javascript
// Example: Filename validation
function validateFileName(name) {
    if (!name || typeof name !== 'string') {
        return false;
    }
    // Path traversal protection
    if (name.includes('..') || name.includes('/') || name.includes('\\')) {
        return false;
    }
    // Length limit
    if (name.length > 255) {
        return false;
    }
    // TCX extension check
    if (!name.toLowerCase().endsWith('.tcx')) {
        return false;
    }
    return true;
}
```

#### 2. Output Sanitization

- [ ] **HTML escaping**: Escape all user-provided data before displaying
- [ ] **Safe filename generation**: Remove dangerous characters from output filenames

```javascript
// Example: HTML escape function
function escapeHtml(unsafe) {
    if (typeof unsafe !== 'string') {
        return '';
    }
    return unsafe
        .replace(/&/g, "&amp;")
        .replace(/</g, "&lt;")
        .replace(/>/g, "&gt;")
        .replace(/"/g, "&quot;")
        .replace(/'/g, "&#x27;");
}

// Example: Safe filename generation
function generateSafeFileName(originalName) {
    const nameWithoutExt = originalName.replace(/\.tcx$/i, '');
    // Remove dangerous characters (preserve Unicode)
    const safeName = nameWithoutExt.replace(/[<>:"/\\|?*\x00-\x1f]/g, '_');
    return `${safeName}_reduced.tcx`;
}
```

#### 3. XML Processing Security

- [ ] **Parse error handling**: Check for `parsererror` after parsing
- [ ] **Structure validation**: Verify expected XML structure
- [ ] **Error messages**: Don't expose internal details in error messages

```javascript
// Example: Safe XML parsing
function parseAndValidateTCX(content) {
    const parser = new DOMParser();
    const xmlDoc = parser.parseFromString(content, 'text/xml');

    // Check for parse errors
    const parserError = xmlDoc.querySelector('parsererror');
    if (parserError) {
        throw new Error('Failed to parse XML');
    }

    // Validate structure
    const root = xmlDoc.documentElement;
    if (!root || root.tagName !== 'TrainingCenterDatabase') {
        throw new Error('Not a valid TCX file');
    }

    return xmlDoc;
}
```

#### 4. Client-Side Only Processing

This application processes all data client-side, which provides inherent security benefits:

- ✅ No server upload (data never leaves the user's browser)
- ✅ No data storage (files are processed in memory only)
- ✅ No network requests for file processing

### Numeric Input Validation

Always validate numeric inputs:

```javascript
function validateSkipCount(value) {
    const num = parseInt(value, 10);
    if (isNaN(num) || num < 1 || num > 100) {
        return false;
    }
    return true;
}
```

## General Guidelines

- Prioritize data security and user privacy
- Keep the application simple and dependency-free
- Maintain backward compatibility with existing TCX files
- Test with various TCX file formats from different GPS devices

