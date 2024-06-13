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
In this task you will create a SQLAlchemy model named `User` for a database table named `users` (by using the mapping declaration of SQLAlchemy).

The model will have the following attributes:

- id, the integer primary key
- email, a non-nullable string
- hashed_password, a non-nullable string
- session_id, a nullable string
- reset_token, a nullable string
