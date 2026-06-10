<script setup>
	import { Notyf } from 'notyf';
	import { ref } from 'vue';

	const notyf = new Notyf();

	const name = ref("");
	const email = ref("");
	const message = ref("");
	const isLoading = ref(false);

	// 1. Web3Forms Access Key
	const WEB3FORMS_ACCESS_KEY = "91c0b0e7-58c2-48d8-a99c-f04255e380e5"; 

	// Email subject that will appear when a form submission is received.
	const subject = "New message from Gladys Ramos Portfolio";

	// The submitForm() function handles the contact form submission.
	const submitForm = async () => {

		// Ensure the user completes the reCAPTCHA challenge before submitting the form.
        // Check if a reCAPTCHA token exists
        // recaptchaToken.value - stores the verification token returned by Google reCAPTCHA
		if(!recaptchaToken.value) {
			notyf.error('Please verify that you are not a robot');
			// Stop the form submission process
			return;
		}

		// While the email is being sent, disable the button and change text to "Sending..."
		isLoading.value = true;

		try {
			// Send HTTP request to Web3Forms API
			const response = await fetch("https://api.web3forms.com/submit", {
				method: "POST",
				headers: {
					"Content-Type": "application/json",
					"Accept": "application/json"
				},
				body: JSON.stringify({
					access_key: WEB3FORMS_ACCESS_KEY,
					subject: subject,
					name: name.value,
					email: email.value,
					message: message.value
				})
			});

			const result = await response.json();

			if (result.success) {
				console.log(result);
				isLoading.value = false;
				notyf.success("Message Sent!");
				
						}
		} catch (error) {
			console.log(error);
			isLoading.value = false;
			notyf.error("Failed to send message.");
		} finally {
			// Reset the reCAPTCHA widget after the submission process completes, whether the request succeeds or fails.
			resetRecaptcha();
	
		}
	}

		/*reCAPTCHA Integration*/

    const SITE_KEY = '';  // Replace with your site key
    // The location where the reCAPTCHA checkbox will appear
    const recaptchaContainer = ref(null);
    // The ID of the reCAPTCHA widget after it is created
    const recaptchaWidgetId = ref(null);
    // Stores the token generated when the user completes reCAPTCHA
    const recaptchaToken = ref('');

    // Runs when the user successfully completes the reCAPTCHA challenge
    function onRecaptchaSuccess(token) {
        recaptchaToken.value = token;
    }


  // Runs when the reCAPTCHA verification expires
    // Token typically expires after about 2 minutes (120 seconds) if the form has not been submitted
    function onRecaptchaExpired() {
        recaptchaToken.value = '';
    }

    // Creates and displays the reCAPTCHA widget
    function renderRecaptcha() {
    	// Check if the Google reCAPTCH script is available
    	// window - refers to the browser window object
        if (!window.grecaptcha) {
            console.error('reCAPTCHA not loaded');
            return;
        }

        // Creates the reCAPTCHA widget and save its ID
        // Google returns a unique widget ID after creating the reCAPTCHA widget
            // render() generates the reCAPTCHA checkbox inside recaptchaContainer.value
            // Call window.grecaptcha.render() to create the widget, then store the widget ID returned by Google in recaptchaWidgetId.value.
        recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
            sitekey: SITE_KEY,
            size: 'normal',
            callback: onRecaptchaSuccess,
            'expired-callback': onRecaptchaExpired,
        });
    }

    // Function the resets the reCAPTCHA widget
    function resetRecaptcha() {
        if (recaptchaWidgetId.value !== null) {
        	window.grecaptcha.reset(recaptchaWidgetId.value);
        	recaptchaToken.value = '';
        }
    }

    onMounted(() => {
    // Check if the Google reCAPTCHA library has finished loading, since it is loaded asynchronously from index.html.
    // This makes sure that the reCAPTCHA widget is rendered only after the library is available.
    // setInterval() is a JavaScript function that repeatedly executes a block of code at a specified time interval.
    const interval = setInterval(() => {
        if (window.grecaptcha && window.grecaptcha.render) {
            renderRecaptcha();
            // Stop checking once the widget has been rendered
            clearInterval(interval);
        }
        // Check every 100 milliseconds if the Google reCAPTCHA library has loaded
    }, 100);

    // Run cleanup code before the component is removed
	onBeforeUnmount(() => {
		// Stop checking once the widget has been rendered
	  clearInterval(interval);
	});
});  

	

</script>

<template>
	<h1 class="text-center my-4 pt-5" id="contact">Contact</h1>
	<div class="contact-section container">
		<div class="row align-items-center mt-4">
			<div class="col-md-6 mb-4 mb-md-0 text-center">
				<div class="p-5 bg-light rounded shadow-sm">
					<h4>Let's Build Something Together!</h4>
					<p class="text-muted">Drop me a line or connect with me via my professional networks below.</p>
				</div>
			</div>
			<div class="col-md-6">
				<form @submit.prevent="submitForm">
	                <div class="mb-3">
	                	<!-- Bind the input field to the reactive variables using v-model -->
	                    <input type="text" v-model="name" class="form-control contact-form-control" placeholder="First Name M.I. Last Name">
	                </div>
	                <div class="mb-3">
	                    <input type="email" v-model="email" class="form-control contact-form-control" placeholder="Email">
	                </div>
	                <div class="mb-3">
	                    <textarea class="form-control contact-form-control" v-model="message" rows="6" placeholder="Message"></textarea>
	                </div>

					<div class="d-flex justify-content-between align-items-center flex-wrap gap-3">
						<button type="submit" class="btn btn-primary" :disabled="isLoading">
							{{ isLoading ? 'Sending...' : 'Send Message' }}
						</button>
					</div>
				</form>
			</div>
		</div>
	</div>
</template>