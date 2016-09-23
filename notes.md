# ==== Overview of Class-based (Generic) View(CBV) ===

- CBV take certain common idioms and patterns found in view development and abstract them in different parent classes(**generic views** and **mixins**) that user can inherit from. 
- Many of Django’s built-in class-based views inherit from other class-based views or various mixins. Because this inheritance chain is very important, the ancestor classes are documented under the section title of Ancestors (MRO). **MRO** is an acronym for Method Resolution Order.
- All class-based generic views have an **as_view()** class method which serves as the callable entry point to your class. The as_view entry point 
    - creates an instance of your class
    - calls its dispatch() method. dispatch looks at the request to determine whether it is a GET, POST, etc, and relays the request to a matching method if one is defined, or raises HttpResponseNotAllowed if not.
- Note that functions don't have inheritance. You need to copy and paste codes. The biggest advantage of CBV is **inheritance**. By inheriting those parent classes, user can inherit common things from parents, and change/override things in the child class view: 
    - **access the parent data**, by accessing attributes or doing supercall of the method in parent class
    - **override the parent data**, by passing new attribute data or by first doing supercall of the method and then change the data returned by supercall
    - **merge the parent data**, by first doing supercall of the method and then add new custom data to the data returned by supercall 

- So it is important to know what attributes and methods each parent generic view has,

Example:
```
from django.views.generic import DetailView
from books.models import Publisher, Book

class PublisherDetail(DetailView):

    model = Publisher

    def get_context_data(self, **kwargs):
        # Call the base implementation first to context data from parent class
        context = super(PublisherDetail, self).get_context_data(**kwargs)
        # Merge a custom QuerySet of all the books into the parent data 
        context['book_list'] = Book.objects.all()
        return context
```

# ==== Base Views ===
There are three the master class-based base views, from which all other class-based views inherit from. They only contain minimun number of methods needed to create Django views
- View
- TemplateView
- RedirectView