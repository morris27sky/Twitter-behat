The chosen framework for this excerice will be behat. This is a php framework based on cucumber. see http://behat.org/ for more fetails.

Pre req
================
you will need the following

php 5.3

selenium server - wget http://selenium.googlecode.com/files/selenium-server-standalone-2.35.0.jar

run java -jar selenium-server-standalone-2.33.0.jar to start server


Installation
================
mkdir behat-twitter

cd behat-twitter

curl -s http://getcomposer.org/installer | php

vi composer.json

Composer
=================
php composer.phar install

New directory should look like this


bin     composer.json   composer.lock   composer.phar   vendor


Now run bin/behat --init which should creat the following

+d features - place your *.feature files here
+d features/bootstrap - place bootstrap scripts and static files here
+f features/bootstrap/FeatureContext.php - place your feature related code here

Feature
================
cd features
vi TwitterDirectMessaging.feature

copy and paste the feature from the feature file TwitterDirectMessaging.feature


featureContext
===============

see featureContext.php file

Running a feature
======================
you will need to be in the project dir

To run a test, simply run the following


bin/behat features/TwitterDirectMessaging.feature

For more info on commands for running or report generating in html format or more, see http://docs.behat.org/guides/6.cli.html
