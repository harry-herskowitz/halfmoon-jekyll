---
layout: post
title: 'HTML Form Submit with Google reCAPTCHA, Vanilla JS and PHP'
date: 2020-12-09
categories: [HTML, PHP, JS]
image: 'recaptchacover.jpg'
author: 'Harry Herskowitz'
author_blurb: 'Harry is a web developer and videographer with a focus on using technology to empower local artists and communities'
author_link: 'https://github.com/roldyclark'
avatar: 'roldy.png'
---

![recaptcha]({{ site.baseurl }}/assets/images/posts/recaptcha.jpg)

Finding simple answers for submitting custom HTML forms are hard to find, especially when adding in Google reCAPTCHA and avoiding jQuery. The following code is a result of hours of hunting Stack Overflow for answers to two main problems: How to submit an HTML form without AJAX and how to prevent Google reCAPTCHA from overriding HTML5 form validation. The first required using a hidden iFrame as the form target, and the second involved placing the reCAPTCHA attributes in a hidden div rather than in the submit button.

```
<!-- form -->
<iframe name="hiddenFrame" width="0" height="0" border="0" style="display: none;"></iframe>
<form action="sendEmail.php" name="contactForm" id="contactForm" method="POST" target="hiddenFrame">
    <div class="form-field">
      <input name="contactName" type="text" id="contactName" placeholder="Name" value="" minlength="2" required />
    </div>

    <div class="form-field">
      <input name="contactEmail" type="email" id="contactEmail" placeholder="Email" value="" required />
    </div>

    <div class="form-field">
      <input name="contactSubject" type="text" id="contactSubject" placeholder="Subject" value="" />
    </div>

    <div class="form-field">
      <textarea name="contactMessage" id="contactMessage" placeholder="Message" rows="10" cols="50" minlength="20"
        required></textarea>
    </div>

    <div id='recaptcha' class="g-recaptcha"
    data-sitekey="YOURKEYHERE"
    data-callback="onCompleted"
    data-size="invisible"></div>

    <div class="form-field">
      <button class="button stroke submitform">Submit</button>
</form>

<!-- contact-success -->
<div id="message-success">
    <i class="fa fa-check"></i>Your message was sent, thank you!<br />
</div>

<script src="https://www.google.com/recaptcha/api.js" async defer>
</script>
  <script>
      var form = document.getElementById("contactForm")
      form.addEventListener("submit", function(event) {
        console.log('form submitted.');

        if (!grecaptcha.getResponse()) {
            console.log('captcha not yet completed.');

            event.preventDefault(); //prevent form submit
            grecaptcha.execute();
        } else {
            console.log('form really submitted.');
        }
    });

    onCompleted = function() {
        console.log('captcha completed.');
        form.submit();
        document.getElementById("message-success").style.display = "flex"
    }
</script>

```

Here is the sendEmail.php file:

```
<?php

// Replace this with your own email address
$siteOwnersEmail = 'YOUR_EMAIL';

if ($_POST) {

	$name = trim(stripslashes($_POST['contactName']));
	$email = trim(stripslashes($_POST['contactEmail']));
	$subject = trim(stripslashes($_POST['contactSubject']));
	$contact_message = trim(stripslashes($_POST['contactMessage']));

	// Check Name
	if (strlen($name) < 2) {
		$error['name'] = "Please enter your name.";
	}
	// Check Email
	if (!preg_match('/^[a-z0-9&\'\.\-_\+]+@[a-z0-9\-]+\.([a-z0-9\-]+\.)*+[a-z]{2}/is', $email)) {
		$error['email'] = "Please enter a valid email address.";
	}
	// Check Message
	if (strlen($contact_message) < 15) {
		$error['message'] = "Please enter your message. It should have at least 15 characters.";
	}
	// Subject
	if ($subject == '') {
		$subject = "Contact Form Submission";
	}

	// Set Message
	$message .= "Email from: " . $name . "<br />";
	$message .= "Email address: " . $email . "<br />";
	$message .= "Message: <br />";
	$message .= $contact_message;
	$message .= "<br /> ----- <br /> This email was sent from your site's contact form. <br />";

	// Set From: header
	$from =  $name . " <" . $email . ">";

	// Email Headers
	$headers = "From: " . $from . "\r\n";
	$headers .= "Reply-To: " . $email . "\r\n";
	$headers .= "MIME-Version: 1.0\r\n";
	$headers .= "Content-Type: text/html; charset=ISO-8859-1\r\n";

	// reCAPTCHA validation
	if (isset($_POST['g-recaptcha-response']) && !empty($_POST['g-recaptcha-response'])) {

		// Google secret API
		$secretAPIkey = 'YOUR_SECRET_KEY';

		// reCAPTCHA response verification
		$verifyResponse = file_get_contents('https://www.google.com/recaptcha/api/siteverify?secret=' . $secretAPIkey . '&response=' . $_POST['g-recaptcha-response']);

		// Decode JSON data
		$response = json_decode($verifyResponse);

		if ($response->success) {

			if (!$error) {

				ini_set("sendmail_from", $siteOwnersEmail); // for windows server
				$mail = mail($siteOwnersEmail, $subject, $message, $headers);

				if ($mail) {
					echo "OK";
				} else {
					echo "Something went wrong. Please try again.";
				}
			} # end if - no validation error

			else {

				$response = (isset($error['name'])) ? $error['name'] . "<br /> \n" : null;
				$response .= (isset($error['email'])) ? $error['email'] . "<br /> \n" : null;
				$response .= (isset($error['message'])) ? $error['message'] . "<br />" : null;

				echo $response;
			} # end if - there was a validation error
		}
	}
}
```
