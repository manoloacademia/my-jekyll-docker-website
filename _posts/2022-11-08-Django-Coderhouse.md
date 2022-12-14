---
layout: post
title: "At Coderhouse, I learned Web Development using Django"
categories: personal
---

## Is it possible to build a Web Page using Python?

Sure it is!! You can use [Django](https://www.djangoproject.com/) which is a high-level Python web framework that encourages rapid development and clean, pragmatic design. In the docs, you can find a tutorial where teaches you how to build a polls app.

![django](https://user-images.githubusercontent.com/76537245/200622274-3175b805-bab9-4c49-a7f7-64b11a0a8545.PNG)

## Not a poll, but something else

Currently, I've been trying to embrace how to grow my content within your participation and I published some polls in [my LinkedIn](https://www.linkedin.com/in/luispablosegovia) (far away from being elected :D!!). So, I decided to go through a [Coderhouse](https://www.coderhouse.com) Python course where I learned Django.

As a final project, my two teammates and I made a [WineStore](https://github.com/manuel14mds/WineStore). We were working for a couple of weeks to fulfill the requirements.

## WineStore requirements and development

It was necessary to build 3 models, connected eachother using relational structured tables. The three models are Wines, Wineries and Varietals. The Wine model is detailed on the next snippet:

``` python
from django.db import models
from varietal.models import varietal
from winery.models import Winery

class Wine(models.Model):
    name = models.CharField(max_length=50)
    winery = models.ForeignKey(Winery, on_delete=models.CASCADE) # --> on Winery table
    varietal = models.ForeignKey(varietal, on_delete=models.CASCADE) # --> on varietal table
    age = models.IntegerField()
    available = models.BooleanField(default=True) 
    price = models.FloatField(default=0)
    image = models.ImageField(upload_to='wines/', null=True, blank=True)

    def __str__(self):
        return self.name
```

Of course, there was necessary to define another model to check the *__User type__* in case you want only check the products, add your profile to purchase or add/delete/update *__CRUD__* products.

`users/forms.py`
```python
from django.contrib.auth.forms import UserCreationForm
from django.contrib.auth.models import User
from django import forms

class User_registration_form(UserCreationForm):
    email = forms.EmailField(required=True)
    password1 = forms.CharField(label='Password', widget=forms.PasswordInput)
    password2 = forms.CharField(label='Password confirmation', widget=forms.PasswordInput)

    class Meta:
        model = User
        fields = ('username', 'email', 'password1', 'password2')

        help_texts = {k:'' for k in fields}
```

`users/models.py`
``` python
from django.db import models

class User_profile(models.Model):
    user = models.OneToOneField('auth.User', on_delete=models.CASCADE, related_name='profile')
    phone = models.CharField(max_length=20, blank=True)
    address = models.CharField(max_length=200, blank=True)
    image = models.ImageField(upload_to='profile_image/', blank=True)

    def __str__(self):
        return self.user.username + ' - profile'
```

## Style and colors

For style, color and stuff I used [Bootstrap](https://getbootstrap.com/) via CDN and HTML/CSS + Django Template languages.

`templates/index.html`
``` html
<div class="jumbotron container">
    <h1 class="display-4">WineStore</h1>
    <p class="lead">Welcome to the best wine store around the world!!</p>
</div>

<section>

    <div class="container ">
        <div id="carouselExampleInterval" class="carousel slide" data-ride="carousel">
            <div class="carousel-inner">
                <div class="carousel-item active" data-interval="2000">
                    <img src="https://i.pinimg.com/originals/81/03/11/810311031d2007d921cf126b3cd298b3.jpg"
                        class="d-block w-100" alt="...">
                </div>
                <div class="carousel-item" data-interval="2000">
                    <img src="https://daily.sevenfifty.com/app/uploads/2022/02/SFD_Lebanon-Beyond-Musar-by-Courtney-Schiessel_Lebanese-wines-on-ililis-wine-list_CO_ilili_Hero_2520x1420.jpg"
                        class="d-block w-100" alt="...">
                </div>
                <div class="carousel-item">
                    <img src="https://post.healthline.com/wp-content/uploads/2022/05/wine-glass-hand-1200x628-facebook-1200x628.jpg" class="d-block w-100"
                        alt="...">
                </div>
            </div>
            <button class="carousel-control-prev" type="button" data-target="#carouselExampleInterval"
                data-slide="prev">
                <span class="carousel-control-prev-icon" aria-hidden="true"></span>
                <span class="sr-only">Previous</span>
            </button>
            <button class="carousel-control-next" type="button" data-target="#carouselExampleInterval"
                data-slide="next">
                <span class="carousel-control-next-icon" aria-hidden="true"></span>
                <span class="sr-only">Next</span>
            </button>
        </div>
    </div>
</section>

```

To see more in detail, please [check the video in this link](https://www.youtube.com/watch?v=2-M1FfZu5y8).

![winestore](https://user-images.githubusercontent.com/76537245/200628771-85c6a36d-d892-400e-91d2-79372382528f.PNG)

Let me know if you need more information about how to create structures, users, use the admin profiles, etcetera and etcetera.

Thanks for your time on the reading :).
## *__Cheers, Pablo.__*
