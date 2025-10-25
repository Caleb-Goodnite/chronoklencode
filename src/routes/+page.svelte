<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Ultimate Cipher Site</title>
    <style>
        :root {
            --background-color: #121212;
            --text-color: #e0e0e0;
            --card-color: #1e1e1e;
            --accent-color: #007bff;
            --accent-color-hover: #0056b3;
            --border-color: #333;
            --spacing: 1.5em;
            --border-radius: 8px;
        }

        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            background-color: var(--background-color);
            color: var(--text-color);
            padding: var(--spacing);
            box-sizing: border-box;
        }

        .container {
            background-color: var(--card-color);
            padding: 2.5em;
            border-radius: var(--border-radius);
            box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);
            width: 90%;
            max-width: 650px;
            border: 1px solid var(--border-color);
            display: flex;
            flex-direction: column;
            gap: var(--spacing);
        }

        h1 {
            text-align: center;
            color: var(--text-color);
            margin-top: 0;
        }

        .controls {
            display: flex;
            flex-direction: column;
            gap: 1em;
        }
       
        .form-group {
            display: flex;
            flex-direction: column;
            gap: 0.5em;
        }

        label {
            font-weight: 600;
            color: var(--text-color);
        }

        textarea, input[type="number"], select {
            width: 100%;
            padding: 0.8em;
            box-sizing: border-box;
            background-color: #2a2a2a;
            border: 1px solid var(--border-color);
            border-radius: var(--border-radius);
            color: var(--text-color);
            font-size: 1em;
            transition: border-color 0.3s, box-shadow 0.3s;
        }
       
        textarea:focus, input:focus, select:focus {
            outline: none;
            border-color: var(--accent-color);
            box-shadow: 0 0 5px var(--accent-color);
        }

        .button-group {
            display: flex;
            gap: 1em;
            flex-wrap: wrap;
        }

        button {
            flex-grow: 1;
            padding: 1em;
            font-size: 1em;
            font-weight: 700;
            color: white;
            background-color: var(--accent-color);
            border: none;
            border-radius: var(--border-radius);
            cursor: pointer;
            transition: background-color 0.3s, transform 0.2s;
        }
       
        button:hover {
            background-color: var(--accent-color-hover);
            transform: translateY(-2px);
        }

        .output-area {
            display: flex;
            flex-direction: column;
            gap: 0.5em;
        }
       
        #result {
            padding: 1em;
            background-color: #2a2a2a;
            border: 1px dashed var(--border-color);
            border-radius: var(--border-radius);
            min-height: 5em;
            white-space: pre-wrap;
            word-wrap: break-word;
            font-family: monospace;
            overflow-y: auto;
        }

        .copyright {
            margin-top: var(--spacing);
            text-align: center;
            font-size: 0.8em;
            color: #777;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Encode.io</h1>
        <div class="controls">
            <div class="form-group">
                <label for="cipherSelect">Select Cipher:</label>
                <select id="cipherSelect">
                    <option value="caesar">Caesar</option>
                    <option value="hex">Hexadecimal</option>
                    <option value="baconian">Baconian</option>
                    <option value="binary">Binary</option>
                </select>
            </div>
            <div id="caesarOptions" class="form-group">
                <label for="shift">Shift Amount:</label>
                <input type="number" id="shift" value="3">
            </div>
            <div class="form-group">
                <label for="inputText">Input Text:</label>
                <textarea id="inputText" rows="4"></textarea>
            </div>
        </div>
        <div class="button-group">
            <button id="encodeBtn">Encode</button>
            <button id="decodeBtn">Decode</button>
        </div>
        <div class="output-area">
            <label for="result">Result:</label>
            <div id="result"></div>
        </div>
        <div class="copyright">
            © 2025 Caleb Goodnite. All rights reserved.
        </div>
    </div>

    <script>
        // --- CORE CIPHER LOGIC ---
        const ciphers = {
            caesar: {
                encode: (text, options) => {
                    const shift = parseInt(options.shift, 10);
                    let result = '';
                    for (let i = 0; i < text.length; i++) {
                        const char = text[i];
                        const code = char.charCodeAt(0);

                        if (code >= 65 && code <= 90) { // Uppercase
                            result += String.fromCharCode(((code - 65 + shift) % 26 + 26) % 26 + 65);
                        } else if (code >= 97 && code <= 122) { // Lowercase
                            result += String.fromCharCode(((code - 97 + shift) % 26 + 26) % 26 + 97);
                        } else {
                            result += char;
                        }
                    }
                    return result;
                },
                decode: (text, options) => {
                    const shift = -parseInt(options.shift, 10);
                    let result = '';
                    for (let i = 0; i < text.length; i++) {
                        const char = text[i];
                        const code = char.charCodeAt(0);

                        if (code >= 65 && code <= 90) { // Uppercase
                            result += String.fromCharCode(((code - 65 + shift) % 26 + 26) % 26 + 65);
                        } else if (code >= 97 && code <= 122) { // Lowercase
                            result += String.fromCharCode(((code - 97 + shift) % 26 + 26) % 26 + 97);
                        } else {
                            result += char;
                        }
                    }
                    return result;
                }
            },
            hex: {
                encode: (text) => {
                    return Array.from(text).map(char => {
                        const hex = char.charCodeAt(0).toString(16);
                        return hex.length === 1 ? '0' + hex : hex;
                    }).join('');
                },
                decode: (text) => {
                    text = text.replace(/\s/g, ''); // Remove spaces
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
                        'A': 'aaaaa', 'B': 'aaaab', 'C': 'aaaba', 'D': 'aaabb', 'E': 'aabaa',
                        'F': 'aabab', 'G': 'aabba', 'H': 'aabbb', 'I': 'abaaa', 'J': 'abaab',
                        'K': 'ababa', 'L': 'ababb', 'M': 'ababa', 'N': 'abbab', 'O': 'abbba',
                        'P': 'abbbb', 'Q': 'baaaa', 'R': 'baaab', 'S': 'baaba', 'T': 'baabb',
                        'U': 'babaa', 'V': 'babab', 'W': 'babba', 'X': 'babbb', 'Y': 'bbaaa', 'Z': 'bbaab'
                    }; // Note: I/J and U/V are often mapped to the same code in classic Baconian. Here, they're distinct.
                    // If you want I/J and U/V mapped to the same codes, you would modify this map:
                    // 'I': 'abaaa', 'J': 'abaaa', 'U': 'babaa', 'V': 'babaa'
                    return text.toUpperCase().split('').map(char => {
                        return baconMap[char] || char; // Preserve non-alpha characters
                    }).join(' ');
                },
                decode: (text) => {
                    const baconMap = {
                        'aaaaa': 'A', 'aaaab': 'B', 'aaaba': 'C', 'aaabb': 'D', 'aabaa': 'E',
                        'aabab': 'F', 'aabba': 'G', 'aabbb': 'H', 'abaaa': 'I', 'abaab': 'J',
                        'ababa': 'K', 'ababb': 'L', 'ababa': 'M', 'abbab': 'N', 'abbba': 'O',
                        'abbbb': 'P', 'baaaa': 'Q', 'baaab': 'R', 'baaba': 'S', 'baabb': 'T',
                        'babaa': 'U', 'babab': 'V', 'babba': 'W', 'babbb': 'X', 'bbaaa': 'Y', 'bbaab': 'Z'
                    }; // Corresponding map for decoding
                    const parts = text.split(' '); // Split by space to get individual Baconian codes
                    return parts.map(part => {
                        return baconMap[part.toLowerCase()] || part; // Handle non-Baconian codes (e.g., spaces, punctuation)
                    }).join('');
                }
            },
            binary: {
                encode: (text) => {
                    return Array.from(text).map(char => {
                        // Convert character to its ASCII code, then to binary, padding to 8 bits
                        return char.charCodeAt(0).toString(2).padStart(8, '0');
                    }).join(' '); // Join with spaces for readability
                },
                decode: (text) => {
                    text = text.replace(/\s+/g, ' ').trim(); // Normalize spaces
                    if (!/^[01\s]+$/.test(text)) { // Basic validation for binary string
                        return 'Invalid binary string. Contains non-binary characters.';
                    }
                    const binaryChunks = text.split(' ');
                    try {
                        return binaryChunks.map(chunk => {
                            if (chunk.length === 0) return ''; // Handle multiple spaces
                            return String.fromCharCode(parseInt(chunk, 2));
                        }).join('');
                    } catch (e) {
                        return 'Error decoding binary. Ensure it is valid.';
                    }
                }
            }
        };

        // --- UI LOGIC ---
        const cipherSelect = document.getElementById('cipherSelect');
        const inputText = document.getElementById('inputText');
        const shiftInput = document.getElementById('shift');
        const encodeBtn = document.getElementById('encodeBtn');
        const decodeBtn = document.getElementById('decodeBtn');
        const resultDiv = document.getElementById('result');
        const caesarOptionsDiv = document.getElementById('caesarOptions');

        const updateUI = () => {
            const selectedCipher = cipherSelect.value;
            // Only show shift amount for Caesar cipher
            caesarOptionsDiv.style.display = (selectedCipher === 'caesar') ? 'flex' : 'none';
            // You could add similar logic here for other cipher-specific options if they are added later
        };

        const handleCipherAction = (action) => {
            const selectedCipher = cipherSelect.value;
            const text = inputText.value;
            const cipherFunction = ciphers[selectedCipher][action];

            if (cipherFunction) {
                const options = { // Pass relevant options
                    shift: shiftInput.value
                };
                const output = cipherFunction(text, options);
                resultDiv.textContent = output;
            } else {
                resultDiv.textContent = `Error: No ${action} function defined for ${selectedCipher}.`;
            }
        };

        // Event listeners
        cipherSelect.addEventListener('change', updateUI);
        encodeBtn.addEventListener('click', () => handleCipherAction('encode'));
        decodeBtn.addEventListener('click', () => handleCipherAction('decode'));

        // Initial UI setup
        updateUI();
    </script>
</body>
</html>