<script>
	let inputText = '';
	let outputText = '';
	let mode = 'encode';
	let copiedMessage = '';
	
	let cipherChain = [
		{ id: 1, type: 'base64', shift: 3 }
	];
	let nextId = 2;

	function addCipher() {
		cipherChain = [...cipherChain, { id: nextId++, type: 'base64', shift: 3 }];
	}

	function removeCipher(id) {
		if (cipherChain.length > 1) {
			cipherChain = cipherChain.filter(c => c.id !== id);
		}
	}

	const ciphers = {
		caesar: {
			encode: (text, shift) => {
				const shiftAmount = parseInt(shift, 10);
				let result = '';
				for (let i = 0; i < text.length; i++) {
					const char = text[i];
					const code = char.charCodeAt(0);
					if (code >= 65 && code <= 90) {
						result += String.fromCharCode(((code - 65 + shiftAmount) % 26 + 26) % 26 + 65);
					} else if (code >= 97 && code <= 122) {
						result += String.fromCharCode(((code - 97 + shiftAmount) % 26 + 26) % 26 + 97);
					} else {
						result += char;
					}
				}
				return result;
			},
			decode: (text, shift) => {
				const shiftAmount = -parseInt(shift, 10);
				let result = '';
				for (let i = 0; i < text.length; i++) {
					const char = text[i];
					const code = char.charCodeAt(0);
					if (code >= 65 && code <= 90) {
						result += String.fromCharCode(((code - 65 + shiftAmount) % 26 + 26) % 26 + 65);
					} else if (code >= 97 && code <= 122) {
						result += String.fromCharCode(((code - 97 + shiftAmount) % 26 + 26) % 26 + 97);
					} else {
						result += char;
					}
				}
				return result;
			}
		},
		base64: {
			encode: (text) => {
				try {
					return btoa(unescape(encodeURIComponent(text)));
				} catch (e) {
					return 'Error encoding to Base64';
				}
			},
			decode: (text) => {
				try {
					return decodeURIComponent(escape(atob(text)));
				} catch (e) {
					return 'Invalid Base64 string';
				}
			}
		},
		hex: {
			encode: (text) => {
				return Array.from(text).map(char => {
					const hex = char.charCodeAt(0).toString(16);
					return hex.length === 1 ? '0' + hex : hex;
				}).join(' ');
			},
			decode: (text) => {
				text = text.replace(/\s/g, '');
				if (text.length % 2 !== 0) {
					return 'Invalid hex string';
				}
				let result = '';
				for (let i = 0; i < text.length; i += 2) {
					const hex = text.substring(i, i + 2);
					result += String.fromCharCode(parseInt(hex, 16));
				}
				return result;
			}
		},
		baconian: {
			encode: (text) => {
				const baconMap = {
					A: 'aaaaa', B: 'aaaab', C: 'aaaba', D: 'aaabb', E: 'aabaa',
					F: 'aabab', G: 'aabba', H: 'aabbb', I: 'abaaa', J: 'abaab',
					K: 'ababa', L: 'ababb', M: 'abbaa', N: 'abbab', O: 'abbba',
					P: 'abbbb', Q: 'baaaa', R: 'baaab', S: 'baaba', T: 'baabb',
					U: 'babaa', V: 'babab', W: 'babba', X: 'babbb', Y: 'bbaaa', Z: 'bbaab'
				};
				return text
					.toUpperCase()
					.split('')
					.map((char) => baconMap[char] || char)
					.join(' ');
			},
			decode: (text) => {
				const baconMap = {
					aaaaa: 'A', aaaab: 'B', aaaba: 'C', aaabb: 'D', aabaa: 'E',
					aabab: 'F', aabba: 'G', aabbb: 'H', abaaa: 'I', abaab: 'J',
					ababa: 'K', ababb: 'L', abbaa: 'M', abbab: 'N', abbba: 'O',
					abbbb: 'P', baaaa: 'Q', baaab: 'R', baaba: 'S', baabb: 'T',
					babaa: 'U', babab: 'V', babba: 'W', babbb: 'X', bbaaa: 'Y', bbaab: 'Z'
				};
				const parts = text.split(' ');
				return parts.map((part) => baconMap[part.toLowerCase()] || part).join('');
			}
		},
		binary: {
			encode: (text) => {
				return Array.from(text)
					.map((char) => char.charCodeAt(0).toString(2).padStart(8, '0'))
					.join(' ');
			},
			decode: (text) => {
				text = text.replace(/\s+/g, ' ').trim();
				if (!/^[01\s]+$/.test(text)) {
					return 'Invalid binary string';
				}
				const binaryChunks = text.split(' ');
				try {
					return binaryChunks
						.map((chunk) => {
							if (chunk.length === 0) return '';
							return String.fromCharCode(parseInt(chunk, 2));
						})
						.join('');
				} catch (e) {
					return 'Error decoding binary';
				}
			}
		}
	};

	$: {
		let result = inputText;
		
		if (mode === 'encode') {
			for (const cipher of cipherChain) {
				if (result && ciphers[cipher.type]) {
					result = ciphers[cipher.type].encode(result, cipher.shift);
				}
			}
		} else {
			for (const cipher of [...cipherChain].reverse()) {
				if (result && ciphers[cipher.type]) {
					result = ciphers[cipher.type].decode(result, cipher.shift);
				}
			}
		}
		
		outputText = result;
	}

	async function copyToClipboard() {
		try {
			await navigator.clipboard.writeText(outputText);
			copiedMessage = 'Copied!';
			setTimeout(() => {
				copiedMessage = '';
			}, 2000);
		} catch (err) {
			copiedMessage = 'Failed to copy';
			setTimeout(() => {
				copiedMessage = '';
			}, 2000);
		}
	}
