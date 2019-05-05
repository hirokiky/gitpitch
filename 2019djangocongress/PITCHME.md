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

## Similar words

* Permission
* scope
* Forbidden (403)

---

## Authz in Django

+++

### Views

```python
def post_change(request, post_id):
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
We need to chnage all of affected Viwes/Templates
(In startup projects, specs often changes dramatically...)

+++

### Django Permission

```python
content_type = ContentType.objects.\
                   get_for_model(Post)
permission = Permission.objects.get(
    codename='change_post',
    content_type=content_type,
)
user.user_permissions.add(permission)
```

+++

### With decorator

```python
@permission_required(blog.change_post')
def post_change(request, post_id):
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

## What can keeper do?

*

+++

## Things keeper can't do

* List filtering
* Cache

+++

### Pre fetching

```python
```

+++

## Similar Libs

* Django's Permission
* django-gurdian
* djagno-rules

---

## keeper in Production

---

## That's all folks
