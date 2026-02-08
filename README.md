# Uploading Images in Flask Using Cloudinary

## Getting Started with Cloudinary

1. Sign up for a free cloudinary account: [Cloudinary Website](https://cloudinary.com/)

![cloudinary homepage](./images/cloudinary-homepage.png)

2. Answer a series of questions (they may be different that what is shown below):

<div style="display: flex; overflow-x: auto; gap: 12px; padding: 8px 0;">
    <img src="./images/cloudinary-signup-question-1.png" width="500" />
    <img src="./images/cloudinary-signup-question-2.png" width="500" />
    <img src="./images/cloudinary-signup-question-3.png" width="500" />
    <img src="./images/cloudinary-signup-question-4.png" width="500" />
    <img src="./images/cloudinary-signup-question-5.png" width="500" />
</div>

## Setting Up Your Flask Code
> This section assumes you already have a Flask application started and want to add image upload to an existing or new SQL Table

1. Open your Flask project and start the virtual environment: `pipenv shell`
2. Install cloudinary: `pipenv install cloudinary`
    > ⚠️ Note: The cloudinary website says to use `pip3 install cloudinary` but this will cause issues with deploy
3. Create a `main.py` file and add the code provided in step 3 from the Cloudinary website to that file
4. Let's create 4 environment variables in our `.env` file so we don't push sensitive information to GitHub. When you click the blue View API Keys button, you will get access to the following information:
```
CLOUD_NAME=your_cloud_name
API_KEY=your_api_key
API_SECRET=your_api_>secret
API_ENVIRONMENT_VARIABLE=your_api_environment_variable
```
### Now lets modify the code in the `main.py`
Replace **everything** in the file with the following, so that you are using the environment variables.
```py
import os
import cloudinary
import cloudinary.uploader
from cloudinary.utils import cloudinary_url

# Configuration
cloudinary.config(
    cloud_name=os.getenv('CLOUD_NAME'),
    api_key=os.getenv('API_KEY'),
    api_secret=os.getenv('API_SECRET'),
    secure=True
)
```
Let's add an `upload_image` function to the bottom of the `main.py` file

```py
def upload_image(file):
    result = cloudinary.uploader.upload(file)
    return result["secure_url"]
```

Now we need to import this file into the blueprint file that has the model we want to add image uploads to.
I'll add it to the `hoots_blueprint.py` file so every hoot has an option of adding an image. 

```py
# hoots_blueprint.py
from main import upload_image # <-- New line of code at the bottom of all other imports
```
Next we need to make sure our hoots table has an `image_url` column.
1. In the terminal enter your psql shell: `psql`
2. Then connect to your database: `\c flask_hoot` <- make sure you know your actual database name
3. Then update the hoots table:
```psql
ALTER TABLE hoots
ADD COLUMN image_url TEXT DEFAULT NULL;
```
Now I need to modify the `create_hoot` function so that it can accept any type of form information (json AND image files)
```py
@hoots_blueprint.route('/hoots', methods=['POST'])
@token_required
def create_hoot():
    try:
        image = request.files.get("image_url")
        # image_url refers to the column name on the hoots table
        # if the user does not upload an image it will default to None
        image_url = None
        # if the user does upload an image, then we update out image_url field to the uploaded image
        if image:
            image_url = upload_image(image)

        # set the author_id to be the id of the currently logged in user
        author_id = g.user["id"]

        # specify the rest of the fields in our table and grab that information
        title = request.form.get("title")
        text = request.form.get("text")
        category = request.form.get("category")

        # connect to the database
        connection = get_db_connection()
        cursor = connection.cursor(
            cursor_factory=psycopg2.extras.RealDictCursor)

        # insert all the form data into the database
        cursor.execute("""
                        INSERT INTO hoots (author, title, text, category, image_url)
                        VALUES (%s, %s, %s, %s, %s)
                        RETURNING id
                        """,
                       (author_id, title, text, category, image_url)
                       )
        hoot_id = cursor.fetchone()["id"]

        # Join the user table and the hoots table
        # Show the newly created information along with the user information
        cursor.execute("""SELECT h.id, 
                            h.author AS hoot_author_id, 
                            h.title, 
                            h.text, 
                            h.category, 
                            h.image_url,
                            u_hoot.username AS author_username
                        FROM hoots h
                        JOIN users u_hoot ON h.author = u_hoot.id
                        WHERE h.id = %s
                       """, (hoot_id,))
        created_hoot = cursor.fetchone()
        connection.commit()
        connection.close()

        # Return the newly created information
        return jsonify(created_hoot), 201
    except Exception as error:
        return jsonify({"error": str(error)}), 500
```

You can test this out in postman! If everything has been setup correctly, we should be able to upload an image to our hoots and see the image in our Cloudinary dashboard.

1. Make a collection in Postman for your Hoots, then create an account in your database. You'll want to make sure to use the generated token to authenticate your requests. If you need help with, view the steps in detail [here](./Postman-Help.md)

2. Create a POST request for hoots in your collection and use this url: `http://127.0.0.1:5000/hoots`
3. Select `Body` then `form-data`
4. Add a new key for each column in your hoots table except for author: title, text, category, image_url
5. Make sure all the keys except for image_url say `Text`. Select `File` for the `image_url`.
6. Fill out all the values with information and for the image_url select any image from your computer
![postman post request form](./images/postman-filled-out-data.png)
7. When you click send you should get a response that looks similar to this:
```json
{
    "author_username": "margot",
    "category": "News",
    "hoot_author_id": 7,
    "id": 42,
    "image_url": "https://res.cloudinary.com/dyw2rw3x6/image/upload/v1770581157/k12frr44kw0suxqyhixl.jpg",
    "text": "this is some big news",
    "title": "Big News"
}
```
> Notice how the image_url has a cloudinary url
8. Now lets check our Cloudinary dashboard and see if the image is there:
- Select `Assets > Media Library > Assets` and you should see your photo
![cloudinary assets](./images/cloudinary-assets.png)