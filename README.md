# apollo-cache-issue

Repro for https://github.com/apollographql/apollo-server/issues/3559

## Steps

1.  run server with existing configuration (with `defaultMaxAge`) defined

    ```
    node index.js
    ```

1.  verify `cache-control` header is set

    ```
    > curl 'http://localhost:4000/graphql' -H 'Accept-Encoding: gzip, deflate, br' -H 'Content-Type: application/json' -H 'Accept: application/json' --data-binary '{"query":"{\n  books{\n    title\n  }\n}"}' -i --compressed

    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: *
    Content-Type: application/json; charset=utf-8
    cache-control: max-age=1000, public
    Content-Length: 99
    ETag: W/"63-rqKuwXrbApDAKA6SS5Pb8FfTZUI"
    Date: Thu, 04 Jun 2020 22:26:22 GMT
    Connection: keep-alive

    {"data":{"books":[{"title":"Harry Potter and the Chamber of Secrets"},{"title":"Jurassic Park"}]}}
    ```

    ✅ `cache-control` header is set

1.  change `cacheControl` in server config to `true` (`index.js:37`)

1.  verify there is no `cache-control` header

    ```
    curl 'http://localhost:4000/graphql' -H 'Accept-Encoding: gzip, deflate, br' -H 'Content-Type: application/json' -H 'Accept: application/json' --data-binary '{"query":"{\n books{\n title\n }\n}"}' -i --compressed

    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: \*
    Content-Type: application/json; charset=utf-8
    Content-Length: 183
    ETag: W/"b7-HxsRFlJyQVitP5MjDhc6vjk3Sjg"
    Date: Thu, 04 Jun 2020 22:33:52 GMT
    Connection: keep-alive

    {"data":{"books":[{"title":"Harry Potter and the Chamber of Secrets"},{"title":"Jurassic Park"}]},"extensions":{"cacheControl":{"version":1,"hints":[{"path":["books"],"maxAge":0}]}}}
    ```

    ✅ `cache-control` header is not set

1.  use `@cacheControl(maxAge: 100)` directive (`index.js:4`)

    ```
    type Book @cacheControl(maxAge: 100) {
    ```

1.  verify `cache-control` header is set

    ```
    curl 'http://localhost:4000/graphql' -H 'Accept-Encoding: gzip, deflate, br' -H 'Content-Type: application/json' -H 'Accept: application/json' --data-binary '{"query":"{\n  books{\n    title\n  }\n}"}' -i --compressed

    HTTP/1.1 200 OK
    Access-Control-Allow-Origin: *
    Content-Type: application/json; charset=utf-8
    Content-Length: 185
    ETag: W/"b9-4uBdjdF/hAQbMaNJy4vncELG+yY"
    Date: Thu, 04 Jun 2020 22:38:08 GMT
    Connection: keep-alive

    {"data":{"books":[{"title":"Harry Potter and the Chamber of Secrets"},{"title":"Jurassic Park"}]},"extensions":{"cacheControl":{"version":1,"hints":[{"path":["books"],"maxAge":100}]}}}
    ```

    🛑 `cache-control` header is not set. Even though the response doesn't contain anything that is cached differently

    ```
    {
      "data": {
        "books": [
          {
            "title": "Harry Potter and the Chamber of Secrets"
          },
          {
            "title": "Jurassic Park"
          }
        ]
      },
      "extensions": {
        "cacheControl": {
          "version": 1,
          "hints": [
            {
              "path": [
                "books"
              ],
              "maxAge": 100
            }
          ]
        }
      }
    }
    ```
