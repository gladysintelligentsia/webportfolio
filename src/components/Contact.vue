<script setup>
	import { Notyf } from 'notyf';
	// FIX 1: Imported missing lifecycle hooks from Vue
	import { ref, onMounted, onBeforeUnmount } from 'vue';

	const notyf = new Notyf();

	const name = ref("");
	const email = ref("");
	const message = ref("");
	const isLoading = ref(false);

	// Web3Forms Access Key
	const WEB3FORMS_ACCESS_KEY = "91c0b0e7-58c2-48d8-a99c-f04255e380e5"; 

	// Email subject that will appear when a form submission is received.
	const subject = "New message from Gladys Ramos Portfolio";

	// The submitForm() function handles the contact form submission.
	const submitForm = async () => {

		// Ensure the user completes the reCAPTCHA challenge before submitting the form.
		if(!recaptchaToken.value) {
			notyf.error('Please verify that you are not a robot');
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
					message: message.value,
					// Optional: send the token along if Web3forms verification is enabled on their dashboard
					"g-recaptcha-response": recaptchaToken.value 
				})
			});

			const result = await response.json();

			if (result.success) {
				console.log(result);
				// Reset form fields upon success
				name.value = "";
				email.value = "";
				message.value = "";
				notyf.success("Message Sent!");
			} else {
				notyf.error(result.message || "Failed to send message.");
			}
		} catch (error) {
			console.log(error);
			notyf.error("Failed to send message.");
		} finally {
			isLoading.value = false;
			// Reset the reCAPTCHA widget after submission
			resetRecaptcha();
		}
	}

	/* reCAPTCHA Integration */

	// FIX 3: Remember to insert your real Google client site key here!
	// Paste the copied key right here
	const SITE_KEY = '6LckbhUtAAAAAHOWunqHB7VyrAk5XBpdkBspHSv-';
	const recaptchaContainer = ref(null);
	const recaptchaWidgetId = ref(null);
	const recaptchaToken = ref('');

	function onRecaptchaSuccess(token) {
		recaptchaToken.value = token;
	}

	function onRecaptchaExpired() {
		recaptchaToken.value = '';
	}

	function renderRecaptcha() {
		if (!window.grecaptcha) {
			console.error('reCAPTCHA not loaded');
			return;
		}

		recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
			sitekey: SITE_KEY,
			size: 'normal',
			callback: onRecaptchaSuccess,
			'expired-callback': onRecaptchaExpired,
		});
	}

	function resetRecaptcha() {
		if (recaptchaWidgetId.value !== null) {
			window.grecaptcha.reset(recaptchaWidgetId.value);
			recaptchaToken.value = '';
		}
	}

	// Create persistent scope wrapper for the interval so clear can target it correctly
	let interval = null;

	onMounted(() => {
		interval = setInterval(() => {
			if (window.grecaptcha && window.grecaptcha.render) {
				renderRecaptcha();
				clearInterval(interval);
			}
		}, 100);
	});  

	onBeforeUnmount(() => {
		if (interval) clearInterval(interval);
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
						<input type="text" id="firstName" v-model="name" class="form-control contact-form-control" placeholder="First Name M.I. Last Name" required>
					</div>
					<div class="mb-3">
						<input type="email" id="emailAddress" v-model="email" class="form-control contact-form-control" placeholder="Email" required>
					</div>
					<div class="mb-3">
						<textarea id="messageBody" class="form-control contact-form-control" v-model="message" rows="6" placeholder="Message" required></textarea>
					</div>

					<div class="mb-3 d-flex justify-content-center justify-content-md-start">
						<div ref="recaptchaContainer"></div>
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