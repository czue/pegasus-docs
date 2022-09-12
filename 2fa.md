# Two-Factor Authentication

Two-Factor authentication (2FA) is configured using the [django-allauth-2fa][1] library which adapts
[django-otp][2] to make it work with [django-allauth][3].

[1]: https://django-allauth-2fa.readthedocs.io/en/latest/
[2]: https://django-otp-official.readthedocs.io/en/latest/
[3]: https://www.intenct.nl/projects/django-allauth/

When using Two-Factor Auth with Pegasus, a new section is added to the user profile
for enabling & configuring the OTP (one-time password) devices for the user.

If a user has a Two-Factor device configured then they will be prompted for a token after logging in.

If you wish to force users to enable Two-Factor Auth on their account you can follow the example
given in the [django-allauth-2fa docs][4].

[4]: https://django-allauth-2fa.readthedocs.io/en/latest/advanced/#forcing-a-user-to-use-2fa
