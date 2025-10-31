# Flux S3 Portal

A static, serverless S3 file browser and documentation portal that runs entirely in the browser with no backend required.

## Overview

Flux S3 Portal provides a secure, user-friendly interface for browsing, viewing, and sharing S3 bucket contents. Perfect for client portals, documentation sites, and file sharing with non-technical users.

### 🎉 NEW Features!
- **Multi-Item Download**: Download any combination of files and folders as ZIP files directly from your browser! Select multiple items with checkboxes, see folder sizes calculated automatically (with smart caching), and download everything in a single ZIP file with intelligent memory management and real-time progress tracking.
- **Smart Filter**: When browsing folders with many items (>20), a filter bar automatically appears to help you quickly find files and folders by name with real-time results.
- **View Toggle**: Seamlessly switch between file browser and documentation viewer modes to read HTML docs and download files without losing your place.

## Key Features

### ✅ **Implemented**

#### 🌐 Static & Serverless Architecture
- Single HTML file with embedded CSS/JavaScript
- No backend server required
- Direct browser-to-S3 communication using AWS SDK v2
- All dependencies loaded from CDN (AWS SDK, Bootstrap, CryptoJS)

#### 📁 S3 File Browser
- Browse folders and files with intuitive navigation
- Breadcrumb navigation for current location
- File type icons for visual identification
- File size display with human-readable formatting
- Back navigation support
- **Smart filter**: Auto-appears when >20 items, real-time filtering by name
- Filter count display: "Showing X of Y" results

#### 📄 HTML Documentation Viewer
- Render HTML files directly in the portal
- Automatic rewriting of relative links and images
- Seamless navigation between documentation pages
- Auto-loads `docs/index.html` if present
- Toggle between file browser and documentation viewer

#### ⬇️ Download Functionality
- Download individual files with AWS Signature v4 presigned URLs
- Proper content-disposition headers for correct filenames
- Download button for each file
- Share file links with others

#### ⬆️ Upload Functionality
- Drag & drop file upload
- Browse to select files
- Multi-file upload support
- Real-time progress tracking
- **Security**: Uploads restricted to `uploads/` folder only

#### 🔗 Share & Link Generation
- Share entire portal with embedded credentials
- Share specific files with presigned URLs
- Share documentation pages with direct links
- One-click copy to clipboard

#### 🔒 Security Features
- **Encrypted credential tokens**: Share URLs use AES-encrypted tokens instead of plaintext credentials
- **Cookie-based credential storage**: Credentials saved as encrypted cookies (30-day expiration) after initial connection
- **Automatic URL cleanup**: Credentials removed from browser address bar after page load
- **Expiring share links**: All share tokens expire after 30 days
- Credentials stored client-side only (not on any server)
- Read-only access pattern with controlled upload location
- **No delete functionality** - prevents accidental/malicious deletion
- CORS-enabled for secure browser access
- Sandboxed uploads to dedicated folder

#### 🎨 Modern UI/UX
- Bootstrap 5 responsive design
- Mobile-friendly interface
- Connection status ribbon
- Real-time status messages
- Loading states and error handling
- Modal-based configuration

#### 📦 Multi-Item Download with ZIP Compression
- Select any combination of files and folders via checkboxes
- Download individual files or entire folder contents as ZIP file
- **Smart size-based batching**: Downloads files in 50MB batches to manage memory efficiently
- **Automatic folder size calculation with caching**:
  - Displays total size for each folder (e.g., "docs/ (450 MB)")
  - Sizes calculated once and cached for performance
  - Cache cleared on refresh or browser reload
- **Two-phase download process**:
  - Phase 1: Analyze - Recursively scan folders and fetch file metadata, calculate total size
  - Phase 2: Download & ZIP - Download files in batches, compress, and save
- **Progress tracking**: Real-time progress by size and file count
- **Size limits and warnings**:
  - Blocks downloads >2GB (browser memory limit)
  - Warns for downloads >500MB (significant time/memory)
  - Pre-download confirmation showing breakdown (e.g., "2 folders + 3 files"), total files and size
- **Memory efficient**: Can handle multi-GB downloads through batching
- **Cancellable**: Cancel button during download process
- ZIP files named intelligently:
  - Single file: `filename-2025-10-31.zip`
  - Multiple files: `files-2025-10-31.zip`
  - Folders: `foldername-2025-10-31.zip`
- Uses JSZip library with STORE compression for speed

## High Priority Objectives

### ✅ All High Priority Objectives Completed!

The multi-item download feature (files and folders) with size-based batching, automatic folder size calculation with caching, and seamless file/folder selection has been successfully implemented and tested.

## Low Priority Objectives

### 📊 Future Enhancements

