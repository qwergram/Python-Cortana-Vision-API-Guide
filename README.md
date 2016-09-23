# Building a Cloud Service App with Cognitive Vision APIs
In this project, we will be building a [Django](https://www.djangoproject.com/) Cloud Service App, and we will be deploying to 
[Azure's Cloud Service](https://azure.microsoft.com/en-us/services/cloud-services/) which uses the 
[PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) paradigm of deployment and programming.

The advantage of using a [PaaS](https://en.wikipedia.org/wiki/Platform_as_a_service) instead of deploying
on, say, [AWS](https://aws.amazon.com/) or on [Virtual Machines](https://azure.microsoft.com/en-us/services/virtual-machines/)
which use [IaaS](https://en.wikipedia.org/wiki/Cloud_computing#Infrastructure_as_a_service_.28IaaS.29) paradigms,
is deployment becomes a lot easier. In addition, you can dedicate entire VMs to either a 
[worker role or web role](https://azure.microsoft.com/en-us/documentation/articles/cloud-services-choose-me/).

In this HOL, we will create a cloud service app that allows users to upload photos, and then
utilizing [Cortana's Vision API](https://www.microsoft.com/cognitive-services/en-us/face-api),
we can sort the images into two containers: one with faces, and one without faces.

Our program outline will look like this:

![](http://i.imgur.com/dmrNiDQ.png)

This HOL will expose us to Azure Cloud Storage, Azure Queues, Azure Cloud Services and Cortana Cognative
services as well.


# Prerequisites
You will need the following to complete this HOL

- [Visual Studio](https://www.visualstudio.com/en-us/visual-studio-homepage-vs.aspx) Installed
- [Azure Account](https://portal.azure.com/)

# Getting Started in Visual Studio

## 1. Start a new project

![](http://i.imgur.com/dfkJZ3E.png)

## 2. Create a "Azure Cloud Service"

![](http://i.imgur.com/vGvCkiL.png)

## 3. Select Worker and Web Roles

![](http://i.imgur.com/syWSZJ5.png)

## 4. Build Project. Your screen should look something like this.

![](http://i.imgur.com/XTFrOMR.png)


# Building the Cloud Service App: Web Role

There are two parts of this app: Uploading and Viewing.
I will be seperating these functions into two seperate apps, namely `upload` and `library` respectively.


## 5. Build a Django App

![](http://i.imgur.com/HCBtqkM.png)

Once you created `upload` and `library`, your directory structure should look something like this:
```
- Django Web Role\ (Root Directory)
    - Django_Web_Role\ (Project Directory)
        - __init__.py
        - settings.py
        - urls.py
        - wsgi.py
    - library\ (App Directory)
        - migrations\
        - templates\
        - __init__.py
        - admin.py
        - models.py
        - tests.py
        - views.py
    - upload\ (App Directory)
        - migrations\
        - templates\
        - __init__.py
        - admin.py
        - models.py
        - tests.py
        - views.py
    - manage.py
    - bin\ (Scripts for deployment)
        - *.ps1 (Some powershell scripts)
```

## 6. Update `Django_web_Role\settings.py` and `manage.py`

In `settings.py`, update `INSTALLED_APPS`. Append the new apps you created to it.
    
```python
INSTALLED_APPS = [
    # Add your apps here to enable them
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'library', # < New apps
    'upload',  # < New apps
]
```

In `manage.py` make sure that the string `"$safeprojectname$"` does not exist.

```python
#!/usr/bin/env python
"""
Command-line utility for administrative tasks.
"""

import os
import sys

if __name__ == "__main__":
    os.environ.setdefault(
        "DJANGO_SETTINGS_MODULE",
        "Django_Web_Role.settings" # < make sure this is a valid python module
                                   # < NOT "$safeprojectname$.settings"
    )

    from django.core.management import execute_from_command_line

    execute_from_command_line(sys.argv)
```



## 7. Update `Django_Web_Role\urls.py`

Update `urlpatterns` to check your new apps when routing.

```python

from django.conf.urls import include, url

# import Django_Web_Role.views                                  # < Delete this import

import upload.urls                                              # < New Import (Doesn't exist yet)
# import library.urls                                           # < New Import (Doesn't exist yet)

urlpatterns = [
    url(r'^<your_path_here>/', include(upload.urls)),           # < New route (Doesn't exist yet)
    # url(r'^<your_path_here>/', include(library.urls)),        # < New Route (Doesn't exist yet)
]
```

__Note__: You can set your path to be root (e.g. `http://example.com/`) with `url(r'^', include(<your_lib>.urls))`.

## 8. Create new `upload\urls.py`

We're going to build the uploading functionality first. Create a new Empty Python File under `upload\`
and name it `urls.py`


![](http://i.imgur.com/DWyVGCy.png)

![](http://i.imgur.com/VwLeW2N.png)

Inside it, create a list called `urlpatterns`:

```python
from django.conf.urls import include, url  # < Import include and urls

urlpatterns = [
    # Empty List
]
```

## 9. Create views under `upload\views.py`

Create a view for just viewing the upload form. Our end goal is something like 
[this](https://html5up.net/upload/demos/eventually/) (obviously we can repurpose this template to accept images, and not emails).

__Note__: [License](http://html5up.net/license) for any template from [html5up](http://html5up.net/)

```python
from django.shortcuts import render, HttpResponse, HttpResponseRedirect
from django.views.generic import View, TemplateView

# Create your views here.

class IndexUploadView(TemplateView):
    template_name = "upload/index.html"
```

## 10. Download Template to `upload\templates\upload\index.html`

Yes, `upload` appears twice in that path, that is not a typo.

> "Now we might be able to get away with putting our templates directly in [upload]/templates (rather than creating another polls subdirectory), but it would actually be a bad idea. Django will choose the first template it finds whose name matches, and if you had a template with the same name in a different application, Django would be unable to distinguish between them. We need to be able to point Django at the right one, and the easiest way to ensure this is by namespacing them. That is, by putting those templates inside another directory named for the application itself."

> [- Source](https://docs.djangoproject.com/en/1.10/intro/tutorial03/)

You should also update to use `{% static %}` tags. So your code should look something like this.

```html
{% load static %}
<!DOCTYPE HTML>
<!--
	Eventually by HTML5 UP
	html5up.net | @ajlkn
	Free for personal and commercial use under the CCA 3.0 license (html5up.net/license)
-->
<html>
	<head>
		<title>Upload an Image!</title>
		<meta charset="utf-8" />
		<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no" />
		<link rel="stylesheet" href="{% static "upload/assets/css/main.css" %}" />
	</head>
	<body>
		<!-- Header -->
			<header id="header">
				<h1>Microsoft Cognitive Face API</h1>
				<p>Use the Microsoft Face recognition API to seperate your images with faces and without faces.</p>
				
			</header>
		<!-- Signup Form -->
			<form id="upload-form" method="post" action="/upload/image/" enctype="multipart/form-data">
				{% csrf_token %}
				<input type="file" name="imageupload" id="imageupload" />
				<input type="submit" value="Upload Image" name="submit" />
			</form>
		<!-- Footer -->
			<footer id="footer">
				<ul class="icons">
					<li><a href="https://github.com/qwergram/CortanaVisionCSA" class="icon fa-github"><span class="label">GitHub</span></a></li>
					<li><a href="https://azure.microsoft.com/" class="icon fa-windows"><span class="label">Microsoft Azure</span></a></li>
				</ul>
				<ul class="copyright">
					<li>&copy; Norton Pengra</li><li>Design: <a href="http://html5up.net">HTML5 UP</a></li>
					<li><a href="/">Upload</a></li><li><a href="/lib/unsorted">Unsorted Images</a></li><li><a href="/lib/noface">Images without Faces</a></li><li><a href="/lib/faces">Images with Faces</a></li>

				</ul>
			</footer>
		<!-- Scripts -->
			<script src="{% static "upload/assets/js/main.js" %}"></script>
	</body>
</html>
```

## 11. Download Static Files to `upload\static\upload\`

Create a directory called `static` and under it create another one called `upload`.
From the [download](https://html5up.net/eventually/download), open the zip and copy the directories `assets\` and `images\`
to `upload\static\upload\`.

![](http://i.imgur.com/qN0vonU.png)

__Note__: [Link](http://aka.ms/CnESymbols) if you're interested in symbols.

## 12. Update `upload\urls.py` to show template

```python
from django.conf.urls import include, url

from upload.views import IndexUploadView    # < New Import

urlpatterns = [
    url(r'^$', IndexUploadView.as_view()),  # < New route
]
```

## 13. Collect Static Files

If you try to run your server as is, None of the static files will show.

![](http://i.imgur.com/HT4FdMw.png)

You will need to Collect Static Files. Right click your project and you should be able to go to `Python > Collect Static Files`

![](http://i.imgur.com/RexhhHp.png)

Your output show show something like this:

```
Python 3.5 interactive window [PTVS 2.2.40623.00-14.0]
Type $help for a list of commands.
Executing manage.py collectstatic --noinput
Copying 'c:\ ... \base.css'

...

XX static files copied to 'c:\ ... \static'.
The Python REPL process has exited
```

If you run the server now, you'll be able to see a lovely website.

![](http://i.imgur.com/TdHC1pV.png)

## 14. Create Upload Form

Create another emtpy python file again and name it `forms.py` under `upload\`

![](http://i.imgur.com/ysoMXIc.png)

In it, define what parameters you want the user to be able to upload, in this case we want the user to be able to upload
an image.

```python
from django import forms

class UploadImageForm(forms.Form):
    imageupload = forms.ImageField()
```

## 15. Install required packages

In this app, we're going to use the following python libraries:

- `requests`
- `azure`
- `Pillow`

Right click `env` and select `Install Python Package...`

![](http://i.imgur.com/nKUgzaW.png)


![](http://i.imgur.com/XedRsoh.png)


![](http://i.imgur.com/B2G3RTa.png)

Install both `requests` and `azure`.

__Note__: If your `azure-mgmt-reqource` is not the latest version:

> Sometimes when installing `azure` with `pip`, the latest version is not installed. To fix this,
right click the `azure` package and install the latest version. At the time of
this writing, the version is `2.0.0rc5`.

> Right click the package under `env` and find `azure-*` and remove it.
Then re-install it using the string `azure==2.0.0rc5`.

Once you're done installing the packages, generate `requirements.txt` for the Cloud Service VMs.

![](http://i.imgur.com/MJU80Uj.png)


## 16. Connect with Azure SDK

in `forms.py`, add an ImageQueue object

```python
from django import forms
import json
from azure.storage.blob import BlockBlobService
from azure.storage.queue import QueueService

from upload.secrets import ACCOUNT_NAME, ACCOUNT_KEY # This doesn't exist yet

class UploadImageForm(forms.Form): ...

class ImageQueue(object):

    queue_name = "imagesqueue"
    container_name = "unsorted-images"

    def __init__(self, account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY):
        self.account_key = account_key
        self.account_name = account_name
        self.init_queue()
        self.init_blob()


    def init_blob(self):
        self.blob = BlockBlobService(account_name=self.account_name, account_key=self.account_key)
        self.blob.create_container(self.container_name)


    def init_queue(self):
        self.queue = QueueService(account_name=self.account_name, account_key=self.account_key)
        self.queue.create_queue(self.queue_name)

    def new_image(self, django_image):
        self.blob.create_blob_from_bytes(self.container_name, django_image['imageupload'].name, django_image['imageupload'].read())
        contents = json.dumps({
            "name": django_image['imageupload'].name,
            "blobname": self.account_name,
            "containername": self.container_name,
        })
        self.queue.put_message(self.queue_name, contents)
```

## 17. Create Storage for Cloud App

Go onto [Azure](http://portal.azure.com), and create a Storage Container.
![](http://i.imgur.com/dHYT1FN.png)

__Note__: Select Resource Manager, not classic.

![](http://i.imgur.com/PtmzJpT.png)

__Note__: Check out resources to see that it actually deployed

![](http://i.imgur.com/VdQwUAx.png)

## 18. Copy Connection Strings

From the Azure Portal, select Access Keys and copy the "Storage account name" and "key1".

![](http://i.imgur.com/qoAzTpf.png)

Then create another file under `upload\` called `secrets.py`

__Note__: Make sure you `.gitignore` this file.

Under `secrets.py` add the following symbols:

```python
# Azure Secrets! GITIGNORE THIS.
ACCOUNT_NAME = "<Your storage account name>"
ACCOUNT_KEY = "<88 digit key1 or key2>"
```

## 19. Create `POST` upload view

Now we actually write the view for processing uploading. In `upload\views.py` add these lines:

```python
...
from upload.forms import UploadImageForm, ImageQueue

# Create your views here.

...

class IndexUploadPost(View):

    def post(self, request):
        "Handle Images being uploaded and push them to Queue and Storage Blob"
        form = UploadImageForm(request.POST, request.FILES)
        if form.is_valid():
            IQ = ImageQueue()
            IQ.new_image(request.FILES)
            return HttpResponseRedirect('/?success=true')
        else:
            return HttpResponseRedirect('/?success=false')
```

## 20. Update `upload/urls.py`

`urlpatterns` should handle post requests like so:

```python
from django.conf.urls import include, url

from upload.views import IndexUploadView, IndexUploadPost

urlpatterns = [
    url(r'^$', IndexUploadView.as_view()),
    url(r'upload/images/$', IndexUploadPost.as_view()),
]
```

## 21. Test uploading a file
Try uploading a file, and once you do. You should see the url bar indicate `/?success=true`.
Once that occurs, goto your [azure portal](http://portal.azure.com) and look at the storage account
you grabbed your access keys from.

![](http://i.imgur.com/F1yay6R.png)
If you open the container `unsorted-images` you should see the image name that you uploaded.

## 22. Create bins for images with faces and images without faces

We're going to create two more containers called `no-face-images` and `has-face-images`. Make sure to set the access
type as "Blob".

![](http://i.imgur.com/8FNmASL.png)

In addition, change the existing `unosrted-images` Access Policy settings to Blob as well.

![](http://i.imgur.com/FIjavKi.png)

## 23. Create views for each bin

We're finished with the `upload` app, and now we can start to build our `library` app. In `library\views.py`,
define a view for each bin. But first, we need to grab all the images from the bin and return an array containing
those images.

```python

from django.shortcuts import render
from django.views.generic import TemplateView
# You can import from upload, or you can create a new secrets.py in this app directory and point it there
from upload.secrets import ACCOUNT_KEY, ACCOUNT_NAME

from azure.storage.blob import BlockBlobService

# Create your views here.

def get_urls_from_container(containername):
    blob = BlockBlobService(account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY)
    return ["https://{}.blob.core.windows.net/{}/{}".format(ACCOUNT_NAME, containername, x.name) for x in blob.list_blobs(containername)]
```

With that in place, you can now add the views:

```python
class UnsortedImageView(TemplateView):
    template_name = "library/index.html"  # Doesn't exist yet.

    def get_context_data(self, *args, **kwargs):
        context = super(UnsortedImageView, self).get_context_data(*args, **kwargs)
        context['image_list'] = get_urls_from_container('unsorted-images')
        context['title'] = "Unsorted Images"
        return context


class FaceImageView(TemplateView):
    template_name = "library/index.html"  # Doesn't exist yet.

    def get_context_data(self, *args, **kwargs):
        context = super(FaceImageView, self).get_context_data(*args, **kwargs)
        context['image_list'] = get_urls_from_container('has-face-images')
        context['title'] = "Images with Faces"
        return context


class NoFaceImageView(TemplateView):
    template_name = "library/index.html"  # Doesn't exist yet.

    def get_context_data(self, *args, **kwargs):
        context = super(NoFaceImageView, self).get_context_data(*args, **kwargs)
        context['image_list'] = get_urls_from_container('no-face-images')
        context['title'] = "Images without Faces"
        return context
```

## 24. Create `library\templates\library\index.html`

I quickly whipped up a bootstrap template for this view. Here's the code behind it.
The only important aspects about it is the `{% for image in image_list %}{% endfor %}`, `{{title}}`
and the links.

```html
<html>
<head>
    <title>{{title}}</title>
    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" />
    <style>
        body {
            padding-top: 50px;
            padding-bottom: 20px;
        }
    </style>
</head>
<body>
    <nav class="navbar navbar-inverse navbar-fixed-top">
      <div class="container">
        <div class="navbar-header">
          <button type="button" class="navbar-toggle collapsed" data-toggle="collapse" data-target="#navbar" aria-expanded="false" aria-controls="navbar">
            <span class="sr-only">Toggle navigation</span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
            <span class="icon-bar"></span>
          </button>
          <a class="navbar-brand" href="#">Microsoft Face Recognition API</a>
        </div>
      </div>
    </nav>
    <div class="jumbotron">
      <div class="container">
        <h1>{{ title }}</h1>
        <p>
            <a href="/">Upload</a> | <a href="/lib/unsorted">Unsorted Images</a> | <a href="/lib/noface">Images without Faces</a> | <a href="/lib/faces">Images with Faces</a>
        </p>
      </div>
    </div>
    <div class="container">
        {% for image in image_list %}
            <div class="col-md-4">
                <img src="{{image}}" width="100%">
            </div>
        {% endfor %}
    </div>
    <div class="container">
        <hr>
        <footer>
            <p>&copy; 2016 Microsoft, Inc.</p>
        </footer>
    </div>
</body>
</html>
```

## 25. Create route in `library\urls.py`

Same as we've done with `upload`, create a `urls.py` and fill it with the right routes.

```python
from django.conf.urls import include, url

from library.views import FaceImageView, UnsortedImageView, NoFaceImageView

urlpatterns = [
    url(r'faces/$', FaceImageView.as_view()),
    url(r'unsorted/$', UnsortedImageView.as_view()),
    url(r'noface/$', NoFaceImageView.as_view()),
]
```

You will also need to update `Django_Web_Role\urls.py`

```python
from django.conf.urls import include, url

import upload.urls
import library.urls

urlpatterns = [
    url(r'^<your_path_here>/', include(upload.urls)),
    url(r'^<your_path_here>/', include(library.urls)),
]
```

In my case, I just set them both as root:

```python
urlpatterns = [
    url(r'^', include(upload.urls)),
    url(r'^lib/', include(library.urls)),
]
```

__Congratulations, if you launch your website, you should have a completed WebRole!__


# Building the Cloud Service App: Worker Role

The worker role will pull images from the queue that the webrole uploads and will
use the Cortana Vision API to see if there are faces in the image. 

## 26. Open `Worker Role\worker.py` and delete everything in it

![](http://i.imgur.com/9ANc5zK.png)

## 27. Create a `secrets.py`

![](http://i.imgur.com/DWyVGCy.png)

and name it `secrets.py`.

## 28. Add account secrets

Copy the `secrets.py` you made in your web role and put the keys in your worker role.

```python
# Azure Secrets!
ACCOUNT_NAME = "<storage account name>"
ACCOUNT_KEY = "<88 char account key>"
```

## 29. Sign up for cognitive services on azure

On [azure portal](https://portal.azure.com), create a new "Cognitive Sercices API" and select "Face API (preview)"

![](http://i.imgur.com/tHEEcTx.png)


## 30. Get Subscription keys

Open up the resource group that cognitive service was deployed on and navigate to the keys

![](http://i.imgur.com/gOYCOAp.png)

![](http://i.imgur.com/xkw3d6p.png)

Copy it and paste it into `secrets.py`

```python
# Azure Secrets!
ACCOUNT_NAME = "<storage account name>"
ACCOUNT_KEY = "<88 char account key>"

COG_ACCOUNT_KEY = "<your key here>"
```

## 31. Copy image queue from `web role\upload\forms.py`

Start out with the ImageQueue we used in our webrole and copy that into `worker.py`.

We can delete `new_image` because the worker will never add images, but it needs to 
pull images. So this is what it will look like:

```python

from secrets import ACCOUNT_KEY, ACCOUNT_NAME, COG_ACCOUNT_KEY


class ImageQueue(object):

    queue_name = "imagesqueue"
    container_name = "unsorted-images"
    face_container = "has-face-images"
    no_face_container = "no-face-images"
    

    def __init__(self, account_name=ACCOUNT_NAME, account_key=ACCOUNT_KEY, workerrole=False):
        self.account_key = account_key
        self.account_name = account_name
        self.workerrole = workerrole
        self.last_image = None
        self.init_queue()
        self.init_blob()

    def init_blob(self):
        self.blob = BlockBlobService(account_name=self.account_name, account_key=self.account_key)
        self.blob.create_container(self.container_name)
        self.blob.create_container(self.face_container)
        self.blob.create_container(self.no_face_container)

    def init_queue(self):
        self.queue = QueueService(account_name=self.account_name, account_key=self.account_key)
        self.queue.create_queue(self.queue_name)

    def get_image(self):
        try:
            message = self.queue.get_messages(self.queue_name, num_messages=1)[0]
        except IndexError:  # there is no message
            return {}
        self.queue.delete_message(self.queue_name, message.id, message.pop_receipt)
        
        if message.content == "test":
            return {}
        message_content = json.loads(message.content)
        self.last_image = message_content
        return message_content

    def upload_image_to_face_container(self, container_name, path):
        if self.last_image is None: return None
        new_name = str(time()).replace('.', '') + '.' + path.split('.')[-1]
        self.blob.create_blob_from_path(container_name, new_name, path)
        self.delete_last_image()

    # http://goo.gl/XN9zkJ :)
    def move_image_to_no_face_container(self):
        if self.last_image is None: return None
        new_name = str(time()).replace('.', '') + '.' + self.last_image['name'].split('.')[-1]
        blob_url = self.blob.make_blob_url(self.last_image['containername'], self.last_image['name'])
        self.blob.copy_blob(self.no_face_container, new_name, blob_url)
        self.delete_last_image()

    def delete_last_image(self):
        self.blob.delete_blob(self.last_image['containername'], self.last_image['name'])
        self.last_image = None

    def __len__(self):
        return self.queue.get_queue_metadata(self.queue_name).approximate_message_count
```

## 31. Create a simple API wrapper

In `worker.py` add a wrapper for our Face API

```python
...

import requests

class ImageQueue(object): ...

class CognativeServicesWrapper(object):

    api_endpoint = "https://api.projectoxford.ai/face/v1.0/detect?returnFaceId=true&returnFaceLandmarks=true&returnFaceAttributes=age,gender,smile,facialHair,headPose,glasses"
    api_key = COG_ACCOUNT_KEY

    def __init__(self, image_dict):
        try:
            assert isinstance(image_dict, dict)
            assert 'name' in image_dict.keys()
            assert 'blobname' in image_dict.keys()
            assert 'containername' in image_dict.keys()
        except AssertionError as e:
            raise ValueError(e)

        self.image_target = self.to_uri(image_dict)

    def to_uri(self, image_dict):
        return "https://{}.blob.core.windows.net/{}/{}".format(image_dict['blobname'], image_dict['containername'], image_dict['name'])

    def hit_api(self):
        post_data = json.dumps({"url": self.image_target})
        header_data = {
            "Ocp-Apim-Subscription-Key": self.api_key
        }
        response = requests.post(self.api_endpoint, data=post_data, headers=header_data)
        return response.json()
```

## 32. Change image if face detected

```python
def show_faces(image_target, results):
    image = requests.get(image_target, stream=True).raw
    image_ext = image_target.split('.')[-1]
    image_path = "image.{}".format(image_ext) 
    with open(image_path, "wb") as context:
        image.decode_content = True
        shutil.copyfileobj(image, context)
    im_obj = Image.open(image_path)
    pen = ImageDraw.Draw(im_obj)

    for person in results:
        height = person['faceRectangle']['height']
        width = person['faceRectangle']['width']
        left = person['faceRectangle']['left']
        top = person['faceRectangle']['top']
        print(person['faceRectangle'], person['faceAttributes']['gender'], person['faceAttributes']['age'])
        pen_color = "blue" if person['faceAttributes']['gender'] == "male" else "pink"
        pen.rectangle([left, top, left+ width, top + height], outline = pen_color)
        for point in person['faceLandmarks'].values():
            x, y = point['x'], point['y']
            pen.ellipse([x - 1, y - 1, x + 1, y + 1], fill = pen_color)

    # im_obj.show()  # This will open microsoft paint or image viewer
    im_obj.save('image.png', 'png')
```

## 33. Create \_\_main__

```python

def process(imagequeue):
    image = imagequeue.get_image()
    print(image)
    if image:
        COG = CognativeServicesWrapper(image)
        image_target = COG.image_target
        results = COG.hit_api()
        if len(results):
            print("Moving to face")
            show_faces(image_target, results)
            imagequeue.upload_image_to_face_container(imagequeue.face_container, "image.png")
        else:
            print("Moving to noface")
            imagequeue.move_image_to_no_face_container()
        sleep(1)
    else:
        sleep(5)


if __name__ == '__main__':
    IQ = ImageQueue()
    while True:
        process(IQ)
```

If you run this, you should be able to upload images to your web role, and your worker role will
sort the images accordingly.

# Deploying your Roles

In IIS, the "right" approach would be configuring [fcgi to work with Django](https://www.toptal.com/django/installing-django-on-iis-a-step-by-step-tutorial). 
Another approach would be to do a [reverse proxy](https://www.nginx.com/resources/admin-guide/reverse-proxy/), which requires a little more
work in IIS than nginx. 

First of all, note that the scripts that Visual Studio generates (as of Sept 2016) are all "suggestions" and don't
work as of right now. Therefore under `DjangoWebRole\bin\` you'll find `ps.cmd` and `ConfigureCloudService.ps1`. 
Open up the powershell script and delete everything inside of it.


## 34. Write Deployment Script for webrole

You can then copy the following powershell script and place it in `DjangoWebRole\bin\ConfigureCloudService.ps1`.

The following code will accomplish the following:
### Get location of approot
Whenever you push your code to Azure via webdeploy, Azure will put all your files on a vhd and swap
the existing vhd on that machine with the new code. Therefore your code's location will alternate
between `E:\` and `F:\`. 

### Download Python
### Install Web Package Installer
### Install ARR with Web Package Installer
ARR is what allows a reverse proxy, which allows us to run the django server on
`localhost` and we can serve it to the public with a stronger web server, much like nginx.
### Create a new Task with `schtasks`
You'll want your local django server to run on boot up. So we'll schedule a task to do it.


```powershell
# Get location of approot
if (Test-Path "E:\approot\") {
    $approot = "E:\approot"
} elseif (Test-Path "F:\approot\") {
    $approot = "F:\approot"
} else {
    throw "approot not found"
}

# Get Task XML
$scheduled_task = "<?xml version=`"1.0`" encoding=`"UTF-16`"?>
<Task version=`"1.4`" xmlns=`"http://schemas.microsoft.com/windows/2004/02/mit/task`">
  <RegistrationInfo>
    <Date>2016-09-06T20:27:10.2939543</Date>
    <Author>Norton Pengra</Author>
  </RegistrationInfo>
  <Triggers>
    <BootTrigger>
      <Enabled>true</Enabled>
    </BootTrigger>
  </Triggers>
  <Principals>
    <Principal id=`"Author`">
      <UserId>S-1-5-18</UserId>
      <RunLevel>LeastPrivilege</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
    <UseUnifiedSchedulingEngine>false</UseUnifiedSchedulingEngine>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>PT0S</ExecutionTimeLimit>
    <Priority>7</Priority>
    <RestartOnFailure>
      <Interval>PT1M</Interval>
      <Count>3</Count>
    </RestartOnFailure>
  </Settings>
  <Actions Context=`"Author`">
    <Exec>
      <Command>C:\python\python.exe</Command>
      <Arguments>$approot\manage.py runserver 127.0.0.1:8080</Arguments>
    </Exec>
  </Actions>
</Task>"



try {
    python -c "print('hello world')" -ErrorAction Stop
} catch {
    # Install Python
    Invoke-WebRequest "https://www.python.org/ftp/python/3.5.2/python-3.5.2.exe" -OutFile "install_python.exe"
    .\install_python.exe /quiet InstallAllUsers=1 PrependPath=1 DefaultAllUsersTargetDir="C:\\python\\"
    cmd /c "C:\python\Scripts\pip.exe install -r $approot\requirements.txt"
}

if (Test-Path "$env:ProgramFiles\microsoft\web platform installer\WebpiCmd-x64.exe") {
    cmd /c "`"$env:ProgramFiles\microsoft\web platform installer\WebpiCmd-x64.exe`" /Install /Products:ARR /accepteula"
} else {
    # Install webpi
    Invoke-WebRequest "https://go.microsoft.com/fwlink/?linkid=226239" -OutFile "install_webpi.msi"
    msiexec /i install_webpi.msi /quiet ADDLOCAL=ALL
    # Install ARR
    cmd /c "`"$env:ProgramFiles\microsoft\web platform installer\WebpiCmd-x64.exe`" /Install /Products:ARR /accepteula"
}

# Launch webserver everytime on boot up
$scheduled_task | Out-File "C:\schedule.xml" -Encoding ascii
schtasks -Delete -TN "WebServer" /F
taskkill /IM python.exe /F
schtasks -Create -XML "C:\schedule.xml" -TN "WebServer"
schtasks -Run -TN "WebServer"
```

## 35. Write Deployment Script for Worker role

The same script can be applied, however parts can be excluded, like the ARR setup.
Under `Worker Role\bin\ConfigureCloudService.ps1`, you can paste this script in.
The other difference is the schedule xml blob calls `worker.py` instead of `manage.py`.

```powershell
# Get location of approot
if (Test-Path "E:\approot\") {
    $approot = "E:\approot"
} elseif (Test-Path "F:\approot\") {
    $approot = "F:\approot"
} else {
    throw "approot not found"
}

# Get Task XML
$scheduled_task = "<?xml version=`"1.0`" encoding=`"UTF-16`"?>
<Task version=`"1.4`" xmlns=`"http://schemas.microsoft.com/windows/2004/02/mit/task`">
  <RegistrationInfo>
    <Date>2016-09-06T20:27:10.2939543</Date>
    <Author>Norton Pengra</Author>
  </RegistrationInfo>
  <Triggers>
    <BootTrigger>
      <Enabled>true</Enabled>
    </BootTrigger>
  </Triggers>
  <Principals>
    <Principal id=`"Author`">
      <UserId>S-1-5-18</UserId>
      <RunLevel>LeastPrivilege</RunLevel>
    </Principal>
  </Principals>
  <Settings>
    <MultipleInstancesPolicy>IgnoreNew</MultipleInstancesPolicy>
    <DisallowStartIfOnBatteries>true</DisallowStartIfOnBatteries>
    <StopIfGoingOnBatteries>true</StopIfGoingOnBatteries>
    <AllowHardTerminate>true</AllowHardTerminate>
    <StartWhenAvailable>false</StartWhenAvailable>
    <RunOnlyIfNetworkAvailable>false</RunOnlyIfNetworkAvailable>
    <IdleSettings>
      <StopOnIdleEnd>true</StopOnIdleEnd>
      <RestartOnIdle>false</RestartOnIdle>
    </IdleSettings>
    <AllowStartOnDemand>true</AllowStartOnDemand>
    <Enabled>true</Enabled>
    <Hidden>false</Hidden>
    <RunOnlyIfIdle>false</RunOnlyIfIdle>
    <DisallowStartOnRemoteAppSession>false</DisallowStartOnRemoteAppSession>
    <UseUnifiedSchedulingEngine>false</UseUnifiedSchedulingEngine>
    <WakeToRun>false</WakeToRun>
    <ExecutionTimeLimit>PT0S</ExecutionTimeLimit>
    <Priority>7</Priority>
    <RestartOnFailure>
      <Interval>PT1M</Interval>
      <Count>3</Count>
    </RestartOnFailure>
  </Settings>
  <Actions Context=`"Author`">
    <Exec>
      <Command>C:\python\python.exe</Command>
      <Arguments>$approot\worker.py</Arguments>
    </Exec>
  </Actions>
</Task>"



try {
    python -c "print('hello world')" -ErrorAction Stop
} catch {
    # Install Python
    Invoke-WebRequest "https://www.python.org/ftp/python/3.5.2/python-3.5.2.exe" -OutFile "install_python.exe"
    .\install_python.exe /quiet InstallAllUsers=1 PrependPath=1 DefaultAllUsersTargetDir="C:\\python\\"
    cmd /c "C:\python\Scripts\pip.exe install -r $approot\requirements.txt"
}

# Launch workerrole everytime on boot up
$scheduled_task | Out-File "C:\schedule.xml" -Encoding ascii
schtasks -Delete -TN "WorkerRole" /F
taskkill /IM python.exe /F
schtasks -Create -XML "C:\schedule.xml" -TN "WorkerRole"
schtasks -Run -TN "WorkerRole"
```

## 36. Right click the Cloud and select Publish

![](http://i.imgur.com/JzE1Q2U.png)

## 37. Select Subscription and Account
![](http://i.imgur.com/CC3dItH.png)

## 38. Create new Cloud Service App
![](http://imgur.com/lciul3N.png)

![](http://imgur.com/yTO68vz.png)

## 39. Finish deployment

![](http://imgur.com/107mv6E.png)

![](http://imgur.com/8mggJ2n.png)

## 40. See deployment in action

![](http://imgur.com/HaIwPFQ.png)

## 41. RDC into Web Role
Open up your azure portal and open your newly deployed cloud app.

![](http://i.imgur.com/qi8yYQA.png)

## 42. Open `inetmgr` and navigate to Url Rewrite

![](http://i.imgur.com/EkC0Isd.png)

![](http://i.imgur.com/nWzp8oS.png)

## 43. Delete all existing rules and create a new one

![](http://imgur.com/zJa7FSq.png)

![](http://imgur.com/FpLWvs9.png)

![](http://imgur.com/ZWpvg4o.png)

## 44. View your site

![](http://imgur.com/Mukt3hk.png)

![](http://imgur.com/YjQCwxD.png)