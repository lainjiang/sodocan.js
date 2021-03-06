# Server API

The server serves as the central hub for all the data that populates the blueprints. It receives this data from both the parser, and users.

### Post requests from Parser:
  ##### *localhost:3000*/create
   All parser results should be posted here. If this is the first time an entry is posted, it is converted to the proper for easy GET requests. If the method already exists, we check to see if the text of your entry is the same as any of the current entries. If the text is the same, we do nothing. If the text does not match, we push it into the array as a new entry.

   *Example Post JSON*
  ```
  {
  "header": {
    "project": "Mongoose"
    "groups": ["mongoose","models","documents","schemas"]
     },
      "body": [
        {
          "functionName": "mongoose.connection",
          "group": "mongoose",
          "params": [{"name":"url", "type":"string"}],
          "returns": [],
          "explanations": {
            "descriptions": "creates the actual connection between your code, and the mongoDB server",
            tips: "You only need to do this once, and then mongoose is setup"
          }
        }
      }
    ]
  }
  ```
  ### Get From Client:
  ##### *localhost*:3000/api
  All get requests, return the requested methods, their params (inputs and outputs) along with the requested number of entries and comments for the requested explanation type. If no amount of entries is specified, we default to 1. If no amount of comments are specified, we default to 0.


  **/api/** - All get requests involving methods, will go through api
  **/api/project** - will request every method in the project, and default to requesting top rated entry, with no comments <br>
  **/api/project/2/all** - will request every method of a project, the top 2 entries for each method, and every comment, for every method <br>
  **/api/project/1/0/tips/0/0/description/all/all** - will request every method in a project. Explicitly stating you want no entries or comments for tips, and every entry and every comment for description. <br>
  **/api/project/ref/methodName** - Will send the top entry and no comments for the entry (for every context) <br>
  **/api/project/ref/methodName/10/5** - Will send the top 10 entries for the specific method, and the top 5 comments (for every context) <br>
  **/api/project/ref/methodName/all/description/5/1** - Will send all entries for for every context that isn’t description, with no comments, will send the top 5 entries and top comment for each of those entries, for description. <br>
  **/api/get/project/ref/methodName/context/'entryID-'entryID/10** - will send the top 10 comments for a specific entry <br>
  **/api/project/ref/methodName/context/'entryID-'entryID/commentID** - will send specific comment <br>

  ```
  [
    {
      "_id": "5625376402941ab4141fa4cb",
      "project": "testProj",
      "functionName": "method1",
      "group": "testGroup",
      "reference": {
        "params": [],
        "returns": []
      },
    "explanations": {
      "descriptions": [
        {
          "text": "descriptionblahblah",
          "upvotes": 0,
          "comments": [],
          "entryID": 1294058334
        }
      ],
      "examples": [
        {
          "text": "examplesblahblah",
          "upvotes": 0,
          "comments": [],
          "entryID": 719765163
        }
      ],
      "tips": [
        {
          "text": "tipsblahblah",
          "upvotes": 0,
          "comments": [],
          "entryID": 907176294
        }
      ]
  ```
  ### Post requests from Client:
    #### For upvoting
    ##### *localhost:3000*/upvote - upvotes entires or comments
    *Example Post JSON*
    ```
    {
      project:
      functionName:
      context:
      entryID:
      [commentID:] -- if omitted, we assume we are upvoting an entry
      [username/ip:] -- not sure what we are doing with this yet, but we probably need a way to stop Lain from just clicking upvote 1,000 times
    }

    ```
  ### For new entries / comments
  #### *localhost:3000*/addEntry
  *Example Post JSON*
  ```
  {
    project:
    functionName:
    context:
    text:
    [entryID:] -- if omitted, we assume we are posting an entry. If included, we will be adding an comment to that entry
    [username/ip:] -- not needed yet, but will probably want this
  }
  ```

  ### Authentication

  -Get requests do not require authentication (anyone can view docs without logging in)

  -POST /auth/register and POST /auth/login are routes for creating an account/logging in. Requests to these
  must have properties called 'username' and 'password' on the body.
  ```
  req.body = { username: "Harold" password: "lept0n"}
  ```
  - GET /auth/github is the route to trigger logging in with Github OAuth.

  -all content modifying post requests (POST /upvote, POST /addEntry) and POST /logout require auth

  -upon logging in, the server will send back a JSON object with the property access_token.  The value
  of this property is an encrypted JSON web token.  this should be placed in localStorage, and must be placed in the body of all the authenticated requests listed above, with the property name access_token.
  Example:
  ```
  req.body = { access_token : 'TOKEN_VALUE' };
  ```

  -tokens will expire after 24hrs (this can be changed in authenticationConfig.js).  Requesting an authenticated route with an expired token, will result in a '401: Unauthorized' response.  The user must log in again to receive a fresh token.

  -logging out will require a post request to the /logout path, and should also remove the token from localStorage.
  Once a user has logged out, the token they were using will no longer work.
