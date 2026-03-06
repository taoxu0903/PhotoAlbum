# Architecture Diagram

PhotoAlbum is an ASP.NET Core 9.0 Razor Pages web application for photo gallery management, using local file storage and SQL Server for persistence.

## Application Architecture

```mermaid
flowchart TD
    Browser["Web Browser\n(HTTP Client)"]

    subgraph App["ASP.NET Core 9.0 Web Application"]
        subgraph Presentation["Presentation Layer\n(Razor Pages)"]
            Index["Index.cshtml\nGallery Grid + Upload"]
            Detail["Detail.cshtml\nPhoto Detail + Navigation"]
            PhotoFile["PhotoFile.cshtml\nFile Retrieval Endpoint"]
        end

        subgraph Service["Service Layer"]
            IPhotoService["IPhotoService\n(Interface)"]
            PhotoService["PhotoService\n- Upload with validation\n- MIME type + size checks\n- Dimension extraction\n- Transactional delete"]
        end

        subgraph Data["Data Access Layer\n(Entity Framework Core 9.0)"]
            DbContext["PhotoAlbumContext\n(DbContext)\nPhotos DbSet"]
            ImageSharp["SixLabors.ImageSharp 3.1\nImage dimension extraction"]
        end
    end

    subgraph Storage["Storage"]
        SQLServer["SQL Server LocalDB\nPhotoAlbumDb\nPhoto metadata"]
        FileSystem["Local File System\nwwwroot/uploads\nGUID-named image files\nJPEG, PNG, GIF, WebP"]
    end

    Browser -- "HTTP requests" --> Presentation
    Index --> IPhotoService
    Detail --> IPhotoService
    PhotoFile --> IPhotoService
    IPhotoService --> PhotoService
    PhotoService --> DbContext
    PhotoService --> ImageSharp
    PhotoService -- "Read/Write image files" --> FileSystem
    DbContext -- "EF Core migrations + queries" --> SQLServer
```
