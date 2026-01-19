<template>
	<div id="app">
		<div class="container">
			<h1>URL Shortener</h1>
			<p class="subtitle">A modern, minimalist, and lightweight URL shortener</p>
			
			<div class="input-section">
				<div class="input-group">
					<input 
						v-model="originalUrl" 
						@keyup.enter="shortenUrl"
						type="text" 
						placeholder="Enter your long URL here..." 
						class="url-input"
						:disabled="loading"
					/>
					<button 
						@click="shortenUrl" 
						class="shorten-btn"
						:disabled="loading || !originalUrl"
					>
						{{ loading ? 'Shortening...' : 'Shorten' }}
					</button>
				</div>
				<p v-if="error" class="error-message">{{ error }}</p>
			</div>

			<div v-if="shortenedUrl" class="result-section">
				<h3>Your shortened URL:</h3>
				<div class="result-container">
					<div class="shortened-url">
						<a :href="fullShortenedUrl" target="_blank">{{ fullShortenedUrl }}</a>
					</div>
					<button @click="copyToClipboard" class="copy-btn">
						{{ copied ? 'Copied!' : 'Copy' }}
					</button>
				</div>
				<p class="original-url">Original: {{ originalUrlResult }}</p>
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
