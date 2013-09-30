Twitter
=======

Prereq
- php 5.3
- composer

Installation steps
=================

mkdir behat-ttwitter; 
cd behat-twitter;
curl -s http://getcomposer.org/installer | php

Composer.json
=================
vi composer.json
enter the following in your composer.json and save

{
    "require": {
        "behat/behat": "2.4.*@stable",
        "behat/mink": "1.4.*@stable",
        "behat/mink-extension": "*",
        "behat/mink-goutte-driver": "*",
        "behat/mink-selenium-driver": "*",
        "behat/mink-selenium2-driver": "*",
        "orangedigital/business-selector-extension": "*"
    },
    "minimum-stability": "dev",
    "config": {
        "bin-dir": "bin/"
    }
}

run bin/behat --init

.Feature steps
====================

cd features
vi TwitterDirectMessaging.feature

Copy and paste the following into your .feature file

Feature: Direct messaging on twitter
  Scenario: Two users who are followers of each other can send direct messages
    Given "Joe" and "Julie" are on twitter
    And Julie is a follower of Joe
    And Joe is a follower of Julie
    When Joe sends a DM to Julie
    Then Julie should see Joe’s message

  Scenario: A user cannot send a direct message to someone who is not a follower
    Given "Joe" and "Julie" are on twitter
    And Julie is NOT following Joe
    And Joe is a follower of Julie
    When Joe sends a DM to Julie
    Then Joe should get an error response
    And Julie should not receive Joe’s message

Feature Context
=====================
vi features/bootstrap/FeatureContext.php

Copy and paste into your featureContext.php

<?php

use Behat\Behat\Context\ClosuredContextInterface,
    Behat\Behat\Context\TranslatedContextInterface,
    Behat\Behat\Context\BehatContext,
    Behat\Behat\Exception\PendingException;
use Behat\Gherkin\Node\PyStringNode,
    Behat\Gherkin\Node\TableNode;

use Behat\MinkExtension\Context\MinkContext;
use Behat\MinkExtension\Context\RawMinkContext;

use Selenium\Client as SeleniumClient;

use OrangeDigital\BusinessSelectorExtension\Context\BusinessSelectorContext;

//
// Require 3rd-party libraries here:
//
require_once 'PHPUnit/Autoload.php';
require_once 'PHPUnit/Framework/Assert/Functions.php';
//

/**
 * Features context.
 */
class FeatureContext extends BehatContext
{
 
    public $output;
    public $parameters;
 
    public function __construct(array $parameters)
    {
        $this->parameters = $parameters;
        $this->useContext('mink', new MinkContext($parameters));
        //$this->useContext('business', new BusinessSelectorContext($parameters));
    }
    
    protected function loadYaml($path) {
        if (!file_exists($path)) {
            throw new \RuntimeException('File: ' . $path . ' does not exist');
        }

        $parser = new \Symfony\Component\Yaml\Parser();

        $string = file_get_contents($path);

        $result = $parser->parse($string);

        if (!$result) {
            throw new \RuntimeException('Unable to parse ' . $path);
        }

        return $result;
    }


    public function getSession($name = null)
    {
      return $this->getSubcontext('mink')->getSession($name);
    } 


// Place your definition and hook methods here:
//
//    /**
//     * @Given /^I have done something with "([^"]*)"$/
//     */
//    public function iHaveDoneSomethingWith($argument)
//    {
//        doSomethingWith($argument);
//    }
 
/**
     * @Given /^"([^"]*)" and "([^"]*)" are on twitter$/
     */
    public function andAreOnTwitter($userA, $userB)
    
    {
        $users = $this->loadYaml('users.yaml');
        $session = $this->getSession();

        $url = 'https://twitter.com/'.$users[$userA]['username'];
        $code = $this->getStatusCode($url);
        if ($code != 200) {
         throw new \RuntimeException("user $userA is not on twitter");
        }

        $url = 'https://twitter.com/'.$users[$userB]['username'];
        $code = $this->getStatusCode($url);
        if ($code != 200) {
         throw new \RuntimeException("user $userB is not on twitter");
        }
    }

    protected function getStatusCode($url){
     // Set up curl (disable body, set url etc.)
        $c = curl_init();
        curl_setopt($c, CURLOPT_URL, $url);
        curl_setopt($c, CURLOPT_HEADER, TRUE);
        curl_setopt($c, CURLOPT_NOBODY, TRUE);
        curl_setopt($c, CURLOPT_RETURNTRANSFER, TRUE);
        $header = curl_exec($c);
        $responseCode = curl_getinfo($c, CURLINFO_HTTP_CODE);
        return $responseCode;
    }

/**
* @Given /^Julie is a follower of Joe$/
     */
public function julieIsAFollowerOfJoe()
{
	//Load the user.yml file to be read
        $users = $this->loadYaml('users.yaml');
        $userA = "Julie";
        $userB = "Joe";
	//visit the page
        $url = 'https://twitter.com';
        $session = $this->getSession();
        $session->visit($url);

	//find the login elements on the page
        $pageElement = $session->getPage();
        $emailElement = $pageElement->find('css', '#signin-email');
        $passwordElement = $pageElement->find('css', '#signin-password');
        //$formElement = $pageElement->find('css', 'form.signin button');
	
	//inputing the username and password
        $emailElement->setValue($users[$userA]['username']);
        $passwordElement->setValue($users[$userA]['password']);
        //$formElement = $pageElement->find('css', 'form.signin');
        //$formElement->submit();
	
	//clicking the login button
	    $formElement = $pageElement->find('css', '.submit.btn.primary-btn.flex-table-btn.js-submit');
        //sleep(1);
        $formElement->click();
        //sleep(3);

	//Visit the page /following
        $url = 'https://twitter.com/following';
        $pageElement = $session->getPage();
        $session->visit($url);
        sleep(1);
        //$followElement = $pageElement->find('xpath', '//*[text()=$userA]');
        $followElement = $pageElement->find('xpath', '//*[text()="@'.$users[$userB]['username'].'"]');
        if(!$followElement)
        {
            throw new \RuntimeException("$userA is not following $userB");
        }
    }
   /**
     * @Given /^Joe is a follower of Julie$/
     */
    public function joeIsAFollowerOfJulie()
    {
    //Load the user.yml file to be read
        $users = $this->loadYaml('users.yaml');
        $userA = "Julie";
        $userB = "Joe";
        
    //visit the page
        $url = 'https://twitter.com/followers';
        $session = $this->getSession();
        $session->visit($url);

        sleep(1);
        //$followElement = $pageElement->find('xpath', '//*[text()=$userA]');
        $pageElement = $session->getPage();
        $followElement = $pageElement->find('xpath', '//*[text()="@'.$users[$userB]['username'].'"]');
        if(!$followElement)
        {
            throw new \RuntimeException("$userB is not following $userA");
        }
    }

