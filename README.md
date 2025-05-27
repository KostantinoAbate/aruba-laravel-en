
# Laravel Deployment on Aruba

[Guida in Italiano qui](https://github.com/KostantinoAbate/aruba-laravel-it)

Deploying a Laravel project from a development environment to production on an [Aruba](https://www.aruba.it/) server requires specific steps.

For this guide, I refer to projects that meet the following conditions:

 - **Laravel 11.x** or later  
 - **Linux Basic** or **Advanced** Hosting (with **SSH** access)  

In other scenarios, the steps might differ, or the deployment might even be impossible.

The steps to follow are:

 1. [Configuring the .env file for production](#env)
 2. [Uploading files via FTP](#ftp)
 3. [Managing .htaccess files](#htaccess)
 4. [Recap](#recap)

You might encounter other issues; in that case, see the [FAQ / Troubleshooting](#faq)

 - [Forbidden 403](#403)
 - [Error 505](505)
 - [Vite fails to load assets](vite)
 - ["$variable is undefined"](undefined)
 - [Other errors](altro)

<a name="env" id="env"></a>
# 1. Configuring the .env file for production

The `.env` file must be modified to work correctly in production.

## APP_ENV


The `.env` file must be modified to work correctly in production.

## APP_ENV

The `APP_ENV` variable, which defaults to `local`, should be set to `production`.

> `APP_ENV` tells Laravel how to behave in various situations. When set to `production` it will ask for confirmation before running migrations, prevent certain commands from running, and introduce other security measures.

So:

```dotenv
APP_ENV=production
```

## APP_DEBUG

The `APP_DEBUG` variable, which defaults to `true`, should be set to `false`.

So

```dotenv
APP_DEBUG=false
```

Optionally, **when uploading the project for the first time**, it may make sense to keep it as `true` to verify everything works correctly, then disable it afterward.

## APP_KEY

Ensure the `APP_KEY` variable has a value; otherwise run:

```bash
php  artisan  key:generate
```

## APP_URL

Ensure the domain you are deploying to uses the `http` or `https` protocol, then set the correct URL.

```dotenv
APP_URL=https://mysite.com
```

## Database Connection

Make sure the project points to the correct database.

```dotenv
DB_HOST=00.00.00.000
```
The host is shown in the MySQL dashboard.
 
```dotenv
DB_DATABASE=Sql0000000_1
```

The database name is shown in the left sidebar; it’s formed by the username + `_1`, `_2`, `_3`, etc.

```dotenv
DB_USERNAME=Sql0000000
```

The username is shown in the MySQL dashboard.

```dotenv
DB_PASSWORD=**********
```
Enter the database password.

## Importing the Database
If you don’t have SSH access, to import the database first export it as a `.sql` file from your development database manager. Then import it into the database you specified in the `.env` file.

If you encounter import errors, manually inspect the `.sql` file. The most common error is related to **foreign keys** and improperly managed chronological references.

<a name="ftp" id="ftp"></a>
# FTP Upload

After fixing the `.env` file and setting up the database, prepare the project for upload.

First, run the following commands:

```bash
php  artisan  clear-compiled
npm  run  build
composer  dump-autoload
```

## Upload
The following method is the fastest way to achieve this result, but if you prefer other methods, they will work as well.

1. Zip the entire project into a `.zip` archive (other formats might not be readable by the FTP client or Aruba’s file manager).

2. Connect via FTP to the server and upload the zip file to the domain’s main root. The result should look like this. You can **delete** any previous files in the root, such as `/cgi-bin`, `index.php`, and `ver.php`.
    
        /www.mysite.it
    	    myproject.zip
3. Go to Aruba’s **File Manager** (found in the Linux Hosting section). Right-click the `.zip` file you uploaded and select **Extract archive → Here**. It will take some time depending on the project size (usually a few minutes).
4. Once the project is unpacked, you can remove the `.zip` file and the structure will look like this:
    
        /www.mysite.it
    	    /app
    	    /bootstrap
    	    /config
    	    /database
    	    /lang
    	    /node_modules
    	    /public
    	    /resources
    	    /routes
    	    /storage
    	    /tests
    	    /vendor
    	    .editorconfig
    	    .env
    	    .gitattributes
    	    .gitignore
    	    artisan
    	    composer.json
    	    composer.lock
    	    package-lock.json
    	    package.json
    	    phpunit.xml
    	    postcss.config.js
    	    README.md
    	    tailwind.config.js
    	    vite.config.js
  
  <a name="htaccess" id="htaccess"></a>  	    
# Managing .htaccess files

To allow the server to point to the `/public` folder and make Laravel work correctly, add a `.htaccess` file in the project’s main root (in `/www.mysite.com`).

The `.htaccess` file content should be:

    RewriteEngine on
    RewriteBase /
    
    RewriteCond %{REQUEST_URI} !(/$|\.)
    RewriteCond %{REQUEST_FILENAME} -d
    RewriteRule ^(.*[^/])$ $1/ [R=301,L]
    
    RewriteCond %{REQUEST_URI} !^/public/
    
    RewriteCond %{REQUEST_URI} !(/$|\.)
    RewriteCond %{DOCUMENT_ROOT}/public%{REQUEST_URI} -d
    RewriteRule ^(.*)$ /$1/ [R=301,L]
    
    RewriteCond %{REQUEST_URI} !^/public/
    RewriteRule ^(.*)$ /public/$1 [L]

The most important part is the last two lines, which instruct the redirect to the `/public` folder.
<a name="recap" id="recap"></a>
# Recap

To recap, the steps are:

1. Update the `.env` for production
2. Import the database
3. Run `php artisan optimize:clear`, `php artisan clear-compiled`, `npm run build`, `composer dump-autoload`
4. Upload the project via `.zip` and extract it in the main root using File Manager
5. Create the [.htaccess file](#htaccess)

<a name="faq" id="faq"></a>
# FAQ / Troubleshooting

Errors and issues of various kinds can occur during deployment. The most common ones, for which I rarely found online answers, are:
<a name="403" id="403"></a>
## Forbidden 403
If you see a “Forbidden 403” error, it’s very likely the `.htaccess` file in the project’s main root wasn’t created or written correctly. This results in a redirection error and the inability to access the `/public` folder properly.
<a name="505" id="505"></a>
## ERROR 505
If you see Laravel’s classic “ERROR 505” page, rejoice: since it’s a Laravel asset, it means Laravel is working correctly but cannot retrieve the desired resources. In this case, update the `.env` file and set `APP_DEBUG` to `true`. This way you can see the error’s origin and fix it directly in production. Remember to set `APP_DEBUG=false` afterward.
<a name="vite" id="vite"></a>
## Vite fails to load assets
If the content loads correctly but scripts and CSS do not, it’s most likely because the hot file was uploaded in the `/public` folder. Simply **remove** this file to get the interface working properly.
<a name="undefined" id="undefined"></a>
## "$variable is undefined"
If you encounter an error like “**$variable is undefined**” and you’ve ruled out communication issues with the server or coding mistakes on your part (i.e., if you have no idea how to solve it)… it’s very likely a **case sensitivity** issue with classes.

To solve it, scan the entire `/app` folder of your project and ensure that both directories and `.php` files start with an uppercase letter. Also update all `.php` files to ensure the namespace matches and the class name is correct.

**Incorrect example:**

    /app
	    /View
		    /Components
			    /test
				    /mycomponent
Where `mycomponent.php` is:
```php
<?php

namespace  App\View\Components\test;
use  Closure;
use  Illuminate\Contracts\View\View;
use  Illuminate\View\Component;

class  mycomponent  extends  Component
{
	...
}
```
**Correct example:**

    /app
	    /View
		    /Components
			    /Test
				    /MyComponent
Where `MyComponent.php` is:
```php
<?php

namespace  App\View\Components\Test;
use  Closure;
use  Illuminate\Contracts\View\View;
use  Illuminate\View\Component;

class  MyComponent extends  Component
{
	...
}
```
<a name="altro" id="altro"></a>
## Other errors
If you encounter other errors, it’s essential to determine whether they originate from the server or from Laravel. Compare Aruba’s logs with Laravel’s. A clean Laravel log alongside numerous Aruba error codes indicates the issue is with Aruba. Conversely, a clean Aruba log and messy Laravel log indicates the problem is in the code.
