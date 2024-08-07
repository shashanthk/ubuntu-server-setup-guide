## Creating Tomcat custom error page

## Global handling

1. Create error HTML pages for all standard error codes either manually or using an automated script. Use filenames like:

    ```
    400.html
    401.html
    ... so on
    ```

Alternatively, use the shell script from the following GitHub Gist to generate these files automatically:

    https://gist.github.com/shashanthk/d79ae47e3eb3ff9e38aaffce308acc3e

Download the script to your local environment and execute

```sh
cd /tmp

wget https://gist.githubusercontent.com/shashanthk/d79ae47e3eb3ff9e38aaffce308acc3e/raw/4d0722f4c86fa91fc90c1f063066cfdbaa9c6866/generate_error_files.sh

sudo chmod 775 generate_error_files.sh

./generate_error_files.sh
```

It will generate files for you.

2. Place the generated error pages in a common location accessible by Tomcat, e.g., `/var/www/tomcat-error`. Make sure Tomcat has read access to this location:

        /var/www/tomcat-error

And grant permissions

    sudo chown -R tomcat:tomcat /var/www/tomcat-error

3. Add the [`Error Report Valve`](https://tomcat.apache.org/tomcat-10.0-doc/config/valve.html#Error_Report_Valve) to the `<tomcat_installation_path>/config/server.xml`

    ```xml
    <Valve className="org.apache.catalina.valves.ErrorReportValve"
	    errorCode.400="/var/www/tomcat-error/400.html"
        errorCode.401="/var/www/tomcat-error/401.html"
        showReport="false"
        showServerInfo="false" 
    />
    ```

To automate this, use a shell script to generate the necessary XML content:

```sh
#!/bin/bash

echo '<Valve className="org.apache.catalina.valves.ErrorReportValve"'

error_codes=(400 401 402 403 404 405 406 407 408 409 410 411 412 413 414 415 416 417 418 421 422 423 424 425 426 428 429 431 451 500 501 502 503 504 505 506 507 508 510 511)

for code in "${error_codes[@]}"; do
    echo "       errorCode.${code}=\"/var/www/tomcat-error/${code}.html\""
done

echo '       showReport="false"'
echo '       showServerInfo="false" />'
```

Copy the generated content into the `server.xml` file between the `<Host></Host>` tags.

The params `showReport="false"` and `showServerInfo="false"` will prevent the full error stack track and Tomcat version disclosure in case Tomcat fails to display the custom error pages.

Save and exit from the text editor.

4. Restart the Tomcat and try with below params to test the custom error pages.

    ```sh
    https://tomcat.domain/%%%            ## 400 bad request
    https://tomcat.domain/sdfhjsdf      ## 404 page
    https://tomcat.domain/manager/html      ## click on cancel button when prompted for credential and it should show 401 error
    ```

## The manual way (individual webapps)

1. Create below error pages

    ```
    400.jsp
    401.jsp
    403.jsp
    404.jsp
    405.jsp
    500.jsp
    502.jsp
    ```

    Open the the file in any text editor and add the content

    ```html
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>401 Unauthorized</title>
    </head>

        <body>
            <h4>
                401 Unauthorized
            </h4>
        </body>

    </html>
    ```

    Update all the files according to the HTTP response code and message.

2. Tomcat's default page is `ROOT.war`. We need to create a directory to keep above created files on the server and tell Tomcat to use these files as default error page.

    Create a directory inside `<tomcat_installation_path>/webapps/ROOT/WEB-INF` path

    ```sh
    cd <tomcat_installation_path>/webapps/ROOT/WEB-INF
    
    sudo mkdir jsp
    ```

    After creating the directory, place all the above files inside it and update the permission

    ```sh
    sudo chown -R tomcat:tomcat jsp
    ```

3. Now, we need to configure these files as default error page

    ```sh
    sudo nano <tomcat_installation_path>/webapps/ROOT/WEB-INF/web.xml
    ```

    Add below lines inside `<web-app>...</web-app>`

    ```xml
    <error-page>
        <error-code>404</error-code>
        <location>/WEB-INF/jsp/404.jsp</location>
    </error-page>
    ```

    Add all the error codes accordingly and change the file names. Save and exit from text editor.

4. Restart the Tomcat service to check the result

    ```sh
    sudo systemctl restart tomcat.service
    ```

5. Visit to any invalid location

    ```
    https://tomcaturl.com/abcd
    ```

    Now, Tomcat should return custom 404 error page.

6. If above config works fine, we need to do the same settings for Tomcat manager page.

    Tomcat manager UI can be found the following location

    ```
    <tomcat_installation_path>/webapps/manager
    ```

    Please note that the manager UI has `jsp` directory and error pages in it by default. Instead of altering existing files, it's recommended to keep a backup of default files.

    ```sh
    cd <tomcat_installation_path>/webapps/manager/WEB-INF

    sudo cp jsp jsp.bak

    sudo cp web.xml web.xml.bak
    ```

    Now, copy the custom error pages to the below location

    ```
    <tomcat_installation_path>/webapps/manager/WEB-INF/jsp
    ```

    Update file permission

    ```sh
    sudo chown -R tomcat:tomcat <tomcat_installation_path>/webapps/manager/WEB-INF/jsp
    ```

    Update `web.xml` file to use custom error pages and update the content inside `<web-app>...</web-app>` block.

    ```sh
    sudo nano <tomcat_installation_path>/webapps/manager/WEB-INF/web.xml
    ```

    Save and exit from editor.

    Note: Same procedure needs to be followed for `host-manager` web app.

7. Restart the Tomcat service

    ```sh
    sudo systemctl restart tomcat.service
    ```
    
8. At last we also need to hide the Tomcat's default page which will give access to the host manager.

    ```sh
    cd <tomcat_installation_path>/webapps/ROOT
    ```

    Keep a copy of `index.jsp` and `favicon.ico` files. If you don't want, you can delete them.

    ```sh 
    cp index.jsp index.jsp.bak
    
    cp favicon.ico favicon.ico.bak

    sudo nano <tomcat_installation_path>/webapps/ROOT/index.jsp
    ```

    Add below HTML content to it

    ```html
    <!DOCTYPE html>
    <html lang="en">

    <head>
        <meta charset="UTF-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Root</title>
    </head>

    <body>
        <h4>
            Root
        </h4>
    </body>

    </html>
    ```

9. Once you change these contents, you will not be able to see the Tomcat's default page. If you want to login to manager, you have to manually enter the URL

    ```
    https://tomcaturl.com/manager/html
    ```

    The above URL propts you to enter the login credentials.
