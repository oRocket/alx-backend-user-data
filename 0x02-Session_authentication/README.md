# 0x02. Session authentication

## api/v1/app.py, api/v1/views/users.py
Copy all your work of the 0x06. Basic authentication project in this new folder.

In this version, you implemented a Basic authentication for giving you access to all User endpoints:

GET /api/v1/users
POST /api/v1/users
GET /api/v1/users/<user_id>
PUT /api/v1/users/<user_id>
DELETE /api/v1/users/<user_id>
Now, you will add a new endpoint: GET /users/me to retrieve the authenticated User object.

Copy folders models and api from the previous project 0x06. Basic authentication
Please make sure all mandatory tasks of this previous project are done at 100% because this project (and the rest of this track) will be based on it.
Update @app.before_request in api/v1/app.py:
Assign the result of auth.current_user(request) to request.current_user
Update method for the route GET /api/v1/users/<user_id> in api/v1/views/users.py:
If <user_id> is equal to me and request.current_user is None: abort(404)
If <user_id> is equal to me and request.current_user is not None: return the authenticated User in a JSON response (like a normal case of GET /api/v1/users/<user_id> where <user_id> is a valid User ID)
Otherwise, keep the same behavior

## api/v1/auth/session_auth.py, api/v1/app.py
Create a class SessionAuth that inherits from Auth. For the moment this class will be empty. It’s the first step for creating a new authentication mechanism:

validate if everything inherits correctly without any overloading
validate the “switch” by using environment variables
Update api/v1/app.py for using SessionAuth instance for the variable auth depending of the value of the environment variable AUTH_TYPE, If AUTH_TYPE is equal to session_auth:

import SessionAuth from api.v1.auth.session_auth
create an instance of SessionAuth and assign it to the variable auth
Otherwise, keep the previous mechanism.

## api/v1/auth/session_auth.py
Update SessionAuth class:

Create a class attribute user_id_by_session_id initialized by an empty dictionary
Create an instance method def create_session(self, user_id: str = None) -> str: that creates a Session ID for a user_id:
Return None if user_id is None
Return None if user_id is not a string
Otherwise:
Generate a Session ID using uuid module and uuid4() like id in Base
Use this Session ID as key of the dictionary user_id_by_session_id - the value for this key must be user_id
Return the Session ID
The same user_id can have multiple Session ID - indeed, the user_id is the value in the dictionary user_id_by_session_id
Now you an “in-memory” Session ID storing. You will be able to retrieve an User id based on a Session ID.

## api/v1/auth/session_auth.py
Update SessionAuth class:

Create an instance method def user_id_for_session_id(self, session_id: str = None) -> str: that returns a User ID based on a Session ID:

Return None if session_id is None
Return None if session_id is not a string
Return the value (the User ID) for the key session_id in the dictionary user_id_by_session_id.
You must use .get() built-in for accessing in a dictionary a value based on key
Now you have 2 methods (create_session and user_id_for_session_id) for storing and retrieving a link between a User ID and a Session ID.

## api/v1/auth/auth.py
Update api/v1/auth/auth.py by adding the method def session_cookie(self, request=None): that returns a cookie value from a request:

Return None if request is None
Return the value of the cookie named _my_session_id from request - the name of the cookie must be defined by the environment variable SESSION_NAME
You must use .get() built-in for accessing the cookie in the request cookies dictionary
You must use the environment variable SESSION_NAME to define the name of the cookie used for the Session ID

## api/v1/app.py
Update the @app.before_request method in api/v1/app.py:

Add the URL path /api/v1/auth_session/login/ in the list of excluded paths of the method require_auth - this route doesn’t exist yet but it should be accessible outside authentication
If auth.authorization_header(request) and auth.session_cookie(request) return None, abort(401)

## api/v1/auth/session_auth.py
Update SessionAuth class:

Create an instance method def current_user(self, request=None): (overload) that returns a User instance based on a cookie value:

You must use self.session_cookie(...) and self.user_id_for_session_id(...) to return the User ID based on the cookie _my_session_id
By using this User ID, you will be able to retrieve a User instance from the database - you can use User.get(...) for retrieving a User from the database.
Now, you will be able to get a User based on his session ID.

## api/v1/views/session_auth.py, api/v1/views/__init__.py
Create a new Flask view that handles all routes for the Session authentication.

In the file api/v1/views/session_auth.py, create a route POST /auth_session/login (= POST /api/v1/auth_session/login):

Slash tolerant (/auth_session/login == /auth_session/login/)
You must use request.form.get() to retrieve email and password parameters
If email is missing or empty, return the JSON { "error": "email missing" } with the status code 400
If password is missing or empty, return the JSON { "error": "password missing" } with the status code 400
Retrieve the User instance based on the email - you must use the class method search of User (same as the one used for the BasicAuth)
If no User found, return the JSON { "error": "no user found for this email" } with the status code 404
If the password is not the one of the User found, return the JSON { "error": "wrong password" } with the status code 401 - you must use is_valid_password from the User instance
Otherwise, create a Session ID for the User ID:
You must use from api.v1.app import auth - WARNING: please import it only where you need it - not on top of the file (can generate circular import - and break first tasks of this project)
You must use auth.create_session(..) for creating a Session ID
Return the dictionary representation of the User - you must use to_json() method from User
You must set the cookie to the response - you must use the value of the environment variable SESSION_NAME as cookie name - tip
In the file api/v1/views/__init__.py, you must add this new view at the end of the file.

## api/v1/auth/session_auth.py, api/v1/views/session_auth.py
Update the class SessionAuth by adding a new method def destroy_session(self, request=None): that deletes the user session / logout:

If the request is equal to None, return False
If the request doesn’t contain the Session ID cookie, return False - you must use self.session_cookie(request)
If the Session ID of the request is not linked to any User ID, return False - you must use self.user_id_for_session_id(...)
Otherwise, delete in self.user_id_by_session_id the Session ID (as key of this dictionary) and return True
Update the file api/v1/views/session_auth.py, by adding a new route DELETE /api/v1/auth_session/logout:

Slash tolerant
You must use from api.v1.app import auth
You must use auth.destroy_session(request) for deleting the Session ID contains in the request as cookie:
If destroy_session returns False, abort(404)
Otherwise, return an empty JSON dictionary with the status code 200
