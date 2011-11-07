======================
django-inline-ordering
======================

This app aims at easing implementing drag & drop reordering of inline data in 
Django Admin. 

Recent version of `Grappelli <http://code.google.com/p/django-grappelli/>`_ includes 
drag and drop inlines, so if you are already using Grappelli, installing 
django-inline-ordering does not make sense. If you didn't evaluate Grappelli yet,
I encourage you to do so!

The django-inline-ordering is being tested during development on: 

- Fx 3.5+
- Google Chrome 10+
- Opera 11+ 

Suggestions on how to improve django-inline-ordering are very welcome.

Installation
------------

1. Put 'dragorder' on your python path

2. Add 'dragorder' to INSTALLED_APPS tuple in settings file 

Usage
-----

For each model that you want to be reorderable with drag & drop, you need to 
setup an admin class and slightly modify model declaration. This way you can 
easily apply reordering to existing model classes. You'll also have to run 
``collectstatic`` task or copy one file to your MEDIA_ROOT when using Django 
< 1.3.

Let's start with a simple example: Gallery has many Images. User might 
want to reorder the photos in the gallery to fit his likings.

1. Setup your admin inline class to inherit after OrderableStackedInline
   
   ::
     
     from dragorder.admin import OrderableStackedInline
     
     class ImageInline(OrderableStackedInline):
    
       model = Image 
 
2. Add jQuery and jQuery.ui to the global Javascript namespace

   ::
     
     class GalleryAdmin(ModelAdmin):
         
         model = Gallery
         inlines = (ImageInline, )
    
         class Media:
             js = (
                 'http://ajax.googleapis.com/ajax/libs/jquery/1/jquery.min.js',
                 'http://ajax.googleapis.com/ajax/libs/jqueryui/1/jquery-ui.min.js',
             )
     
   Altough this could be done automatically, some widgets like for 
   example the WYMEditor widget, use their own way of loading jQuery UI, which 
   could lead to conflicts with inline-ordering. 

3. Setup your model to inherit after Orderable
   
   ::
   
     from dragorder.models import Orderable
     
     class Image(Orderable):
       src = models.ImageField(upload_to='gallery/images')
       title = models.CharField(max_length=255)
       alt = models.TextField()
     
     class Meta(Orderable.Meta):
       pass
    
   As ordering column was named ``dragorder_position``, avoid using
   this name in your model.

   The Meta class declaration is NOT necessary - add it only if you need to set
   your own meta attributes. 
    
4. Make ``dragorder.js`` accessible over HTTP

   Since Django 1.3, there is a `staticfiles app`_ that django-inline-ordering is 
   aware of. Just run ``manage.py collectstatic`` to copy/symlink media files.
   
.. _staticfiles app: http://docs.djangoproject.com/en/1.3/ref/contrib/staticfiles/

   The simplest way, back in Django 1.2, was to copy 
   ``media/dragorder.js`` to your ``MEDIA_ROOT``.

   If you however believe that would make a mess, take advantage of the 
   'INLINE_ORDERING_JS' setting and set it to a location which should be requested 
   when accessing orderable inlines:

   ``INLINE_ORDERING_JS = STATIC_URL + '/js/third_party/dragorder.js'``
  
Known issues
------------

1. Reordering won't work for new records until saved. This needs a onchange 
   handler for the record form or some model refactoring. 

2. At this point there is no support for tabular inlines. If you would like to 
   approach this problem, fork the project on Github.

Development
-----------

A simple test project has been added in fdc2189 under tests/testproject. Use 
tools/buildenv to build a virtualenv for development and syncdb to create necessary
models and admin permissions. No unit tests at this point as this is a browser-side 
thing mostly, improvements welcome.

Kudos
-----
simon for http://djangosnippets.org/snippets/1053/
