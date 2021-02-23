# Django

## Django - Preventing translation of available language names in template

According to the [translation documentation](https://docs.djangoproject.com/en/dev/topics/i18n/translation/) you can use the available language tools either in the template or in the python code.

In the template, using the [get_language_info](https://docs.djangoproject.com/en/dev/topics/i18n/translation/#get-language-info) template tag:
{% raw %}
'''jinja
{% get_language_info for "pl" as lang %}

Language code: {{ lang.code }}<br />
Name of language: {{ lang.name_local }}<br />
Name in English: {{ lang.name }}<br />
Bi-directional: {{ lang.bidi }}
Name in the active language: {{ lang.name_translated }}
'''
{% endraw %}

which can be combined with other tags and build a mechanism that allows you to [change languages](https://docs.djangoproject.com/en/dev/topics/i18n/translation/#switching-language-in-templates):

{% raw %}

{% for lang_code, lang_name in languages %}  
   {% if lang_code != LANGUAGE_CODE %}      
     {% get_language_info for lang_code as lang_info %}
     {% language lang_code %}                            
     {% url request.resolver_match.url_name as no_slug %}
     {% url request.resolver_match.url_name slug=object.slug as yes_slug %}  
     <p>Link to: {% firstof yes_slug no_slug %} Local name: {{ lang_info.name_local }}</p>
     {% endlanguage %}
   {% endif %}
 {% endfor %}

{% endraw %}
