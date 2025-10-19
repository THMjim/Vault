# curl
Set Proxy and POST request
```bash
# curl looks for http_proxy env variable by default, so curl will use it when it's set
export http_proxy=http://127.0.0.1:8080/

curl -i -F name=test.png -F myFile=@sample.png http://IP/exiftest.php -v -L
# multipart form data request
# -i   : Inclue headers
# -F   : Form Data
# -v   : verbose
# -L   : follow redirects

```