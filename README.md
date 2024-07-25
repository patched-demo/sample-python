# Documentation for dsvw.py

## Overview

This Python script is designed to start a simple HTTP server that demonstrates various web vulnerabilities for educational and testing purposes. It includes examples of SQL Injection, XSS, CSRF, and other common web vulnerabilities. The script creates a minimal web application to help users understand, identify, and test security vulnerabilities in web applications.

## Inputs

The script can be started using the command line, and once running, it listens for HTTP requests on a specified IP address and port.

### Command Line Execution

To start the server, simply run:
```shell
python dsvw.py
```

### HTTP Requests

The server responds to various URL patterns that simulate different vulnerabilities:
- **SQL Injection**: `/login?username=admin&amp;password=' OR '1'='1`
- **XSS**: `/?v=0.2%3Cscript%3Ealert(%22arbitrary%20javascript%22)%3C%2Fscript%3E`
- **CSRF**: `/login?username=admin&password=csrf_token`

## Outputs

### Console Output

Upon execution, the server displays information about the server and listens for incoming connections:
```plaintext
Damn Small Vulnerable Web (DSVW) < 100 LoC (Lines of Code) #v0.2b
 by: Miroslav Stampar (@stamparm)

[i] running HTTP server at 'http://127.0.0.1:65412'...
```

### HTTP Responses

The server generates HTTP responses based on the vulnerability being demonstrated:
- **SQL Injection**: Returns user data if the injection is successful.
- **XSS**: Executes JavaScript code in the visitor's browser.
- **CSRF**: Simulates unauthorized actions if not properly protected.

## Detailed Functionality

### Initialization (init function)
- Initializes an in-memory SQLite database.
- Populates the database with sample user data from an XML structure.

### HTTP Request Handler (ReqHandler class)
- Extends `http.server.BaseHTTPRequestHandler`.
- Processes `GET` requests and simulates various vulnerabilities.
- Handles routes like `/login`, `/?id=2`, and more sophisticated payloads.

### Vulnerability Examples
The following vulnerabilities are demonstrated with corresponding URLs and payloads:
- **Blind SQL Injection (boolean)**: `/vulnerable?id=2%20AND%20SUBSTR((SQL_PAYLOAD),1,1)%3D'7'`
- **Cross Site Scripting (reflected)**: `/?v=0.2<script>alert('XSS')</script>`
- **XML External Entity (local)**: `/?xml=XML_PAYLOAD`
- **Cross Site Request Forgery**: `/login?username=admin&password=csrf_token`

### Running in Docker
The script can be containerized using Docker, and the Dockerfile is provided:
```dockerfile
# Example: docker build . -t dsvw && docker run -p 65412:65412 dsvw

FROM python:3.10-alpine3.18 AS build

RUN apk --no-cache add libxml2-dev libxslt-dev gcc python3 python3-dev py3-pip musl-dev linux-headers

RUN python3 -m ensurepip --upgrade && python3 -m pip install pex~=2.1.47
RUN mkdir /source
COPY requirements.txt /source/
RUN pex -r /source/requirements.txt -o /source/pex_wrapper

FROM python:3.10-alpine3.18 AS final

RUN apk upgrade --no-cache
WORKDIR /dsvw
RUN adduser -D dsvw && chown -R dsvw:dsvw /dsvw

COPY dsvw.py .
RUN sed -i 's/127.0.0.1/0.0.0.0/g' dsvw.py
COPY --from=build /source /

EXPOSE 65412
USER dsvw
CMD ["/dsvw/pex_wrapper", "dsvw.py"]
```

### Docker Compose
The service can also be managed using Docker Compose:
```yml
version: '3.9'

services:
  dsvw:
    build: .
    expose:
      - 65412
    ports:
      - 65412:65412
    restart: always
```

## Dependencies

The script requires the following dependencies:
- Python 3.10+
- `lxml`

Dependencies can be installed using the `requirements.txt`:
```txt
lxml
```

## License

The project is licensed under the Unlicense, making it free and open-source with no restrictions on its use.

For more detailed information, refer to the LICENSE file included in the project.