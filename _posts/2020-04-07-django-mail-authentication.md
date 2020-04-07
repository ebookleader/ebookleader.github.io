---
layout: post
title:  "[Django] 회원가입 이메일인증"
date:   2020-04-07
categories: Django
tags:	BackEnd python Django SMTP
---

### 사전 설정

1.Gmail 보안수준이 낮은 앱 허용 설정  

&nbsp;&nbsp;&nbsp;&nbsp;gmail 로그인 후 <a href='https://www.google.com/settings/security/lesssecureapps'>Gmail 보안 설정</a>에서 사용으로 바꿔준다.

2.settings.py에 메일 호스트 추가

{% highlight python %}
# settings.py

# Gmail 기준
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_PORT = 587
EMAIL_HOST_USER = 'example@gmail.com'
EMAIL_HOST_PASSWORD = 'password'
EMAIL_USE_TLS = True
{% endhighlight %}

--------------------------------------------
### Django forms 사용

app 폴더 내에 forms.py 생성
{% highlight python %}
# forms.py

# 장고에서 디폴트로 제공하는 User 모델을 사용
from django import forms
from django.contrib.auth.models import User
from django.contrib.auth.forms import UserCreationForm

# built-in으로 제공하는 UserCreationForm 사용
class UserSignUpForm(UserCreationForm):
    # email 필드 추가
    email = forms.EmailField(max_length=128)
    class Meta:
        # User 모델을 사용하고
        model = User
        # user가 보게 될 필드를 정의
        fields = ('username', 'email', 'password1', 'password2')
{% endhighlight %}

--------------------------------------------

### Token을 만드는 class 작성
{% highlight python %}
# token_generator.py
from django.contrib.auth.tokens import PasswordResetTokenGenerator
import six

class TokenGenerator(PasswordResetTokenGenerator):
    def _make_hash_value(self, user, timestamp):
        return(
            six.text_type(user.pk)+six.text_type(timestamp)+six.text_type(user.is_active)
        )

account_activation_token = TokenGenerator()

{% endhighlight %}

--------------------------------------------
