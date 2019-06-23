# Filebrowser

[https://github.com/filebrowser/filebrowser](https://github.com/filebrowser/filebrowser)

This program is not like h5ai. h5ai is a "modern file indexer for HTTP web servers" that runs from your WWW directory powered by apache or nginx. Filebrowers is a golang program that does something different.

What this does is allow you to manage your slot files directly from the web browser since it runs via ssh and is proxypassed using nginx in this example.

You can create separate configured users that are limited to a specific folder and permissions. Then you can create users with limited permissions to access the data in that folder via a browser.

[https://filebrowser.github.io/configuration/](https://filebrowser.github.io/configuration/)

## The setup in this readme is this:

The binary is installed to and run from the `~/bin`.

We point the binary to a configuration database `~/.config/filebrowser/filebrowser.db` using `-d`

We proxypass it using nginx to be available by https only.

We start filebrowser in a screen called `filebrowser` using the custom configuration file.

You will have will be able to access the progam via `http://servername.feralhosting.com/username/filebrowser` to login as admin

The default username is `admin` and the default password is `admin` ( you should change this in the settings/profile section )

The below commands can be copy and pasted in one go and will download, install and configure filebrowser with nginx on Feralhosting. The commands can be tweaked to work with other providers.

~~~bash
mkdir -p ~/{bin,.config/filebrowser}
wget -qO ~/filebrowser.tar.gz "$(curl -sNL https://git.io/fxQ38 | grep -Po 'ht(.*)linux-amd64(.*)gz')"
tar xf ~/filebrowser.tar.gz --exclude LICENSE --exclude README.md -C ~/bin
#
# Proxypass
wget -qO ~/.nginx/conf.d/000-default-server.d/filebrowser.conf https://git.io/vpSav
sed -i 's|# rewrite /(.*) /username/$1 break;|rewrite /(.*) /username/$1 break;|g' ~/.nginx/conf.d/000-default-server.d/filebrowser.conf
sed -i 's|HOME|'"$HOME"'|g' ~/.nginx/conf.d/000-default-server.d/filebrowser.conf
sed -i 's|generic|filebrowser|g' ~/.nginx/conf.d/000-default-server.d/filebrowser.conf
sed -i 's|username|'"$(whoami)"'|g' ~/.nginx/conf.d/000-default-server.d/filebrowser.conf
#
# Port Generation
app_port="$(shuf -i 10001-32001 -n 1)" && while [[ "$(ss -ln | grep -co ''"$app_port"'')" -ge "1" ]]; do app_port="$(shuf -i 10001-32001 -n 1)"; done
sed -i 's|PORT|'"$app_port"'|g' ~/.nginx/conf.d/000-default-server.d/filebrowser.conf
#
# Configuration
filebrowser config init -d ~/.config/filebrowser/filebrowser.db > /dev/null 2>&1
filebrowser config set -a 10.0.0.1 -p $app_port -b "/$(whoami)/filebrowser" -l ~/.config/filebrowser/filebrowser.log -d ~/.config/filebrowser/filebrowser.db > /dev/null 2>&1
filebrowser users add admin admin --perm.admin -d ~/.config/filebrowser/filebrowser.db > /dev/null 2>&1
#
# Reload nginx - ignore the errors
/usr/sbin/nginx -s reload -c ~/.nginx/nginx.conf 2> /dev/null
#
# Start the program in a screen called filebrowser
screen -dmS "filebrowser" && screen -S "filebrowser" -p 0 -X stuff "filebrowser -d $HOME/.config/filebrowser/filebrowser.db^M"
#
echo "Just an echo to return our prompt if you copied and pasted all these commands"
~~~