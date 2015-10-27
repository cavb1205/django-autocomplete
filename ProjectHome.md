# AutoComplete for ForeignKey and ManyToManyField #
uses same syntax as the search for django-admin

up:
the latest version in Downloads

  1. - fixed compatibility with Django 1.3
  1. - cleaned java scripts
  1. - added support for inline AutocompleteStackedInline, AutocompleteTabularInline
  1. - added a field WildModelSearchInput, it can be used in the admin forms

## requirements ##
> copy to the appropriate folder
  1. media\admin\js\jquery.autocomplete.js
  1. media\jquery.autocomplete.css

## example ##
![http://s59.radikal.ru/i165/0902/07/dbfae244c7dc.png](http://s59.radikal.ru/i165/0902/07/dbfae244c7dc.png)

## use a fairly simple ##

### models.py ###
**in m2m field need to specify related\_name**

```
class Celebrity(models.Model):
	name = models.CharField()

class Film(models.Model):
	type	= models.ForeignKey( Type )
	director= models.ManyToManyField( Celebrity, related_name="director")
	actor	= models.ManyToManyField( Celebrity, related_name="actor")
```
**NEW**: for inline
```
class Ceremony( PageSeo ):
	name	=models.CharField( max_length = 200 )
	guiding	=models.ManyToManyField( Celebrity, related_name = "guiding", blank = True )
	best_film=models.ForeignKey( Film, default = None, null = True, blank = True )

class Nominee( models.Model ):
	ceremony =models.ForeignKey( Ceremony )
	film	 =models.ForeignKey( Film, blank = True, null = True, on_delete = models.SET_NULL )
	celebrity=models.ForeignKey( Celebrity, blank = True, null = True, on_delete = models.SET_NULL )
```
### admin.py ###
```
from apps.autocomplete.widgets import *

class FilmAdmin(AutocompleteModelAdmin):
	related_search_fields = { 

		'type': ('title',),
		'actor': ('^name',),
		'director': ('^name',),
	}
admin.site.register( Film, FilmAdmin )
```
**NEW**: for inline
```
class NomineeInline( AutocompleteTabularInline ):
	model=Nominee
	extra=0
	
	""" describe the model fields for the search is only used for rendering the widget """
	related_search_fields={
		'film':			( 'name', ),
		'celebrity':	( 'name', ),
	}

class CeremonyAdmin( AutocompleteModelAdmin ):
	inlines=[
		NomineeInline,
	]

	related_search_fields={
	""" describe the model fields for the search """
		'guiding':		( 'name', ),
		'best_film':	( 'name', ),

	""" re-describes the same field as in inline is used to search for """
		'film':			( 'name', ),
		'celebrity':	( 'name', ),
	}
```

"related\_search\_fields" parameter is used to specify on what fields you want to search
'actor' and  'director' ties are the names given in "related\_name"
query syntax is similar to searching in admin panel  http://docs.djangoproject.com/en/dev/ref/contrib/admin/#django.contrib.admin.ModelAdmin.search_fields

I use the  " 'actor': ('^name',) "
operator ^ means the beginning of the field. and eventually will be formed about the substitution request form
For example, if related\_search\_fieldsis set to ('^name',') and a user searches for john lennon, Django will do the equivalent of this SQL WHERE clause:

WHERE name ILIKE 'john%' AND name ILIKE 'lennon%'

## NEW ##
You can also use the admin form and the additional fields are not described in the model which will refer to any existing model by WildModelSearchInput widget

search\_fields format - ( "search\_field", "result\_field")
```
class FilmAdminForm( forms.ModelForm ):
	class Meta:
		model=Film
	discussion = forms.CharField( required=False, widget=WildModelSearchInput(app_label='forum',model_name='Topic', search_fields=('name','id')))
```
when saving in the discussion will be chosen topic id and you can handle it on your own