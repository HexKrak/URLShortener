# URLShortener
Easy URL shortener
Check out an example of this live at [hau.me][http://hau.me]

## Prerequisites 
You'll need a machine or virtual machine capable of running docker. If you're running on a windows environment I suggest booting up a [coreos virtual machine using vagrant.][https://coreos.com/os/docs/latest/booting-on-vagrant.html]

## Installation
Clone this git repository and build the docker containers
```
$ git clone https://github.com/HexKrak/URLShortener.git
$ cd URLShortener
$ docker build -t shorten-app ./shorten-app
$ docker build -t shorten-api ./shorten-api
$ docker build -t shorten-db ./shorten-db
```

Run the container instances
```
#first a mariadb instance, add "-p 3306:3306" if you need to access the database via an external tool.
$ docker run --name shorten-db -v /var/www/URLShortener/shorten-db/docker-resource/:/var/lib/mysql/Shorten/ -e MYSQL_ROOT_PASSWORD=my-secret-pw -d shorten-db
#second spin up the api with a public port of 8081 and link it to the mariadb instance
$ docker run --name shorten-api -v /var/www/URLShortener/shorten-api/api:/usr/share/nginx/html --link shorten-db:mysql -p 8081:80 -d shorten-api
#second spin up the app with a public port of 80
$ docker run --name shorten-app -v /var/www/URLShortener/shorten-app/app:/usr/share/nginx/html -p 80:80 -i -t -d shorten-app
```

## How this project came together
First, I created a docker stragegy that will run the app and the api and a database all on individual docks. The api container gets NginX installed with zend framework. The app will run NginX without any extras.
I'll use a prebuilt mariadb database container for my database, and import the table structure during container build time.

After investigating pre-built scaffolds like angular-seed and yeoman's angular generator I decided to piecemeal my frontend build with angular, grunt, and various grunt plugins. Grunt will handle the build process, testing, compiling sass, and minifying both the css and js. Angular is an MVC style javscript framework that will help create a seemless organized oop front end.

First I built a basic package.json file in the root of the project. This is what npm will use as a project manafest. Then using npm install I added grunt and it's plugins:
```
$ npm install grunt grunt-contrib-compass grunt-contrib-cssmin grunt-contrib-jshint grunt-contrib-uglify grunt-contrib-watch jshint-stylish karma --save-dev
```
The --save-dev flag adds the modules to my package.json file so the next time I clone this from git I can just run npm install and it will pull all the plugins down for me automatically.

Next I started configuring Gruntfile.js adding build and watcher instructions. Jshint is a syntax checker, which parses my js files checking for errors and warnings. Uglify puts all of my js files together in one file and minifys. Compass builds my scss files into css, and sssmin puts the style sheets together and minifys them. By configuring watch I can run $ grunt watch and any time I save a css or js file it will automatically run uglify or compass and cssmin based on which type of file is updated. This way I can just add the production files in my html and still maintain a clean development archetecture. 

With grunt properly configured I put together my angular core, main controller, and main service, and added them to my index. I configured the list display and add url methods with dummy data, and now it's time to work on the api.

I set up a docker file with the nginx base to install and configure php in a way that is compatable with zend. I cloned the top level of zf1 into my api directory and used the zf command to create a project scaffold. I then included the relative path to the library in the index.php file. Then I added all the appropriate abstract functions to the class and added some dummy data. At this point I realize a full database server is probably unneccesary, and I could use memcache or mongoDB instead, but for the purpose of the demo I'll continue with a mariaDB/mysql server.

Now that zend framework is returning dummy data, I'll replace the dummy data in the front end with content provided by the api. To do this I'll add api interfaces in the main.service and call those from the controller.

Next get the database set up and configure all the input and output, and link it up to the API. After that's done, and the page is populating with live data I need to configure the redirect portion.

TODO:
Fix stack size error that occures when adding a url, following the redirect then pressing back until back to the index.
Add database structure to mariadb initialization.
Change display to show full short url for easy copy. 

