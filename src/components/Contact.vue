<script setup>
	import { Notyf } from 'notyf';
	import { ref, onMounted, onBeforeUnmount } from 'vue';

	const notyf = new Notyf();

	const name = ref("");
	const email = ref("");
	const message = ref("");
	const isLoading = ref(false);

	// 1. Web3Forms Access Key
	const WEB3FORMS_ACCESS_KEY = "91c0b0e7-58c2-48d8-a99c-f04255e380e5"; 

	// Email subject that will appear when a form submission is received.
	const subject = "New message from Gladys Ramos Portfolio";

	/* reCAPTCHA Integration Setup */
	const SITE_KEY = '6LdfC0MfAAAAAF963m998U0ky9snF_1E_z8isY6v';  

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
		// Clean rendering path: Verify the element is ready and target standard engine
		if (window.grecaptcha && window.grecaptcha.render && recaptchaContainer.value) {
			try {
				// Clear any lingering rendering artifacts before drawing the fresh widget
				recaptchaContainer.value.innerHTML = "";
				
				recaptchaWidgetId.value = window.grecaptcha.render(recaptchaContainer.value, {
					sitekey: SITE_KEY,
					size: 'normal',
					callback: onRecaptchaSuccess,
					'expired-callback': onRecaptchaExpired,
				});
			} catch (error) {
				console.warn("reCAPTCHA rendering handled natively:", error);
			}
		}
	}

	function resetRecaptcha() {
		if (recaptchaWidgetId.value !== null && window.grecaptcha && window.grecaptcha.reset) {
			window.grecaptcha.reset(recaptchaWidgetId.value);
			recaptchaToken.value = '';
		}
	}

	const submitForm = async () => {
		if (!recaptchaToken.value) {
			notyf.error('Please verify that you are not a robot.');
			return;
		}

		isLoading.value = true;

		try {
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
					"g-recaptcha-response": recaptchaToken.value
				})
			});

			const result = await response.json();

			if (result.success) {
				notyf.success("Message Sent!");
				name.value = "";
				email.value = "";
				message.value = "";
				resetRecaptcha();
			} else {
				notyf.error(result.message || "Failed to send message.");
				resetRecaptcha();
			}
		} catch (error) {
			console.error(error);
			notyf.error("Failed to send message.");
			resetRecaptcha();
		} finally {
			isLoading.value = false;
		}
	}

	// Native Single-Page-App Lifecycle Management
	let checkInterval = null;

	onMounted(() => {
		// If the core script has loaded already, execute immediately
		if (window.grecaptcha && window.grecaptcha.render) {
			renderRecaptcha();
		} else {
			// Fail-safe interval polling: checks every 300ms if script finishes late
			checkInterval = setInterval(() => {
				if (window.grecaptcha && window.grecaptcha.render) {
					renderRecaptcha();
					clearInterval(checkInterval);
				}
			}, 300);
		}
	});

	onBeforeUnmount(() => {
		if (checkInterval) {
			clearInterval(checkInterval);
		}
	});
</script>

<template>
	<h1 class="text-center my-4 pt-5" id="contact">Contact Gladys Ramos</h1>
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
						<input type="text" id="name" v-model="name" class="form-control" placeholder="Name" required>
					</div>
					<div class="mb-3">
						<input type="email" id="email" v-model="email" class="form-control" placeholder="Email Address" required>
					</div>
					<div class="mb-3">
						<textarea id="message" class="form-control" v-model="message" rows="6" placeholder="Your Message" required></textarea>
					</div>

					<div class="mb-4 d-flex justify-content-start" style="min-height: 78px;">
						<div ref="recaptchaContainer"></div>
					</div>

					<div class="d-flex justify-content-between align-items-center flex-wrap gap-3">
						<div class="social-icons d-flex gap-3 fs-4">
							<a href="https://www.linkedin.com/in/gladysramos" target="_blank" id="linkedin"><i class="fab fa-linkedin"></i></a>
							<a href="https://git.zuitt.co/gladysintelligentsia" target="_blank" id="gitlab"><i class="fab fa-gitlab"></i></a>
							<a href="https://github.com/gladysintelligentsia" target="_blank" id="github"><i class="fab fa-github"></i></a>
						</div>
						<button type="submit" class="btn btn-primary px-4" :disabled="isLoading">
							{{ isLoading ? "Sending..." : "Submit" }}
						</button>
					</div>
				</form>
			</div>
		</div>
	</div>
</template>

<style scoped>
	.social-icons a {
		color: #495057;
		transition: color 0.2s ease-in-out;
	}
	.social-icons a:hover {
		color: #0d6efd;
	}
</style>