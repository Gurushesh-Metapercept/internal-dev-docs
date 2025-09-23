# DOCX to DITA Migration Backend - Developer Documentation

## Table of Contents
1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Technology Stack](#technology-stack)
4. [Project Structure](#project-structure)
5. [Core Libraries and Dependencies](#core-libraries-and-dependencies)
6. [Processing Pipeline](#processing-pipeline)
7. [API Endpoints](#api-endpoints)
8. [Database Schema](#database-schema)
9. [State Management](#state-management)
10. [Utility Functions](#utility-functions)
11. [DITA Schema Validation](#dita-schema-validation)
12. [File Processing Flow](#file-processing-flow)
13. [Error Handling](#error-handling)
14. [Security Features](#security-features)
15. [Deployment Configuration](#deployment-configuration)
16. [Development Setup](#development-setup)
17. [Testing Guidelines](#testing-guidelines)
18. [Performance Considerations](#performance-considerations)

## Project Overview

The DOCX to DITA Migration Backend is a Node.js-based web service that converts Microsoft Word documents (.docx) to DITA (Darwin Information Typing Architecture) format. The system provides a complete conversion pipeline from document upload to structured DITA output with proper topic separation, metadata handling, and hierarchical organization.

### Key Features
- **Document Conversion**: Converts DOCX files to DITA XML format
- **Topic Separation**: Automatically separates content into individual DITA topics
- **Hierarchical Structure**: Maintains document hierarchy through DITA maps
- **Image Processing**: Handles embedded images and media files
- **User Management**: Authentication and file history tracking
- **Validation**: Pre-flight checks and DITA schema validation
- **Download Management**: Packaged output with cleanup mechanisms

## System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Frontend      │    │   Backend API   │    │   Database      │
│   Client        │◄──►│   Express.js    │◄──►│   MongoDB       │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                              │
                              ▼
                    ┌─────────────────┐
                    │ Processing      │
                    │ Pipeline        │
                    └─────────────────┘
                              │
                    ┌─────────────────┐
                    │ File System     │
                    │ Management      │
                    └─────────────────┘
```

### Processing Architecture
1. **Input Layer**: File upload and validation
2. **Conversion Layer**: DOCX to HTML to JSON to DITA
3. **Processing Layer**: Content transformation and structuring
4. **Output Layer**: DITA file generation and packaging
5. **Cleanup Layer**: Temporary file management

## Technology Stack

### Core Technologies
- **Runtime**: Node.js (JavaScript)
- **Framework**: Express.js
- **Database**: MongoDB with Mongoose ODM
- **Authentication**: JWT (JSON Web Tokens)
- **File Processing**: Mammoth.js, Cheerio, xml-js

### Key Libraries by Category

#### Document Processing
- **mammoth**: DOCX to HTML conversion
- **cheerio**: Server-side HTML manipulation
- **html-to-json-parser**: HTML to JSON transformation
- **xml-js**: XML/JSON conversion utilities
- **xml-formatter**: XML formatting and beautification

#### File Management
- **express-fileupload**: File upload handling
- **adm-zip**: ZIP file creation and extraction
- **archiver**: Archive creation utilities
- **fs-extra**: Enhanced file system operations
- **shelljs**: Cross-platform shell commands

#### Image Processing
- **convert-base64-to-image**: Base64 image conversion
- **jsdom**: DOM manipulation for image processing

#### Validation and Parsing
- **fast-xml-parser**: XML parsing and validation
- **he**: HTML entity encoding/decoding
- **compromise**: Natural language processing

#### Security and Authentication
- **bcrypt**: Password hashing
- **jsonwebtoken**: JWT token management
- **cors**: Cross-origin resource sharing
- **cookie-parser**: Cookie parsing middleware

#### Utilities
- **dotenv**: Environment variable management
- **uuid**: Unique identifier generation
- **useragent**: User agent parsing
- **geoip-lite**: IP geolocation

## Project Structure

```
Docx_DITA_Backend/
├── src/
│   ├── api.js                 # Main Express server and API routes
│   ├── index.js              # Core conversion logic
│   ├── schema.js             # DITA schema definitions
│   ├── database/
│   │   └── db.js             # MongoDB connection
│   ├── middleware/
│   │   └── authentication.js # JWT authentication middleware
│   ├── models/               # Database models
│   │   ├── tag/
│   │   │   ├── ditaTagModel.js
│   │   │   └── htmlTagModel.js
│   │   ├── fileHistoryModel.js
│   │   └── userModel.js
│   ├── state/                # Application state management
│   │   ├── allVariables.js
│   │   ├── ditaMap.js
│   │   ├── logData.js
│   │   ├── tagList.js
│   │   └── topicData.js
│   └── utils/                # Utility functions (50+ files)
├── xslt/                     # XSLT transformation files
├── output/                   # Generated DITA files
├── .env                      # Environment configuration
├── package.json              # Dependencies and scripts
└── README.md                 # Project documentation
```

## Core Libraries and Dependencies

### Document Conversion Pipeline

#### 1. Mammoth.js
**Purpose**: Converts DOCX files to HTML
**Usage**: Primary conversion engine from binary DOCX to structured HTML
```javascript
const mammothOptions = {
  styleMap: [
    "p.Title => h1:fresh",
    "p.AltTitle => h2:fresh",
    "p.Quote => blockquote:fresh",
    // ... style mappings
  ],
};
```

#### 2. Cheerio
**Purpose**: Server-side HTML manipulation
**Usage**: DOM manipulation, content extraction, and HTML processing
- Table structure analysis
- Image processing
- ID and href cleaning
- Content restructuring

#### 3. html-to-json-parser
**Purpose**: Converts HTML to JSON structure
**Usage**: Intermediate format for content manipulation
- Preserves document hierarchy
- Enables programmatic content transformation

#### 4. xml-js
**Purpose**: XML/JSON conversion utilities
**Usage**: Final DITA XML generation from processed JSON

### File Management

#### 1. Express-fileupload
**Purpose**: Handles multipart file uploads
**Configuration**: Supports DOCX file validation and temporary storage

#### 2. AdmZip
**Purpose**: Creates downloadable ZIP packages
**Usage**: Packages converted DITA files with media assets

#### 3. Shelljs
**Purpose**: Cross-platform shell operations
**Usage**: Directory management, file cleanup, and system operations

## Processing Pipeline

### Step-by-Step Conversion Flow

#### 1. Pre-flight Check (`/api/pre-flight-check`)
```javascript
// Validates DOCX file structure and formatting
const result = await checkDocxStartsWithHeading(filePath);
```
- File type validation
- Document structure verification
- Style and formatting checks

#### 2. DOCX to HTML Conversion
```javascript
const { value: rawHtml } = await mammoth.convertToHtml(
  { path: filePath },
  { ...mammothOptions }
);
```
- Style mapping application
- Content extraction
- Table of contents removal

#### 3. HTML Processing and Enhancement
```javascript
const modifiedHtml = processHtml(fullHtml, OutputPath);
```
- Image extraction and conversion
- Table structure enhancement
- ID and reference cleaning

#### 4. HTML to JSON Transformation
```javascript
let result = await HTMLToJSON(contentWithHmtlAsRootElement, false);
```
- Hierarchical structure preservation
- Content normalization
- Metadata extraction

#### 5. Content Processing and Cleanup
```javascript
result = await removeUnwantedElements(result, {}, "", userId);
result = characterToEntity(result);
```
- Unwanted element removal
- Character entity conversion
- Footnote processing

#### 6. DITA Structure Generation
```javascript
const modifiedDitaCode = codeRestructure(
  await JSONToHTML(characterToEntity(safeJson))
);
```
- Topic separation
- DITA tag application
- Hierarchy establishment

#### 7. File Separation and Organization
```javascript
let topicWise = fileSeparator(cleanedOlTagSecondRound);
```
- Individual topic file creation
- Folder structure generation
- DITA map creation

#### 8. Final Processing and Packaging
- XSLT transformations
- Media file copying
- ZIP package creation

## API Endpoints

### 1. Health Check
```
GET /
Response: { message: "Docx2Dita Backend", status: "Online" }
```

### 2. Pre-flight Check
```
POST /api/pre-flight-check
Body: { userId: string, file: multipart }
Response: { message: "ok", status: 201, result: object }
```

### 3. Document Conversion
```
POST /api/convertDocxToDita
Body: { userId: string }
Response: { message: "Files converted successfully", downloadLink: string, status: 200 }
```

### 4. File Download
```
GET /api/download/:downloadId
Response: ZIP file download
```

### 5. DITA Tag Management
```
POST /api/insertDitaTag - Insert/update DITA tags
GET /api/insertDitaTag - Retrieve DITA tags
```

### 6. User Management
```
POST /api/register - User registration
POST /api/login - User authentication
POST /api/fileHistory - File processing history
```

## Database Schema

### User Model
```javascript
{
  email: String,
  password: String (hashed),
  lastLogin: Date,
  timestamps: true
}
```

### File History Model
```javascript
{
  fileName: String,
  userId: ObjectId,
  email: String,
  os: String,
  ip: String,
  location: { country: String, city: String },
  browser: String,
  timestamps: true
}
```

### DITA Tag Model
```javascript
{
  key: String,
  value: String,
  timestamps: true
}
```

## State Management

### Global State Variables (`src/state/`)

#### 1. allVariables.js
- Input/output folder management
- User session data
- File name tracking

#### 2. ditaMap.js
- DITA map structure storage
- Topic hierarchy management

#### 3. logData.js
- Processing logs
- Error tracking

#### 4. tagList.js
- AI-generated tag lists
- Custom tag management

#### 5. topicData.js
- Topic metadata
- Content structure information

## Utility Functions

### Core Utilities (`src/utils/`)

#### File Processing
- `fileSeperator.js`: Topic extraction and separation
- `ditaMapMaker.js`: DITA map generation
- `processTopics.js`: Topic content processing

#### Content Transformation
- `characterToEntity.js`: Character encoding
- `removeUnwantedElements.js`: Content cleanup
- `codeRestructure.js`: DITA structure application

#### Image and Media
- `addIdtoFigTag.js`: Figure element processing
- `fixImagePath.js`: Image path resolution

#### Validation
- `fileValidator.js`: File format validation
- `tagValidator.js`: DITA tag validation
- `checkDocxStartsWithHeading.js`: Document structure validation

#### Helper Functions
- `helper.js`: Common utilities
- `generateRandomId.js`: Unique ID generation
- `createDirectory.js`: Directory management

## DITA Schema Validation

### Schema Structure (`src/schema.js`)
The system includes comprehensive DITA schema definitions covering:

#### Topic Types
- **topic**: Base topic structure
- **concept**: Conceptual information
- **task**: Procedural information
- **reference**: Reference material

#### Content Elements
- **title**: Topic titles with inline elements
- **body**: Main content container
- **section**: Content sections
- **p**: Paragraphs with mixed content
- **ol/ul**: Ordered and unordered lists
- **table**: Table structures
- **fig**: Figures and images

#### Attributes
- Standard DITA attributes (id, conref, props, etc.)
- Localization attributes (xml:lang, dir)
- Linking attributes (href, scope, format)

## File Processing Flow

### Input Processing
1. **File Upload**: Multipart form handling
2. **Validation**: MIME type and structure checks
3. **Storage**: Temporary file system storage
4. **Pre-processing**: Document analysis

### Conversion Processing
1. **HTML Generation**: Mammoth conversion
2. **Content Enhancement**: Image and table processing
3. **Structure Analysis**: Hierarchy detection
4. **JSON Transformation**: Intermediate format creation

### DITA Generation
1. **Topic Separation**: Content splitting by headings
2. **DITA Tagging**: Semantic markup application
3. **Map Creation**: Hierarchical navigation structure
4. **File Organization**: Folder structure creation

### Output Processing
1. **File Writing**: Individual DITA topic files
2. **Media Copying**: Image and asset management
3. **Package Creation**: ZIP archive generation
4. **Cleanup**: Temporary file removal

## Error Handling

### Error Categories
1. **Validation Errors**: File format, structure issues
2. **Processing Errors**: Conversion failures
3. **System Errors**: File system, database issues
4. **Authentication Errors**: JWT, user validation

### Error Response Format
```javascript
{
  message: "Error description",
  status: HTTP_STATUS_CODE,
  error: "Detailed error information" // Optional
}
```

### Cleanup Mechanisms
- Automatic temporary file cleanup
- User session cleanup
- Failed conversion cleanup
- Memory management

## Security Features

### Authentication
- JWT-based authentication
- Password hashing with bcrypt
- Session management

### File Security
- MIME type validation
- File size limits
- Temporary file isolation
- Automatic cleanup

### API Security
- CORS configuration
- Input validation
- Error message sanitization
- Rate limiting considerations

## Deployment Configuration

### Environment Variables
```
PORT=8445
BASE_URL=https://docxapi.met-r.io
MONGODB_URI=mongodb+srv://...
JWT_SECRET=...
```

### Production Considerations
- MongoDB Atlas connection
- File system permissions
- Process management
- Logging configuration
- Error monitoring

## Development Setup

### Prerequisites
- Node.js (v14+)
- MongoDB (local or Atlas)
- Git

### Installation
```bash
git clone <repository>
cd Docx_DITA_Backend
npm install
```

### Configuration
1. Copy `.env.example` to `.env`
2. Configure MongoDB connection
3. Set JWT secret
4. Configure base URL

### Running
```bash
npm start  # Production
npm run dev  # Development with watch
```

## Testing Guidelines

### Test Categories
1. **Unit Tests**: Individual utility functions
2. **Integration Tests**: API endpoint testing
3. **Conversion Tests**: End-to-end document processing
4. **Performance Tests**: Large file handling

### Test Data
- Sample DOCX files with various structures
- Edge cases (empty documents, complex formatting)
- Invalid file formats
- Large documents

## Performance Considerations

### Optimization Strategies
1. **Memory Management**: Stream processing for large files
2. **Concurrent Processing**: Multiple user handling
3. **Caching**: Processed content caching
4. **Cleanup**: Aggressive temporary file cleanup

### Monitoring Points
- Memory usage during conversion
- File system space utilization
- Database connection pooling
- Response times for different file sizes

### Scalability Considerations
- Horizontal scaling with load balancers
- Database sharding for user data
- CDN for static assets
- Queue-based processing for large files

---

## Conclusion

This DOCX to DITA Migration Backend provides a comprehensive solution for document conversion with robust error handling, security features, and scalable architecture. The modular design allows for easy maintenance and feature extensions while maintaining high performance and reliability.

For additional information or support, refer to the individual utility documentation or contact the development team.