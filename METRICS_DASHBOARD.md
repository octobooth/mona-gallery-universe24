# Mona Gallery Metrics Dashboard Documentation

## Overview

The Mona Gallery application tracks various metrics about user galleries and art pieces. While there isn't a dedicated visual dashboard interface, you can access and analyze these metrics through the database and API endpoints. This guide explains what metrics are available and how to access them.

## What Are Metrics?

Metrics are measurements that help you understand how the Mona Gallery application is being used. They answer questions like:
- How many galleries have been created?
- How many art pieces are in each gallery?
- What are the most popular art pieces (based on star ratings)?
- When were galleries and art pieces created or updated?

## Available Metrics

### Gallery Metrics

The application tracks the following information about galleries:

| Metric | Description | How to Use It |
|--------|-------------|---------------|
| **Total Galleries** | The number of galleries created in the system | Shows how many users are actively using the application |
| **Gallery Title** | The name given to each gallery | Helps identify which galleries belong to which users |
| **Gallery Description** | A text description of the gallery | Provides context about what the gallery contains |
| **Created At** | When the gallery was first created | Shows when users started using the application |
| **Updated At** | When the gallery was last modified | Indicates recent activity and engagement |

### Art Piece Metrics

Each art piece in a gallery has the following trackable metrics:

| Metric | Description | How to Use It |
|--------|-------------|---------------|
| **Total Art Pieces** | The number of art pieces across all galleries | Shows overall content volume in the application |
| **Art Piece Title** | The name of each art piece | Identifies individual artworks |
| **Art Piece Description** | Details about the art piece | Provides context and information about the artwork |
| **Star Rating** | A rating from 0 to 3 stars | Shows which art pieces are most appreciated (higher stars = more popular) |
| **URI** | The location of the image file | Points to where the actual image is stored |
| **Is File Upload** | Whether the art was uploaded or linked | Distinguishes between uploaded files and external links |
| **Created At** | When the art piece was added | Tracks when content was created |
| **Updated At** | When the art piece was last modified | Shows recent edits or changes |

## How to Access Metrics

### Through the Web Interface

When you log into the Mona Gallery application, you can see your personal metrics:

1. **View Your Gallery**: Navigate to `/gallery` to see your gallery information
2. **View Your Art Pieces**: Your gallery page displays all your art pieces with their titles, descriptions, and star ratings
3. **Individual Art Details**: Click "View" on any art piece to see its full details

### Through the API

The application provides REST API endpoints to retrieve metrics programmatically:

#### Get Gallery Information
```
GET http://localhost:8081/gallery
```
**What it returns**: Information about your gallery including ID, title, and description

**Example response**:
```json
{
  "id": 1,
  "title": "This gallery belongs to Mona",
  "description": "Mona loves your art!"
}
```

#### Get All Art Pieces
```
GET http://localhost:8081/gallery/art
```
**What it returns**: A list of all art pieces in your gallery

**Example response**:
```json
[
  {
    "id": 1,
    "title": "Octocat",
    "description": "The original GitHub mascot",
    "stars": 3,
    "uri": "https://octodex.github.com/images/original.png",
    "is_file_upload": false
  },
  {
    "id": 2,
    "title": "Mona Lisa Cat",
    "description": "A masterpiece",
    "stars": 2,
    "uri": "http://localhost:8082/blob/abc123",
    "is_file_upload": true
  }
]
```

#### Get Specific Art Piece
```
GET http://localhost:8081/gallery/art/{id}
```
**What it returns**: Detailed information about a single art piece

**Note**: You must be authenticated to access these endpoints. Include your authorization token in the request header.

### Through the Database

For administrators or developers, metrics can be accessed directly from the SQLite database:

#### Database Location
The database file is specified in the `config.json` file (typically `./data/gallery.db`)

#### Tables and Schema

**Gallery Table**:
```sql
CREATE TABLE 'gallery' (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  login TEXT UNIQUE,
  title TEXT NOT NULL,
  description TEXT,
  created_at DATETIME,
  updated_at DATETIME
);
```

**Art Piece Table**:
```sql
CREATE TABLE 'art_piece' (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  gallery_id INTEGER,
  uri TEXT,
  title TEXT,
  description TEXT,
  is_file_upload BOOL DEFAULT 0,
  stars INTEGER CHECK (stars BETWEEN 0 AND 3) DEFAULT 0,
  created_at DATETIME,
  updated_at DATETIME,
  FOREIGN KEY(gallery_id) REFERENCES gallery(id) ON DELETE CASCADE
);
```

