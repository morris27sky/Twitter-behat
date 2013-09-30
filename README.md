Pre req:

php 5.3

Installation
================
mkdir behat-twitter; 
cd behat-testing-ftw;
curl -s http://getcomposer.org/installer | php
vi composer.json

Composer
=================
php composer.phar install

Dir should look like this
bin     composer.json   composer.lock   composer.phar   vendor


Now run bin/behat --init which should creat the following

+d features - place your *.feature files here
+d features/bootstrap - place bootstrap scripts and static files here
+f features/bootstrap/FeatureContext.php - place your feature related code here

Feature
================
cd features
vi TwitterDirectMessaging.feature

copy and paste the feature f