#### JWT-Style Signed Tokens
- **Status**: Not implemented
- **Priority**: Low
- **Description**: Replace AES encryption with JWT-style cryptographically signed tokens
- **Current Behavior**: Uses AES encryption with a static key embedded in the code
- **Benefits**:
  - Stronger security through asymmetric cryptography
  - Token tampering detection via signature verification
  - Industry-standard token format
  - Better key management options
- **Implementation**: Use Web Crypto API or jose library for JWT creation/verification
- **Reference**: `docs/index.html:845-877` (current encryption implementation)

#### PDF Viewer
- **Status**: Not implemented
- **Priority**: Low
- **Description**: Inline PDF rendering using PDF.js
- **Current Behavior**: PDFs show download button only
- **Implementation**: Add PDF.js library and viewer component
- **Reference**: `docs/index.html:1080` (PDF icon mapping)

#### Markdown Viewer
- **Status**: Not implemented
- **Priority**: Low
- **Description**: Inline Markdown rendering with syntax highlighting
- **Current Behavior**: `.md` files show download button only
- **Implementation**: Add marked.js parser and optional highlight.js for code blocks
- **Use Case**: View README.md, documentation files, etc. without downloading

#### Enhanced File Type Support
- **Status**: Partial
- **Priority**: Low
- **Description**: Preview more file types (images, videos, text files, code)
- **Current Behavior**: Icons shown, but no preview
- **Potential**: Modal viewers for common file types

## Setup Instructions

### Prerequisites
- AWS S3 bucket
- AWS IAM user with S3 access (read, and write to specific prefix)
- Bucket CORS configuration (see below)

### 1. Configure S3 Bucket CORS

Your bucket must have CORS enabled for browser-based access:

```json
{
  "CORSRules": [
    {
      "AllowedHeaders": ["*"],
      "AllowedMethods": ["GET", "HEAD", "PUT", "POST"],
      "AllowedOrigins": ["*"],
      "ExposeHeaders": [
        "ETag",
        "x-amz-server-side-encryption",
        "x-amz-request-id",
        "x-amz-id-2"
      ],
      "MaxAgeSeconds": 3000
    }
  ]
}
```

Apply CORS configuration:
```bash
aws s3api put-bucket-cors \
  --bucket YOUR_BUCKET_NAME \
  --cors-configuration file://s3-cors-config.json
```

Verify CORS configuration:
```bash
aws s3api get-bucket-cors --bucket YOUR_BUCKET_NAME
```

### 2. Configure IAM Permissions

Create an IAM user with appropriate S3 permissions:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket",
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME",
        "arn:aws:s3:::YOUR_BUCKET_NAME/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject"
      ],
      "Resource": [
        "arn:aws:s3:::YOUR_BUCKET_NAME/uploads/*"
      ]
    }
  ]
}
```

**Note**: This policy allows read access to the entire bucket but restricts uploads to the `uploads/` folder only.

### 3. Deploy the Portal

Simply copy `docs/index.html` to your web server or S3 bucket:

```bash
# Option 1: Copy to your web server
cp docs/index.html /var/www/html/s3-portal.html

# Option 2: Upload to S3 and enable static website hosting
aws s3 cp docs/index.html s3://YOUR_BUCKET_NAME/index.html --acl public-read
aws s3 website s3://YOUR_BUCKET_NAME/ --index-document index.html

