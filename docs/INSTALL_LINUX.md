## Instalación en Linux
Sailor es compatible con una variedad de sistemas operativos y servidores web. En este ejemplo, utilizaremos Ubuntu y Apache 2.4.

### Instalando Lua

Si aún no lo tienes, instala Lua. Sailor es compatible con las versiones 5.1 y 5.2 
    sudo apt-get install lua5.1

### Instalando Apache 2.4

En Ubuntu 13.10 y superiores, utiliza el siguiente comando para instalar Apache 2.4.x:

    sudo apt-get install apache2

Para Ubuntu 13.04 e inferiores tienes Apache 2.2 instalado por defecto, pero no puedes utilizar los scripts de Lua. Se recomienda que utilices la versión más reciente de Apache que puedas. Puedes seguir las instrucciones de instalación en http://httpd.apache.org. Si esto te suena un poco complicado, puedes obtener Apache 2.4.12 añadiendo añadiendo el siguiente repositorio e instalandolo via apt-get:

    sudo apt-add-repository ppa:ondrej/apache2
    sudo apt-get update
    sudo apt-get install apache2

If you are compiling apache by yourself it might be necessary to add the flag --with-lua during ./configure.

Enable mod_lua

    a2enmod lua

On some versions where a2enmod command is not available, you may enable it alternatively by uncommenting this line on httpd.conf

    LoadModule lua_module modules/mod_lua.so

Restart Apache

    service apache2 restart

### Installing Sailor
You can either clone it directly from the repository, download the zip containing the master branch or download and install it through LuaRocks. We will go through LuaRocks since it will also download and install almost all the required dependencies except for luasql-mysql if you want to persist your models.

    sudo apt-get install luarocks
    luarocks install sailor

We are almost done! You can now use `sailor` to create your web applications. In this example, we will create an app called "Hey Arnold" on the directory Apache is reading from (usually /var/www). After you're done, you can open your browser and access it on http://localhost/hey_arnold

    sailor create 'Hey Arnold' /var/www

Alternatively, you can manually copy the files in the /src/sailor/blank-app directory of this repository to /var/www/hey_arnold and access it at <http://localhost/hey_arnold> and if you didn't install sailor through LuaRocks, you must open .htaccess and replace {{path}} with the full path on your system to Sailor's src directory.

#### Dependencies
If you want to persist your models you need luasql. Sailor could work with other drivers but so far we've only tested with mysql and don't offer support for others.

    luarocks install luasql-mysql

If you installed Sailor through LuaRocks, there is no need to worry, all next dependencies will be installed with it and you can ignore the rest of this section. If you just cloned the repository or downloaded the zip, you should install these dependencies:

Lua File System and valua are required.

    luarocks install luafilesystem
    luarocks install valua

If you want to save your models in a database, you will need LuaSQL. I believe it should work with every database LuaSQL supports, but so far I have only tested with MySQL. LuaSQL-MySQL requires you to have mysql installed.

    luarocks install LuaSQL-MySQL

If you want to use our mailer module, get these rocks so we are able to send stuff via SMTP.

    luarocks install LuaSocket
    luarocks install LuaSec

LuaSec requires OpenSSL as a dependency, if you don't have it already please install it and try getting LuaSec again. Remember to install "devel" packages, if your distro has them, to get the header files! If LuaSec can't find your OpenSSL dir, try using these flags, depending on your system's architecture (the examples below work on some Linux distros).

    luarocks install LuaSec OPENSSL_LIBDIR=/usr/lib/x86_64-linux-gnu
or

    luarocks install LuaSec OPENSSL_LIBDIR=/usr/lib/i386-linux-gnu

###Alternative Installation with Nginx

Install Nginx as explained at <http://wiki.nginx.org/Install#Official_Debian.2FUbuntu_packages>

Install the ngx_lua module (aka HttpLuaModule) as explained at <http://wiki.nginx.org/HttpLuaModule#Installation>

Open the nginx.conf file and add to the http block:

    lua_package_path 'path/to/lua/?.lua;path/to/html/hey_arnold/?.lua;';
    lua_package_cpath 'path/to/clibs/?.so;';

Don't forget to replace path/to with the actual path to your Lua installation and the Nginx document root.

You must also add to the server block:

    location ~ ^/hey_arnold/(.+) {
        lua_need_request_body on;
        lua_code_cache off;
        content_by_lua_file html/hey_arnold/$1;
        index  index.lua index.lp;
    }
    location ~* ^.+\.(?:css|eot|js|json|png|svg|ttf|woff)$ { }

Now run nginx and go to http://localhost/hey_arnold/index.lua?r=main in your browser.

###Alternative Installation with Lighttpd

Install Lighttpd as explained at <http://redmine.lighttpd.net/projects/lighttpd/wiki/GetLighttpd>

Copy the files in the `src/sailor/demo-app` directory of this repository to the htdocs/sailor directory of the Lighttpd dir.

####Configuring Lighttpd

Open the `conf\lighttpd.conf` file, uncomment mod_magnet in server.modules, and add the following lines right after index-file.names:

    $HTTP["url"] =~ "^/sailor/index.lua" {                    
        magnet.attract-physical-path-to = ( server_root + "/htdocs/sailor/index-magnet.lua")
    }
    $HTTP["url"] =~ "^/sailor/(conf|controllers|models|runtime|views)/" {                
        url.access-deny = ("")
        dir-listing.activate = "disable" 
    }
    $HTTP["url"] =~ "^/sailor/themes/" {                
        url.access-deny = (".lp")
        dir-listing.activate = "disable" 
    }
    
####Done!

Now run Lighttpd and go to <http://localhost/sailor/index.lua?r=main> in your browser.