## Common Metrics Queries

Here are some helpful database queries you can run to get useful metrics:

### Count Total Galleries
```sql
SELECT COUNT(*) as total_galleries FROM gallery;
```
This tells you how many galleries exist in the system.

### Count Total Art Pieces
```sql
SELECT COUNT(*) as total_art_pieces FROM art_piece;
```
This shows the total number of art pieces across all galleries.

### Find Most Popular Art (Highest Stars)
```sql
SELECT title, description, stars 
FROM art_piece 
ORDER BY stars DESC 
LIMIT 10;
```
This shows the top 10 most highly-rated art pieces.

### Count Art Pieces Per Gallery
```sql
SELECT g.title as gallery_name, COUNT(a.id) as art_count
FROM gallery g
LEFT JOIN art_piece a ON g.id = a.gallery_id
GROUP BY g.id, g.title
ORDER BY art_count DESC;
```
This shows which galleries have the most art pieces.

### Find Recently Added Art
```sql
SELECT title, created_at 
FROM art_piece 
ORDER BY created_at DESC 
LIMIT 10;
```
This shows the 10 most recently added art pieces.

### Calculate Average Star Rating
```sql
SELECT AVG(stars) as average_rating 
FROM art_piece;
```
This calculates the average star rating across all art pieces.

### Find Uploaded vs Linked Art
```sql
SELECT 
  is_file_upload,
  COUNT(*) as count
FROM art_piece
GROUP BY is_file_upload;
```
This shows how many art pieces are uploaded files versus external links.

## Understanding the Data

### What Do the Timestamps Mean?

- **created_at**: This timestamp is automatically set when a gallery or art piece is first created. It never changes and helps you track when things were added to the system.

- **updated_at**: This timestamp is automatically updated whenever a gallery or art piece is modified. It helps you see which items have been recently edited.

### What Do Star Ratings Mean?

Star ratings range from 0 to 3 and represent how much someone appreciates an art piece:
- **0 stars**: Not yet rated or neutral
- **1 star**: Somewhat appreciated
- **2 stars**: Well appreciated
- **3 stars**: Highly appreciated

Higher average star ratings across a gallery might indicate high-quality or engaging content.

### File Uploads vs External Links

Art pieces can come from two sources:
- **File uploads** (`is_file_upload = true`): Images that users have uploaded from their computer, stored in the blob storage service (MinIO)
- **External links** (`is_file_upload = false`): Images linked from external sources like the GitHub Octodex

This metric can help you understand how users prefer to add content.

## Exporting Metrics

### Exporting to CSV

You can export metrics data to a CSV file for analysis in spreadsheet applications:

```bash
sqlite3 -header -csv ./data/gallery.db "SELECT * FROM art_piece;" > art_pieces.csv
```

This creates a CSV file with all art piece data that you can open in Excel, Google Sheets, or other tools.

### Exporting to JSON

You can also export data as JSON for use in other applications:

```bash
sqlite3 ./data/gallery.db "SELECT json_group_array(json_object('id', id, 'title', title, 'stars', stars)) FROM art_piece;" > art_pieces.json
```

## Tips for Analyzing Metrics

1. **Track trends over time**: Compare metrics from different time periods to see if usage is growing
2. **Identify popular content**: Look at star ratings to understand what resonates with users
3. **Monitor activity**: Use the `updated_at` timestamps to see if users are actively engaging
4. **Check content mix**: See if users prefer uploading their own images or linking to external ones

## Need Help?

If you need assistance with metrics or have questions about the data:
- Review the main [README.md](./README.md) for general application information
- Check the [SUPPORT.md](./SUPPORT.md) file for support processes
- Examine the database schema in the [gallery/main.go](./gallery/main.go) file for technical details

## Security Note

This application is **deliberately vulnerable** and designed for security training purposes. In a production environment, you would want to:
- Add role-based access controls for viewing metrics
- Implement audit logging for metric queries
- Use parameterized queries to prevent SQL injection
- Add rate limiting to API endpoints
- Encrypt sensitive data at rest and in transit

For security training purposes, refer to the [SECURITY.md](./SECURITY.md) file.
