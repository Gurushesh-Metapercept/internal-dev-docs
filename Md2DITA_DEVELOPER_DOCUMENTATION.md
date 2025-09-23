# Markdown to DITA Parser - Developer Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [Core Processing Flow](#core-processing-flow)
6. [Key Components](#key-components)
7. [Database Schema](#database-schema)
8. [API Endpoints](#api-endpoints)
9. [Processing Pipeline](#processing-pipeline)
10. [State Management](#state-management)
11. [Utility Functions](#utility-functions)
12. [Configuration](#configuration)
13. [Development Setup](#development-setup)
14. [Deployment](#deployment)
15. [Error Handling](#error-handling)
16. [Performance Considerations](#performance-considerations)
17. [Security Features](#security-features)
18. [Testing](#testing)
19. [Troubleshooting](#troubleshooting)

## Project Overview

The **Markdown to DITA Parser** is a Node.js-based web service that converts Markdown (.md) and MDX (.mdx) files into DITA (Darwin Information Typing Architecture) XML format. The system provides a RESTful API for file upload, processing, and download of converted DITA files.

### Key Features
- **Batch Processing**: Handles multiple markdown files in ZIP archives
- **User Authentication**: JWT-based authentication system
- **File Validation**: Pre-flight checks for file structure and content
- **DITA Compliance**: Generates valid DITA XML with proper DTD declarations
- **Task Generation**: Automatically creates DITA task files from markdown content
- **Image Handling**: Copies and references images in the output
- **Hierarchical Structure**: Maintains document hierarchy in DITA map files
- **Logging**: Comprehensive logging of processing steps and errors

## System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client App    │    │   Express API   │    │   MongoDB       │
│                 │◄──►│                 │◄──►│                 │
│ - File Upload   │    │ - Authentication│    │ - User Data     │
│ - Download      │    │ - Processing    │    │ - Tag Mappings  │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ Processing      │
                    │ Pipeline        │
                    │                 │
                    │ - Markdown-It   │
                    │ - Cheerio       │
                    │ - HTML-to-JSON  │
                    │ - DITA Schema   │
                    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ File System     │
                    │                 │
                    │ - Input Files   │
                    │ - Output DITA   │
                    │ - Images        │
                    │ - Temp Storage  │
                    └─────────────────┘
```

## Technology Stack

### Backend Framework
- **Node.js**: Runtime environment
- **Express.js**: Web application framework
- **Nodemon**: Development server with auto-restart

### Markdown Processing
- **markdown-it**: Core markdown parser
- **markdown-it-emoji**: Emoji support
- **markdown-it-sup/sub**: Superscript/subscript
- **markdown-it-footnote**: Footnote handling
- **markdown-it-mark**: Highlighting support
- **markdown-it-deflist**: Definition lists
- **markdown-it-task-lists**: Task list support
- **markdown-it-inline-comments**: Comment handling
- **markdown-it-attrs**: Attribute support
- **markdown-it-container**: Custom containers
- **markdown-it-jsx**: JSX component support

### HTML/XML Processing
- **cheerio**: Server-side jQuery for HTML manipulation
- **html-to-json-parser**: HTML to JSON conversion
- **xml-formatter**: XML formatting and beautification
- **xmldom**: XML DOM manipulation

### Database
- **MongoDB**: Document database
- **Mongoose**: MongoDB object modeling

### Authentication & Security
- **bcrypt**: Password hashing
- **jsonwebtoken**: JWT token generation
- **cors**: Cross-origin resource sharing

### File Handling
- **express-fileupload**: File upload middleware
- **adm-zip**: ZIP file handling
- **archiver**: Archive creation
- **fs-extra**: Enhanced file system operations
- **gray-matter**: Front matter parsing

### Utilities
- **uuid**: Unique identifier generation
- **shelljs**: Shell command execution
- **useragent**: User agent parsing
- **request-ip**: IP address extraction
- **compromise**: Natural language processing

## Project Structure

```
MD_DITA_Backend/
├── .github/workflows/          # CI/CD pipeline configuration
│   └── CI-CD.yml
├── database/                   # Database connection
│   └── db.js
├── downloads/                  # Temporary download storage
├── input/                      # User input files (temporary)
├── mardownIt-custom-plugin/    # Custom markdown-it plugins
│   ├── addTopicTag__notInUse.js
│   ├── ignore-import-statements.js
│   └── ignoreCustomTags.js
├── middleware/                 # Express middleware
│   ├── generateToken.js
│   └── user.js
├── models/                     # Mongoose data models
│   ├── ditaTagModel.js
│   ├── mdTagModel.js
│   ├── userAgentDetailsModel.js
│   └── userModel.js
├── output/                     # Generated DITA files (temporary)
├── state/                      # Application state management
│   ├── allVeriables.js
│   ├── ditaMap.js
│   ├── logData.js
│   ├── tagList.js
│   └── topicData.js
├── utils/                      # Utility functions
│   ├── [30+ utility files]
├── .env                        # Environment variables
├── .gitignore
├── api.js                      # Main Express application
├── nodemon.json               # Nodemon configuration
├── package.json               # Dependencies and scripts
├── readme.md                  # Basic project information
└── schema.js                  # DITA schema definitions
```

## Core Processing Flow

### 1. File Upload & Validation
```javascript
POST /api/pre-flight-check
├── User uploads ZIP file
├── Extract ZIP to user-specific input folder
├── Validate markdown files exist
├── Check each file has required title
└── Return validation results
```

### 2. Main Processing
```javascript
POST /api/markdowntodita
├── Load DITA tag mappings from database
├── Process each markdown file:
│   ├── Parse front matter with gray-matter
│   ├── Convert markdown to HTML with markdown-it
│   ├── Add table structure (tgroup, colspec)
│   ├── Convert HTML to JSON representation
│   ├── Remove unwanted elements
│   ├── Apply character entity conversion
│   ├── Handle footnotes
│   ├── Process task lists
│   ├── Generate DITA XML structure
│   └── Write DITA file with proper DTD
├── Generate DITA map file
├── Copy images to output folder
├── Create downloadable ZIP archive
└── Return download link
```

### 3. File Download
```javascript
GET /api/download/:userId/:downloadId
├── Validate download request
├── Stream ZIP file to client
├── Clean up temporary files
└── Reset user session data
```

## Key Components

### Main Application (api.js)
The central Express application that handles:
- Route definitions
- Middleware setup
- User authentication
- File upload processing
- Error handling
- Database connections

### Processing Engine (utils/mainMethod.js)
Core processing logic that:
- Configures markdown-it with all plugins
- Processes files recursively
- Handles different file types (.md, .mdx)
- Manages concurrent processing with Promise.all
- Generates processing logs

### Schema Validation (schema.js)
Comprehensive DITA schema definitions including:
- Element hierarchies
- Attribute specifications
- Content models
- Validation rules
- Topic types (concept, task, reference)

### State Management (state/)
Manages application state across processing:
- **allVeriables.js**: Global variables and user-specific paths
- **ditaMap.js**: DITA map structure building
- **logData.js**: Processing logs and error tracking
- **tagList.js**: HTML to DITA tag mappings
- **topicData.js**: Topic hierarchy management

## Database Schema

### User Model
```javascript
{
  email: String (required, unique),
  password: String (required, hashed),
  lastLogin: Date,
  timestamps: true
}
```

### DITA Tag Model
```javascript
{
  key: String (HTML tag identifier),
  value: String (DITA tag equivalent)
}
```

### MD Tag Model
```javascript
{
  key: String (markdown element identifier),
  value: String (HTML tag equivalent)
}
```

### User Agent Details Model
```javascript
{
  userAgent: String,
  ipAddress: String,
  timestamp: Date
}
```

## API Endpoints

### Authentication
- `POST /api/register` - User registration
- `POST /api/login` - User authentication

### File Processing
- `POST /api/pre-flight-check` - Validate uploaded files
- `POST /api/markdowntodita` - Convert markdown to DITA
- `GET /api/download/:userId/:downloadId` - Download processed files

### Tag Management
- `POST /api/insertDitaTag` - Update DITA tag mappings
- `GET /api/insertDitaTag` - Retrieve DITA tag mappings

## Processing Pipeline

### Stage 1: Markdown Parsing
```javascript
// Configure markdown-it with plugins
const md = new markdownIt()
  .use(emoji)
  .use(require("markdown-it-sup"))
  .use(require("markdown-it-sub"))
  .use(require("markdown-it-footnote"))
  .use(require("markdown-it-mark"))
  .use(require("markdown-it-deflist"))
  .use(require("markdown-it-task-lists"))
  .use(require("markdown-it-inline-comments"))
  .use(require("markdown-it-attrs"))
  .use(require("markdown-it-container"), "warning")
  // ... additional containers
  .use(require("markdown-it-jsx"));
```

### Stage 2: HTML Enhancement
```javascript
// Add table structure with Cheerio
$("table").each((index, element) => {
  const numCols = $(element).find("thead th").length;
  $(element).prepend(`<tgroup cols="${numCols}">`);
  for (let i = 1; i <= numCols; i++) {
    $(element).find("tgroup").append(`<colspec colname="c${i}"/>`);
  }
});
```

### Stage 3: JSON Transformation
```javascript
// Convert HTML to JSON for processing
let result = await HTMLToJSON(contentWithHmtlAsRootElement, false);

// Remove unwanted elements
result = await removeUnwantedElements(
  result,
  {},
  "",
  userId
);
```

### Stage 4: DITA Generation
```javascript
// Convert back to HTML/XML
const modifiedDitaCode = await JSONToHTML(characterToEntity(result));

// Add DITA-specific elements
const prologContent = `
<prolog base="ai-intent">
  <author>XYZ</author>
  <critdates base="ai-intent">
    <revised base="ai-intent" modified="2024-01-11"></revised>
  </critdates>
  <metadata base="ai-intent">
    <category base="ai-intent">XYZ</category>
  </metadata>
</prolog>`;
```

## State Management

### User-Specific State
Each user session maintains isolated state:
- Input/output folder paths
- Processing logs
- Tag mappings
- Topic hierarchies
- DITA map structure

### Global State
Application-wide configurations:
- AI tags list
- Schema definitions
- Default tag mappings

### State Cleanup
Automatic cleanup after processing:
- Remove temporary files
- Reset user state
- Clear memory allocations

## Utility Functions

### File Operations
- **createDirectory.js**: Recursive directory creation
- **fileValidator.js**: File type and content validation
- **ValidDirectory.js**: Directory structure validation
- **fixImagePath.js**: Image path resolution

### Content Processing
- **characterToEntity.js**: HTML entity conversion
- **removeUnwantedElements.js**: DOM cleanup
- **taskFileMaker.js**: DITA task file generation
- **processTopicWise.js**: Topic-based processing

### Data Transformation
- **moveTitleAboveBody.js**: Structure reorganization
- **SortTopicsTags.js**: Topic hierarchy sorting
- **xRefPathFixer.js**: Cross-reference path correction

### Logging & Monitoring
- **logFileGenerator.js**: Processing log creation
- **historyTracker.js**: Operation history tracking

## Configuration

### Environment Variables
```bash
# Database
MONGODB_URI=mongodb://localhost:27017/mdtodita

# Authentication
JWT_SECRET=your-secret-key

# Server
PORT=8448
BASE=http://localhost:8448

# Node Environment
NODE_ENV=development
```

### Nodemon Configuration
```json
{
  "watch": ["*.js", "utils/*.js", "models/*.js"],
  "ext": "js,json",
  "ignore": ["node_modules/", "downloads/", "input/", "output/"],
  "delay": "1000"
}
```

## Development Setup

### Prerequisites
- Node.js (v14 or higher)
- MongoDB (v4.4 or higher)
- Git

### Installation Steps
```bash
# Clone repository
git clone <repository-url>
cd MD_DITA_Backend

# Install dependencies
npm install

# Set up environment variables
cp .env.example .env
# Edit .env with your configuration

# Start MongoDB service
mongod

# Start development server
npm start
```

### Development Workflow
1. Make code changes
2. Nodemon automatically restarts server
3. Test API endpoints
4. Check processing logs
5. Validate DITA output

## Deployment

### Production Environment
```bash
# Set production environment
NODE_ENV=production

# Use PM2 for process management
npm install -g pm2
pm2 start api.js --name "md-dita-parser"

# Set up reverse proxy (nginx)
# Configure SSL certificates
# Set up monitoring
```

### Docker Deployment
```dockerfile
FROM node:16-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 8448
CMD ["node", "api.js"]
```

## Error Handling

### Validation Errors
- File format validation
- Content structure validation
- Schema compliance checking

### Processing Errors
- Markdown parsing errors
- HTML conversion errors
- DITA generation errors

### System Errors
- Database connection errors
- File system errors
- Memory allocation errors

### Error Response Format
```json
{
  "message": "Error description",
  "status": 400,
  "error": "Detailed error information"
}
```

## Performance Considerations

### Concurrent Processing
- Uses Promise.all for parallel file processing
- Implements user-specific processing queues
- Manages memory usage during large file processing

### File System Optimization
- Temporary file cleanup
- Efficient ZIP handling
- Stream-based file operations

### Database Optimization
- Indexed user queries
- Connection pooling
- Query optimization

### Memory Management
- Garbage collection optimization
- Large file streaming
- Buffer management

## Security Features

### Authentication
- JWT token-based authentication
- Password hashing with bcrypt
- Session management

### File Security
- File type validation
- Content sanitization
- Path traversal prevention

### API Security
- CORS configuration
- Rate limiting considerations
- Input validation

### Data Protection
- User data isolation
- Temporary file cleanup
- Secure file handling

## Testing

### Unit Tests
- Utility function testing
- Schema validation testing
- Data transformation testing

### Integration Tests
- API endpoint testing
- Database integration testing
- File processing pipeline testing

### End-to-End Tests
- Complete workflow testing
- Error scenario testing
- Performance testing

### Test Data
- Sample markdown files
- Expected DITA outputs
- Error case scenarios

## Troubleshooting

### Common Issues

#### File Processing Failures
- **Symptom**: Files not converting properly
- **Causes**: Invalid markdown syntax, missing titles, unsupported elements
- **Solutions**: Check pre-flight validation, review markdown syntax, update schema mappings

#### Memory Issues
- **Symptom**: Server crashes during large file processing
- **Causes**: Insufficient memory allocation, memory leaks
- **Solutions**: Increase Node.js memory limit, implement streaming, optimize processing

#### Database Connection Issues
- **Symptom**: Authentication failures, tag mapping errors
- **Causes**: MongoDB connection problems, network issues
- **Solutions**: Check MongoDB status, verify connection string, implement retry logic

#### File System Errors
- **Symptom**: Cannot create output files, permission errors
- **Causes**: Insufficient permissions, disk space issues
- **Solutions**: Check file permissions, verify disk space, implement error handling

### Debug Mode
Enable detailed logging:
```bash
DEBUG=* npm start
```

### Log Analysis
Check processing logs in:
- Console output
- Generated log files
- Database error logs

### Performance Monitoring
Monitor:
- Memory usage
- CPU utilization
- File processing times
- Database query performance

## Maintenance

### Regular Tasks
- Clean up temporary files
- Monitor disk usage
- Update dependencies
- Review error logs

### Database Maintenance
- Index optimization
- Data cleanup
- Backup procedures
- Performance monitoring

### Security Updates
- Dependency updates
- Security patch application
- Vulnerability scanning
- Access log review

---

This documentation provides a comprehensive overview of the Markdown to DITA Parser system. For specific implementation details, refer to the individual source files and their inline documentation.