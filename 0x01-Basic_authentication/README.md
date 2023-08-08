# Basic authentication

# Task 0
Simple basic API

# Task 1
What the HTTP status code for a request unauthorized? 401 of course!

Edit api/v1/app.py:

	* Add a new error handler for this status code, the response must be:
		* a JSON: {"error": "Unauthorized"}
		* status code 401
		* you must use jsonify from Flask
For testing this new error handler, add a new endpoint in api/v1/views/index.py:

	* Route: GET /api/v1/unauthorized
	* This endpoint must raise a 401 error by using abort - Custom Error Pages
By calling abort(401), the error handler for 401 will be executed.

In the first terminal:
bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 python3 -m api.v1.app
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
....

In a second terminal:
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/unauthorized"
{
  "error": "Unauthorized"
}
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/unauthorized" -vvv
*   Trying 0.0.0.0...
* TCP_NODELAY set
* Connected to 0.0.0.0 (127.0.0.1) port 5000 (#0)
> GET /api/v1/unauthorized HTTP/1.1
> Host: 0.0.0.0:5000
> User-Agent: curl/7.54.0
> Accept: */*
> 
* HTTP 1.0, assume close after body
< HTTP/1.0 401 UNAUTHORIZED
< Content-Type: application/json
< Content-Length: 30
< Server: Werkzeug/0.12.1 Python/3.4.3
< Date: Sun, 24 Sep 2017 22:50:40 GMT
< 
{
  "error": "Unauthorized"
}
* Closing connection 0
bob@dylan:~$

# Task 2: Error handler: Forbidden
What the HTTP status code for a request where the user is authenticate but not allowed to access to a resource? 403 of course!

Edit api/v1/app.py:

	* Add a new error handler for this status code, the response must be:
		* a JSON: {"error": "Forbidden"}
		* status code 403
		* you must use jsonify from Flask
For testing this new error handler, add a new endpoint in api/v1/views/index.py:

	* Route: GET /api/v1/forbidden
	* This endpoint must raise a 403 error by using abort - Custom Error Pages
By calling abort(403), the error handler for 403 will be executed.

In the first terminal:
bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 python3 -m api.v1.app
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
....

In a second terminal:
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/forbidden"
{
  "error": "Forbidden"
}
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/forbidden" -vvv
*   Trying 0.0.0.0...
* TCP_NODELAY set
* Connected to 0.0.0.0 (127.0.0.1) port 5000 (#0)
> GET /api/v1/forbidden HTTP/1.1
> Host: 0.0.0.0:5000
> User-Agent: curl/7.54.0
> Accept: */*
> 
* HTTP 1.0, assume close after body
< HTTP/1.0 403 FORBIDDEN
< Content-Type: application/json
< Content-Length: 27
< Server: Werkzeug/0.12.1 Python/3.4.3
< Date: Sun, 24 Sep 2017 22:54:22 GMT
< 
{
  "error": "Forbidden"
}
* Closing connection 0
bob@dylan:~$

# Task 3: Auth class
Now you will create a class to manage the API authentication.

	* Create a folder api/v1/auth
	* Create an empty file api/v1/auth/__init__.py
	* Create the class Auth:
		* in the file api/v1/auth/auth.py
		* import request from flask
		* class name Auth
		* public method def require_auth(self, path: str, excluded_paths: List[str]) -> bool: that returns False - path and excluded_paths will be used later, now, you don’t need to take care of them
		* public method def authorization_header(self, request=None) -> str: that returns None - request will be the Flask request object
		* public method def current_user(self, request=None) -> TypeVar('User'): that returns None - request will be the Flask request object
This class is the template for all authentication system you will implement.
bob@dylan:~$ cat main_0.py
#!/usr/bin/env python3
""" Main 0
"""
from api.v1.auth.auth import Auth

a = Auth()

print(a.require_auth("/api/v1/status/", ["/api/v1/status/"]))
print(a.authorization_header())
print(a.current_user())

bob@dylan:~$ 
bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 ./main_0.py
False
None
None
bob@dylan:~$

# Task 4: Define which routes don't need authentication
Update the method def require_auth(self, path: str, excluded_paths: List[str]) -> bool: in Auth that returns True if the path is not in the list of strings excluded_paths:

	* Returns True if path is None
	* Returns True if excluded_paths is None or empty
	* Returns False if path is in excluded_paths
	* You can assume excluded_paths contains string path always ending by a /
	* This method must be slash tolerant: path=/api/v1/status and path=/api/v1/status/ must be returned False if excluded_paths contains /api/v1/status/

bob@dylan:~$ cat main_1.py
#!/usr/bin/env python3
""" Main 1
"""
from api.v1.auth.auth import Auth

a = Auth()

print(a.require_auth(None, None))
print(a.require_auth(None, []))
print(a.require_auth("/api/v1/status/", []))
print(a.require_auth("/api/v1/status/", ["/api/v1/status/"]))
print(a.require_auth("/api/v1/status", ["/api/v1/status/"]))
print(a.require_auth("/api/v1/users", ["/api/v1/status/"]))
print(a.require_auth("/api/v1/users", ["/api/v1/status/", "/api/v1/stats"]))

bob@dylan:~$
bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 ./main_1.py
True
True
True
False
False
True
True
bob@dylan:~$