# Option 3: Open directly from file system (file://)
open docs/index.html
```

### 4. Access the Portal

**Method 1: Manual Configuration**
1. Open `index.html` in your browser
2. Click "Configure" button
3. Enter your AWS credentials:
   - S3 Bucket Name
   - AWS Access Key ID
   - AWS Secret Access Key
   - AWS Region
4. Click "Connect"

**Method 2: Share URL with Encrypted Token (Recommended)**
Use the built-in "Share" button to generate a secure share URL with encrypted credentials:
```
https://your-domain.com/index.html?token=ENCRYPTED_TOKEN
```

Or with a specific path:
```
https://your-domain.com/index.html?token=ENCRYPTED_TOKEN&path=docs/index.html
```

**How it works**:
- Credentials are AES-encrypted into a single token parameter
- Token expires after 30 days
- After page loads, credentials are stored in encrypted cookies
- URL parameters are automatically cleaned from the address bar
- Users can bookmark the clean URL and rely on cookies for subsequent visits

**Legacy Format (Backward Compatible)**:
```
https://your-domain.com/index.html?bucket=BUCKET&key=ACCESS_KEY&secret=SECRET_KEY&region=REGION
```
This format still works but is less secure as credentials are visible in plaintext in the URL.

**Security Notes**:
- Encrypted tokens are much more secure than plaintext credentials
- Tokens expire automatically after 30 days
- Share URLs only with trusted users
- Consider using temporary credentials or IAM roles with limited permissions
- Credentials are never sent to any server except AWS S3

## Usage

### Browsing Files
- Click folders to navigate into them
- Click ".. (Back)" to go up one level
- View breadcrumb trail at top for current location
- Click "Refresh" to reload current folder
- **Filtering** (auto-appears when >20 items):
  - Type in the filter field to search files/folders by name
  - Real-time filtering as you type (case-insensitive, contains match)
  - See "Showing X of Y" count update dynamically
  - Click "Clear" button or delete text to show all items
  - Filter automatically hides when viewing folders with ≤20 items

### Viewing Documentation
- Click any `.html` file to view it inline
- Relative links and images are automatically resolved
- **Toggle between views**:
  - When viewing documentation: Button shows "📁 File View" - click to return to file browser
  - When in file browser: Button shows "📄 Doc View" - click to return to previously viewed documentation
- Click "Share Page" to copy documentation link
- **Use case**: View HTML documentation, switch to file browser to download files, then return to documentation without losing your place

### Downloading Files
- Click "Download" button next to any file
- File will download with original filename
- Uses AWS presigned URLs (valid for 5 minutes)

### Downloading Multiple Items (Files and Folders)
- Check the box next to any combination of files and/or folders
- Folder sizes are automatically calculated and displayed (e.g., "docs/ (450 MB)")
- File sizes are shown inline (e.g., "report.pdf (2.3 MB)")
- Click "Download (X)" button in the top ribbon showing the count of selected items
- Review the confirmation showing:
  - Breakdown of selection (e.g., "2 folders + 3 files")
  - Total files to be downloaded
  - Total size
- Wait for the download process (progress shown in real-time)
- All selected items packaged into a single ZIP file
- ZIP filename format:
  - Single file: `filename-2025-10-31.zip`
  - Multiple files: `files-2025-10-31.zip`
  - Folders: `foldername-2025-10-31.zip`

### Uploading Files
- Click "Upload Files" button
- Drag & drop files or click "Browse Files"
- Files automatically uploaded to `uploads/` folder
- Progress bar shows upload status
- Click "Exit Upload" to return to file browser

### Sharing
- **Share Portal**: Click "Share" button to copy portal URL with credentials
- **Share File**: Click "Share" next to any file to copy presigned URL
- **Share Page**: Click "Share Page" while viewing documentation

## Architecture

### Technology Stack
- **Frontend**: Vanilla JavaScript, Bootstrap 5, Bootstrap Icons
- **AWS Integration**: AWS SDK for JavaScript v2
- **Cryptography**: CryptoJS for AWS Signature v4
- **File Browser**: Custom implementation with recursive navigation
- **HTML Rendering**: DOMParser for documentation processing

### Key Components

#### S3 Client (`docs/index.html:819-881`)
- Initializes AWS SDK with user credentials
- Creates S3 service object
- Manages connection state
- Tests connection on initialization

#### File Listing (`docs/index.html:940-994`)
- Lists objects using `listObjectsV2` with delimiter
- Separates folders (CommonPrefixes) from files (Contents)
- Filters out folder markers (0-byte objects ending with `/`)
- Displays results with icons and metadata

#### File Filter (`docs/index.html:1114-1200`)
- Smart auto-show filter for listings with >20 items
- Real-time filtering as user types (case-insensitive contains search)
- Separate rendering logic for efficient re-filtering
- Count display showing filtered vs total items
- Preserves folder selection and checkbox state during filtering

#### Documentation Renderer (`docs/index.html:1104-1127`)
- Fetches HTML content from S3
- Rewrites relative links using `rewriteRelativeLinks()`
- Generates presigned URLs for images
- Intercepts HTML link clicks for in-portal navigation

#### Download Handler (`docs/index.html:1243-1279`)
- Generates AWS Signature v4 presigned URLs
- Sets proper `Content-Disposition` headers
- Triggers browser download

#### Upload Handler (`docs/index.html:1429-1511`)
- Handles drag-and-drop and file browse
- Shows upload progress with progress bar
- Uses S3 `upload()` method with multipart support
- Prefixes all uploads with `uploads/`

#### Multi-Item Download Handler (`docs/index.html:1659-1923`)
- Calculates and displays folder sizes asynchronously with caching
- Supports selecting any combination of files and folders
- Two-phase download process: analyze then download
- Size-based batching for memory efficiency (50MB batches)
- Recursive folder listing with S3 pagination support
- Direct file metadata fetching for individual files
- Real-time progress tracking by bytes and file count
- JSZip integration with STORE compression
- Size limit enforcement (warns >500MB, blocks >2GB)
- Cancellation support during download

### Security Model

**Encrypted Tokens**: Share URLs use AES-encrypted tokens (30-day expiration) instead of plaintext credentials. Tokens are automatically generated when sharing and include all necessary credentials in a single encrypted parameter.

**Cookie-Based Persistence**: After successful connection, credentials are stored as encrypted cookies with 30-day expiration. This allows users to bookmark clean URLs without credentials in the address bar.

**Automatic URL Cleanup**: When loading from a share URL (token or legacy format), credentials are automatically removed from the browser address bar using `history.replaceState()` after the page loads, preventing exposure in browser history.

**Client-Side Credentials**: All AWS credentials are stored only in the browser as encrypted cookies (never sent to any server except AWS S3). The encryption key is embedded in the application code.

**Read-Mostly Access**: Portal designed for reading files with controlled upload capability.

**No Delete**: Intentionally no delete functionality to prevent accidental data loss.

**Sandboxed Uploads**: All uploads restricted to `uploads/` folder via hard-coded prefix.

**CORS Protection**: Bucket must explicitly allow browser access via CORS configuration.

**Backward Compatibility**: Legacy plaintext URL parameters (`?bucket=...&key=...&secret=...&region=...`) are still supported for existing share links, but new shares use encrypted tokens.

## Project Structure

```
FluxS3Portal/
├── docs/
│   └── index.html          # Main portal application (single file)
├── s3-cors-config.json     # CORS configuration for S3 bucket
└── README.md               # This file
```

## Troubleshooting

### "Network failure" Error
**Problem**: Clicking Refresh shows "Failed to load files: Network failure"

**Solution**: Configure CORS on your S3 bucket (see Setup Instructions)

### "S3 client not initialized"
**Problem**: Portal shows this error when trying to perform actions

**Solution**: Click "Configure" and enter valid AWS credentials

### Files Not Loading
**Problem**: Connected successfully but files don't appear

**Possible Causes**:
1. IAM user lacks `s3:ListBucket` permission
2. Bucket name incorrect
3. Region mismatch
4. CORS not configured properly

### Upload Fails
**Problem**: Upload button doesn't work or fails silently

**Possible Causes**:
1. IAM user lacks `s3:PutObject` permission on `uploads/*` prefix
2. Bucket CORS doesn't allow PUT method
3. File too large (check S3 limits)

### Documentation Images Don't Load
**Problem**: HTML renders but images are broken

**Possible Causes**:
1. Image paths are absolute URLs (should be relative)
2. IAM user lacks `s3:GetObject` permission
3. Presigned URL expired (shouldn't happen immediately)

## Development

### Adding New Features

The entire application is in `docs/index.html`. Structure:

- **Lines 1-595**: HTML structure and CSS styling
- **Lines 596-751**: HTML body and main structure
- **Lines 752-790**: Bootstrap modal and progress modal
- **Lines 791-792**: Bootstrap JS imports
- **Lines 793-2050+**: JavaScript application logic
  - Core functions, S3 client setup
  - File listing and display
  - Filter functionality
  - Folder download with ZIP compression
  - Upload, download, share features

### Testing Locally

```bash
# Simple HTTP server (Python 3)
python3 -m http.server 8000

# Access at:
open http://localhost:8000/docs/index.html
```

### Building Share URLs

Use the included script:
```bash
./build-portal-url.sh
```

Or manually construct:
```
{BASE_URL}?bucket={BUCKET}&key={ACCESS_KEY}&secret={SECRET_KEY}&region={REGION}&path={OPTIONAL_PATH}
```

## Use Cases

- **Client Portals**: Share project deliverables and documentation with clients
- **Documentation Sites**: Static documentation backed by S3
- **File Sharing**: Share files with non-technical users without AWS console access
- **Demo/Presentation Materials**: Provide easy access to presentation resources
- **Internal Knowledge Base**: Browse company documentation stored in S3
- **Asset Distribution**: Distribute design assets, videos, or other large files

## License

This project is provided as-is. Modify and use as needed for your purposes.

## Contributing

Contributions welcome! Priority areas:
1. ~~**Multi-item download feature (files + folders)**~~ ✅ **COMPLETED**
2. PDF viewer integration (low priority)
3. Markdown viewer integration (low priority)
4. Enhanced file type previews (low priority)
5. Performance optimizations
6. UI/UX improvements
7. Additional authentication methods (IAM roles, temporary credentials)

## Credits

- **AWS SDK for JavaScript**: https://aws.amazon.com/sdk-for-javascript/
- **Bootstrap**: https://getbootstrap.com/
- **CryptoJS**: https://code.google.com/archive/p/crypto-js/
- **Bootstrap Icons**: https://icons.getbootstrap.com/
- **JSZip**: https://stuk.github.io/jszip/ - For folder download ZIP compression

## Support

For issues or questions:
1. Check the Troubleshooting section above
2. Review AWS S3 documentation for CORS and IAM permissions
3. Check browser console for detailed error messages
