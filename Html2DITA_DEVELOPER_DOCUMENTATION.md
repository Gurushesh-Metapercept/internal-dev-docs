# HTML to DITA Conversion System - Developer Documentation

## Table of Contents

1. [System Overview](#system-overview)
2. [Architecture](#architecture)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [Core Components](#core-components)
6. [Data Flow](#data-flow)
7. [API Endpoints](#api-endpoints)
8. [Processing Pipeline](#processing-pipeline)
9. [State Management](#state-management)
10. [Database Schema](#database-schema)
11. [Utility Functions](#utility-functions)
12. [Configuration](#configuration)
13. [Deployment](#deployment)
14. [Development Setup](#development-setup)
15. [Testing](#testing)
16. [Troubleshooting](#troubleshooting)

## System Overview

The HTML to DITA Conversion System is a Node.js-based web service that converts HTML documents to DITA (Darwin Information Typing Architecture) format. The system processes uploaded HTML files, transforms them into structured DITA topics, and provides downloadable ZIP archives containing the converted content.

### Key Features

- Multi-user support with authentication
- Batch HTML file processing
- Automatic DITA topic generation
- Hierarchical content structure preservation
- Image asset management
- Comprehensive logging and error handling
- RESTful API interface

## Architecture

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Client App    │───▶│   Express API   │───▶│   MongoDB       │
│   (Frontend)    │    │   Server        │    │   Database      │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                       ┌─────────────────┐
                       │  File System    │
                       │  (Temp Storage) │
                       └─────────────────┘
```

### System Components

1. **API Layer**: Express.js server handling HTTP requests
2. **Authentication**: JWT-based user authentication
3. **File Processing Engine**: HTML to DITA conversion pipeline
4. **State Management**: User session and processing state tracking
5. **Database Layer**: MongoDB for user data and tag mappings
6. **File System**: Temporary storage for input/output files

## Technology Stack

### Core Technologies

- **Runtime**: Node.js (v14+)
- **Framework**: Express.js (v4.19.2)
- **Database**: MongoDB with Mongoose ODM (v8.6.1)
- **Authentication**: JWT (jsonwebtoken v9.0.2) + bcrypt (v5.1.1)

### Key Libraries

- **HTML Processing**:
  - `cheerio` (v1.0.0-rc.12) - Server-side jQuery implementation
  - `html-to-json-parser` (v2.0.1) - HTML to JSON conversion
  - `js-beautify` (v1.15.1) - HTML formatting
- **XML/DITA Processing**:
  - `xml-formatter` (v3.6.2) - XML formatting
  - `xmldom` (v0.6.0) - XML DOM manipulation
- **File Operations**:
  - `adm-zip` (v0.5.12) - ZIP file handling
  - `archiver` (v7.0.1) - Archive creation
  - `express-fileupload` (v1.5.0) - File upload handling
  - `shelljs` (v0.8.5) - Shell command execution
- **Utilities**:
  - `uuid` (v9.0.1) - Unique ID generation
  - `compromise` (v14.14.2) - Natural language processing
  - `cors` (v2.8.5) - Cross-origin resource sharing

## Project Structure

```
HTML_DITA_Backend/
├── .github/
│   └── workflows/
│       └── CI-CD.yml              # GitHub Actions CI/CD pipeline
├── database/
│   ├── db.js                      # MongoDB connection
│   └── getAllTags.js              # Tag retrieval utilities
├── downloads/                     # Temporary download files
├── input/                         # User input files (temporary)
├── middleware/
│   ├── generateToken.js           # JWT token generation
│   └── user.js                    # User authentication middleware
├── models/
│   ├── ditaTagModel.js           # DITA tag database schema
│   ├── htmlTagModel.js           # HTML tag database schema
│   └── userModel.js              # User database schema
├── output/                        # Processed output files (temporary)
├── state/
│   ├── allVeriables.js           # Global state management
│   ├── ditaMap.js                # DITA map state
│   ├── logData.js                # Logging state
│   ├── rootDita.js               # Root DITA structure
│   ├── tagList.js                # Tag mapping state
│   └── topicData.js              # Topic processing state
├── utils/                         # Utility functions (40+ files)
├── api.js                         # Main Express server
├── schema.js                      # DITA schema definitions
├── package.json                   # Dependencies and scripts
└── nodemon.json                   # Development configuration
```

## Core Components

### 1. Express Server (api.js)

The main application server that handles:

- HTTP request routing
- File upload processing
- User authentication
- Response formatting
- Error handling

### 2. State Management System

Multi-user state management with isolated user sessions:

- **allVeriables.js**: User-specific I/O paths and global settings
- **ditaMap.js**: DITA map structure for each user
- **topicData.js**: Topic processing state
- **logData.js**: Processing logs and error tracking
- **tagList.js**: HTML to DITA tag mappings

### 3. Processing Pipeline

Sequential processing stages:

1. **File Validation**: Check file types and structure
2. **HTML Parsing**: Extract and clean HTML content
3. **Tag Conversion**: Map HTML tags to DITA equivalents
4. **Structure Analysis**: Identify topics and hierarchies
5. **DITA Generation**: Create DITA files and maps
6. **Asset Management**: Handle images and resources
7. **Archive Creation**: Package output files

### 4. Database Models

- **User Model**: Authentication and user management
- **DITA Tag Model**: HTML to DITA tag mappings
- **HTML Tag Model**: HTML tag definitions

## Data Flow

### 1. Pre-flight Check Flow

```
Client Upload → File Validation → HTML Structure Check → Response
```

### 2. Main Conversion Flow

```
Authenticated Request → File Processing → HTML Parsing →
Tag Conversion → Topic Generation → DITA Map Creation →
Asset Copying → Archive Creation → Download Link
```

## API Endpoints

### File Processing Endpoints

- `POST /api/pre-flight-check` - Validate uploaded files
- `POST /api/htmltodita` - Convert HTML to DITA
- `GET /api/download/:userId/:downloadId` - Download converted files

### Configuration Endpoints

- `POST /api/insertDitaTag` - Update DITA tag mappings
- `GET /api/insertDitaTag` - Retrieve DITA tag mappings

### Health Check

- `GET /` - Server status check

## Processing Pipeline

### Stage 1: File Upload and Validation

1. **File Reception**: Express-fileupload middleware handles ZIP uploads
2. **User Directory Creation**: Create user-specific input/output folders
3. **ZIP Extraction**: Extract uploaded ZIP to user input directory
4. **File Validation**: Check for HTML files and validate structure
5. **Pre-flight Check**: Validate HTML structure and title presence

### Stage 2: HTML Processing

1. **File Discovery**: Recursively scan input directory for HTML files
2. **HTML Beautification**: Format HTML using js-beautify
3. **DOM Loading**: Parse HTML using Cheerio
4. **Table Processing**: Handle table structures and column groups
5. **Title Deduplication**: Remove duplicate titles

### Stage 3: Content Transformation

1. **HTML Extraction**: Extract content from HTML root element
2. **Tag Insertion**: Apply DITA tag mappings
3. **JSON Conversion**: Convert HTML to JSON structure
4. **Element Removal**: Remove unwanted HTML elements
5. **Character Entity Conversion**: Convert special characters
6. **Content Restructuring**: Organize content for DITA format

### Stage 4: DITA Generation

1. **Topic Separation**: Split content into DITA topics
2. **Hierarchy Analysis**: Determine topic relationships
3. **Folder Structure**: Create organized folder structure
4. **DITA File Creation**: Generate individual DITA topic files
5. **DITA Map Generation**: Create master DITA map file

### Stage 5: Post-processing

1. **Cross-reference Fixing**: Update internal links
2. **Output Class Addition**: Add DITA-specific classes
3. **Child Map Processing**: Handle nested DITA maps
4. **Attribute Cleanup**: Remove unnecessary attributes
5. **Asset Copying**: Copy images and resources

### Stage 6: Archive and Delivery

1. **ZIP Creation**: Package all output files
2. **Download Link Generation**: Create secure download URL
3. **Cleanup Scheduling**: Schedule temporary file removal
4. **Response Delivery**: Send download link to client

## State Management

### User-Based State Isolation

Each user session maintains isolated state:

```javascript
// User-specific paths
USER_BASED_IO_PATH[userId] = {
  input_dir: "input/userId/",
  output_dir: "output/userId/",
};

// User-specific processing data
TOPIC_DATA[userId] = { topics: [], metadata: {} };
DITA_MAP_DATA[userId] = { structure: {}, files: [] };
LOG_DATA[userId] = { errors: [], warnings: [], skipped: [] };
```

### State Lifecycle

1. **Initialization**: Create user state on first request
2. **Processing**: Update state during conversion
3. **Persistence**: Maintain state across processing stages
4. **Cleanup**: Reset state after completion

## Database Schema

### User Schema

```javascript
{
  email: String (required, unique),
  password: String (required, hashed),
  lastLogin: Date,
  timestamps: true
}
```

### DITA Tag Schema

```javascript
{
  key: String (required),      // HTML tag name
  value: String (required),    // DITA tag name
  timestamps: true
}
```

## Utility Functions

### Core Utilities (utils/ directory)

- **addTagsToDatabase.js**: Database tag management
- **addTopicTag.js**: Topic tag insertion
- **characterToEntity.js**: Character entity conversion
- **childDitamap.js**: Child DITA map processing
- **colFixerWithoutHeadTag.js**: Table column fixing
- **colMergeFixer.js**: Column merge handling
- **createDirectory.js**: Directory creation utilities
- **deduplicateTitles.js**: Title deduplication
- **dtdConcept.js**: Concept DTD handling
- **dtdReference.js**: Reference DTD handling
- **dtdTask.js**: Task DTD handling
- **extractHTML.js**: HTML content extraction
- **fileSeparator.js**: File separation logic
- **fileValidator.js**: File validation
- **FinalTagsCleanUp.js**: Final tag cleanup
- **fixImagePath.js**: Image path correction
- **generateRandomId.js**: Random ID generation
- **helper.js**: General helper functions
- **insertTags.js**: Tag insertion logic
- **logFileGenerator.js**: Log file creation
- **mainMethod.js**: Main processing orchestration
- **moveTgroupClosingTagBeforeTable.js**: Table group handling
- **outputClassAdder.js**: Output class addition
- **preFlightChecker.js**: Pre-flight validation
- **processAndWriteData.js**: Data processing and writing
- **processTableElements.js**: Table element processing
- **processTopicWise.js**: Topic-wise processing
- **removeOutputclassAttributes.js**: Attribute removal
- **removeUnwantedElements.js**: Element cleanup
- **replaceSpecialCharactersWithEntities.js**: Character replacement
- **rootDitaMaker.js**: Root DITA creation
- **SortTopicsTags.js**: Topic tag sorting
- **subStepsMover.js**: Sub-step handling
- **tagCheckerInDB.js**: Database tag checking
- **tagsValidator.js**: Tag validation
- **taskListFixerJson.js**: Task list fixing
- **taskStepHandler.js**: Task step processing
- **validateURL.js**: URL validation
- **ValidDirectory.js**: Directory validation
- **xrefDataAttribute.js**: Cross-reference attributes
- **xRefPathFixer.js**: Cross-reference path fixing

## Configuration

### Environment Variables

```bash
PORT=8449                    # Server port
BASE=http://localhost:8449   # Base URL
MONGODB_URI=mongodb://...    # MongoDB connection string
JWT_SECRET=your_secret_key   # JWT signing secret
```

### DITA Schema Configuration

The system includes comprehensive DITA schema definitions in `schema.js` covering:

- Topic types (concept, task, reference)
- Element hierarchies and relationships
- Attribute specifications
- Content models

### AI Tags List

Predefined list of DITA elements that receive AI-intent attributes:

- Body elements (body, bodydiv, conbody, etc.)
- Structural elements (section, div, topic, etc.)
- Content elements (p, ul, ol, table, etc.)
- Specialized elements (step, cmd, note, etc.)

## Deployment

### Production Deployment

1. **Environment Setup**: Configure production environment variables
2. **Database Setup**: Set up MongoDB Atlas or self-hosted MongoDB
3. **File System**: Ensure adequate disk space for temporary files
4. **Process Management**: Use PM2 or similar for process management
5. **Reverse Proxy**: Configure Nginx or Apache for production
6. **SSL/TLS**: Implement HTTPS for secure communication

### CI/CD Pipeline

GitHub Actions workflow (`.github/workflows/CI-CD.yml`) handles:

- Automated testing
- Dependency installation
- Build process
- Deployment to staging/production

## Development Setup

### Prerequisites

- Node.js (v14 or higher)
- MongoDB (local or Atlas)
- Git

### Installation Steps

1. **Clone Repository**:

   ```bash
   git clone <repository-url>
   cd HTML_DITA_Backend
   ```

2. **Install Dependencies**:

   ```bash
   npm install
   ```

3. **Environment Configuration**:

   ```bash
   cp .env.example .env
   # Edit .env with your configuration
   ```

4. **Database Setup**:

   ```bash
   # Start MongoDB locally or configure Atlas connection
   ```

5. **Start Development Server**:
   ```bash
   npm start
   # Server starts on http://localhost:8449
   ```

### Development Commands

- `npm start` - Start development server with nodemon
- `npm test` - Run test suite (if implemented)
- `npm run lint` - Code linting (if configured)

## Testing

### Test Categories

1. **Unit Tests**: Individual function testing
2. **Integration Tests**: API endpoint testing
3. **End-to-End Tests**: Complete workflow testing
4. **Performance Tests**: Load and stress testing

### Test Data

- Sample HTML files for conversion testing
- Expected DITA output for validation
- Edge cases and error scenarios

## Troubleshooting

### Common Issues

#### 1. File Upload Failures

- **Cause**: File size limits, invalid ZIP format
- **Solution**: Check file size limits, validate ZIP structure

#### 2. Conversion Errors

- **Cause**: Invalid HTML structure, missing titles
- **Solution**: Validate HTML before upload, ensure proper structure

#### 3. Memory Issues

- **Cause**: Large file processing, memory leaks
- **Solution**: Implement streaming, optimize memory usage

#### 4. Database Connection Issues

- **Cause**: Network issues, authentication problems
- **Solution**: Check MongoDB connection string, network connectivity

#### 5. Authentication Failures

- **Cause**: Invalid JWT tokens, expired sessions
- **Solution**: Implement token refresh, proper session management

### Debugging Tools

- **Logging**: Comprehensive logging throughout the pipeline
- **Error Tracking**: Detailed error messages and stack traces
- **State Inspection**: User state debugging utilities
- **File System Monitoring**: Track temporary file creation/deletion

### Performance Optimization

- **Memory Management**: Proper cleanup of temporary data
- **File System**: Efficient file operations and cleanup
- **Database**: Optimized queries and indexing
- **Concurrency**: Handle multiple user sessions efficiently

## Security Considerations

### Authentication Security

- Password hashing with bcrypt
- JWT token expiration
- Secure token storage

### File Security

- File type validation
- Size limits
- Temporary file cleanup
- Path traversal prevention

### API Security

- CORS configuration
- Rate limiting (recommended)
- Input validation
- Error message sanitization

## Monitoring and Maintenance

### Health Monitoring

- Server uptime monitoring
- Database connection health
- File system space monitoring
- Memory usage tracking

### Maintenance Tasks

- Regular cleanup of temporary files
- Database optimization
- Log rotation
- Security updates

### Backup Strategy

- Database backups
- Configuration backups
- Code repository backups

## Future Enhancements

### Planned Features

- Real-time processing status updates
- Batch processing improvements
- Enhanced error recovery
- Performance optimizations
- Additional DITA topic types
- Custom tag mapping interface

### Scalability Considerations

- Horizontal scaling support
- Load balancing
- Distributed file storage
- Microservices architecture

---

This documentation provides a comprehensive overview of the HTML to DITA Conversion System. For specific implementation details, refer to the individual source files and their inline documentation.
