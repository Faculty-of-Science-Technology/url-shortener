<template>
	<div id="app">
		<div class="header">
			<div class="logo">
				<span class="logo-main">Short URL</span>
				<span class="logo-by">ShortUrl by NameSilo.com</span>
			</div>
		</div>

		<div class="container">
			<div class="main-section" v-if="!shortenedUrl">
				<h1>Paste the URL to be shortened</h1>
				
				<div class="input-section">
					<input 
						v-model="originalUrl" 
						@keyup.enter="shortenUrl"
						type="text" 
						placeholder="Enter the link here" 
						class="url-input"
						:disabled="loading"
					/>
					<button 
						@click="shortenUrl" 
						class="shorten-btn"
						:disabled="loading || !originalUrl"
					>
						{{ loading ? 'Shortening...' : 'ShortURL!' }}
					</button>
					<p v-if="error" class="error-message">{{ error }}</p>
				</div>

				<div class="info-section">
					<p class="info-text">
						<strong>ShortURL</strong> is a free tool to shorten URLs and generate short links
					</p>
					<p class="info-text">
						URL shortener allows to create a shortened link making it easy to share
					</p>
				</div>
			</div>

			<div class="result-page" v-if="shortenedUrl">
				<h1 class="result-title">Your shortened URL</h1>
				<p class="result-subtitle">Copy the short link and share it in messages, texts, posts, websites and other locations.</p>

				<div class="result-box">
					<div class="url-display">
						<a :href="fullShortenedUrl" target="_blank" class="shortened-link">{{ fullShortenedUrl }}</a>
						<button @click="copyToClipboard" class="copy-button" :class="{ copied: copied }">
							{{ copied ? '✓ Copied' : 'Copy' }}
						</button>
					</div>
					<div class="long-url-display">
						<strong>Long URL:</strong> {{ originalUrlResult }}
					</div>
				</div>

				<button @click="resetForm" class="shorten-another-btn">Shorten another URL</button>

				<div class="note-section">
					<p class="note-text">* Short URLs that do not have at least one click per month are disabled</p>
				</div>

				<div class="share-section">
					<h3>Share it on social networks</h3>
					<div class="social-buttons">
						<a :href="'https://www.facebook.com/sharer/sharer.php?u=' + encodeURIComponent(fullShortenedUrl)" target="_blank" class="social-btn facebook">
							Facebook
						</a>
						<a :href="'https://twitter.com/intent/tweet?url=' + encodeURIComponent(fullShortenedUrl)" target="_blank" class="social-btn twitter">
							Twitter
						</a>
						<a :href="'https://www.linkedin.com/sharing/share-offsite/?url=' + encodeURIComponent(fullShortenedUrl)" target="_blank" class="social-btn linkedin">
							LinkedIn
						</a>
						<a :href="'https://api.whatsapp.com/send?text=' + encodeURIComponent(fullShortenedUrl)" target="_blank" class="social-btn whatsapp">
							WhatsApp
						</a>
					</div>
				</div>
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

.logo-by {
	font-size: 14px;
	color: #666;
	display: block;
	margin-top: 5px;
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

/* Result Page Styles */
.result-page {
	text-align: center;
}

.result-title {
	color: #333;
	font-size: 32px;
	margin-bottom: 15px;
	font-weight: normal;
}

.result-subtitle {
	color: #666;
	font-size: 16px;
	margin-bottom: 30px;
	line-height: 1.6;
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

.shorten-another-btn {
	padding: 15px 40px;
	font-size: 16px;
	font-weight: bold;
	color: white;
	background-color: #2196F3;
	border: none;
	border-radius: 4px;
	cursor: pointer;
	margin-bottom: 30px;
	transition: background-color 0.3s;
}

.shorten-another-btn:hover {
	background-color: #0b7dda;
}

.note-section {
	margin-bottom: 30px;
}

.note-text {
	color: #f44336;
	font-size: 14px;
	font-style: italic;
}

.share-section h3 {
	color: #333;
	font-size: 24px;
	margin-bottom: 20px;
	font-weight: normal;
}

.social-buttons {
	display: flex;
	flex-wrap: wrap;
	gap: 15px;
	justify-content: center;
}

.social-btn {
	padding: 12px 25px;
	color: white;
	text-decoration: none;
	border-radius: 4px;
	font-size: 14px;
	font-weight: bold;
	transition: opacity 0.3s;
}

.social-btn:hover {
	opacity: 0.8;
}

.social-btn.facebook {
	background-color: #3b5998;
}

.social-btn.twitter {
	background-color: #1DA1F2;
}

.social-btn.linkedin {
	background-color: #0077b5;
}

.social-btn.whatsapp {
	background-color: #25D366;
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
	
	.social-buttons {
		flex-direction: column;
	}
	
	.social-btn {
		width: 100%;
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
