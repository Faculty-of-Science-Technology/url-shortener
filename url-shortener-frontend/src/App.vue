<template>
	<div id="app">
		<div class="header">
			<div class="logo">
				<span class="logo-main">
					<span class="logo-icon">⚡</span>
					ShortCircuit
					<span class="logo-icon">⚡</span>
				</span>
				<span class="logo-tagline">Lightning-fast URL shortening</span>
			</div>
		</div>

		<div class="container">
			<!-- Main Input Section - Always visible -->
			<div class="main-section">
				<h1>{{ shortenedUrl ? 'Shorten Another URL' : 'Power Up Your Links' }}</h1>
				
				<div class="input-section">
					<input 
						v-model="originalUrl" 
						@keyup.enter="shortenUrl"
						type="text" 
						placeholder="Paste your long URL here" 
						class="url-input"
						:disabled="loading"
					/>
					<button 
						@click="shortenUrl" 
						class="shorten-btn"
						:disabled="loading || !originalUrl"
					>
						<span v-if="loading">⚡ Processing...</span>
						<span v-else>⚡ Short Circuit</span>
					</button>
					<p v-if="error" class="error-message">{{ error }}</p>
				</div>

				<div class="info-section" v-if="!shortenedUrl">
					<p class="info-text">
						⚡ Instantly transform long URLs into compact, shareable links
					</p>
					<p class="info-text">
						Fast, simple, and reliable URL shortening
					</p>
				</div>
			</div>

			<!-- Result Section - Shows after shortening -->
			<div class="result-section" v-if="shortenedUrl">
				<h2 class="result-title">⚡ Circuit Complete!</h2>
				<p class="result-subtitle">Your URL has been shortened</p>

				<div class="result-box">
					<div class="url-display">
						<a :href="fullShortenedUrl" target="_blank" class="shortened-link">{{ fullShortenedUrl }}</a>
						<button @click="copyToClipboard" class="copy-button" :class="{ copied: copied }">
							{{ copied ? '✓ Copied' : 'Copy' }}
						</button>
					</div>
					<div class="long-url-display">
						<strong>Original URL:</strong> {{ originalUrlResult }}
					</div>
				</div>

				<button @click="resetForm" class="back-btn">⚡ Create Another</button>
			</div>
		</div>
	</div>
</template>

<script>
export default {
	name: "App",
	data() {
		return {
			originalUrl: '',
			originalUrlResult: '',
			shortenedUrl: '',
			loading: false,
			error: '',
			copied: false
		};
	},
	computed: {
		fullShortenedUrl() {
			if (!this.shortenedUrl) return '';
			const baseUrl = window.location.origin;
			return `${baseUrl}/${this.shortenedUrl}`;
		}
	},
	methods: {
		async shortenUrl() {
			if (!this.originalUrl.trim()) {
				this.error = 'Please enter a URL';
				return;
			}

			// Basic URL validation
			try {
				const url = new URL(this.originalUrl.trim());
				if (!['http:', 'https:'].includes(url.protocol)) {
					this.error = 'URL must start with http:// or https://';
					return;
				}
			} catch (e) {
				this.error = 'Please enter a valid URL (e.g., https://example.com)';
				return;
			}

			this.loading = true;
			this.error = '';
			this.copied = false;

			try {
				// Call the backend API
				const response = await fetch('/shorten', {
					method: 'POST',
					headers: {
						'Content-Type': 'text/plain',
					},
					body: this.originalUrl
				});

				if (!response.ok) {
					let errorMessage = 'Failed to shorten URL';
					try {
						const errorData = await response.json();
						errorMessage = errorData.message || errorMessage;
					} catch (jsonError) {
						// If response is not JSON, use status text
						errorMessage = response.statusText || errorMessage;
					}
					throw new Error(errorMessage);
				}

				const data = await response.json();
				this.shortenedUrl = data.shortenCode;
				this.originalUrlResult = data.url || this.originalUrl;
				this.originalUrl = ''; // Clear input after successful shortening
			} catch (err) {
				this.error = err.message || 'An error occurred while shortening the URL';
			} finally {
				this.loading = false;
			}
		},
		async copyToClipboard() {
			try {
				await navigator.clipboard.writeText(this.fullShortenedUrl);
				this.copied = true;
				setTimeout(() => {
					this.copied = false;
				}, 2000);
			} catch (err) {
				console.error('Failed to copy:', err);
			}
		},
		resetForm() {
			this.originalUrl = '';
			this.originalUrlResult = '';
			this.shortenedUrl = '';
			this.error = '';
			this.copied = false;
		}
	}
};
</script>

