# CURL

### Downloading

```sh
curl -o <file.name> https://example.com/index.html
```

Below command will download file name of remote document, if no filename is specified in the URL, this fails:

```sh
curl -O https://example.com/files/abc.mp3
```

### Uploading

Upload all data on stdin to a specified server:

```sh
curl -T - https://example.com/myfile
```

Upload data from a specified file to server:

```sh
curl -T uploadfile https://example.com/myfile
```

### Verbose/Debug

> flag `-v` : Verbose

To get even more details an information on what curl does, try using the `--trace` or `--trace-ascii` options with a given filename to log to:

```sh
curl --trace trace.txt www.example.com
```

> flag `-I` or `--head` : Show detailed information about a single file
>
> for HTTP, can use `-i` or `--include`

Store HTTP headers in a separate file (header.txt)

```curl
curl --dump-header header.txt www.example.com
```

### POST (HTTP)

> flag `-d` : Data. The post data must be urlencoded.

```sh
curl -d "name=Rafael%20Sagula&phone=123456" https://example.com/guest.cgi
```

Or automatically URL encode the data:

```sh
curl --data-urlencode "name=Rafael Sagula&phone=123456" https://example.com/guest.cgi
```

The data is a form as below:

```html
<form aciton="guest.cgi" method="post">
  <input name="Rafael Sagula">
  <input phone="123456">
</form>
```

> While `-d` uses the _application/x-www-form-urlencoded_ mime-type, curl also supports the more capable _multipart/form-data_ type.
>
> `-F` accepts parameters like -F "name=contents". If U want the contents to be read from a file, use _@filename_ as contents. If the content-type is not specified, curl tries to guess from the file extension, or use the previously specified type or else uses the default type _application/octet-stream_

Example:

```sh
curl -F "coolfiles=@fill.gif;type=image/gif,fil2.txt,fil3.html" https://example.com/postit.cgi
```

```sh
curl -F "file=@cooltext.txt" -F "yourname=Daniel" -F "filedescription=Cool text file with cool text inside" https://example.com/postit.cgi
```

To send 2 files in 1 post you can do it in 2 ways:

Send multiple files in a single field with a single field name:

```sh
curl -F "pictures=@dog.gif,cat.gif" $URL
```

Send 2 fields with 2 field names:

```sh
curl -F "dogpicture=@dog.gif" -F "catpicture=@cat.gif" $URL
```

> To send a field value literally without interpetting a leading @ or <, or an embedded ;type=, use `--form-string` instead of `-F`. This is recommended when the value is obtained form a user or some other unpredictable source. Uner these circumstances, using `-F` instead of `--form-string` could allow a user to trick curl into uploading a file.

### Referrer

Use flag `-e` to specify referrer

```sh
curl -e www.example.org https://example.com
```

### User Agent

Use flag `-A` to specify User-Agent

```sh
curl -A 'Mozilla/3.0 (Win95; I)' https://example.com
```

### Config File

Put `.curlrc` file at user's home directory (like .zshrc, .gitconfig,...)

### Extra headers

Use flag `-H` to specify header

```sh
curl -H "Content-Type: application/json" https://example.com
```

---

## For HTTP (RESTful API)

### Bypass SSL certificate validation

Use flag `-k` or `--insecure` to bypass SSL cert validation

```sh
curl -k https://example.com
```

### Specifiy certificate

```sh
curl --cert mycert.pem https://example.com
```

Or can use CA Certificate to verify server's certificate

```sh
curl --cacert ca-bundle.pem https://example.com
```



### Using HTTP Proxy

Use flag `-x` or `--proxy`

```sh
curl -x "http://proxy.example.com:80" -X GET 'https://www.example.com/resources'
```

```sh
curl -x "socks5://proxy.example.com:1080" -X GET 'https://www.example.com/resources'
```



### GET

Insomnia style

```sh
curl --request GET \
  --url 'http://localhost:8080/articles' \
  --header 'x-api-key: apikey123'
```

Postman style

```sh
curl --location 'http://localhost:8080/articles' \
	--header 'x-api-key: apikey123'
```

OpenAPI style

```sh
curl -X 'http://localhost:8080/articles' \
	-H 'x-api-key: apikey123'
```

##### Notice

When using query string

Method 1:

```sh
curl -X GET -G 'http://localhost:8080/articles' \
	--data-urlencode 'page=1' \
	--data-urlencode 'limit=10'
	-H 'x-api-key: apikey123'
```

> With `-G`, `--data-urlencode` (or `-d` , `--data`, `--data-binary`) will be appended to the URL as a query string

Method 2:

```sh
curl -X GET 'http://localhost:8080/articles' \
	--url-query 'page=1' \
	--url-query 'limit=10'
	-H 'x-api-key: apikey123'
```

### POST

Insomnia style

```sh
curl --request POST \
	--url 'http://localhost:8080/articles' \
	--header 'Content-Type: application/json' \
	--header 'x-api-key: apikey123' \
	--data '{
	"title": "This is title",
	"description": "This is description",
	"content": "This is content"
	}'
```

Postman style

```sh
curl --location 'http://localhost:8080/articles' \
	--header 'x-api-key: apikey123' \
	--header 'Content-Type: application/json' \
	--data '{
	"title": "This is title",
	"description": "This is description",
	"content": "This is content"
	}'
```

OpenAPI style

```sh
curl -X POST 'http://localhost:8080/articles' \
	-H 'x-api-key: apikey123' \
	-H 'Content-Type: application/json' \
	-d '{
	"title": "This is title",
	"description": "This is description",
	"content": "This is content"
	}'
```

> You can pass a file containing request body to the data option (`-d` or `--data`) like this:

```sh
curl -X POST 'http://localhost:8080/articles' \
	-H 'x-api-key: apikey123' \
	-H 'Content-Type: application/json' \
	-d @request.json
```

Other methods are similar

> Note that `-d` or `--data` is implicit POST method, `-I` for HEAD. To specify method, you can use `--request` or `-X` followed by method Name

### Formatting

To write `verbose` to 1 file and `response-payload` to another but still print out the result:

```sh
curl -X GET \
  'https://www.example.com/resources' \
  -H "Authorization: $TOKEN" \
  -v 2> >(tee verbose.log >&2) | awk 'END{print}' | tee response-payload.json | jq
```

Then when we want to re-print the result with these file:

```sh
cat verbose.log >&2; cat response-payload.json  | jq
```

