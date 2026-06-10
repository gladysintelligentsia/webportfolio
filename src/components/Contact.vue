<script setup>
	import { Notyf } from 'notyf';
	import { ref, onMounted } from 'vue';

	const notyf = new Notyf();

	const name = ref("");
	const email = ref("");
	const message = ref("");
	const isLoading = ref(false);

	// Web3Forms Access Key
	const WEB3FORMS_ACCESS_KEY = "91c0b0e7-58c2-48d8-a99c-f04255e380e5"; 
	const subject = "New message from Gladys Ramos Portfolio";

	const submitForm = async () => {
		// 1. Check if user completed the Google reCAPTCHA checkbox
		const recaptchaResponse = window.grecaptcha.getResponse();
		
		if (!recaptchaResponse) {
			notyf.error('Please verify that you are not a robot using Google reCAPTCHA');
			return;
		}

		isLoading.value = true;

		try {
			// Send submission payload to Web3Forms
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
					// Sends validation token safely to backend context
					"g-recaptcha-response": recaptchaResponse
				})
			});

			const result = await response.json();

			if (result.success) {
				isLoading.value = false;
				notyf.success("Message Sent!");
				
				// Reset inputs & recaptha checkbox state
				name.value = "";
				email.value = "";
				message.value = "";
				window.grecaptcha.reset();
			} else {
				isLoading.value = false;
				notyf.error(result.message || "Failed to send message.");
			}
		} catch (error) {
			console.log(error);
			isLoading.value = false;
			notyf.error("Failed to send message.");
		}
	}
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
						<input type="text" v-model="name" class="form-control" placeholder="Full Name" required>
					</div>
					<div class="mb-3">
						<input type="email" v-model="email" class="form-control" placeholder="Email Address" required>
					</div>
					<div class="mb-3">
						<textarea class="form-control" v-model="message" rows="6" placeholder="Your Message" required></textarea>
					</div>
					
					<div class="mb-3 d-flex justify-content-center justify-content-md-start">
						<div class="g-recaptcha" data-sitekey="PASTE_YOUR_GOOGLE_RECAPTCHA_SITE_KEY_HERE"></div>
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