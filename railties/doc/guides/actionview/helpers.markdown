Helpers
====================

Helper Basics
------------------------

Helpers allow you to encapsulate rendering tasks as reusable functions.  Helpers are modules, not classes, so their methods execute in the context in which they are called. They get included in a controller (typically the ApplicationController) using the helper function, like so

	Class ApplicationController < ActionController::Base
		…
		helper :menu

		def …
		end
	end

In this way, methods in the menu helper are made available to any view or partial in your application. These methods can accept parameters, for example controller instance variables (eg; records or record collections gathered by you current controller), items from the view or partial’s locals[] hash or items from the params[] hash. You may wish to pass your controller instance variables and items from the params[] hash to the locals hash before rendering (See the section on partials). Helper methods can also accept an executable block of code.

It is important to remember, though, that helpers are for rendering, and that they become available once a controller method has returned, while Rails is engaged in rendering the contents generated by a controller method. This means that helper methods are not available from within the methods of your controllers.

Helpers can accomplish a variety of tasks, from formatting a complex tag for embedding content for a browser plugin (eg; Flash), to assembling a menu of options appropriate for the current context of your application, to generating sections of forms that get assembled on-the-fly.

Helpers are organized around rendering tasks, so it is not necessary (nor necessarily desirable) to organize them around your application’s controllers or models. In fact, one of the benefits of helpers is that they are not connected via a rendering pipeline to specific controllers, like views and partials are. They can and should handle more generalized tasks.

Here is a very simple, pseudo-example:

	module  MenuHelper
		def menu(records, menu_options={})
			item_options = menu_options.merge({<some stuff>})
			items = records.collect |record| do
				menu_item(record, options)
			end
			content_tag(“ul”, items, options)
		end

		def menu_item(record, item_options={}))
			action = item_options[:action]
			action ||= “show”
			content_tag(“li”, link_to(record.title, :action => action, item_options)
		end
	end


This helper will require that records passed into it have certain fields (notably :title). The helper could be written to use this as a default, allowing the field to be overwritten by an element of item_options.

Look at the Rails API for examples of helpers included in Rails, eg; [`ActionView::Helpers::ActiveRecordHelper`](http://api.rubyonrails.org/classes/ActionView/Helpers/ActiveRecordHelper.html).

Passing Blocks to Helper Methods
------------------------

We mentioned before that blocks can be passed to helper methods. This allows for an interesting wrinkle: a block passed to a helper method can cause it to render a partial, which can then be wrapped by the helper method’s output. This can make your helper method much more reusable. It doesn’t need to know anything about the internals about what it is rendering, it just contextualizes it for the page. You can also use the helper to modify the locals hash for the partial, based on some configuration information unique to the current controller. You could implement a flexible themes system in this way.


Partials vs. Helpers?
------------------------

In general, the choice between using a partial vs. using a helper depends on the amount of flexibility you need. If the task is more about reacting to conditions than performing actual rendering, you may likely want a helper method. If you want to be able to call it from a variety of views, again, you may want to use a helper method. You can expect to extract helper methods out of code in views and partials during refactoring.


Tutorial -- Calling a Helper [UNFINISHED]
------------------------

1. Create a Rails application using `rails helper_test`
Notice the code:

		class ApplicationController < ActionController::Base
			helper :all # include all helpers, all the time
For this tutorial, we'll keep this code, but you will likely want to exert more control over loading your helpers.

2. Configure a database of your choice for the app.

3. Inside of the `/app/helpers/` directory, create a new file called, `menu_helper.rb`. Write this in the file:

		module  MenuHelpers
		   def menu(records, item_proc=nil)
		      items = records.collect{ |record|
		        menu_item(record, item_proc)
		      }
		      content_tag("ul", items)
		   end

		   def menu_item(record, item_proc=nil)
		      item_url = item_proc.call(record)
		      item_url ||= { :action => :show }
		      content_tag("li", link_to(record.name, item_url))
		   end
		end

4. Create a scaffold for some object in your app, using `./script/generate scaffold widgets`.
5. Create a database table for your widgets, with at least the fields `name` and `id`. Create a few widgets.
6. Call the menu command twice from `index.html.erb`, once using the default action, and once supplying a Proc to generate urls.