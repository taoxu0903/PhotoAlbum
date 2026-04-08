# Architecture Diagram

PhotoAlbum is an ASP.NET Core 9.0 Razor Pages web application for photo gallery management with local file storage and SQL Server database persistence.

## Application Architecture

```mermaid
flowchart TD
    Browser["Browser\nUser Interface"]

    subgraph App["ASP.NET Core 9.0 Web Application"]
        subgraph Pages["Presentation Layer - Razor Pages"]
            IndexPage["Index.cshtml\nGallery Grid + Upload"]
            DetailPage["Detail.cshtml\nFull-size View + Metadata"]
            PhotoFilePage["PhotoFile.cshtml\nFile Retrieval Endpoint"]
        end

        subgraph Services["Service Layer"]
            IPhotoService["IPhotoService\nInterface"]
            PhotoService["PhotoService\nUpload / Retrieve / Delete\nSixLabors.ImageSharp\nImage Dimension Extraction"]
        end

        subgraph Data["Data Access Layer"]
            PhotoAlbumContext["PhotoAlbumContext\nEntity Framework Core 9.0"]
            PhotoModel["Photo Model\nOriginalFileName, StoredFileName\nFilePath, FileSize, MimeType\nWidth, Height, UploadedAt"]
        end
    end

    subgraph Storage["Storage"]
        SQLServer["SQL Server LocalDB\nPhotoAlbumDb\nPhotos Table"]
        FileSystem["Local File System\nwwwroot/uploads\nGUID-named image files"]
    end

    Browser -->|"HTTP Requests"| Pages
    Pages -->|"Calls"| IPhotoService
    IPhotoService -->|"Implemented by"| PhotoService
    PhotoService -->|"Validates MIME type and size\nExtracts dimensions"| PhotoService
    PhotoService -->|"EF Core queries"| PhotoAlbumContext
    PhotoAlbumContext -->|"Reads and writes"| SQLServer
    PhotoAlbumContext -->|"Uses"| PhotoModel
    PhotoService -->|"Stores and retrieves files"| FileSystem
    Pages <-->|"Serves static files"| FileSystem
```