<style>
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
}

body {
	font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
	background: linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%);
	min-height: 100vh;
}

#app {
	min-height: 100vh;
	background: linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%);
}

.header {
	background: linear-gradient(135deg, #1a1a2e 0%, #16213e 100%);
	padding: 25px 0;
	box-shadow: 0 4px 20px rgba(0, 255, 255, 0.2);
	border-bottom: 2px solid #00ffff;
	margin-bottom: 40px;
}

.logo {
	max-width: 1200px;
	margin: 0 auto;
	padding: 0 20px;
	text-align: center;
}

.logo-main {
	font-size: 36px;
	font-weight: bold;
	color: #00ffff;
	display: flex;
	align-items: center;
	justify-content: center;
	gap: 10px;
	text-shadow: 0 0 10px rgba(0, 255, 255, 0.5),
				0 0 20px rgba(0, 255, 255, 0.3),
				0 0 30px rgba(0, 255, 255, 0.2);
	letter-spacing: 2px;
}

.logo-icon {
	font-size: 32px;
	animation: pulse 2s infinite;
}

@keyframes pulse {
	0%, 100% {
		opacity: 1;
		text-shadow: 0 0 10px rgba(255, 255, 0, 0.8);
	}
	50% {
		opacity: 0.7;
		text-shadow: 0 0 20px rgba(255, 255, 0, 1);
	}
}

.logo-tagline {
	font-size: 14px;
	color: #66d9ef;
	display: block;
	margin-top: 8px;
	letter-spacing: 3px;
	text-transform: uppercase;
}

.container {
	max-width: 800px;
	margin: 0 auto;
	padding: 0 20px 40px;
}

.main-section h1 {
	color: #00ffff;
	font-size: 32px;
	text-align: center;
	margin-bottom: 30px;
	font-weight: 600;
	text-shadow: 0 0 10px rgba(0, 255, 255, 0.3);
}

.input-section {
	background: rgba(22, 33, 62, 0.9);
	padding: 30px;
	border-radius: 12px;
	box-shadow: 0 8px 32px rgba(0, 255, 255, 0.1);
	margin-bottom: 30px;
	border: 1px solid rgba(0, 255, 255, 0.2);
	backdrop-filter: blur(10px);
}

.url-input {
	width: 100%;
	padding: 15px;
	font-size: 16px;
	border: 2px solid #66d9ef;
	border-radius: 8px;
	outline: none;
	margin-bottom: 15px;
	transition: all 0.3s;
	background: rgba(255, 255, 255, 0.95);
	color: #1a1a2e;
}

.url-input:focus {
	border-color: #00ffff;
	box-shadow: 0 0 15px rgba(0, 255, 255, 0.4);
	background: white;
}

.url-input:disabled {
	background-color: rgba(200, 200, 200, 0.3);
	cursor: not-allowed;
}

.shorten-btn {
	width: 100%;
	padding: 15px;
	font-size: 18px;
	font-weight: bold;
	color: #1a1a2e;
	background: linear-gradient(135deg, #00ffff 0%, #66d9ef 100%);
	border: none;
	border-radius: 8px;
	cursor: pointer;
	transition: all 0.3s;
	text-transform: uppercase;
	letter-spacing: 1px;
	box-shadow: 0 4px 15px rgba(0, 255, 255, 0.3);
}

.shorten-btn:hover:not(:disabled) {
	background: linear-gradient(135deg, #66d9ef 0%, #00ffff 100%);
	box-shadow: 0 6px 20px rgba(0, 255, 255, 0.5);
	transform: translateY(-2px);
}

.shorten-btn:active:not(:disabled) {
	transform: translateY(0);
}

.shorten-btn:disabled {
	opacity: 0.5;
	cursor: not-allowed;
}

.error-message {
	color: #ff6b6b;
	font-size: 14px;
	margin-top: 10px;
	text-align: center;
	background: rgba(255, 107, 107, 0.1);
	padding: 10px;
	border-radius: 6px;
	border: 1px solid rgba(255, 107, 107, 0.3);
}

.info-section {
	text-align: center;
}

.info-text {
	color: #a8dadc;
	font-size: 16px;
	margin-bottom: 10px;
	line-height: 1.6;
}

/* Result Section Styles */
.result-section {
	text-align: center;
	margin-top: 20px;
}

.result-title {
	color: #00ffff;
	font-size: 32px;
	margin-bottom: 10px;
	font-weight: 600;
	text-shadow: 0 0 15px rgba(0, 255, 255, 0.5);
	animation: slideIn 0.5s ease-out;
}

@keyframes slideIn {
	from {
		opacity: 0;
		transform: translateY(-20px);
	}
	to {
		opacity: 1;
		transform: translateY(0);
	}
}

.result-subtitle {
	color: #a8dadc;
	font-size: 16px;
	margin-bottom: 30px;
}

.result-box {
	background: rgba(22, 33, 62, 0.9);
	padding: 30px;
	border-radius: 12px;
	box-shadow: 0 8px 32px rgba(0, 255, 255, 0.15);
	margin-bottom: 30px;
	border: 1px solid rgba(0, 255, 255, 0.2);
	backdrop-filter: blur(10px);
	animation: fadeIn 0.5s ease-out;
}

@keyframes fadeIn {
	from {
		opacity: 0;
		transform: scale(0.95);
	}
	to {
		opacity: 1;
		transform: scale(1);
	}
}

.url-display {
	display: flex;
	align-items: center;
	gap: 15px;
	margin-bottom: 20px;
	padding: 15px;
	background: rgba(0, 255, 255, 0.05);
	border-radius: 8px;
	border: 2px solid #00ffff;
	box-shadow: 0 0 20px rgba(0, 255, 255, 0.2);
}

.shortened-link {
	flex: 1;
	color: #00ffff;
	text-decoration: none;
	font-size: 18px;
	font-weight: bold;
	word-break: break-all;
	text-align: left;
	text-shadow: 0 0 10px rgba(0, 255, 255, 0.3);
}

.shortened-link:hover {
	text-decoration: underline;
	color: #66d9ef;
}

.copy-button {
	padding: 10px 20px;
	background: linear-gradient(135deg, #00ffff 0%, #66d9ef 100%);
	color: #1a1a2e;
	border: none;
	border-radius: 6px;
	cursor: pointer;
	font-size: 14px;
	font-weight: bold;
	white-space: nowrap;
	transition: all 0.3s;
	text-transform: uppercase;
	letter-spacing: 1px;
}

.copy-button:hover {
	background: linear-gradient(135deg, #66d9ef 0%, #00ffff 100%);
	box-shadow: 0 4px 15px rgba(0, 255, 255, 0.4);
}

.copy-button.copied {
	background: linear-gradient(135deg, #51cf66 0%, #37b24d 100%);
	color: white;
}

.long-url-display {
	color: #a8dadc;
	font-size: 14px;
	text-align: left;
	word-break: break-all;
	padding: 12px;
	background: rgba(0, 0, 0, 0.2);
	border-radius: 6px;
	border-left: 3px solid #66d9ef;
}

.back-btn {
	padding: 12px 30px;
	font-size: 16px;
	font-weight: bold;
	color: #00ffff;
	background: rgba(0, 255, 255, 0.1);
	border: 2px solid #00ffff;
	border-radius: 8px;
	cursor: pointer;
	transition: all 0.3s;
	text-transform: uppercase;
	letter-spacing: 1px;
}

.back-btn:hover {
	background: linear-gradient(135deg, #00ffff 0%, #66d9ef 100%);
	color: #1a1a2e;
	box-shadow: 0 4px 15px rgba(0, 255, 255, 0.4);
	transform: translateY(-2px);
}

@media (max-width: 600px) {
	.logo-main {
		font-size: 28px;
	}
	
	.logo-icon {
		font-size: 24px;
	}
	
	.main-section h1,
	.result-title {
		font-size: 24px;
	}
	
	.url-display {
		flex-direction: column;
	}
	
	.shortened-link {
		text-align: center;
	}
}
</style>
