# Postman Help
## About 
These instructions will show how to setup Postman

## Content
- [Create a Collection](#create-a-collection)
- [Sign Up a New User](#sign-up-a-new-user)
- [Authorize All Routes](#authorize-all-routes)

## Create a Collection
1. Create a new collection
![postman new collection](./images/postman-new-collection.png)
2. Select blank collection
![postman blank collection](./images/postman-blank-collection.png)
3. Name your collection **Hoots Flask Api**
![postman name collection](./images/postman-name-collection.png)

## Sign Up a New User
Create a POST request to sign up a new user
> Note: Make sure your backend is running

![postman new request](./images/postman-new-request.png)

Make sure you have the following settings:
1. Select `POST`
2. The url should be: http://127.0.0.1:5000/auth/sign-up
3. Select `Body`, `raw` and `JSON`
4. Add in a username and password:
```json
{
    "username": "cooluser",
    "password": "pass123"
}
```

![postman signup](./images/postman-signup.png)

## Authorize All Routes
After hitting send, you should see a token. 
1. Copy the token (just the part in between the quotes)
2. Click on the name of your collection on the left: **Hoot Flask Api**
3. Click the Authorization tab
4. Select Bearer Token as the Auth Type
5. Paste in your token

![postman authorization setup](./images/postman-setup-auth.png)

**You are now setup with Postman! 🎉**

Get started adding Cloudinary image uploads here: [Flask Instructions](./Flask-Image-Upload-Instructions.md)