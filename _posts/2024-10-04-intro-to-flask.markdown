---
layout: post
title:  "Python - Learning the Flask framework with a side project"
date:   2024-10-04 16:02:29 -0400
categories: jekyll update
---

Click [here][repo-link] for code repository.


Flask is fun to work with. I'm currently working on templates, and how the concept of inheritance comes into play while working with files having many lines of repeated code. There is something called as `block elements`, which help to prevent rewriting of code.

The following functions were used to build an initial version of the blogging application I'm working on:
`render_remplate()`
`url_for()`
`app.route()`

`render_template()` is used to load the HTML pages from the templates folder, when a particular URL is called. This is done primarily from the `app.route()` function. `url_for()` is used to call the exact function name used within the Flask codebase, rather than relying on URLs. For instance, there is a URL defined in the `app.route()` function as `/home`. Instead of calling the URL direclty, which can probably change in the future, we instead call the function associated with that URL. This prevents configuration issues when the web URL is to be changed.

## 8/28/2024

Today we went through Flask Forms. Flask Forms are an excellent feature provided by Flask, which helps to set up HTML forms with validations very quickly. I'll discuss some of the highlights from the code.

<br/>

### Form Classes

What we can see below is an HTML class for Flask. It has fields of username, email, password, and confirm password. Each of the filed has its own set of validations, which is defined in the `validators` variable. These are imported from the `flask_wtf` and the `wtforms` library.

![image tooltip here]({{ "/assets/flask/image_2.png" | relative_url }})

<br/>

### Validate on submit, Flash messages

There's lot to unpack from the below code snippet. The form class which was created in the above section, is now accessed by the application. `form` stores an instance of the `RegistrationForm` class, which will be passed to the `register.html` page using the `render_template()` function.
The purpose of the `validate_on_submit()` method is to capture whether the form was submitted with the valid details. the `flash` method is responsible for displaying a prompt of whether there was an error or not.
The `redirect` method is a new concept, wherein a different page can be called in case of errors.

![image tooltip here]({{ "/assets/flask/image_3.png" | relative_url }})

<br/>

### Integrating Flash messages in HTML layout

Let'discuss how the flash message can be used. We have a common HTML page called `layout.html`, which stores all the repeated lines of HTML code that is used by other HTML pages within the application. This is why the prompt for flash messages is added in the `layout.html` page, so that it can be consumed by all the other pages. We get the messages, if any, and iterate it over to display on the HTML page. The below code snippet shows how the linking between the Python code and the HTML page for flash messages is done.

![image tooltip here]({{ "/assets/flask/image_4.png" | relative_url }})

<br/>

### Showing errors for individual fields on HTML form

The below code helps to print out the error for each field.

![image tooltip here]({{ "/assets/flask/image_4.png" | relative_url }})

<br/>

### Other points covered:
* Error with email_validator, using dummy data to test login page.

<br/>

## 8/29/2024

* When ORM is used for database interaction, different databases can be used without altering the Python code.
* Each class represents a table in the database. Below is an example:

![image tooltip here]({{ "/assets/flask/image_1.png" | relative_url }})

* The `__repr__` method is a representation method. The `///` logic is that the temporary database will be created in the home path relative to the current directory.
* There was an issue while creating the database. The line `app.app_context().push()` is used to execute the database creating command within the application context.
* The `db.relationship` code is used to establish the one to many relationship between the User and Post table. The `author` field is shared in this context.

<br/>

## 8/31/2024

## Refactoring modules into packages

* It is always a good idea to convert modules into packages. This requires changing of import statements. While doing this update, imports failed, due to a concept called circular imports. The issue was that one of the database models tried to access the database (db) instance, before the db instance was created and imported into the app.
* This concept is discussed in a talk called Flask at Scale. Packages is a very important topic.

<br/>

## 9/10/2024

## All about user authentication and sessions

Use of brcypt ensures the following: the password is stored as a hash in the database, and same passwords will have different hash in the database.

![image tooltip here]({{ "/assets/flask/user_session/001_bcrypt.png" | relative_url }})

<br/>

Handling of duplicate usernames - This is done by Jinja templates. A ValidationError is raised in case conditions are not met. The error is thrown in the HTML for itself.

![image tooltip here]({{ "/assets/flask/user_session/003_custom_validation_forms.png" | relative_url }})

<br/>

The Login Manager instance is loaded into the app:

![image tooltip here]({{ "/assets/flask/user_session/004_login_manager.png" | relative_url }})

<br/>

We need a decorator when performing user session login. This is implemented in the models.py class. The decorator function loads the user to perform authentication.

![image tooltip here]({{ "/assets/flask/user_session/005_user_loader.png" | relative_url }})

<br/>

