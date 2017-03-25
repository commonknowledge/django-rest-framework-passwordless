drfpasswordless is a quick way to integrate ‘passwordless’ auth into your Django Rest Framework project using a user’s email address or mobile number only (herein referred to as an alias).

Built to work with DRF’s own TokenAuthentication system, it sends the user a 6-digit callback token to a given email address or a mobile number. The user sends it back correctly and they’re given an authentication token (again, provided by Django Rest Framework’s TokenAuthentication system).

Callback tokens by default expire after 15 minutes.

## Example Usage:

	curl -X POST -d “email=aaron@email.com” localhost:8000/auth/email/


Email to aaron@email.com:

	…
	<h1>Your login token is 815381.</h1>
	…

Return Stage

	curl -X POST -d "token=815381" localhost:8000/callback/auth/

	> HTTP/1.0 200 OK
	> {"token":"76be2d9ecfaf5fa4226d722bzdd8a4fff207ed0e”}

## Requirements

	Python 3
	Django
	Django Rest Framework + AuthToken
	Python-Twilio (Optional, for mobile.)

## Quickstart
1. Add Django Rest Framework’s Token Authentication to your Django Rest Framework project.

	REST_FRAMEWORK = {
			'DEFAULT_AUTHENTICATION_CLASSES': (
			//…
	   'rest_framework.authentication.TokenAuthentication',
	    ),
	}

	INSTALLED_APPS = [
    …
    'rest_framework.authtoken',
    …
	]

And run `manage.py migrate`.

2. Set which types of contact points are allowed for auth in your Settings.py. The available options are `EMAIL` and `MOBILE`.

		PASSWORDLESS_AUTH = {
	    //…
	    ‘PASSWORDLESS_AUTH_TYPES’: [‘EMAIL’, ‘MOBILE’],
	    //…
		}

3. Add `drfpasswordless.urls` to your urls.py

		urlpatterns = [
		//..
		url(r'^', include('drfpasswordless.urls')),
		//..
		]

4. Add an email or mobile number field to your User model. By default drfpasswordless looks for fields named `email` or `mobile` on the User model. If an alias provided doesn’t belong to any given user, a new user is created.

  4a. If you’re using `email`, see the Configuring Email section below.

  4b. If you’re using `mobile`, see the Configuring Email section below.

5. You can now POST to either of the endpoints:

		curl -X POST -d "email=aaron@email.com" localhost:8000/auth/email/

		curl -X POST -d "mobile=+15552143912" localhost:8000/mobile/

A 6 digit callback token will be sent to the contact point.

6. The client has 15 minutes to use the 6 digit callback token correctly. If successful, they get an authorization token in exchange which the client can then use with Django Rest Framework’s TokenAuthentication scheme.

		curl -X POST -d "token=815381" localhost:8000/callback/auth/

		> HTTP/1.0 200 OK
		> {"token":"76be2d9ecfaf5fa4226d722bzdd8a4fff207ed0e”}

### Configuring Emails
Specify the email address you’d like to send the callback token from with the `PASSWORDLESS_EMAIL_NOREPLY_ADDRESS` setting.

You’ll also need to set up an SMTP server to send emails ([See Django Docs](https://docs.djangoproject.com/en/1.10/topics/email/)), but for development you can set up a dummy development smtp server to test emails. Sent emails will print to the console. [Read more here.](https://docs.djangoproject.com/en/1.10/topics/email/#configuring-email-for-development)

	# Settings.py
	…
	EMAIL_HOST = 'localhost'
	EMAIL_PORT = 1025

Then run the following:

	python -m smtpd -n -c DebuggingServer localhost:1025

### Configuring Mobile
You’ll need to have the python twilio module installed

    pip install twilio

and set the `TWILIO_ACCOUNT_SID` and `TWILIO_AUTH_TOKEN` environment variables.

You’ll also need to specify the number you send the token from with the `PASSWORDLESS_MOBILE_NOREPLY_NUMBER` setting.

## Templates
If you’d like to use a custom email template for your email callback token, specify your template name with this setting:

	PASSWORDLESS_AUTH = {
		//…
		'PASSWORDLESS_EMAIL_TOKEN_HTML_TEMPLATE_NAME': "mytemplate.html"
	}

The template renders a single variable `{{ callback_token }}` which is the 6 digit callback token being sent.

## Contact Point Validation
Endpoints can automatically mark themselves as validated when a user logs in with a token sent to a specific endpoint. They can also automatically mark themselves as invalid when a user changes a contact point.

This is off by default but can be turned on with `PASSWORDLESS_USER_MARK_VERIFIED_EMAIL` or `PASSWORDLESS_USER_MARK_VERIFIED_MOBILE`. By default when these are enabled they look for the User model fields `email_verified` or `mobile_verified`.

## Registration
all unrecognized emails and mobile numbers create new accounts by default. New accounts are automatically set with `set_unusable_password()` but it’s recommended that admins have some type of password.

This can be turned off with the `PASSWORDLESS_REGISTER_NEW_USERS` setting.

## Other Settings
Here’s a full list of the configurable defaults.

	DEFAULTS = {
    # Allowed auth types, can be EMAIL, MOBILE, or both.
    'PASSWORDLESS_AUTH_TYPES': ['EMAIL'],

    # Amount of time that tokens last, in seconds
    'PASSWORDLESS_TOKEN_EXPIRE_TIME': 15 * 60,

    # The user's email field name
    'PASSWORDLESS_USER_EMAIL_FIELD_NAME': 'email',

    # The user's mobile field name
    'PASSWORDLESS_USER_MOBILE_FIELD_NAME': 'mobile',

    # Marks itself as verified the first time a user completes auth via token.
    # Automatically unmarks itself if email is changed.
    'PASSWORDLESS_USER_MARK_VERIFIED_EMAIL': False,
    'PASSWORDLESS_USER_EMAIL_VERIFIED_FIELD_NAME': 'email_verified',

    # Marks itself as verified the first time a user completes auth via token.
    # Automatically unmarks itself if mobile number is changed.
    'PASSWORDLESS_USER_MARK_VERIFIED_MOBILE': False,
    'PASSWORDLESS_USER_MOBILE_VERIFIED_FIELD_NAME': 'mobile_verified',

    # The email the callback token is sent from
    'PASSWORDLESS_EMAIL_NOREPLY_ADDRESS': None,

    # The email subject
    'PASSWORDLESS_EMAIL_SUBJECT': "Your Login Token",

    # A plaintext email message overridden by the html message. Takes one string.
    'PASSWORDLESS_EMAIL_PLAINTEXT_MESSAGE': "Enter this token to sign in: %s",

    # The email template name.
    'PASSWORDLESS_EMAIL_TOKEN_HTML_TEMPLATE_NAME': "passwordless_default_token_email.html",

    # The SMS sent to mobile users logging in. Takes one string.
    'PASSWORDLESS_MOBILE_MESSAGE': "Use this code to log in: %s"

    # Registers previously unseen aliases as new users.
    'PASSWORDLESS_REGISTER_NEW_USERS': True
		}


### Todo
- Support non-US mobile numbers
- Tests
- Custom URLs
