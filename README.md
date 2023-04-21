# cheat-sheet-flask
cheat sheet | راهنمای کامل فلاسک به صورت خلاصه

## Creating a Simple App
- Create a module called `app.py`:
```python
from flask import Flask
app = Flask(__name__)
@app.route('/')
def home():
	return 'Hello!'
  
if __name__ == '__main__':
	app.run(debug=True)
```

## Creating Models with SQLAlchemy
- In `models.py`:
```python
from flask_sqlalchemy import SQLAlchemy
from datetime import datetime

class Post(db.Model):
    category_id = db.Column(db.Integer, db.ForeignKey(Category.id), nullable=True)
    user_id = db.Column(db.Integer, db.ForeignKey(User.id), nullable=True)
    id = db.Column(db.Integer, primary_key=True)
    title = db.Column(db.String(128), nullable=False)
    date = db.Column(db.DateTime, nullable=False, default=datetime.utcnow)
    content = db.Column(db.Text, nullable=True)
    slug = db.Column(db.String(128), nullable=False)
    rep = db.Column(db.Boolean, default=False, nullable=False)

    def __repr__(self):
        return '<Posts %r>' % self.title
```

## Creating Forms
- In `forms.py`:
```python
from wtforms import Form
from wtforms import (StringField, TextAreaField, IntegerField, BooleanField,
                     RadioField, PasswordField, EmailField, URLField, FileField)
from wtforms.validators import InputRequired, Length, EqualTo, Email, URL


class Register(Form):
    name = StringField('نام و نام خانوادگی', [InputRequired('فیلد نام و نام خانوادگی خالی است')])
    username = StringField('نام کاربری', [InputRequired('فیلد نام کاربری خالی است')])
    phone = StringField('موبایل', [InputRequired('فیلد موبایل خالی است'),
                                   Length(min=11, max=11, message='شماره موبایل اشتباه است')])
    password = PasswordField('گذرواژه',
                             [InputRequired('فیلد گذرواژه خالی است'), EqualTo('confirm', message='گذرواژه هماهنگ نیست'),
                              Length(min=8, message='گذرواژه باید 8 نویسه باشد')])
    confirm = PasswordField('تکرار گذرواژه')
```

## Class base View
- In `app.py`:
```python
# App_Url
from controller import admin, user, home, blog

controller_home = home.Home()
controller_user = user.Serclass()
controller_admin = admin.Admin()
controller_blog = blog.Blog()


# Home
app.add_url_rule('/', 'home', controller_home.home)

# User
app.add_url_rule('/register', 'register', controller_user.register, methods=['POST', 'GET'])
app.add_url_rule('/login', 'login', controller_user.login, methods=['POST', 'GET'])
app.add_url_rule('/logout', 'logout', controller_user.logout)
```

## Database created
- Database created `__init__.py`:
```python
with app.app_context():
    db.create_all()
    print("Database created and modified successfully!")
```

## Register 
- Register `user.py`:
```python
def register(self):
        if current_user.is_authenticated:
            return redirect(url_for('home'))
        form = Register(request.form)
        if request.method == 'POST' and form.validate():
            hashed_pass = bcrypt.generate_password_hash(form.password.data).decode('utf-8')
            phone_filter = User.query.filter_by(phone=form.phone.data).first()
            if not phone_filter:
                user_re = User(phone=form.phone.data, fullname=form.name.data, username=form.username.data, password=hashed_pass)
                db.session.add(user_re)
                db.session.commit()
                flash('با موفقیت ثبت نام کردید', 'success')
                return redirect(url_for('login'))
            else:
                flash('شماره موبایل وارد شده تکراری است', 'info')
                return redirect(url_for('register'))
        return render_template('user/register.html', form=form)
```

## Flask_Login
- Config Flask-Login `settings.py`:
```python
from flask_login import LoginManager
from models import User

# Flask-Login
login_manager = LoginManager(app)
login_manager.login_view = 'login'
login_manager.login_message = 'برای وارد شدن اطلاعات خود را وارد کنید'
login_manager.login_message_category = 'info'


# Flask_Login
@login_manager.user_loader
def load_user(user_id):
    return User.query.get(user_id)
    
# Models
from flask_login import UserMixin
class User(db.Model, UserMixin):
    id = db.Column(db.Integer, primary_key=True)
    ..........
```

## Login
- Login `user`:
```python
def login(self):
        if current_user.is_authenticated:
            return redirect(url_for('home'))
        form = Login(request.form)
        if request.method == 'POST' and form.validate():
            user = User.query.filter_by(phone=form.phone.data).first()
            if user and bcrypt.check_password_hash(user.password, form.password.data):
                login_user(user)
                next_page = request.args.get('next')
                flash('شما با موفقیت وارد شدید', 'success')
                return redirect(next_page if next_page else url_for('home'))
            else:
                flash('شماره موبایل یا گذرواژه اشتباه است', 'danger')
                return redirect(url_for('login'))

        return render_template('user/login.html', form=form)
```

## Logout
- Logout `user`:
```python
from flask_login import login_required, logout_user
@login_required
    def logout(self):
        logout_user()
        flash('شما با موفقیت خارج شدید', 'success')
        return redirect(url_for('login'))
```

## Admin
- Admin page all user `admin.py`:
```python
from flask.views import MethodView
from models import User

class Admin(MethodView):
    def __init__(self, *args, **kwargs):
        pass

    @login_required
    def admin(self):
        return render_template('admin/admin.html')

    @login_required
    def users(self):
        user = User.query.all()
        return render_template('admin/users.html', user=user)
```

## Admin
- Admin page delete_user `admin.py`:
```python
@login_required
def user_delete(self, user_id):
user = User.query.get_or_404(user_id)
db.session.delete(user)
db.session.commit()
flash(f' کاربر {user.fullname} با موفقیت حذف شد', 'info')
return redirect(url_for('users'))
```

