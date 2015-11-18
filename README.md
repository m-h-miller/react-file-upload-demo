# React File Upload Demo
This demo shows how to upload images using React, Paperclip, and AWS S3.

## Key Files
- [application.rb](./config/application.rb)
- [post.rb](./app/models/post.rb)
- [_post.json.jbuilder](./app/views/api/posts/_post.json.jbuilder)
- [posts_controller.rb](./app/controllers/api/posts_controller.rb)
- [posts_store.js](./app/assets/javascripts/stores/posts_store.js)
- [api_util.js](./app/assets/javascripts/util/api_util.js)
- [post_form.js.jsx](./app/assets/javascripts/components/post_form.js.jsx)

## Useful Docs
- [Paperclip](https://github.com/thoughtbot/paperclip#paperclip)
- [Figaro] (https://github.com/laserlemon/figaro#why-does-figaro-exist)
- [AWS] (http://aws.amazon.com/)
- [FileReader] (https://developer.mozilla.org/en-US/docs/Web/API/FileReader)
- [FormData] (https://developer.mozilla.org/en-US/docs/Web/API/FormData)

### Image Preview
On change of the file input component we instantiate a new [FileReader]
(https://developer.mozilla.org/en-US/docs/Web/API/FileReader) object. We ask
this reader using the [readAsDataUrl] (https://developer.mozilla.org/en-US/docs/Web/API/FileReader.readAsDataURL)
method to load the file as a base64 encoded string using the data property. A
base64 encoded string can be used as an inline source for an img. In the callback
to `onloadend` we `setState` for both the source string and the file itself.

### Image Uploading
To upload the file we will instantiate a new
[FormData] (https://developer.mozilla.org/en-US/docs/Web/API/FormData) object.
We then use the [append](https://developer.mozilla.org/en-US/docs/Web/API/FormData/append)
method to add key/values to send to the server. One of the key/value pairs will be the binary
file we grab from `this.state.file`. Be mindful to have your keys match whatever your Rails
controller is expecting in the params. In our case this is `post[image]`. We will use
`ApiUtil.createPost()` to make the AJAX request and create an action on success. In the
options for the `$.ajax` request we need to set `processData` and `contentType` both to
`false`. This is to prevent default jQuery behaviour from trying to convert our FormData
object and sending up the wrong header. See more in this [SO post](http://stackoverflow.com/a/8244082).

### Image Missing and Jbuilder
Use a Jbuilder template when generating JSON from Rails. When providing JSON that represents
an image url to your attached image (for instance on S3), make sure always to wrap the URL in
the asset_path helper as such:

`json.image_url asset_path(post.image.url(:original))`

### AWS Paperclip Config

```ruby
# config/application.rb

config.paperclip_defaults = {
  :storage => :s3,
  :s3_credentials => {
    :bucket => ENV["s3_bucket"],
    :access_key_id => ENV["s3_access_key_id"],
    :secret_access_key => ENV["s3_secret_access_key"]
  }
}
```

Store your API keys securely with Figaro.

```ruby

# config/application.yml
development:
  s3_bucket: "XXXX-BUCKET-NAME-DEV"

production:
  s3_bucket: "XXXX-BUCKET-NAME-PRO"

s3_access_key_id: "XXXX"
s3_secret_access_key: "XXXX"
```

Make sure to use separate buckets for development and production. You may want to use the following security policy for your Amazon AWS IAM user.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "Stmt1420751757000",
      "Effect": "Allow",
      "Action": [
        "s3:*"
      ],
      "Resource": [
        "arn:aws:s3:::XXXX-BUCKET-NAME-DEV",
        "arn:aws:s3:::XXXX-BUCKET-NAME-DEV/*",
        "arn:aws:s3:::XXXX-BUCKET-NAME-PRO",
        "arn:aws:s3:::XXXX-BUCKET-NAME-PRO/*"
      ]
    }
  ]
}
```


