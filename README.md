#ES6 implementation of custom errors in Express app
Tutorial about how easy is it to create your own custom error classes in ES6.

##Simple example
```es6
class CustomError extends Error {
    constructor(message, someProperty) {
        super();
        Error.captureStackTrace(this, this.constructor);
        this.name = this.constructor.name;
        this.message = message;
        this.someProperty = someProperty;
    }
}

class SomeError extends CustomError {
    constructor() {
        super('Some error occurred.', 'Foo');
    }
}

class SomeAnotherError extends CustomError {
    constructor() {
        super('Some another error occurred.', 'Bar');
    }
}

const err = new SomeError();
console.log(err instanceof Error);              // true
console.log(err instanceof CustomError);        // true
console.log(err instanceof SomeAnotherError);   // false
```

##Example in Express app:
libs/responseErrors.es6:
```es6
class ResponseError extends Error {
    constructor(message, type, status) {
        super();
        Error.captureStackTrace(this, this.constructor);
        this.name = this.constructor.name;
        this.message = message;
        this.type = type;
        this.status = status;
    }
}

export class ForbiddenError extends ResponseError {
    constructor() {
        super('Site access denied.', 'Forbidden', 403);
    }
}

export class InvalidTokenError extends ResponseError {
    constructor() {
        super('Specified token is invalid.', 'InvalidToken', 401);
    }
}
```
app.es6:
```es6
import express from 'express';
import {ForbiddenError, InvalidTokenError} from './libs/responseErrors';

const app = express();

//Get item with id = 1
app.get('/items/1', (req, res, next) => {
    res.json({name: 'Foo', description: 'Bar'});
});

//Create new item
app.post('/items', (req, res, next) => {
    //Some token failure validation
    next(new InvalidTokenError());
});

//Handle all other requests as Forbidden
app.use((req, res, next) => {
    next(new ForbiddenError());
});

//Error handler
app.use((err, req, res, next) => {
    const statusCode = err.status || 500;
    const type = err.type || 'UnknownError';
    const message = err.message || 'Something went wrong.';
    res.status(statusCode).json({type, message});
});

//Start server
const server = app.listen(3000, () => {
    console.log('Listening on port %s', server.address().port);
});
```
###Install dependencies (Babel, Express)
from [package.json](https://github.com/PetrKohut/es6-custom-errors-express-app/blob/master/package.json)
```sh
npm install
```

###Start server
```sh
npm start
```

###Test APIs to see our custom errors
```sh
curl -X GET localhost:3000/items/1
# response: {"name":"Foo","description":"Bar"}
# statusCode: 200

curl -X POST localhost:3000/items -d '{"name":"Bar","description":"Foo"}'
# response: {"type":"InvalidToken","message":"Specified token is invalid."}
# statusCode: 401

curl -X GET localhost:3000/anything
# response: {"type":"Forbidden","message":"Site access denied."}
# statusCode: 403
```