We use the current_user.is_authenticated method to check whether user session is loaded. The below image shows conditonals in the layout.html page, wherein based on the state of the user session, appropriate navigation links are displayed:

![image tooltip here]({{ "/assets/flask/user_session/006_nav_bar.png" | relative_url }})

<br/>

When user tries to access register page after login, the current_user.is_authenticated method checks if there is any user session loaded. If yes, it will redirect to the home page:

![image tooltip here]({{ "/assets/flask/user_session/007_register_check_user.png" | relative_url }})

<br/>

Same applies for login page:

![image tooltip here]({{ "/assets/flask/user_session/008_login_check_user.png" | relative_url }})

<br/>

When a user tries to access the links he's not allowed to access when not logged in (for instance, the accounts page), the functionality was to redirect the user to the login page, giving an error stating that the user don't have access to that page. However when the user logs in, he is redirected to the home page. It would be nice to redirect the user to the link he was trying to access:

![image tooltip here]({{ "/assets/flask/user_session/002_check_user_and_password_in_db.png" | relative_url }})

<br/>

## 9/11/2024

## User account and profile picture

Changes are made in the Account tab to provide means to update username, email, and profile picture.

We added a new form to handle updates:

![image tooltip here]({{ "/assets/flask/user_account/009_update_username_email.png" | relative_url }})

<br/>

We update the update form to add provision for updating picture:

![image tooltip here]({{ "/assets/flask/user_account/010_picture_form.png" | relative_url }})

<br/>

The account() function below does several functions - it updates the profile picture in the application and database, updates username and email, it then redirects to the Account page after form is validated.

![image tooltip here]({{ "/assets/flask/user_account/011_account_route_code.png" | relative_url }})

<br/>

The below code performs the following objectives: create a randomized name for the uploaded photos, properly store them in the application directory once updated, and using the Image library, resize the image so that less space is taken in the application.

![image tooltip here]({{ "/assets/flask/user_account/012_save_picture.png" | relative_url }})

<br/>

## Creating, Updating, and Deleting posts

We create a new form for handling post updates. We have a title and content field:

![image tooltip here]({{ "/assets/flask/modify_post/013_update_form.png" | relative_url }})

<br/>

Based on whether the user is logged in or not, the app displays the link to create a new post:

![image tooltip here]({{ "/assets/flask/modify_post/014_new_post_nav_bar.png" | relative_url }})

<br/>

Once the New Post link is hit, the app loads the PostForm form instance. This uses the create_post.html page for creating new post. Once validated on submit, it commits data to the database, and redirects to home page.

![image tooltip here]({{ "/assets/flask/modify_post/015_new_post_route.png" | relative_url }})

<br/>

Each post is assigned a post ID by the database. On the home page when a particular post is clicked, the post opens on a new link, with the URL as post/post_id:

![image tooltip here]({{ "/assets/flask/modify_post/016_display_post.png" | relative_url }})

<br/>

To update any post, 2 criterias need to be satisfied: the user is logged in, and the user who has created the post is the one trying to update the post. This is handled by the @login_required and the post.author != current user conditional respectively. abort(403) is raised if the user is not authorised to update the post. Note that both create and update post functions use the same create_post.html page to perform their actions. The request.method == 'GET' condtional is triggered when the post is first loaded after the user clicks on the post from home page. The code inside the conditinal populates the title and content field.

![image tooltip here]({{ "/assets/flask/modify_post/017_update_post_route.png" | relative_url }})

<br/>

The app uses BootStrap Modal code to provide an extra layer of confirmation to the user, when the user tries to delete a post. This is provided in the post.html page. The below function is called when the modal is hit, and deletes the post from database:

![image tooltip here]({{ "/assets/flask/modify_post/018_delete_post_route.png" | relative_url }})

<br/>

## Pagination

It isn't ideal for all the posts to be displayed on one page. Pagination helps in distribute the posts across several pages. This can be controlled using the paginate() method. In the below code, the paginate() method describes how many posts are to be displayed per page:

![image tooltip here]({{ "/assets/flask/pagination/019_pagination_arguments.png" | relative_url }})

<br/>

To reflect this on the HTML page, the below code is used. The below code also handles a case wherein the page on which the user is currently will be highlighted, so that the user knows what page he's on:

![image tooltip here]({{ "/assets/flask/pagination/020_display_page_count.png" | relative_url }})

<br/>

One additional functionality is provided by the app: when the user clicks on the username on the home page, the posts written by that username will be displayed. The pagination logic remains the same, except this time the records are filtered down by the author name:

![image tooltip here]({{ "/assets/flask/pagination/021_user_posts_only.png" | relative_url }})

<br/>

To reflect these user-specific changes in HTML, we need to pass an extra argument, that is the username:

![image tooltip here]({{ "/assets/flask/pagination/022_display_page_count_users.png" | relative_url }})

