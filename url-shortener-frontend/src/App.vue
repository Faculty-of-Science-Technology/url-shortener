<template>
	<div id="app">
		<div class="header">
			<div class="logo">
				<span class="logo-main">URL Shortener</span>
			</div>
		</div>

		<div class="container">
			<!-- Main Input Section - Always visible -->
			<div class="main-section">
				<h1>{{ shortenedUrl ? 'Shorten Another URL' : 'Enter URL to shorten' }}</h1>
				
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
						{{ loading ? 'Shortening...' : 'Shorten URL' }}
					</button>
					<p v-if="error" class="error-message">{{ error }}</p>
				</div>

				<div class="info-section" v-if="!shortenedUrl">
					<p class="info-text">
						A modern, minimalist, and lightweight URL shortener
					</p>
					<p class="info-text">
						Create short links that are easy to share and remember
					</p>
				</div>
			</div>

			<!-- Result Section - Shows after shortening -->
			<div class="result-section" v-if="shortenedUrl">
				<h2 class="result-title">Success!</h2>
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

				<button @click="resetForm" class="back-btn">Back to Home</button>
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
					const errorData = await response.json();
					throw new Error(errorData.message || 'Failed to shorten URL');
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
	font-family: Arial, Helvetica, sans-serif;
	background-color: #f5f5f5;
}

#app {
	min-height: 100vh;
	background-color: #f5f5f5;
}

.header {
	background-color: #fff;
	padding: 20px 0;
	box-shadow: 0 2px 5px rgba(0,0,0,0.1);
	margin-bottom: 40px;
}

.logo {
	max-width: 1200px;
	margin: 0 auto;
	padding: 0 20px;
}

.logo-main {
	font-size: 28px;
	font-weight: bold;
	color: #333;
	display: block;
}

.container {
	max-width: 800px;
	margin: 0 auto;
	padding: 0 20px 40px;
}

.main-section h1 {
	color: #333;
	font-size: 32px;
	text-align: center;
	margin-bottom: 30px;
	font-weight: normal;
}

.input-section {
	background: white;
	padding: 30px;
	border-radius: 8px;
	box-shadow: 0 2px 10px rgba(0,0,0,0.1);
	margin-bottom: 30px;
}

.url-input {
	width: 100%;
	padding: 15px;
	font-size: 16px;
	border: 2px solid #ddd;
	border-radius: 4px;
	outline: none;
	margin-bottom: 15px;
	transition: border-color 0.3s;
}

.url-input:focus {
	border-color: #4CAF50;
}

.url-input:disabled {
	background-color: #f5f5f5;
	cursor: not-allowed;
}

.shorten-btn {
	width: 100%;
	padding: 15px;
	font-size: 18px;
	font-weight: bold;
	color: white;
	background-color: #4CAF50;
	border: none;
	border-radius: 4px;
	cursor: pointer;
	transition: background-color 0.3s;
}

.shorten-btn:hover:not(:disabled) {
	background-color: #45a049;
}

.shorten-btn:disabled {
	background-color: #ccc;
	cursor: not-allowed;
}

.error-message {
	color: #f44336;
	font-size: 14px;
	margin-top: 10px;
	text-align: center;
}

.info-section {
	text-align: center;
}

.info-text {
	color: #666;
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
	color: #4CAF50;
	font-size: 28px;
	margin-bottom: 10px;
	font-weight: normal;
}

.result-subtitle {
	color: #666;
	font-size: 16px;
	margin-bottom: 30px;
}

.result-box {
	background: white;
	padding: 30px;
	border-radius: 8px;
	box-shadow: 0 2px 10px rgba(0,0,0,0.1);
	margin-bottom: 30px;
}

.url-display {
	display: flex;
	align-items: center;
	gap: 15px;
	margin-bottom: 20px;
	padding: 15px;
	background-color: #f9f9f9;
	border-radius: 4px;
	border: 2px solid #4CAF50;
}

.shortened-link {
	flex: 1;
	color: #4CAF50;
	text-decoration: none;
	font-size: 18px;
	font-weight: bold;
	word-break: break-all;
	text-align: left;
}

.shortened-link:hover {
	text-decoration: underline;
}

.copy-button {
	padding: 10px 20px;
	background-color: #4CAF50;
	color: white;
	border: none;
	border-radius: 4px;
	cursor: pointer;
	font-size: 14px;
	font-weight: bold;
	white-space: nowrap;
	transition: background-color 0.3s;
}

