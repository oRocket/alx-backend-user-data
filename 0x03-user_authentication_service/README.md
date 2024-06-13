# User authentication service

## Setup
You will need to install `bcrypt`
```
pip3 install bcrypt
```

## Tasks
```
0. User model
```
In this task you will create a SQLAlchemy model named `User` for a database table named `users` (by using the `mapping declaration of SQLAlchemy`).

The model will have the following attributes:

- `id`, the integer primary key
- `email`, a non-nullable string
- `hashed_password`, a non-nullable string
- `session_id`, a nullable string
- `reset_token`, a nullable string

```
bob@dylan:~$ cat main.py
#!/usr/bin/env python3
"""
Main file
"""
from user import User

print(User.__tablename__)

for column in User.__table__.columns:
    print("{}: {}".format(column, column.type))

bob@dylan:~$ python3 main.py
users
users.id: INTEGER
users.email: VARCHAR(250)
users.hashed_password: VARCHAR(250)
users.session_id: VARCHAR(250)
users.reset_token: VARCHAR(250)
bob@dylan:~$
```
**Repo**:

- GitHub repository: `alx-backend-user-data`
- Directory: `0x03-user_authentication_service`
- File: `user.py`

```
1. create user
```
In this task, you will complete the `DB` class provided below to implement the `add_user` method.
```
"""DB module
"""
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from sqlalchemy.orm.session import Session

from user import Base


class DB:
    """DB class
    """

    def __init__(self) -> None:
        """Initialize a new DB instance
        """
        self._engine = create_engine("sqlite:///a.db", echo=True)
        Base.metadata.drop_all(self._engine)
        Base.metadata.create_all(self._engine)
        self.__session = None

    @property
    def _session(self) -> Session:
        """Memoized session object
        """
        if self.__session is None:
            DBSession = sessionmaker(bind=self._engine)
            self.__session = DBSession()
        return self.__session
```
Note that `DB._session` is a private property and hence should NEVER be used from outside the `DB` class.

Implement the `add_user` method, which has two required string arguments: `email` and `hashed_password`, and returns a `User` object. The method should save the user to the database. No validations are required at this stage.
```
bob@dylan:~$ cat main.py
#!/usr/bin/env python3
"""
Main file
"""

from db import DB
from user import User

my_db = DB()

user_1 = my_db.add_user("test@test.com", "SuperHashedPwd")
print(user_1.id)

user_2 = my_db.add_user("test1@test.com", "SuperHashedPwd1")
print(user_2.id)

bob@dylan:~$ python3 main.py
1
2
bob@dylan:~$
```
**Repo**:

- GitHub repository: `alx-backend-user-data`
- Directory: `0x03-user_authentication_service`
- File: `db.py`


```
4. Hash password
```
In this task you will define a `_hash_password` method that takes in a `password` string arguments and returns bytes.

The returned bytes is a salted hash of the input password, hashed with `bcrypt.hashpw`.
```
bob@dylan:~$ cat main.py
#!/usr/bin/env python3
"""
Main file
"""
from auth import _hash_password

print(_hash_password("Hello Holberton"))

bob@dylan:~$ python3 main.py
b'$2b$12$eUDdeuBtrD41c8dXvzh95ehsWYCCAi4VH1JbESzgbgZT.eMMzi.G2'
bob@dylan:~$
```
**Repo**:

- GitHub repository: `alx-backend-user-data`
- Directory: `0x03-user_authentication_service`
- File: `auth.py`


```
6. Basic Flask app
```
In this task, you will set up a basic Flask app.

Create a Flask app that has a single `GET` route (`"/"`) and use `flask.jsonify` to return a JSON payload of the form:
```
{"message": "Bienvenue"}
```
Add the following code at the end of the module:
```
if __name__ == "__main__":
    app.run(host="0.0.0.0", port="5000")
```
**Repo**:

- GitHub repository: `alx-backend-user-data`
- Directory: `0x03-user_authentication_service`
- File: `app.py`


```
20. End-to-end integration test
```
Start your app. Open a new terminal window.

Create a new module called `main.py`. Create one function for each of the following tasks. Use the `requests` module to query your web server for the corresponding end-point. Use `assert` to validate the response’s expected status code and payload (if any) for each task.
- register_user(email: str, password: str) -> None
- log_in_wrong_password(email: str, password: str) -> None
- log_in(email: str, password: str) -> str
- profile_unlogged() -> None
- profile_logged(session_id: str) -> None
- log_out(session_id: str) -> None
- reset_password_token(email: str) -> str
- update_password(email: str, reset_token: str, new_password: str) -> None
Then copy the following code at the end of the `main` module:
```
EMAIL = "guillaume@holberton.io"
PASSWD = "b4l0u"
NEW_PASSWD = "t4rt1fl3tt3"


if __name__ == "__main__":

    register_user(EMAIL, PASSWD)
    log_in_wrong_password(EMAIL, NEW_PASSWD)
    profile_unlogged()
    session_id = log_in(EMAIL, PASSWD)
    profile_logged(session_id)
    log_out(session_id)
    reset_token = reset_password_token(EMAIL)
    update_password(EMAIL, reset_token, NEW_PASSWD)
    log_in(EMAIL, NEW_PASSWD)
```
Run `python main.py`. If everything is correct, you should see no output.

**Repo**:

- GitHub repository: `alx-backend-user-data`
- Directory: `0x03-user_authentication_service`
- File: `main.py`

