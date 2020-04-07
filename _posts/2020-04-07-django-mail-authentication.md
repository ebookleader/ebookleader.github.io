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
<a href='https://simpleisbetterthancomplex.com/tutorial/2016/08/24/how-to-create-one-time-link.html'>
password token generator 참고</a>
{% highlight python %}
# token_generator.py

# PasswordResetTokenGenerator를 사용하여 토큰 생성
from django.contrib.auth.tokens import PasswordResetTokenGenerator
# for wrapping over differences between python2 and python3
import six

class TokenGenerator(PasswordResetTokenGenerator):
    def _make_hash_value(self, user, timestamp):
        return(
            six.text_type(user.pk)+six.text_type(timestamp)+six.text_type(user.is_active)
        )

account_activation_token = TokenGenerator()

{% endhighlight %}

<ol>
<li>사용자 데이터와 관련해서 hash값을 만들고 비밀번호가 리셋되면 값이 바뀌는 형식</li>
<li>text_type은 유니코드 정수로부터 유니코드 문자열을 가지고 오며, 사용자의 pk, 현재시간, 활성화를 합쳐서 token을 생성해 리턴한다</li>
</ol>
--------------------------------------------

### 회원가입 View
{% highlight python %}
# views.py

from django.http import HttpResponse
from django.shortcuts import render, redirect
from django.contrib.auth import login, authenticate
from .forms import UserSignUpForm
from django.contrib.sites.shortcuts import get_current_site
from django.utils.encoding import force_bytes, force_text
from django.utils.http import urlsafe_base64_encode, urlsafe_base64_decode
from django.template.loader import render_to_string
from .token_generator import account_activation_token
from django.contrib.auth.models import User
from django.core.mail import EmailMessage

# signup method
def signupuser(request):
    if request.method == 'GET':
        # 위에서 만들어두었던 UserSignUpForm 사용
        form = UserSignUpForm()
    else:
        form = UserSignUpForm(request.POST)
        if form.is_valid():
            user = form.save(commit=False)
            user.is_active = False
            user.save()
            current_site = get_current_site(request)
            email_subject = 'Activate your account'
            message = render_to_string('todoapp/activate_account.html',
                                       {
                                           'user':user,
                                           'domain':current_site.domain,
                                            'uid':urlsafe_base64_encode(force_bytes(user.pk)),
                                           'token':account_activation_token.make_token(user),
                                        }
                                       )
            to_email = form.cleaned_data.get('email')
            email = EmailMessage(email_subject,message,to=[to_email])
            email.send()
            return HttpResponse('we have sent you an email')
    return render(request,'todoapp/signup.html',{'form':form})
{% endhighlight %}

<ol>
<li>`GET`이면 UserSignUpForm을 전달</li>
<li>`POST`면 입력받은 값이 유효한지 체크하고 이메일 인증을 완료한 사용자만 로그인 할 수 있도록 가입 시에는 is_active를 False로 설정한다</li>
<li></li>
</ol>