<br/>

Currently, the href did not have any value, so clicking on the username would do nothing. So the URL us updated as follows:

![image tooltip here]({{ "/assets/flask/pagination/023_change_href.png" | relative_url }})

<br/>

## Reset password using link sent in email

To perform password reset, it is divided into 2 steps - the first form will take the email from the user, which will be used as a receipent for the password reset link. Once the user opens the link from the email, he will be redirected to a different reset password form page, where he will have to enter the password and confirm password fields:

![image tooltip here]({{ "/assets/flask/reset_password/024_reset_forms.png" | relative_url }})

<br/>

To manage the time limit for such links, tokens are generated based on the user ID. These are timed, after which they will be invalid or expired:

![image tooltip here]({{ "/assets/flask/reset_password/025_generate_token.png" | relative_url }})

<br/>

The below function handles the sending of email to the user. The send_reset_email() functions manages the operation:

![image tooltip here]({{ "/assets/flask/reset_password/026_reset_request_route.png" | relative_url }})

<br/>

The reset password option is provided by the Forgot Password link on the login page. The href for this is set as the form defined in the reset_request() definition:

![image tooltip here]({{ "/assets/flask/reset_password/027_link_for_reset.png" | relative_url }})

<br/>

The below code consists of the config details for the mail server. I have not set this up, so this needs to be done:

![image tooltip here]({{ "/assets/flask/reset_password/028_mail_config.png" | relative_url }})

<br/>

The below function handles the sending of the email. The app fetches the token generated from the models.py class:

![image tooltip here]({{ "/assets/flask/reset_password/029_send_mail_function.png" | relative_url }})

<br/>

The below function does the following: it makes sure that the reset password functionality is not available if the user is logged in, it verifies whether the token is expired or invalid, and once that is done, it updates the password:

![image tooltip here]({{ "/assets/flask/reset_password/030_reset_password_route.png" | relative_url }})

<br/>

## 9/12/2024

## Blueprints

The application is structured as a blueprint. It helps to split the functions, which are all bunched up, into their respective domains. For instance, all routes and forms related to users are placed in the users package, and all routes and forms related to posts are moved into the posts package. Standalone functions are moved into the main package, which are then placed in a utils.py file.

Python knows if a directory is a package, if it contains a `__init__.py` file. It can be empty.

Once these functions are moved, the import statements also need to be handled.

Since the functions are moved into their respective blueprints, the `url_for` functionality will not work for the current function names. They need to be accessed using the blueprint name. For instance, the `home` route is placed in the `main` blueprint package. This for `url_for` to access this function, it needs to use `main.home`, instead of just `home`. These changes are mostly found in the routes file and some template files.

Another big change is the refactoring of the `__init__.py` file inside the root directory. We first move all the config code that is involved, such as the database `SECRET_KEY`, the URI, and the config properties related to the mail server. These are not stored in the application files for security purposes. A `config.py` file is created, and all these config details are placed inside a `Config` class. This class will be accessed by the `__init__.py` file when the app initializes during app start up.

It is to be noted that we need to create blueprints inside this init file for Flask to know we are using blueprints. There is more info about why this needs to be done, and how these blueprints are then structured inside a function for creating objects, refer the video for more understanding.

<br/>

The below function contains all initialization of the components used within the app:

![image tooltip here]({{ "/assets/flask/blueprint/031_create_app_blueprint.png" | relative_url }})

<br/>

The below Config class contains all the config details:

![image tooltip here]({{ "/assets/flask/blueprint/032_config_class.png" | relative_url }})

<br/>

The starting point for the app:

![image tooltip here]({{ "/assets/flask/blueprint/033_run_app.png" | relative_url }})

<br/>

## Custom error pages

<br/>

The below code defines error handlers for a Flask app using a Blueprint named "errors." Three error handlers are defined: one for 404 (Not Found), one for 403 (Forbidden), and one for 500 (Internal Server Error). Each function renders an appropriate HTML error page for the respective error code.

![image tooltip here]({{ "/assets/flask/error_pages/034_error_handler.png" | relative_url }})

<br/>

This HTML template handles a 403 (Forbidden) error. It extends a base layout and includes a section with a message indicating that the user doesn't have permission to perform the action. The same is done for other status codes.

![image tooltip here]({{ "/assets/flask/error_pages/035_error_page.png" | relative_url }})

<br/>

The below code initializes the Flask application. It sets up various components like the database (db), bcrypt for hashing (bcrypt), login manager (login_manager), and mail service (mail). The app registers several blueprints for different parts of the application, such as users, posts, main content, and error handling routes.

![image tooltip here]({{ "/assets/flask/error_pages/036_add_init.png" | relative_url }})

<br/>

[repo-link]: https://github.com/siddhesh2263/flask_corey