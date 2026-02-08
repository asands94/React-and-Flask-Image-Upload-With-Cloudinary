# Setting Up Your React Frontend for Image Uploads
## About 
These instructions will show how to modify a React form so it can send uploaded images to a backend

```jsx
// HootForm.jsx

import { useState, useEffect } from 'react'
import { useParams } from 'react-router'

import * as hootService from '../../services/hootService'

const HootForm = ({ handleAddHoot, handleUpdateHoot }) => {
    const { hootId } = useParams()
    const [formData, setFormData] = useState({
        title: '',
        text: '',
        category: 'News',
    })
    // Add in a new useState for your image
    const [imageFile, setImageFile] = useState(null)

    useEffect(() => {
        const fetchHoot = async () => {
            if (!hootId) return
            const hootData = await hootService.show(hootId)
            setFormData({
                title: hootData.title || '',
                text: hootData.text || '',
                category: hootData.category || 'News',
                // add image_url to the formData for update purposes
                // we want to see if a hoot already has an image
                image_url: hootData.image_url || '',
            })
        }
        fetchHoot()
    }, [hootId])

    const handleChange = (evt) => {
        setFormData({ ...formData, [evt.target.name]: evt.target.value })
    }

    // update the handleSubmit
    const handleSubmit = (evt) => {
        evt.preventDefault()

        // FormData allows us to send text and files to our backend
        const data = new FormData()

        // append (add) the form values to FormData
        data.append('title', formData.title)
        data.append('text', formData.text)
        data.append('category', formData.category)

        if (imageFile) {
            data.append('image_url', imageFile)
        }

        if (hootId) {
            // send the updated data to the backend
            handleUpdateHoot(hootId, data)
        } else {
            // send the new data to the backend
            handleAddHoot(data)
        }
    }

    return (
        <main>
            <h1>{hootId ? 'Edit Hoot' : 'New Hoot'}</h1>
            <form onSubmit={handleSubmit}>
                <label htmlFor='title-input'>Title</label>
                <input
                    required
                    type='text'
                    name='title'
                    id='title-input'
                    value={formData.title}
                    onChange={handleChange}
                />
                <label htmlFor='text-input'>Text</label>
                <textarea
                    required
                    type='text'
                    name='text'
                    id='text-input'
                    value={formData.text}
                    onChange={handleChange}
                />
                <label htmlFor='category-input'>Category</label>
                <select
                    required
                    name='category'
                    id='category-input'
                    value={formData.category}
                    onChange={handleChange}
                >
                    <option value='News'>News</option>
                    <option value='Games'>Games</option>
                    <option value='Music'>Music</option>
                    <option value='Movies'>Movies</option>
                    <option value='Sports'>Sports</option>
                    <option value='Television'>Television</option>
                </select>
                {/* add a new label and input for the image */}
                <label htmlFor='image_url-input'>Image</label>
                <input
                    type='file'
                    name='image_url'
                    id='image_url-input'
                    accept='image/*'
                    onChange={(e) => setImageFile(e.target.files[0])}
                />
                {/* if the hoot is being updated, show a preview of the previously uploaded image */}
                {hootId && formData.image_url && (
                    <div>
                        <p>Current image:</p>
                        <img
                            src={formData.image_url}
                            alt='Current hoot'
                            style={{ maxWidth: '200px' }}
                        />
                    </div>
                )}

                <button type='submit'>SUBMIT</button>
            </form>
        </main>
    )
}

export default HootForm
```

We also need to make a small change to our services so that the functions accept form data and not just JSON:
```js
// hootService.js


const create = async (hootFormData) => {
    try {
        const res = await fetch(`${BASE_URL}`, {
            method: 'POST',
            headers: {
                Authorization: `Bearer ${localStorage.getItem('token')}`,
                // we removed 'Content-Type': 'application/json' since we are sending back more than JSON data now 
            },
            // we removed JSON.stringify()
            body: hootFormData,
        })
        return res.json()
    } catch (err) {
        console.log(err)
    }
}

// We'll do the same thing for the update function:

const updateHoot = async (hootId, hootFormData) => {
    try {
        const res = await fetch(`${BASE_URL}/${hootId}`, {
            method: 'PUT',
            headers: {
                Authorization: `Bearer ${localStorage.getItem('token')}`,
            },
            body: hootFormData,
        })
        return res.json()
    } catch (err) {
        console.log(err)
    }
}
```

## AND WE'RE DONE 🎉🎉🎉
View the entire backend here:
> The only difference in the full version is that a created_at column has been added to the hoots table
View the entire frontend here:
> The only difference in the full version is that the created_at date has been fixed to show the date a hoot was created instead of Invalid Date