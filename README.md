# File Bud Backend

Backend API for File Bud, a file management application inspired by Google Drive and file systems. The service handles authentication, folder hierarchy, file uploads, private media access, downloads, thumbnails, video streaming, and storage usage tracking.

The project is built with Express, MongoDB, and Cloudinary. MongoDB stores users, folders, and file metadata, while Cloudinary is used for authenticated file storage and media delivery.

The backend is designed around file-system-like behavior rather than flat CRUD resources. It works with nested folders, recursive operations, file metadata consistency, temporary local files, external CDN state, signed media delivery, and multi-step flows that need to stay reliable when one part fails.

## [Demo](https://file-bud-frontend.vercel.app/)

## Features

-   User registration and login with JWT authentication
-   HTTP-only cookie support and bearer token support
-   Automatic root folder creation for each user
-   Nested folder structure with parent-child relationships
-   File upload with temporary local storage and Cloudinary persistence
-   Folder-level filename conflict handling
-   Private file access through signed Cloudinary URLs
-   Signed download URLs for authenticated files
-   Thumbnail generation for supported images, videos, and PDFs
-   HLS streaming URL support for videos
-   Recursive folder deletion
-   User storage usage tracking
-   MongoDB transactions for multi-step write operations
-   Cleanup handling for temporary files and failed Cloudinary operations

## Tech Stack

-   Node.js
-   Express.js
-   MongoDB
-   Mongoose
-   Cloudinary
-   JWT
-   bcrypt
-   Multer
-   cookie-parser
-   CORS

## Project Structure

```text
src/
  app.js                    Express app setup and route registration
  index.js                  Server entry point
  constants.js              Shared constants
  controllers/              Route handlers
  db/                       Database connection
  middlewares/              Auth and upload middleware
  models/                   Mongoose models
  routes/                   API routes
  utils/                    Shared API and Cloudinary utilities

public/
  temp/                     Temporary upload directory
```

## Data Model

The backend uses three main models:

-   `User`: stores account details, auth token state, root folder reference, storage usage, and storage limit.
-   `Folder`: represents a directory in the file tree with references to files, subfolders, owner, and parent folder.
-   `File`: stores metadata for Cloudinary-backed assets, including public ID, owner, parent folder, file type, size, and format.

This structure allows the API to work with nested folders while keeping file binaries outside the database.

## Architecture Notes

-   Folders are modeled as a tree using parent and child references, which allows the API to represent nested directories and fetch folder contents with populated files and subfolders.
-   File binaries are stored in Cloudinary, while MongoDB stores ownership, placement, size, format, resource type, and delivery metadata.
-   Upload and delete flows update multiple resources, so MongoDB transactions are used where database state must stay consistent.
-   Cloudinary operations sit outside MongoDB transactions, so the backend includes cleanup paths for failed uploads and logs failed remote deletions for retry.
-   File access is handled through signed URLs instead of public asset links, keeping private files behind the API while still using Cloudinary for delivery and streaming.

## API Routes

All routes are prefixed with:

```text
/api/v1
```

### Users

| Method | Endpoint          | Description                                |
| ------ | ----------------- | ------------------------------------------ |
| `POST` | `/users/register` | Register a user and create the root folder |
| `POST` | `/users/login`    | Login and issue tokens                     |
| `POST` | `/users/logout`   | Logout and clear auth cookies              |
| `GET`  | `/users/getUser`  | Get the authenticated user                 |

### Folders

| Method   | Endpoint                   | Description                              |
| -------- | -------------------------- | ---------------------------------------- |
| `POST`   | `/folders/create`          | Create a folder                          |
| `GET`    | `/folders/fetch/:folderId` | Fetch a folder with files and subfolders |
| `DELETE` | `/folders/delete`          | Delete a folder and its contents         |

### Files

| Method   | Endpoint               | Description                   |
| -------- | ---------------------- | ----------------------------- |
| `POST`   | `/files/upload`        | Upload a file                 |
| `GET`    | `/files/fetch`         | Get a signed file access URL  |
| `GET`    | `/files/download`      | Get a signed download URL     |
| `DELETE` | `/files/delete`        | Delete a file                 |
| `GET`    | `/files/thumbnail/:id` | Get a signed thumbnail URL    |
| `GET`    | `/files/stream`        | Get a signed video stream URL |

Protected routes require a valid access token.

## Authentication

The API uses JWT-based authentication. Protected routes accept the access token from either:

-   `accessToken` cookie
-   `Authorization: Bearer <token>` header

Passwords are hashed before storage. File and folder operations verify resource ownership before returning or modifying data.

## File Handling

Uploads are first stored temporarily using Multer, then uploaded to Cloudinary as authenticated resources. The database stores only file metadata and Cloudinary identifiers.

For file access, the API generates signed Cloudinary URLs instead of exposing public asset URLs. Videos can also be served through signed HLS stream URLs.

## Consistency and Cleanup

Several operations update multiple records or systems. The backend uses MongoDB transactions where database writes need to remain consistent, such as user registration, folder creation, uploads, and delete flows.

Because Cloudinary operations cannot be part of a MongoDB transaction, the code includes cleanup handling for cases where a file is uploaded but a later database step fails. Failed Cloudinary deletions are logged so they can be retried later.

## Getting Started

### Prerequisites

-   Node.js
-   npm
-   MongoDB connection string
-   Cloudinary account credentials

### Installation

```bash
git clone <your-repository-url>
cd file-bud-backend
npm install
```

Create a `.env` file in the project root:

```env
PORT=8000
CORS_ORIGIN=http://localhost:YOUR_FRONTEND_PORT
MONGODB_URI=
ACCESS_TOKEN_SECRET=
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=
REFRESH_TOKEN_EXPIRY=10d
CLOUDINARY_CLOUD_NAME=
CLOUDINARY_API_KEY=
CLOUDINARY_API_SECRET=
```

Start the development server:

```bash
npm run dev
```

The API runs on:

```text
http://localhost:8000
```

## Environment Variables

| Variable                | Description              |
| ----------------------- | ------------------------ |
| `PORT`                  | Server port              |
| `CORS_ORIGIN`           | Allowed frontend origin  |
| `MONGODB_URI`           | MongoDB connection URI   |
| `ACCESS_TOKEN_SECRET`   | JWT access token secret  |
| `ACCESS_TOKEN_EXPIRY`   | Access token expiry      |
| `REFRESH_TOKEN_SECRET`  | JWT refresh token secret |
| `REFRESH_TOKEN_EXPIRY`  | Refresh token expiry     |
| `CLOUDINARY_CLOUD_NAME` | Cloudinary cloud name    |
| `CLOUDINARY_API_KEY`    | Cloudinary API key       |
| `CLOUDINARY_API_SECRET` | Cloudinary API secret    |

## Development Notes

-   The default MongoDB database name is `filebud`.
-   Uploaded files are temporarily stored in `public/temp`.
-   The upload middleware currently limits files to 100 MB.
-   The backend is intended to be used with a separate frontend configured through `CORS_ORIGIN`.

## License

This project is licensed under the ISC license.
