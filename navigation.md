{% assign menus = site.pages | where: 'nav', true | sort: 'order' | group_by: 'category' %}
{% for cat in menus %}
	{% if cat.name %}
		<!-- get  the  first item-->
		{% assign root_item = cat.items.first %}
		{% assign items = cat.items | shift%}
		
		
		## [{{root_item.title}}]({{root_item.url}})

		{% for item in items %}
			[{{item.title}}]({{item.url}})
		{% endfor %}
	{% endif %}
{% endfor %}