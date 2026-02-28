# PhotoAlbum Application Architecture Diagram

```mermaid
flowchart TD
    Browser["Web Browser\n(User)"]

    subgraph App["ASP.NET Core 9.0 - PhotoAlbum"]
        subgraph Presentation["Presentation Layer (Razor Pages)"]
            Index["Index.cshtml\n(Gallery Grid + Upload)"]
            Detail["Detail.cshtml\n(Full-size Photo + Metadata)"]
            PhotoFile["PhotoFile.cshtml\n(File Retrieval Endpoint)"]
        end

        subgraph Business["Business Logic Layer"]
            IPhotoService["IPhotoService\n(Interface)"]
            PhotoService["PhotoService\n(Upload, Validate, Retrieve, Delete)"]
            ImageSharp["SixLabors.ImageSharp\n(Image Dimension Extraction)"]
        end

        subgraph DataAccess["Data Access Layer (EF Core 9.0)"]
            DbContext["PhotoAlbumContext\n(EF Core DbContext)"]
            PhotoModel["Photo Model\n(Filename, Size, MIME, Dimensions, Timestamp)"]
        end
    end

    subgraph Storage["Data Storage"]
        SqlServer["SQL Server LocalDB\n(PhotoAlbumDb)"]
        FileSystem["Local File System\nwwwroot/uploads\n(GUID-named image files)"]
    end

    Browser -->|"HTTP Requests"| Presentation
    Presentation -->|"Uses"| IPhotoService
    IPhotoService -->|"Implemented by"| PhotoService
    PhotoService -->|"Image processing"| ImageSharp
    PhotoService -->|"CRUD operations"| DbContext
    PhotoService -->|"Read / Write files"| FileSystem
    DbContext -->|"SQL queries"| SqlServer
    DbContext --> PhotoModel
```

## Architecture Overview

| Layer | Technology | Responsibility |
|-------|-----------|----------------|
| Presentation | ASP.NET Core Razor Pages | Gallery UI, upload form, photo display |
| Business Logic | PhotoService (C#) | File validation, upload, retrieval, deletion |
| Image Processing | SixLabors.ImageSharp 3.1 | Extract image dimensions from uploaded files |
| Data Access | EF Core 9.0 + SQL Server | Persist photo metadata with descending timestamp index |
| File Storage | Local file system (`wwwroot/uploads`) | Store GUID-named image files |
| Database | SQL Server LocalDB | Persist photo records |

## Key Design Notes

- **Service abstraction**: `IPhotoService` interface enables future swap to Azure Blob Storage
- **Transactional consistency**: File is deleted on DB save failure to avoid orphaned files
- **Configuration-driven**: File size limits (10 MB) and allowed MIME types in `appsettings.json`
- **Auto-migrations**: EF Core migrations applied automatically on startup (skipped in test environment)
