# Authz in Django

May 18 2019 / DjangoCongress JP 2019

Hiroki Kiyohara @hirokiky

---

## About this talk

+++

## On this talk

You can learn

* Difference between Authn / Authz
* Nicer to handle Authz in Django

+++

## Authorization

認可 (Nin-Ka)

+++

## More simply

WHAT you can do

+++

### Older version

This talk is updated version of this

http://slides.hirokiky.org/pyconjp2017.html

---

## Who am I

+++

## @hirokiky

* Hiroki Kiyohara
* Created [PyQ](https://pyq.jp/), [dig-en.com](https://dig-en.com/), [PileMd](https://pilemd.com/)
* Chairperson of DjangoCongress JP 2018, 2019

+++

### PyQ

![PyQ-ScreenShots](2019djangocongress/images/pyq_ss.png)

---

## Authn / Authz

+++

## Authn / Authz

* Authn: 認証 (Nin-Sho)
* Authz: 認可 (Nin-Ka)

+++

## Authentication

WHO you are

+++

## Authorization

WHAT you can do

+++

## What's WHAT

can WHO DO THIS ?

+++

## Related things

* Permission
* scope
* Forbidden (403)

---

## Authz in Django

+++

### Views

```python
def post_edit(request, post_id):
    post = get_object_or_404(Post, id=post_id)
    if request.user is not post.author:
        return HttpResponseForbidden()
    ...
```

+++

### Views

* Pros:
    * Easy
* Cons:
    * Complication
    * Duplication

+++

### Decorators

```python
@login_required
@premium_sub_required
def post_detail(request, post_id):
    ...
```

+++

### Decorators

* Pros
    * Smarter
    * Enoughly Work
* Cons
    * Hard to change condition
    * Can't use in Template
    
+++

Now "Standard" users can use some "Premium" features.
We need to change all of affected Viwes/Templates
(In startup projects, specs often changes dramatically...)

+++

### Django Permission

```python
content_type = ContentType.objects.\
                   get_for_model(Post)
permission = Permission.objects.get(
    codename='edit',
    content_type=content_type,
)
user.user_permissions.add(permission)
```

+++

### With decorator

```python
@permission_required(blog.edit)
def post_edit(request, post_id):
    ...
```

+++

### Pros and Cons

* Pros
    * Default feature
    * Integration with Admin
* Cons
    * Dynamic data
    * Hard to change conditions

---

## django-keeper

+++

## At a glance

+++

### ACLs in models

```python
from keeper.operators import Everyone, IsUser
from keeper.security import Allow

class Post(models.Model):
    ...
    def __acl__(self):
        return [
            (Allow, Everyone, 'view'),
            (Allow, IsUser(self.author), 'edit'),
        ]
```

+++

### Decorators in views

```python
from keeper.views import keeper

@keeper(
    'edit',
    model=Post,
    mapper=lambda request, post_id: {'id': post_id},
    
)
def post_edit(request, post_id):
    request.k_context  # Post has post_id
```

+++

### Tags in templates

```html
{% load keeper %}

{% has_permission post 'delete' as can_delete %}
{% if can_detele %}
    <a href="{% url 'blog:post_delete' post_id=post.id %}">Delete</a>
{% endif %}
```

+++

### In models

List contains tuples (**Action**, **Who**, **Permission**).

```python
(Allow, IsUser(self.author), 'edit'),
```

It will identify what kind of "permissions" each requests has.
(If `post.author` is `request.user`, the request can `"edit"`).

+++

### Dynamic ACLs

```python
class Post(models.Model):
    def __acl__(self):
        if self.draft:
            return [(Allow, IsUser(self.author),
                     ('view', 'edit', 'delete'))]
        else:
            return [(Allow, Everyone, 'view'),
                    (Allow, IsUser(self.author),
                     (edit', 'delete'))]
```

+++

### Dynamic ACLs

Even if you write large ACL, views will be so simple.

```python
@keeper('view', factory=post_factory)
def post_detail(request, post_id):
   ...

@keeper(edit', factory=post_factory)
def post_edit(request, post_id):
   ...

@keeper('delete', factory=post_factory)
def post_delete(request, post_id):
   ...

```

+++

### Operators

These `Everyone`, `IsUser` are "Operators".
It should be callable to take HttpRequests and return Bool.

* [Default operators](https://github.com/hirokiky/django-keeper/blob/master/keeper/operators.py)

+++

### Own Operators

```python
from keeper.operators import Authenticated

class HasSubscription(Authenticated):
    def __init__(self, plan_code):
        self.plan_code = plan_code

    def __call__(self, request):
        if not super().__call__(request):
            return False
        return request.user.sub.plan.code is \
            self.plan_code
```

+++

### Own Operators

```python
class Post(models.Model):
    def __acl__(self):
        if self.exclusive:
            return [
                (Allow,
                 HasSubscription(PLAN_PREMIUM),
                 'view')
            ]
        else:
            return [(Allow, Everyone, 'view')]
```

No need to change Views.

+++

### keeper allows you

* To handle complex authz
* To describe permissions declaratively
* Not to change views

+++

### @keeper decorator

```python
@keeper(
    'view',
    model=Post,
    mapper=lambda request, post_id: {'id': post_id}
)
```

It specifies way to get target models (`Post.objects.get(id=post_id)`)
and required permission.

+++

### Changing on_fail action

```python
from keeper.views import not_found

@keeper(
    ...,
    on_fail=not_found,
)
```

Displays 404 page if user can't access the page
(like GitHub's 404 page for private repos).

* [Default on_fail actions](https://github.com/hirokiky/django-keeper/blob/master/keeper/views.py)

+++

### Template tags

```html
{% load keeper %}

{% has_permission post, 'delete' as can_delete %}
{% if can_detele %}
  <a href="{% url 'blog:post_delete' post_id=post.id %}">Delete</a>
{% endif %}
```

+++

### Global Context

Not related with objects.

```python
from keeper.operators import Authenticated
from keeper.security import Allow

class GlobalContext:
    def __acl__(self):
        return [
            (Allow, Authenticated, 'search_posts'),
        ]
```

+++

### Things keeper can't do

* List filtering
* Cache

+++

## FAQ

* Should I use django-keeper?
    * Too much for small projects
    * If you have some problems now
* Should I use @keeper for all views?
    * No

+++

## Similar Libs

* Django's Permission
* django-gurdian
* djagno-rules

---

## keeper in Production

+++

### Production ready?

Yes.
But I can't guarantee safety.

+++

### Use case

* Personal / Team users
* Subscription
* Contents
    * Default
    * Plan only

+++

Try django-keeper and send Pull-Requests 🙏

---

### Conclution

* Authz is detect WHAT each requests can do
* Many ways for authz in Django
* django-keeper will help you in complex projects

---

## That's all folks
