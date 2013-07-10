----------------------------------------------------
Django-JFU - A Django Library for jQuery File Upload 
----------------------------------------------------

Django-JFU is designed to simplify the tasks involved in integrating jQuery File
Upload into Django. Django-JFU assumes very little and leaves the model/view
design up to the user. 

Other Django - jQuery File Upload implementations are full-featured but
generally serve more as demonstrations than libraries for existing
applications.

If you seek a tool to ease the integration of jQuery File Upload into your
Django application while still having a great degree of freedom, you may find
this package useful.

Installation
------------

1. `pip install django-jfu`.
2. Add 'jfu' to `INSTALLED_APPS` in your project `settings.py` file.
3. Add 'django.core.context_processors.request' to `TEMPLATE_CONTEXT_PROCESSORS` in settings.py as well.
4. Run `python manage.py collectstatic`.


Usage
-----

Django-JFU provides simple customizable template tags and override-able templates that
do the work of integrating the jQuery File Upload CSS and JavaScipt and the
HTML implementation found in the jQuery File Upload demo.

To place the jQuery File Upload widget in a template, simply insert the
following within it::
    
    {% load jfutags %}
    {% jfu %}

Then create a view that will handle the uploaded files. By default, the
URL for the view must be named **'jfu_upload'**.

Here is an example implementation:

In your `urls.py` file::

    ...
    url( r'upload/', views.upload, name = 'jfu_upload' ),

    # You may optionally define a delete url as well
    url( r'^delete/(?P<pk>\d+)$', views.upload_delete, name = 'jfu_delete' ),

In your `views.py` file::

    import settings
    from django.views.decorators.http import require_POST
    from jfu.http import upload_receive, UploadResponse, Django-JFUResponse

    @require_POST
    def upload( request ):

        # The assumption here is that jQuery File Upload 
        # has been configured to send files one at a time.
        # If multiple files can be uploaded simulatenously,
        # 'file' will be a list of files.

        file = upload_receive( request )

        # Assuming file_field is a FileField that saves to the 'media' directory.
        instance = YourUploadModel( file_field = file )
        instance.save()
        
        file_dict = {
            'name' : instance.file_field.file.name,
            'size' : instance.file_field.file.size,
            'url': settings.MEDIA_URL + instance.file.file_field.name,
            'thumbnail_url': settings.MEDIA_URL + instance.file.file_field.name,
            'delete_url':  reverse('jfu_delete', kwargs = { 'pk': instance.pk } ),
            "delete_type": "POST",
        }

        return UploadResponse( request, file_dict )

    @require_POST
    def upload_delete( request, pk ):
        # An example implementation.
        success = True
        try:
            YourUploadModel.objects.get( pk = pk ).delete()
        except YourUploadModel.DoesNotExist:
            success = False

        return Django-JFUResponse( request, success )


Customization
-------------

Django-JFU is designed to be very customizable.  

The Django-JFU template tag optionally takes two arguments: the name of the
template to load and the name of the URL pointing to the upload-handling
view.::

    {% load jfutags %}
    {% jfu 'your_fileuploader.html' 'your_uploader' %}

A custom template can extend from the master jfu template
`jfu/upload_form.html`.  There are several blocks which may be overriden for
the purpose of customization:

* JS_OPTS - The options supplied to the jQuery File Upload `fileupload` function. 
* JS_INIT - The initializing JavaScript
* JS_FORM - Loads existing files.
* FILE_INPUT - The file input for the upload form.

The blocks above are most-likely what you will want to override when seeking to
customize. For instance, one would go about adding a few options to the
fileupload function in this manner::

    # your_fileuploader.html
    {% extends 'jfu/upload_form.html' %}
    
    {% block JS_OPTS %}
    autoUpload: true,
    maxNumberOfFiles: 5,
    sequentialUploads: true,
    {% endblock %}

There are several other blocks too:


HTML Components
===============

* MODAL_GALLERY - The modal gallery
* UPLOAD_FORM - The file upload form used as target for the file upload widget.
* UPLOAD_FORM_LISTING - The table listing the files available for upload/download.
* UPLOAD_FORM_LINDICATOR - The loading indicator shown during file processing.
* UPLOAD_FORM_PROGRESS_BAR - The global progress information.
* UPLOAD_FORM_BUTTON_BAR - The fileupload-buttonbar contains buttons to add/delete files and start/cancel the upload.
    * UPLOAD_FORM_BUTTON_BAR_CONTROL 
    * UPLOAD_FORM_BUTTON_BAR_ADD 

CSS Components
==============

* CSS_BOOTSTRAP 
* CSS_BLUEIMP_GALLERY 
* CSS_JQUERY_FILE_UPLOAD_UI
* CSS_HTML5_SHIM 

JS Components
=============

* JS_DIALOG 
* JS_DOWNLOAD_TEMPLATE 
    * JS_DOWNLOAD_TEMPLATE_DELETE 
    * JS_DOWNLOAD_TEMPLATE_DOWNLOAD  
    * JS_DOWNLOAD_TEMPLATE_PREVIEW 
    * JS_DOWNLOAD_TEMPLATE_ERROR 
    * JS_DOWNLOAD_TEMPLATE_FSIZE 
* JS_UPLOAD_TEMPLATE 
* JS_JQUERY 
* JS_JQUERY_UI_WIDGET
* JS_TEMPLATES 
* JS_LOAD_IMAGE
* JS_CANVAS_TO_BLOB 
* JS_BOOTSTRAP 
* JS_BLUEIMP_GALLERY 
* JS_BOOTSTRAP_IFRAME_TRANSPORT
* JS_JQUERY_FILE_UPLOAD
* JS_JQUERY_FILE_UPLOAD_FP
* JS_JQUERY_FILEUPLOAD_UI 


The included JavaScript and CSS can be updated or suppressed by overriding these blocks ::

    # your_fileuploader.html
    {% extends 'jfu/upload_form.html' %}

    {% block JS_JQUERY %}
        <script src={{STATIC_URL}}/js/my.newer.jquery.js />
    {% endblock %}

    {% block CSS_BOOTSTRAP %}
        {% comment %}
        This is already included.
        {% endcomment %}
    {% endblock %}


Contribution
------------           
Django-JFU is wholly open source and welcomes contributions of any kind.  Feel
free to either extend it, report bugs, or provide suggestions for improvements.
The author can be contacted at alem@cidola.com