.copy-button:hover {
	background-color: #45a049;
}

.copy-button.copied {
	background-color: #2196F3;
}

.long-url-display {
	color: #666;
	font-size: 14px;
	text-align: left;
	word-break: break-all;
	padding: 10px;
	background-color: #f9f9f9;
	border-radius: 4px;
}

.back-btn {
	padding: 12px 30px;
	font-size: 16px;
	font-weight: bold;
	color: #4CAF50;
	background-color: white;
	border: 2px solid #4CAF50;
	border-radius: 4px;
	cursor: pointer;
	transition: all 0.3s;
}

.back-btn:hover {
	background-color: #4CAF50;
	color: white;
}

@media (max-width: 600px) {
	.logo-main {
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

			this.loading = true;
			this.error = '';
			this.shortenedUrl = '';
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
					const errorData = await response.json();
					throw new Error(errorData.message || 'Failed to shorten URL');
				}

				const data = await response.json();
				this.shortenedUrl = data.shortenCode;
				this.originalUrlResult = data.url || this.originalUrl;
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

#app {
	font-family: Avenir, Helvetica, Arial, sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
	min-height: 100vh;
	background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
	display: flex;
	align-items: center;
	justify-content: center;
	padding: 20px;
}

.container {
	background: white;
	border-radius: 20px;
	padding: 40px;
	max-width: 600px;
	width: 100%;
	box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
}

h1 {
	color: #2c3e50;
	margin-bottom: 10px;
	font-size: 2.5em;
	text-align: center;
}

.subtitle {
	color: #7f8c8d;
	text-align: center;
	margin-bottom: 30px;
	font-size: 1em;
}

.input-section {
	margin-bottom: 30px;
}

.input-group {
	display: flex;
	gap: 10px;
	margin-bottom: 10px;
}

.url-input {
	flex: 1;
	padding: 15px 20px;
	font-size: 16px;
	border: 2px solid #e0e0e0;
	border-radius: 10px;
	outline: none;
	transition: border-color 0.3s;
}

.url-input:focus {
	border-color: #667eea;
}

.url-input:disabled {
	background-color: #f5f5f5;
	cursor: not-allowed;
}

.shorten-btn {
	padding: 15px 30px;
	font-size: 16px;
	font-weight: bold;
	color: white;
	background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
	border: none;
	border-radius: 10px;
	cursor: pointer;
	transition: transform 0.2s, box-shadow 0.2s;
	white-space: nowrap;
}

.shorten-btn:hover:not(:disabled) {
	transform: translateY(-2px);
	box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.shorten-btn:active:not(:disabled) {
	transform: translateY(0);
}

.shorten-btn:disabled {
	opacity: 0.6;
	cursor: not-allowed;
}

.error-message {
	color: #e74c3c;
	font-size: 14px;
	margin-top: 10px;
}

.result-section {
	background: #f8f9fa;
	border-radius: 15px;
	padding: 25px;
	margin-top: 20px;
}

.result-section h3 {
	color: #2c3e50;
	margin-bottom: 15px;
	font-size: 1.2em;
}

.result-container {
	display: flex;
	gap: 10px;
	margin-bottom: 15px;
}

.shortened-url {
	flex: 1;
	background: white;
	padding: 15px 20px;
	border-radius: 10px;
	border: 2px solid #667eea;
	word-break: break-all;
}

.shortened-url a {
	color: #667eea;
	text-decoration: none;
	font-weight: bold;
}

.shortened-url a:hover {
	text-decoration: underline;
}

.copy-btn {
	padding: 15px 25px;
	font-size: 16px;
	font-weight: bold;
	color: white;
	background: #27ae60;
	border: none;
	border-radius: 10px;
	cursor: pointer;
	transition: background 0.3s;
	white-space: nowrap;
}

.copy-btn:hover {
	background: #229954;
}

.original-url {
	color: #7f8c8d;
	font-size: 14px;
	word-break: break-all;
}

@media (max-width: 600px) {
	.container {
		padding: 30px 20px;
	}
	
	h1 {
		font-size: 2em;
	}
	
	.input-group {
		flex-direction: column;
	}
	
	.result-container {
		flex-direction: column;
	}
}
</style>
