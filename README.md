# Saint ParseServer SMTP Email Adapter
### This fork caters for the twig template rendering.

With this adapter you can send email for reset password and email verification in parse with SMTP access and custom twig templates.

### Installation

Install npm module in your parse server project

```sh
$ npm install --save @saintborn/parse-smtp-adapter
```

### Use

In the configuration of your parse server you must pass `@saintborn/parse-smtp-adapter` as email adapter and set your SMTP access for send emails also the path to your jade template and its less file.

This is an example using parse server as express module:

```js
"use strict";

const Express = require('express');
const ParseServer = require('parse-server').ParseServer;

const app = Express();
const APP_PORT = 1337;

let options = {

};

let api = new ParseServer({
	appName: "Parse Test",
	appId: "12345",
	masterKey: "abcde12345",
	serverURL: "http://localhost:1337/parse",
	publicServerURL: "http://localhost:1337/parse",
	databaseURI: "mongodb://user:pass@host:27017/parse",
	port: APP_PORT,
	//This is the config for email adapter
	emailAdapter: {
		module: "@saintborn/parse-smtp-adapter",
		options: {
			fromAddress: 'your@sender.address',
			user: 'email@email.com', //#"required for service SMTP"
			password: 'AwesomePassword',  //#"required for service SMTP"
			host: 'your.smtp.host',  //#"required for service SMTP"
			isSSL: true, //True or false if you are using ssl  //#"required for service SMTP"
			port: 465, //SSL port or another port //#"required for service SMTP"
			name: 'your domain name', //  optional, used for identifying to the server 
			//Somtimes the user email is not in the 'email' field, the email is search first in
			//email field, then in username field, if you have the user email in another field
			//You can specify here
			emailField: 'username', 
			templates: {
				templateExtension: 'twig',
				templateRoot: __dirname + '/views/email/templates/',
				resetPassword: {
					template: 'reset-password',
				},
				verifyEmail: {
					template: 'verify-email',
				}
			}
		}
	}
});

/**
 * Parse Server endpoint
 */
app.use('/parse', api);

app.listen(APP_PORT, function () {
	console.log(`Parse Server Ready and listening on port ${APP_PORT}`);
});
```

### Template
The path you pass to the email adapter must be a directory and not a file, this path must contain 2 mandatory files `html.twig` and `subject.twig` you can do your template as you like with the [CSS rules that emails supports](https://www.campaignmonitor.com/css/) in the template you can use 3 variables:

- appName //This is the name of your parse app
- link //This is the link for reset the password
- user //This is a Parse object with the current user, so you can use any field in your User class of parse for example the user name `{{ user.get('username') }}`

in templates use `<link rel="stylesheet" href="/css/style.css" data-inline>` in the `<head>` of your html and it uses your css from `rootTemplateFolder/css/style.css`

### Example

#### layout.twig
```twig
<!DOCTYPE html>
<html>

<head>
    {% block head %}
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="x-apple-disable-message-reformatting">
        <title>{% block title %}{% endblock %}</title>
        <link rel="stylesheet" href="/css/style.css" data-inline>
    {% endblock %}
</head>

<body>
{% block body %}{% endblock %}
</body>

</html>
```


#### template-name/html.twig
```twig
{% extends '../layout.twig' %}

{% block title %}{{ subject }}{% endblock %}

{% block body %}
    <p>your reset password link is: {{ link }}</p>
{% endblock %}
```

#### template-name/subject.twig
```twig
{{ appName }}: Reset password
```

#### Folder structure

```sh
.
├── app.js
├── ...
└── views
        └── email
                └── templates
                            ├── layout.twig
                            ├── css
                                  └── style.css
                            ├── reset-password
                                             ├── html.twig
                                             └── subject.twig
                            └── verify-email
                                           ├── html.twig
                                           └── subject.twig
```

### License MIT