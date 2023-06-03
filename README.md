## Cloudinary Image Upload

This is a simple project to upload images to cloudinary using RUby on Rails. It uses the cloudinary gem to upload images to cloudinary. It also uses the cloudinary jquery plugin to upload images to cloudinary.


####  PROCESS

First, we need to sign up for Cloudinary. Follow this link and register your account if you don’t have one:

Cloudinary Account Sign Up(https://cloudinary.com/users/register/free)
Edit description
cloudinary.com

We will be redirected to the Cloudinary Dashboard after a successful sign-up.

In the Dashboard, we can see the ‘Account Details’ section.

For now, we will only need the ‘Cloud name’, ‘API Key’, and ‘API Secret’ to set up the Rails API.

And, that’s it, as far as it goes for setting up Cloudinary. Now we can move to the Rails API.

##### Rails API
We will be using a Rails API to store the URLs of images we upload to Cloudinary.

##### Let’s generate a new Rails app:
	rails new cloundinary-image-upload --database=postgresql --api
We will generate the app as an API only app
We will use PostgreSQL as the database
First, we will need to update the Gemfile of our app.

Uncomment gem ‘rack-cors’
Add gem ‘cloudinary’
Run bundle install
Now let’s move on to setting up the model and controller.

Model and Controller
For our app, we will generate an Item model. The model will have an attribute of image, which will be the actual URL of the uploaded image.

##### Generating the model:

	rails g model Item image:string
Generating the controller:

	rails g controller items
After creating our model and controller it’s time to create and migrate our database:

	rails db:create
	rails db:migrate
	Cors config file
Now let’s navigate to config/initializers/cors.rb file.

In this file we need to uncomment these lines:

		Rails.application.config.middleware.insert_before 0, Rack::Cors do
			allow do
				origins 'example.com'
				resource '*',
					headers: :any,
					methods: [:get, :post, :put, :patch, :delete, :options, :head]
			end
		end

We also need to replace this line:

	origins ‘example.com’ 
with this:

	origins ‘*’
Cloudinary config file
Now, let’s create a new file in the config/initializers folder, named cloudinary.rb

In this file we need to add the following configuration:

		Cloudinary.config do |config|
			config.cloud_name = 'CLOUD_NAME'
			config.api_key = 'API_KEY'
			config.api_secret = 'API_SECRET'
			config.secure = true
			config.cdn_subdomain = true
		end

Be sure to replace the CLOUD_NAME , API_KEY, and API_SECRET with the information from your Cloudinary account details. And make sure they are wrapped in strings.

Items Controller
In our ItemsController we will set up the create method.

The create method will handle the actual uploading of the image from our front-end to Cloudinary. When the image is uploaded Cloudinary will send a response that contains the URL of the stored image.

We will not be storing the actual image in our Rails database, but the URL pointing to the image stored on Cloudinary.

Since we have already added the Cloudinary gem to our app, we now have access to the methods from Cloudinary.

The one that we will be using is:

	Cloudinary::Uploader.upload
This method accepts the image data as the one argument.

	image = Cloudinary::Uploader.upload(params[:image])
And after the image is successfully uploaded, Cloudinary will send out a response containing the URL of the image. We will access the URL using the ‘url’ key.

And next, we need to create the Item instance:

	item = Item.create(image: image['url'])
Lastly, we can render a JSON response.

And to wrap it up, we now have the full create method for our ItemsController :

	class ItemsController < ApplicationController
		def create
			image = Cloudinary::Uploader.upload(params[:image])
			item = Item.create(image: image['url'])
			render json: {
				status: 200,
				item: item
			}
		end
	end
	
Routes file
And lastly, we need to set up a route for our create method for Item.

Let’s navigate to config/routes.rb and update the file:

	Rails.application.routes.draw do
		resources :items, only: [:create]
	end

####  FRONTE-END 

The frontend part is in React Js with Typescript:

	import React, { useState } from 'react';

	export interface ImageUploadFormProps {
			id: number;
			image: string;
			name: string;
	}



	const ImageUploadForm = () => {
			const [image, setImage] = useState<File | null>(null);

		 const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
					if (e.target.files) {
							setImage(e.target.files[0]);
					}
			};

			const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
					e.preventDefault();
					const data = new FormData();
					data.append('image', image!);

					fetch('http://localhost:3000/items', {
							method: 'POST',
							body: data,
					});
			};

			return (
					<div className="justify-evenly w-full p-8 ">
							<p className="text-xl my-8 ">Airbnb</p>
							<form onSubmit={handleSubmit}>
									<label>Image upload</label>
									<input type="file" name="image" onChange={handleChange} />
									<button type="submit">Submit</button>
							</form>
					</div>
			)
	};

	export default ImageUploadForm;