</script>

<svelte:head>
	<title>Encode.io - Advanced Cipher Tool</title>
</svelte:head>

<div class="page">
	<header class="header">
		<h1>Chronokl Encoder</h1>
		<p class="subtitle">Real-time encoding with cipher chaining</p>
	</header>

	<main class="main-content">
		<div class="cipher-chain">
			<div class="chain-header">
				<h2>Cipher Chain</h2>
				<button class="add-btn" on:click={addCipher}>+ Add Cipher</button>
			</div>
			
			<div class="chain-list">
				{#each cipherChain as cipher, index (cipher.id)}
					<div class="cipher-item">
						<span class="cipher-number">{index + 1}</span>
						<select bind:value={cipher.type} class="cipher-select">
							<option value="base64">Base64</option>
							<option value="hex">Hexadecimal</option>
							<option value="binary">Binary</option>
							<option value="caesar">Caesar</option>
							<option value="baconian">Baconian</option>
						</select>
						
						{#if cipher.type === 'caesar'}
							<input
								type="number"
								bind:value={cipher.shift}
								min="1"
								max="25"
								class="shift-input"
								placeholder="Shift"
							/>
						{/if}
						
						{#if cipherChain.length > 1}
							<button class="remove-btn" on:click={() => removeCipher(cipher.id)}>×</button>
						{/if}
					</div>
				{/each}
			</div>

			<div class="mode-toggle">
				<button 
					class="mode-btn {mode === 'encode' ? 'active' : ''}" 
					on:click={() => mode = 'encode'}
				>
					Encode
				</button>
				<button 
					class="mode-btn {mode === 'decode' ? 'active' : ''}" 
					on:click={() => mode = 'decode'}
				>
					Decode
				</button>
			</div>
		</div>

		<div class="editor-container">
			<div class="editor-panel">
				<label class="panel-label">Input</label>
				<textarea
					bind:value={inputText}
					placeholder="Enter text here..."
					class="editor-textarea"
				></textarea>
			</div>

			<div class="arrow">→</div>

			<div class="editor-panel">
				<div class="panel-header">
					<label class="panel-label">Output</label>
					{#if outputText}
						<button class="copy-btn" on:click={copyToClipboard}>
							{copiedMessage || 'Copy'}
						</button>
					{/if}
				</div>
				<textarea
					value={outputText}
					readonly
					placeholder="Output will appear here..."
					class="editor-textarea output"
				></textarea>
			</div>
		</div>
	</main>

	<footer class="footer">
		<p>© 2025 Caleb Goodnite. All rights reserved.</p>
	</footer>
</div>

<style>
	:global(body) {
		margin: 0;
		padding: 0;
		font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif;
		background: #0a0a0a;
		color: #e0e0e0;
	}

	.page {
		min-height: 100vh;
		display: flex;
		flex-direction: column;
		padding: 2rem;
		max-width: 1400px;
		margin: 0 auto;
	}

	.header {
		text-align: center;
		margin-bottom: 2rem;
	}

	.header h1 {
		font-size: 3rem;
		margin: 0;
		background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
		-webkit-background-clip: text;
		-webkit-text-fill-color: transparent;
		background-clip: text;
	}

	.subtitle {
		color: #888;
		margin-top: 0.5rem;
		font-size: 1.1rem;
	}

	.main-content {
		flex: 1;
		display: flex;
		gap: 2rem;
		flex-wrap: wrap;
	}

	.cipher-chain {
		background: #1a1a1a;
		border-radius: 12px;
		padding: 1.5rem;
		min-width: 300px;
		max-width: 350px;
		flex-shrink: 0;
		border: 1px solid #2a2a2a;
		height: fit-content;
	}

	.chain-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1.5rem;
	}

	.chain-header h2 {
		margin: 0;
		font-size: 1.3rem;
		color: #fff;
	}

	.add-btn {
		background: #667eea;
		color: white;
		border: none;
		padding: 0.5rem 1rem;
		border-radius: 6px;
		cursor: pointer;
		font-weight: 600;
		font-size: 0.9rem;
		transition: all 0.2s;
	}

	.add-btn:hover {
		background: #5568d3;
		transform: translateY(-1px);
	}

	.chain-list {
		display: flex;
		flex-direction: column;
		gap: 0.75rem;
		margin-bottom: 1.5rem;
	}

	.cipher-item {
		display: flex;
		align-items: center;
		gap: 0.75rem;
		background: #0f0f0f;
		padding: 0.75rem;
		border-radius: 8px;
		border: 1px solid #2a2a2a;
	}

	.cipher-number {
		background: #667eea;
		color: white;
		width: 28px;
		height: 28px;
		border-radius: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		font-weight: 700;
		font-size: 0.85rem;
		flex-shrink: 0;
	}

	.cipher-select {
		flex: 1;
		background: #2a2a2a;
		border: 1px solid #3a3a3a;
		color: #e0e0e0;
		padding: 0.5rem;
		border-radius: 6px;
		font-size: 0.9rem;
		cursor: pointer;
	}

	.cipher-select:focus {
		outline: none;
		border-color: #667eea;
	}

	.shift-input {
		width: 60px;
		background: #2a2a2a;
		border: 1px solid #3a3a3a;
		color: #e0e0e0;
		padding: 0.5rem;
		border-radius: 6px;
		font-size: 0.9rem;
	}

	.shift-input:focus {
		outline: none;
		border-color: #667eea;
	}

	.remove-btn {
		background: #ff4444;
		color: white;
		border: none;
		width: 28px;
		height: 28px;
		border-radius: 50%;
		cursor: pointer;
		font-size: 1.3rem;
		display: flex;
		align-items: center;
		justify-content: center;
		transition: all 0.2s;
		flex-shrink: 0;
	}

	.remove-btn:hover {
		background: #cc0000;
		transform: scale(1.1);
	}

	.mode-toggle {
		display: flex;
		gap: 0.5rem;
		background: #0f0f0f;
		padding: 0.25rem;
		border-radius: 8px;
	}

	.mode-btn {
		flex: 1;
		background: transparent;
		color: #888;
		border: none;
		padding: 0.75rem;
		border-radius: 6px;
		cursor: pointer;
		font-weight: 600;
		transition: all 0.2s;
	}

	.mode-btn.active {
		background: #667eea;
		color: white;
	}

	.mode-btn:hover:not(.active) {
		background: #1a1a1a;
		color: #e0e0e0;
	}

	.editor-container {
		flex: 1;
		display: flex;
		gap: 1.5rem;
		min-width: 0;
	}

	.editor-panel {
		flex: 1;
		display: flex;
		flex-direction: column;
		background: #1a1a1a;
		border-radius: 12px;
		padding: 1.5rem;
		border: 1px solid #2a2a2a;
		min-width: 0;
	}

	.panel-header {
		display: flex;
		justify-content: space-between;
		align-items: center;
		margin-bottom: 1rem;
	}

	.panel-label {
		font-weight: 700;
		font-size: 1.1rem;
		color: #fff;
		margin-bottom: 1rem;
	}

	.copy-btn {
		background: #2a2a2a;
		color: #e0e0e0;
		border: 1px solid #3a3a3a;
		padding: 0.5rem 1rem;
		border-radius: 6px;
		cursor: pointer;
		font-weight: 600;
		font-size: 0.85rem;
		transition: all 0.2s;
	}

	.copy-btn:hover {
		background: #667eea;
		border-color: #667eea;
		color: white;
	}

	.editor-textarea {
		flex: 1;
		background: #0f0f0f;
		border: 1px solid #2a2a2a;
		color: #e0e0e0;
		padding: 1rem;
		border-radius: 8px;
		font-family: 'Consolas', 'Monaco', 'Courier New', monospace;
		font-size: 0.95rem;
		resize: none;
		line-height: 1.6;
		min-height: 400px;
	}

	.editor-textarea:focus {
		outline: none;
		border-color: #667eea;
	}

	.editor-textarea.output {
		background: #0a0a0a;
	}

	.arrow {
		display: flex;
		align-items: center;
		font-size: 2rem;
		color: #667eea;
		font-weight: 700;
	}

	.footer {
		text-align: center;
		margin-top: 3rem;
		padding-top: 2rem;
		border-top: 1px solid #2a2a2a;
		color: #666;
		font-size: 0.9rem;
	}

	@media (max-width: 1024px) {
		.main-content {
			flex-direction: column;
		}

		.cipher-chain {
			max-width: 100%;
		}

		.editor-container {
			flex-direction: column;
		}

		.arrow {
			transform: rotate(90deg);
			margin: 0;
		}
	}

	@media (max-width: 640px) {
		.page {
			padding: 1rem;
		}

		.header h1 {
			font-size: 2rem;
		}

		.cipher-item {
			flex-wrap: wrap;
		}

		.shift-input {
			width: 100%;
		}
	}
</style>