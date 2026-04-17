This guideline will help you to use CleanTalk check_bot API method via the special library that can be downloaded from this repo.
## Usage
1. Install the package via composer

`composer install cleantalk/php-checkbot`

2. Include the composer autoloader into your app

`require_once 'vendor/autoloader.php';`

3. Add the CleanTalk Bot-detector JS library wrapper `https://fd.cleantalk.org/ct-bot-detector-wrapper.js` as `<script>`tag to the HTML page contains the form you want to protect.

```html
<script src="https://fd.cleantalk.org/ct-bot-detector-wrapper.js"></script>
```

4. Prepare config and create a new CleanTalk\CheckBot object, provide $_POST or filtered POST data to the constructor.

```php
$check_bot_config = array(
    'access_key' => "YOUR-CLEANTALK-API-KEY"
);
```

5. Add necessary parameter `event_token_enabled` into config
If you correctly added the JS library wrapper to the HTML page, you can use the `event_token_enabled` parameter in the config. This will allow you to get more accurate bot detection results.

```php
$check_bot_config = array(
    'access_key' => "YOUR-CLEANTALK-API-KEY",
    'event_token_enabled' => true // <- Only if CleanTalk Bot-detector JS library wrapper has been included to the page
);
```

6. Then create the CleanTalk\CheckBot object, provide $_POST or filtered POST data to the constructor:

```php
$config = new \Cleantalk\CheckBot\CheckBotConfig($check_bot_config);
$bot_checker = new \Cleantalk\CheckBot\CheckBot($config, $_POST);
```

7. Then perform the check:
```php
$is_bot = $bot_checker->check()->getVerdict();
```

8. Then do the actions depending on the verdict.
```php
if ( $is_bot ) {
  die ($bot_checker->getBlockMessage());
}
```

9. How it looks in the suggested files structure:
- `index.html`:
```html
<!DOCTYPE html>
<html lang="en">
   <head>
   <meta charset="UTF-8">
   <title>Title</title>
   <!--Bot-detector JS library wrapper. This script must be added to the HTML of the page.-->
   <script src="https://fd.cleantalk.org/ct-bot-detector-wrapper.js"></script>
</head>
<body>
   <form method="post" action="your_form_handler.php">
      <label for="search_field">What do you search?</label>
      <input type="text" name="search_field" id="search_field" /> <br />
      <input type="submit" />
   </form>
</body>
</html>
```
- `your_form_handler.php`:
```php
<?php

require_once 'vendor/autoload.php';

if ( empty($_POST) ) {
    return;
} else {
    handle_search_form($_POST);
}

/**
 * Main search from handler.
 * @param $post
 * @return void
 */
function handle_search_form($post)
{
    if ( empty($post['search_field']) ) {
        return;
    }

    //create a new CheckBot object 
    $check_bot_config = array(
      'access_key' => "YOUR-CLEANTALK-API-KEY"
    );
    $config = new \Cleantalk\CheckBot\CheckBotConfig($check_bot_config);
    $bot_checker = new \Cleantalk\CheckBot\CheckBot($config, $_POST);

    //call visitor check and make decision
    $is_bot = $bot_checker->check()->getVerdict();
    if ( $is_bot ) {
        die ($bot_checker->getBlockMessage());
    }

    //implement your further search form handlers here replacing echo
    echo('You searched for this: ' . $post['search_field']);
}
```
## Config setup explanation

### Available config array elements
```php
<?php
$check_bot_config = array(
    'access_key' => "",
    'trust_cleantalk_decision' => true,
    'block_no_js_visitors' => false,
    'common_block_message' => 'Visitor blocked. It seems to be a bot.',
    'bot_expectation' => 0.5,
    'ip_frequency_24hour' => 50,
    'ip_frequency_1hour' => 15,
    'ip_frequency_10min' => 5,
    'do_log' => false
);
```

### Common params

- `access_key (string)`
  <br><br>
Your CleanTalk Anti-Spam access key.
  <br><br>
- `trust_cleantalk_decision (bool)`
  <br><br>
Set this to true if you do not want to set custom checking settings. Applicable in the most cases. 
  <br><br>
- `block_no_js_visitors (bool)`
  <br><br>
Set this to true if you want to block any visitor that could not execute JS script (bot-like behavior). Applicable in the most cases.
  <br><br>
- `common_block_message (string)`
  <br><br>
A message for blocked visitor.
  <br><br>
- `do_log (bool)`
  <br><br>
Set to true if you want to see the log in the PHP error log, false otherwise.

### Custom params
<em>Params below affected only if the property "trust_cleantalk_decision is set to false.</em>

- `bot_expectation`
<br><br>
Set maximum bot probability percentage. For example, 0.5 is 50%. If CleanTalk API responsed with bot_expectation 0.53 - visitor will be blocked, if 0.47 - passed.
<br><br>
- `ip_frequency_24hour`,`ip_frequency_1hour`,`ip_frequency_10min`
<br><br>
Custom checks - set how to block a visitor whose IP address detected by CleanTalk service in the period. For example, if CleanTalk response contains ip_frequency_24hour = 1000, and the config property ip_frequency_24hour = 500, visitor will be blocked.

## How to test
1. Disable JavaScript implementation in your web-browser. If the param `trust_cleantalk_decision` is set to `true`, you are blocked when you try to send the form.
2. Install the solution on a dev site with visiting opened. Then set the param `do_log` to `true`. You will see how the CheckBot works in the PHP error log.

## Examples
Examples of the form with CheckBot integrated can be found in the `/examples` folder. Note: the examples does not contain the lib itself.
