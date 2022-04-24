Using Google Drive API with R
================

## Introduction

This document aims to introduce how to use [Google Drive
API](https://developers.google.com/drive/api) with R. Readers should
first [create a service
account](https://cloud.google.com/docs/authentication/production) and
acquire the JSON key for authentication before reading this document.
Below will cover how to authenticate using the service account JSON via
R and demonstrate the download and upload methods.

## Authentication

``` r
# https://developers.google.com/drive/api/v3/reference/files/get
scopes <- c(
  "https://www.googleapis.com/auth/drive",
  "https://www.googleapis.com/auth/drive.file",
  "https://www.googleapis.com/auth/drive.readonly",
  "https://www.googleapis.com/auth/drive.metadata.readonly",
  "https://www.googleapis.com/auth/drive.appdata",
  "https://www.googleapis.com/auth/drive.metadata",
  "https://www.googleapis.com/auth/drive.photos.readonly"
)

endpoints <- httr::oauth_endpoints("google")
secrets <- jsonlite::fromJSON("./serviceAccountKey.json")

# https://www.rdocumentation.org/packages/httr/versions/1.4.2/topics/oauth_service_token
oauth_token <- httr::oauth_service_token(
  endpoint = endpoints,
  secrets = secrets,
  scope = scopes
)
```

## Download Files

Access the file without authentication.

``` r
fileId <- "1_zeJqQP8umrTk-evSAt3wCLxAkTKo0lC"
url <- sprintf("https://www.googleapis.com/drive/v3/files/%s?alt=media", fileId)
httr::GET(url)
```

    ## Response [https://www.googleapis.com/drive/v3/files/1_zeJqQP8umrTk-evSAt3wCLxAkTKo0lC?alt=media]
    ##   Date: 2022-04-24 20:38
    ##   Status: 403
    ##   Content-Type: application/json; charset=UTF-8
    ##   Size: 379 B
    ## {
    ##  "error": {
    ##   "errors": [
    ##    {
    ##     "domain": "usageLimits",
    ##     "reason": "dailyLimitExceededUnreg",
    ##     "message": "Daily Limit for Unauthenticated Use Exceeded. Continued use r...
    ##     "extendedHelp": "https://code.google.com/apis/console"
    ##    }
    ##   ],
    ## ...

Access the file with authentication.

``` r
httr::GET(url, oauth_token)
```

    ## Response [https://www.googleapis.com/drive/v3/files/1_zeJqQP8umrTk-evSAt3wCLxAkTKo0lC?alt=media]
    ##   Date: 2022-04-24 20:38
    ##   Status: 200
    ##   Content-Type: application/x-zip-compressed
    ##   Size: 7.04 MB
    ## <BINARY BODY>

Saving the file.

``` r
httr::GET(url, oauth_token, httr::write_disk("/tmp/tmp.zip", overwrite = TRUE))
```

    ## Response [https://www.googleapis.com/drive/v3/files/1_zeJqQP8umrTk-evSAt3wCLxAkTKo0lC?alt=media]
    ##   Date: 2022-04-24 20:39
    ##   Status: 200
    ##   Content-Type: application/x-zip-compressed
    ##   Size: 7.04 MB
    ## <ON DISK>  /tmp/tmp.zip

## Uploading Files

To upload files to Google Drive via the API, the `POST` method will be
used. The request body is put in a `metadata.json` as seen below. The
keys can be found from the [API
document](https://developers.google.com/drive/api/v3/reference/files/create).

``` bash
cat metadata.json
```

    ## {
    ##   "parents": [
    ##     "1IG2_jnDiyg9CFIEbv5s-EWP9slu7ezKC"
    ##   ],
    ##   "mimeType": "application/x-zip-compressed",
    ##   "name": "donwloaded.zip"
    ## }

The script below uploads the downloaded zip file to another folder in Google Drive. 

``` r
# https://stackoverflow.com/questions/31080363/how-to-post-multipart-related-content-with-httr-for-google-drive-api
url <- "https://www.googleapis.com/upload/drive/v3/files"

body <- list(
  metadata = httr::upload_file("./metadata.json", type = "application/json; charset=UTF-8"),
  media = httr::upload_file("/tmp/tmp.zip", type = "application/x-zip-compressed")
)

httr::POST(
  url = url,
  body = body,
  oauth_token
)
```

    ## Response [https://www.googleapis.com/upload/drive/v3/files]
    ##   Date: 2022-04-24 20:40
    ##   Status: 200
    ##   Content-Type: application/json; charset=UTF-8
    ##   Size: 142 B
    ## {
    ##  "kind": "drive#file",
    ##  "id": "1fLdFQwJl6zkLtXEUKGKLI6XkcfWOZ21Y",
    ##  "name": "donwloaded.zip",
    ##  "mimeType": "application/x-zip-compressed"
    ## }