    /**
     * @When /^Joe sends a DM to Julie$/
     */
    public function joeSendsADmToJulie()
    {

        $users = $this->loadYaml('users.yaml');
        $userA = "Julie";
        $userB = "Joe";

        $url = 'https://twitter.com/logout';
        $session = $this->getSession();
        $session->visit($url);

        $pageElement = $session->getPage();
        $logoutElement = $pageElement->find('css', '.js-submit');
        $logoutElement->click();

        $url = 'https://twitter.com/';
        $session = $this->getSession();
        $session->visit($url);

        $pageElement = $session->getPage();
        $emailElement = $pageElement->find('css', '#signin-email');
        $passwordElement = $pageElement->find('css', '#signin-password');
        //$formElement = $pageElement->find('css', 'form.signin button');
    
    //inputing the username and password
        $emailElement->setValue($users[$userB]['username']);
        $passwordElement->setValue($users[$userB]['password']);
        //$formElement = $pageElement->find('css', 'form.signin');
        //$formElement->submit();
    
    //clicking the login button
        $formElement = $pageElement->find('css', '.submit.btn.primary-btn.flex-table-btn.js-submit');
        //sleep(1);
        $formElement->click();
        //sleep(3);

        //Visit the page /following
        $url = 'https://twitter.com/following';
        $pageElement = $session->getPage();
        $session->visit($url);
        sleep(1);
        $dropdownElement = $pageElement->find('css', '.user-dropdown');
        $dropdownElement->click();
        $dmLinkElement = $pageElement->find('css', '.dm-text.dropdown-link');
        $dmLinkElement->click();
        sleep(5);
        $dmtextElement = $pageElement->find('css', '#tweet-box-dm-new-conversation');
        sleep(5);
        //$dmtextElement->keyPress('a');
        $dmtextElement->setValue('This is a test');
        sleep(5);
        $tweetactionElement = $pageElement->find('css', '#dm_dialog_conversation.modal-content .twttr-dialog-body .tweet-button button');
        sleep(5);
        $tweetactionElement->click();



        //$tweetactionElement = $pageElement->find('css', '#dm_dialog_conversation.modal-content .twttr-dialog-body .tweet-button');
        //sleep(5);
        //$tweetactionElement->click();

    }

    /**
     * @Then /^Julie should see Joe’s message$/
     */
    public function julieShouldSeeJoeSMessage()
    {
        $users = $this->loadYaml('users.yaml');
        $userA = "Julie";
        $userB = "Joe";

        $url = 'https://twitter.com/logout';
        $session = $this->getSession();
        $session->visit($url);

        $pageElement = $session->getPage();
        $logoutElement = $pageElement->find('css', '.js-submit');
        $logoutElement->click();

        $url = 'https://twitter.com/';
        $session = $this->getSession();
        $session->visit($url);

        $pageElement = $session->getPage();
        $emailElement = $pageElement->find('css', '#signin-email');
        $passwordElement = $pageElement->find('css', '#signin-password');
        //$formElement = $pageElement->find('css', 'form.signin button');
    
    //inputing the username and password
        $emailElement->setValue($users[$userA]['username']);
        $passwordElement->setValue($users[$userA]['password']);
        //$formElement = $pageElement->find('css', 'form.signin');
        //$formElement->submit();
    
    //clicking the login button
        $formElement = $pageElement->find('css', '.submit.btn.primary-btn.flex-table-btn.js-submit');
        //sleep(1);
        $formElement->click();

        $dropdownElement = $pageElement->find('css', '.user-dropdown');
        $dropdownElement->click();
        $dmtoggleElement = $pageElement->find('css', '.js-dm-dialog');
        $dmtoggleElement->click();
        $formElement = $pageElement->find('css', '.dm-thread-snippet:contains("this is a test")');

}



/*
        $url = 'https://twitter.com/';
        $session = $this->getSession();
        $session->visit($url);
        $pageElement = $session->getPage();
        
        $users = $this->loadYaml('users.yaml');
        
        $emailElement = $pageElement->find('css', '#signin-email');
        $passwordElement = $pageElement->find('css', '#signin-password');
        $formElement = $pageElement->find('css', 'form.signin');

        $emailElement->setValue($users[$userA]['username']);
        $passwordElement->setValue($users[$userA]['password']);
        $formElement->submit();

        $url = 'https://twitter.com/'.$users[$userB]['username'];
        $session->visit($url);
        $scopeElement = $session->getPage();
        $dmTextElement = $scopeElement->find('css', 'div.dropdown-menu li.dm-text');
        $dmTextElement->click();
    }
*/
}

