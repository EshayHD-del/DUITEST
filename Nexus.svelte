<script lang="ts">
	import { fade } from 'svelte/transition';
	import { onMount } from 'svelte';

	let current = 0;
	let activeMenu: any[] = [];
	let showing = false;
	let itemRefs: HTMLElement[] = [];

	// NEW: Highlight bar positioning
	let highlightTop = 0;
	let highlightHeight = 0;
	let menuContentElement: HTMLElement;

	// NEW: Scrollbar indicator positioning
	let scrollbarIndicatorHeight = 0;
	let scrollbarThumbTop = 0;
	let scrollbarThumbHeight = 0;

	// Text input dialog state
	let showTextInput = false;
	let textInputValue = '';
	let textInputQuestion = '';
	let textInputPlaceholder = '';
	let textInputMaxLength = 100;
	let textInputType = 'general';

	// Key selection dialog state
	let showKeySelection = false;
	let keySelectionTitle = '';
	let keySelectionInstruction = '';
	let keySelectionHint = '';
	let selectedKey = null;
	let selectedKeyName = '';

	// Keybind setup dialog state
	let showKeybindSetup = false;
	let keybindSetupTitle = '';
	let keybindFeatureName = '';
	let keybindInstruction = '';
	let keybindHint = '';
	let keybindSelectedKey = null;
	let keybindSelectedKeyName = '';

	// Banner settings
	let bannerLink = 'https://share.creavite.co/68add0153ab89cb91d355b6e.gif';
	let bannerColor = '#e74c3c';
	let bannerSpacing = 5;
	let breadcrumb = 'Home';

	// NEW: Image display state for weapons and models
	let showImage = false;
	let currentImageUrl = '';
	let currentImageLabel = '';
	let currentImageType = ''; // 'weapon' or 'model'

	// Dialog active state tracking
	let isDialogActive = false;

	type Notification = {
		id: number;
		message: string;
		type: 'info' | 'success' | 'warn' | 'error';
	};

	let notifications: Notification[] = [];

	const showNotification = (message: string, type: Notification['type']) => {
		const id = Date.now();
		notifications = [...notifications, { id, message, type }];
		setTimeout(() => {
			notifications = notifications.filter((n) => n.id !== id);
		}, 4000);
	};

	// NEW: Update highlight bar position
	const updateHighlightPosition = () => {
		if (!itemRefs[current] || !menuContentElement) return;
		
		const activeItem = itemRefs[current];
		const menuContentRect = menuContentElement.getBoundingClientRect();
		const activeItemRect = activeItem.getBoundingClientRect();
		
		// Calculate position relative to menu content container
		highlightTop = activeItemRect.top - menuContentRect.top + menuContentElement.scrollTop;
		highlightHeight = activeItemRect.height;
	};

	// NEW: Update scrollbar indicator position
	const updateScrollbarIndicator = () => {
		if (!menuContentElement || activeMenu.length === 0) {
			scrollbarIndicatorHeight = 0;
			return;
		}
		
		const { scrollTop, scrollHeight, clientHeight } = menuContentElement;
		
		// Only show scrollbar if content overflows
		if (scrollHeight <= clientHeight) {
			scrollbarIndicatorHeight = 0;
			return;
		}
		
		// Set track height to match menu content area
		scrollbarIndicatorHeight = clientHeight - 1; // Small margin
		
		// Calculate thumb height (minimum 3vh for visibility)
		const visibleRatio = clientHeight / scrollHeight;
		scrollbarThumbHeight = Math.max(scrollbarIndicatorHeight * visibleRatio, 3);
		
		// Calculate thumb position
		const maxScroll = scrollHeight - clientHeight;
		const scrollRatio = maxScroll > 0 ? scrollTop / maxScroll : 0;
		const maxThumbTop = scrollbarIndicatorHeight - scrollbarThumbHeight;
		scrollbarThumbTop = maxThumbTop * scrollRatio;
		
		console.log('Scrollbar Debug:', {
			scrollHeight,
			clientHeight,
			scrollTop,
			scrollRatio,
			thumbTop: scrollbarThumbTop,
			thumbHeight: scrollbarThumbHeight,
			trackHeight: scrollbarIndicatorHeight
		});
	};

	// Update both highlight and scrollbar when content changes
	const updatePositions = () => {
		updateHighlightPosition();
		updateScrollbarIndicator();
	};

	// Watch for current item changes to update highlight
	$: if (current >= 0 && itemRefs[current]) {
		setTimeout(updatePositions, 10);
	}

	// Watch for menu changes to update scrollbar
	$: if (activeMenu.length > 0 && menuContentElement) {
		setTimeout(updatePositions, 10);
	}

	// Send message to Lua
	const sendToLua = (action: string, data: any = {}) => {
		console.log('Sending to Lua:', action, data);
	};

	// Enhanced key mapping for FiveM compatibility
	const mapKeyToControlId = (event: KeyboardEvent): { controlId: number | null, keyName: string } => {
		const key = event.key;
		const code = event.code;
		let controlId = null;
		let keyName = '';

		if ((showKeySelection || showKeybindSetup) && (key === 'Enter' || key === 'Escape')) {
			return { controlId: null, keyName: '' };
		}

		// Map to FiveM control IDs
		switch(code) {
			case 'PageDown': controlId = 11; keyName = 'Page Down'; break;
			case 'KeyQ': controlId = 44; keyName = 'Q Key'; break;
			case 'KeyE': controlId = 38; keyName = 'E Key'; break;
			case 'KeyF': controlId = 23; keyName = 'F Key'; break;
			case 'F2': controlId = 75; keyName = 'F2 Key'; break;
			case 'Insert': controlId = 121; keyName = 'Insert Key'; break;
			case 'Delete': controlId = 178; keyName = 'Delete Key'; break;
			case 'Home': controlId = 213; keyName = 'Home Key'; break;
			case 'End': controlId = 214; keyName = 'End Key'; break;
			case 'ControlLeft': controlId = 96; keyName = 'Left Ctrl'; break;
			case 'AltLeft': controlId = 97; keyName = 'Left Alt'; break;
			case 'ShiftLeft': controlId = 21; keyName = 'Left Shift'; break;
			case 'KeyR': controlId = 19; keyName = 'R Key'; break;
			case 'KeyT': controlId = 245; keyName = 'T Key'; break;
			case 'KeyY': controlId = 246; keyName = 'Y Key'; break;
			case 'KeyU': controlId = 303; keyName = 'U Key'; break;
			// Function keys F1-F12
			case 'F1': controlId = 3000; keyName = 'F1 Key'; break;
			case 'F2': controlId = 3001; keyName = 'F2 Key'; break;
			case 'F3': controlId = 3002; keyName = 'F3 Key'; break;
			case 'F4': controlId = 3003; keyName = 'F4 Key'; break;
			case 'F5': controlId = 3004; keyName = 'F5 Key'; break;
			case 'F6': controlId = 3005; keyName = 'F6 Key'; break;
			case 'F7': controlId = 3006; keyName = 'F7 Key'; break;
			case 'F8': controlId = 3007; keyName = 'F8 Key'; break;
			case 'F9': controlId = 3008; keyName = 'F9 Key'; break;
			case 'F10': controlId = 3009; keyName = 'F10 Key'; break;
			case 'F11': controlId = 3010; keyName = 'F11 Key'; break;
			case 'F12': controlId = 3011; keyName = 'F12 Key'; break;
			default:
				// Letters A-Z
				if (code.startsWith('Key') && code.length === 4) {
					const letter = code.substring(3);
					const letterIndex = letter.charCodeAt(0) - 65;
					if (letterIndex >= 0 && letterIndex <= 25) {
						controlId = 1000 + letterIndex;
						keyName = letter + ' Key';
					}
				}
				// Numbers 0-9
				else if (code.startsWith('Digit') && code.length === 6) {
					const digit = parseInt(code.substring(5));
					controlId = 2000 + digit;
					keyName = digit + ' Key';
				}
				break;
		}

		return { controlId, keyName };
	};

	// F9 handler for keybind setup
	const handleF9KeyPress = (event: KeyboardEvent) => {
		if (event.key !== 'F9') return;
		
		event.preventDefault();
		event.stopPropagation();

		if (isDialogActive) {
			showNotification('Close current dialog first before using F9', 'warn');
			return;
		}

		if (!showing) {
			showNotification('Open menu first to use F9 keybind setup', 'info');
			return;
		}

		const currentItem = activeMenu[current];
		if (!currentItem) {
			showNotification('No menu item selected', 'error');
			return;
		}

		if (!currentItem.canBind) {
			showNotification('This feature cannot be bound to a key', 'warn');
			return;
		}

		startKeybindSetup(currentItem);
	};

	// Keybind setup functions
	const startKeybindSetup = (feature: any) => {
		showTextInput = false;
		showKeySelection = false;
		isDialogActive = true;
		showKeybindSetup = true;
		
		keybindSetupTitle = 'Keybind Setup';
		keybindFeatureName = feature.bindName || feature.label;
		keybindInstruction = 'Press any key to bind to this feature';
		keybindHint = 'ESC to cancel';
		keybindSelectedKey = null;
		keybindSelectedKeyName = '';
		
		setTimeout(() => window.focus(), 100);
	};

	const handleKeybindSetupInput = (event: KeyboardEvent) => {
		if (!showKeybindSetup) return;
		
		event.preventDefault();
		event.stopPropagation();

		if (event.key === 'Escape') {
			cancelKeybindSetup();
			return;
		}

		if (event.key === 'Enter') {
			if (keybindSelectedKey !== null) {
				confirmKeybindSetup();
			} else {
				showNotification('Select a key first', 'warn');
			}
			return;
		}

		const { controlId, keyName } = mapKeyToControlId(event);
		if (controlId && keyName) {
			keybindSelectedKey = controlId;
			keybindSelectedKeyName = keyName;
		} else {
			showNotification('Key cannot be bound', 'warn');
		}
	};

	const confirmKeybindSetup = () => {
		sendToLua('keybindSetupConfirm', { 
			controlId: keybindSelectedKey, 
			keyName: keybindSelectedKeyName,
			featureName: keybindFeatureName
		});
		showNotification(`Keybind set: ${keybindSelectedKeyName} -> ${keybindFeatureName}`, 'success');
		closeKeybindSetup();
	};

	const cancelKeybindSetup = () => {
		sendToLua('keybindSetupCancel');
		showNotification('Keybind setup canceled', 'info');
		closeKeybindSetup();
	};

	const closeKeybindSetup = () => {
		showKeybindSetup = false;
		isDialogActive = false;
		keybindSetupTitle = '';
		keybindFeatureName = '';
		keybindInstruction = '';
		keybindHint = '';
		keybindSelectedKey = null;
		keybindSelectedKeyName = '';
	};

	// Key selection functions
	const handleKeySelectionInput = (event: KeyboardEvent) => {
		if (!showKeySelection) return;
		
		event.preventDefault();
		event.stopPropagation();

		if (event.key === 'Escape') {
			cancelKeySelection();
			return;
		}

		if (event.key === 'Enter' && selectedKey) {
			confirmKeySelection();
			return;
		}

		const { controlId, keyName } = mapKeyToControlId(event);
		if (controlId && keyName) {
			selectedKey = controlId;
			selectedKeyName = keyName;
		}
	};

	const confirmKeySelection = () => {
		sendToLua('keySelectionConfirm', { 
			controlId: selectedKey, 
			keyName: selectedKeyName 
		});
		closeKeySelection();
	};

	const cancelKeySelection = () => {
		sendToLua('keySelectionCancel');
		closeKeySelection();
	};

	const closeKeySelection = () => {
		showKeySelection = false;
		isDialogActive = false;
		keySelectionTitle = '';
		keySelectionInstruction = '';
		keySelectionHint = '';
		selectedKey = null;
		selectedKeyName = '';
	};

	// Text input functions
	const isValidCharacter = (char: string, inputType: string): boolean => {
		if (inputType === 'numeric') {
			return /^[0-9]$/.test(char);
		} else if (inputType === 'alphanumeric') {
			return /^[a-zA-Z0-9_]$/.test(char);
		}
		return /^[\x20-\x7E]$/.test(char);
	};

	const handleTextInputKeyboard = (event: KeyboardEvent) => {
		if (!showTextInput) return;
		
		if (event.key !== 'F5' && event.key !== 'F12') {
			event.preventDefault();
		}

		if (event.key === 'Enter') {
			submitTextInput();
			return;
		}

		if (event.key === 'Escape') {
			cancelTextInput();
			return;
		}

		if (event.key === 'Backspace') {
			if (textInputValue.length > 0) {
				textInputValue = textInputValue.slice(0, -1);
			}
			return;
		}

		if (event.key === 'Delete') {
			textInputValue = '';
			return;
		}

		if (event.ctrlKey) {
			if (event.key.toLowerCase() === 'v') {
				handlePaste();
				return;
			}
			if (event.key.toLowerCase() === 'a') {
				textInputValue = '';
				return;
			}
			return;
		}

		if (event.key.length === 1) {
			const char = event.key;
			if (isValidCharacter(char, textInputType)) {
				if (textInputValue.length < textInputMaxLength) {
					textInputValue += char;
				}
			}
		}
	};

	const handlePaste = async () => {
		try {
			const text = await navigator.clipboard.readText();
			const validText = text.split('').filter(char => 
				isValidCharacter(char, textInputType)
			).join('');
			
			const newValue = textInputValue + validText;
			if (newValue.length <= textInputMaxLength) {
				textInputValue = newValue;
			} else {
				textInputValue = newValue.substring(0, textInputMaxLength);
			}
		} catch (err) {
			sendToLua('requestClipboard');
		}
	};

	const submitTextInput = () => {
		sendToLua('textInputSubmit', { value: textInputValue });
		closeTextInput();
	};

	const cancelTextInput = () => {
		sendToLua('textInputCancel');
		closeTextInput();
	};

	const closeTextInput = () => {
		showTextInput = false;
		isDialogActive = false;
		textInputValue = '';
		textInputQuestion = '';
		textInputPlaceholder = '';
		textInputMaxLength = 100;
		textInputType = 'general';
	};

	// NEW: Image display functions
	const showImageDisplay = (imageType: string, imageUrl: string, label: string) => {
		currentImageType = imageType;
		currentImageUrl = imageUrl;
		currentImageLabel = label;
		showImage = true;
	};

	const hideImageDisplay = () => {
		showImage = false;
		currentImageUrl = '';
		currentImageLabel = '';
		currentImageType = '';
	};

	// Global keyboard handler
	const handleGlobalKeyDown = (event: KeyboardEvent) => {
		if (event.key === 'F9') {
			handleF9KeyPress(event);
			return;
		}

		if (showKeybindSetup) {
			handleKeybindSetupInput(event);
		} else if (showKeySelection) {
			handleKeySelectionInput(event);
		} else if (showTextInput) {
			handleTextInputKeyboard(event);
		}
	};

	// Update dialog active state when dialogs change
	$: isDialogActive = showTextInput || showKeySelection || showKeybindSetup;

	onMount(() => {
		window.addEventListener('keydown', handleGlobalKeyDown);
		
		// Update positions on window resize and scroll
		const handleResize = () => setTimeout(updatePositions, 10);
		const handleScroll = () => setTimeout(updatePositions, 10);
		
		window.addEventListener('resize', handleResize);
		
		return () => {
			window.removeEventListener('keydown', handleGlobalKeyDown);
			window.removeEventListener('resize', handleResize);
		};
	});

	// Enhanced message handler
	window.addEventListener('message', (event) => {
		const data = event.data;
		const action = data.action;

		try {
			switch(action) {
				case 'setCurrent':
					current = data.current - 1;
					activeMenu = data.menu;
					showing = true;
					setTimeout(() => {
						itemRefs[current]?.scrollIntoView({
							behavior: 'smooth',
							block: 'nearest'
						});
						updatePositions();
					}, 100);
					break;

				case 'setVisible':
					showing = data.visible;
					if (showing) {
						setTimeout(() => {
							itemRefs[current]?.scrollIntoView({
								behavior: 'smooth',
								block: 'nearest'
							});
							updatePositions();
						}, 100);
					} else {
						hideImageDisplay();
					}
					break;

				case 'notify':
					showNotification(data.message, data.type || 'info');
					break;

				case 'openTextInput':
					textInputValue = data.value || '';
					textInputQuestion = data.question || 'Enter text:';
					textInputPlaceholder = data.placeholder || 'Type here...';
					textInputMaxLength = data.maxLength || 100;
					textInputType = data.inputType || 'general';
					showTextInput = true;
					isDialogActive = true;
					setTimeout(() => window.focus(), 100);
					break;

				case 'updateTextInput':
					textInputValue = data.value || '';
					break;

				case 'closeTextInput':
					closeTextInput();
					break;

				case 'openKeySelection':
					keySelectionTitle = data.title || 'Select Key';
					keySelectionInstruction = data.instruction || 'Press any key to bind';
					keySelectionHint = data.hint || 'ESC to cancel';
					selectedKey = null;
					selectedKeyName = '';
					showKeySelection = true;
					isDialogActive = true;
					setTimeout(() => window.focus(), 100);
					break;

				case 'updateKeySelection':
					selectedKeyName = data.keyName || '';
					selectedKey = data.keyCode || null;
					break;

				case 'closeKeySelection':
					closeKeySelection();
					break;

				case 'openKeybindSetup':
					keybindSetupTitle = data.title || 'Keybind Setup';
					keybindFeatureName = data.featureName || '';
					keybindInstruction = data.instruction || 'Press any key to bind';
					keybindHint = data.hint || 'ESC to cancel';
					keybindSelectedKey = null;
					keybindSelectedKeyName = '';
					showKeybindSetup = true;
					isDialogActive = true;
					setTimeout(() => window.focus(), 100);
					break;

				case 'updateKeybindSetup':
					keybindSelectedKeyName = data.keyName || '';
					keybindSelectedKey = data.keyCode || null;
					break;

				case 'closeKeybindSetup':
					closeKeybindSetup();
					break;

				case 'updateBanner':
					bannerLink = data.bannerLink || bannerLink;
					bannerColor = data.bannerColor || bannerColor;
					bannerSpacing = data.bannerSpacing || bannerSpacing;
					break;

				case 'updateBreadcrumb':
					breadcrumb = data.breadcrumb || 'Home';
					break;

				case 'requestClipboard':
					handlePaste();
					break;

				// NEW: Image display actions
				case 'showImage':
					showImageDisplay(data.imageType, data.imageUrl, data.label);
					break;

				case 'hideImage':
					hideImageDisplay();
					break;
			}
		} catch (error) {
			console.error('Error handling message:', error);
		}
	});
</script>

<svelte:head>
	<script src="https://unpkg.com/@phosphor-icons/web"></script>
</svelte:head>

<style>
	* {
		background: none;
	}

	.hide-scrollbar {
		overflow-y: auto;
		scrollbar-width: none; /* Hide Firefox scrollbar */
		padding-right: 1vh;
	}
	
	/* Hide webkit scrollbar completely */
	.hide-scrollbar::-webkit-scrollbar {
		display: none;
	}

	/* Custom Scrollbar Indicator */
	.scrollbar-indicator {
		position: absolute;
		top: 0.5vh;
		right: 0.2vh;
		width: 0.4vh;
		background: rgba(0, 0, 0, 0.4);
		border-radius: 0.2vh;
		z-index: 10;
		pointer-events: none;
	}

	.scrollbar-thumb-indicator {
		background: var(--accent-color, #e74c3c);
		border-radius: 0.2vh;
		width: 100%;
		transition: top 0.1s ease, height 0.1s ease;
		box-shadow: 0 0 0.2vh rgba(0, 0, 0, 0.3);
	}

	/* NEW: Dynamic highlight bar styles */
	.menu-highlight-bar {
		overflow: hidden;
		position: absolute;
		left: 0;
		right: 0;
		border-radius: 0.3vh;
		background-color: var(--accent-color, #e74c3c);
		opacity: 0.15;
		box-shadow: inset 0 0 0 0.10vh var(--accent-color, #e74c3c);
		z-index: 0;
		transition: top 120ms ease, height 120ms ease;
		pointer-events: none;
	}

	.notification-container {
		position: absolute;
		top: 1.75vh;
		left: 50%;
		transform: translateX(-50%);
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1vh;
		z-index: 9999;
		pointer-events: none;
	}

	.notification {
		min-width: 25vh;
		max-width: 50vh;
		background-color: #000000e5;
		color: white;
		padding: 1.2vh 2vh;
		border-radius: 0.75vh;
		font-size: 1.3vh;
		font-family: 'Segoe UI', sans-serif;
		box-shadow: 0 0.3vh 1vh rgba(0, 0, 0, 0.5);
		text-align: center;
		pointer-events: auto;
	}

	.notification.info { border: 0.1vh solid var(--accent-color, #3498db); }
	.notification.success { border: 0.1vh solid #2ecc71; }
	.notification.warn { border: 0.1vh solid #f39c12; }
	.notification.error { border: 0.1vh solid #e74c3c; }

	/* NEW: Image display styles */
	.image-display-container {
		position: fixed;
		top: 50%;
		right: 3vh;
		transform: translateY(-50%);
		z-index: 1000;
		background: linear-gradient(135deg, #000000f0 0%, #1a1a1af0 100%);
		border: 0.2vh solid var(--accent-color, #e74c3c);
		border-radius: 1vh;
		padding: 1.5vh;
		box-shadow: 0 1vh 3vh rgba(0, 0, 0, 0.8);
		max-width: 25vh;
		max-height: 35vh;
	}

	.image-display-title {
		color: var(--accent-color, #e74c3c);
		font-size: 1.2vh;
		font-weight: bold;
		text-align: center;
		margin-bottom: 1vh;
		font-family: 'Segoe UI', sans-serif;
	}

	.image-display-img {
		width: 100%;
		height: auto;
		max-height: 25vh;
		object-fit: contain;
		border-radius: 0.5vh;
		border: 0.1vh solid rgba(255, 255, 255, 0.2);
	}

	.image-display-label {
		color: white;
		font-size: 1vh;
		text-align: center;
		margin-top: 1vh;
		font-family: 'Segoe UI', sans-serif;
		opacity: 0.8;
	}

	/* Keybind Setup Dialog */
	.keybind-setup-container {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: 10002;
		width: 40vh;
		max-width: 90vw;
	}

	.keybind-setup-box {
		background: linear-gradient(135deg, #000000f0 0%, #1a1a1af0 100%);
		border: 0.2vh solid var(--accent-color, #e74c3c);
		border-radius: 1vh;
		padding: 2vh;
		color: white;
		font-family: 'Segoe UI', sans-serif;
		box-shadow: 0 1vh 3vh rgba(0, 0, 0, 0.8);
		text-align: center;
		width: 100%;
		box-sizing: border-box;
	}

	.keybind-setup-title {
		font-size: 1.4vh;
		color: var(--accent-color, #e74c3c);
		margin-bottom: 1vh;
		font-weight: bold;
	}

	.keybind-setup-feature {
		font-size: 1.2vh;
		color: var(--accent-color, #e74c3c);
		margin-bottom: 1.5vh;
		font-weight: 500;
		background: rgba(231, 76, 60, 0.1);
		padding: 0.5vh 1vh;
		border-radius: 0.5vh;
		border: 0.05vh solid var(--accent-color, #e74c3c);
	}

	.keybind-setup-instruction {
		font-size: 1.1vh;
		color: #ccc;
		margin-bottom: 1.5vh;
	}

	.keybind-setup-display {
		background: rgba(255, 255, 255, 0.1);
		border: 0.1vh solid rgba(255, 255, 255, 0.2);
		border-radius: 0.5vh;
		padding: 1vh;
		color: white;
		font-size: 1.3vh;
		width: 100%;
		min-height: 2vh;
		font-family: 'Consolas', 'Monaco', monospace;
		margin-bottom: 1vh;
		position: relative;
		text-align: center;
		font-weight: bold;
	}

	.keybind-waiting-text {
		color: #888;
		font-style: italic;
		animation: pulse 2s infinite;
	}

	.keybind-selected-key {
		color: var(--accent-color, #e74c3c);
		font-weight: bold;
		text-shadow: 0 0 0.5vh rgba(231, 76, 60, 0.5);
		animation: glow 1.5s ease-in-out infinite alternate;
	}

	@keyframes glow {
		from { text-shadow: 0 0 0.5vh var(--accent-color, #e74c3c); }
		to { text-shadow: 0 0 1vh var(--accent-color, #e74c3c), 0 0 1.5vh var(--accent-color, #e74c3c); }
	}

	.keybind-setup-hint {
		font-size: 1vh;
		color: #888;
		margin-top: 0.5vh;
	}

	.keybind-setup-confirm {
		font-size: 1vh;
		color: #2ecc71;
		margin-top: 0.5vh;
		font-weight: bold;
	}

	/* Key Selection Dialog */
	.key-selection-container {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: 10001;
		width: 40vh;
		max-width: 90vw;
	}

	.key-selection-box {
		background: linear-gradient(135deg, #000000f0 0%, #1a1a1af0 100%);
		border: 0.2vh solid var(--accent-color, #e74c3c);
		border-radius: 1vh;
		padding: 2vh;
		color: white;
		font-family: 'Segoe UI', sans-serif;
		box-shadow: 0 1vh 3vh rgba(0, 0, 0, 0.8);
		text-align: center;
		width: 100%;
		box-sizing: border-box;
	}

	.key-selection-title {
		font-size: 1.4vh;
		color: var(--accent-color, #e74c3c);
		margin-bottom: 1vh;
		font-weight: bold;
	}

	.key-selection-instruction {
		font-size: 1.1vh;
		color: #ccc;
		margin-bottom: 1.5vh;
	}

	.key-selection-display {
		background: rgba(255, 255, 255, 0.1);
		border: 0.1vh solid rgba(255, 255, 255, 0.2);
		border-radius: 0.5vh;
		padding: 1vh;
		color: white;
		font-size: 1.3vh;
		width: 100%;
		min-height: 2vh;
		font-family: 'Consolas', 'Monaco', monospace;
		margin-bottom: 1vh;
		position: relative;
		text-align: center;
		font-weight: bold;
	}

	.waiting-text {
		color: #888;
		font-style: italic;
		animation: pulse 2s infinite;
	}

	.selected-key {
		color: var(--accent-color, #e74c3c);
		font-weight: bold;
		text-shadow: 0 0 0.5vh rgba(231, 76, 60, 0.5);
	}

	@keyframes pulse {
		0%, 100% { opacity: 0.5; }
		50% { opacity: 1; }
	}

	.key-selection-hint {
		font-size: 1vh;
		color: #888;
		margin-top: 0.5vh;
	}

	.key-selection-confirm {
		font-size: 1vh;
		color: #2ecc71;
		margin-top: 0.5vh;
		font-weight: bold;
	}

	/* Text Input Dialog */
	.text-input-container {
		position: fixed;
		top: 50%;
		left: 50%;
		transform: translate(-50%, -50%);
		z-index: 10000;
		width: 40vh;
		max-width: 90vw;
	}

	.text-input-box {
		background: linear-gradient(135deg, #000000e5 0%, #1a1a1ae5 100%);
		border: 0.2vh solid var(--accent-color, #e74c3c);
		border-radius: 1vh;
		padding: 2vh;
		color: white;
		font-family: 'Segoe UI', sans-serif;
		box-shadow: 0 1vh 3vh rgba(0, 0, 0, 0.7);
		width: 100%;
		box-sizing: border-box;
	}

	.text-input-title {
		font-size: 1.2vh;
		color: var(--accent-color, #e74c3c);
		margin-bottom: 0.8vh;
		text-align: center;
		font-weight: bold;
	}

	.text-input-field {
		background: rgba(255, 255, 255, 0.1);
		border: 0.1vh solid rgba(255, 255, 255, 0.2);
		border-radius: 0.5vh;
		padding: 1vh;
		color: white;
		font-size: 1.3vh;
		width: 100%;
		min-height: 2vh;
		font-family: 'Consolas', 'Monaco', monospace;
		margin-bottom: 1vh;
		position: relative;
		white-space: nowrap;
		overflow: hidden;
	}

	.text-content {
		display: inline-block;
		position: relative;
	}

	.cursor {
		display: inline-block;
		width: 0.1vh;
		height: 1.4vh;
		background-color: white;
		margin-left: 0.1vh;
		animation: blink 1s infinite;
		vertical-align: text-bottom;
	}

	@keyframes blink {
		0%, 50% { opacity: 1; }
		51%, 100% { opacity: 0; }
	}

	.text-input-info {
		font-size: 1vh;
		color: #888;
	}

	.text-input-hint {
		font-size: 1vh;
		color: #666;
		text-align: center;
		margin-top: 0.5vh;
	}

	.placeholder-text {
		color: #666;
		font-style: italic;
	}

	.menu-container {
		position: absolute;
		top: 50%;
		transform: translateY(-50%);
		left: 6.5vh;
		width: 33.5vh;
		background: #000000e5;
		border-radius: 1.5vh;
		box-shadow: 0 0 1vh rgba(0, 0, 0, 0.5);
		display: flex;
		flex-direction: column;
		overflow: hidden;
		font-family: 'Segoe UI', sans-serif;
	}

	.menu-header {
		position: relative;
		width: 100%;
		height: auto;
		max-height: 9.5vh;
		overflow: hidden;
		border-radius: 1.5vh 1.5vh 0 0;
	}

	.menu-header img {
		width: 100%;
		height: auto;
		object-fit: cover;
		display: block;
	}

	.banner-spacing {
		height: var(--spacing);
		background: transparent;
	}

	.breadcrumb-container {
		background: rgba(0, 0, 0, 0.9);
		color: white;
		font-size: 1vh;
		padding: 0.8vh 1.2vh;
		text-align: center;
		border-bottom: 0.05vh solid rgba(255, 255, 255, 0.1);
		font-family: 'Segoe UI', sans-serif;
		font-weight: 400;
	}

	.breadcrumb-text {
		color: var(--accent-color, #e74c3c);
		font-weight: 500;
	}

	.menu-content {
		max-height: 33.6vh;
		overflow-y: auto;
		padding: 0.5rem 0;
		position: relative;
	}

	.menu-item {
		padding: 1.05vh 1.4vh;
		color: white;
		justify-content: space-between;
		display: flex;
		align-items: center;
		font-size: 1.4vh;
		gap: 1vh;
		font-weight: 400;
		cursor: pointer;
		transition: background 0.2s ease-in-out, box-shadow 0.2s ease-in-out;
		margin: 0.3vh 0;
		background-color: transparent;
		position: relative;
		z-index: 1;
	}

	.menu-item.active {
		background-color: transparent;
		border-left: 0.3vh solid var(--accent-color, #e74c3c);
	}

	.menu-icon {
		font-size: 1.3vh;
		color: var(--accent-color, #e74c3c);
		min-width: 18px;
	}

	.menu-label {
		flex-grow: 1;
		text-align: left;
		font-size: 1.2vh;
	}

	.toggle-switch {
		width: 3vh;
		height: 1.5vh;
		background-color: #555;
		border-radius: 1vh;
		position: relative;
		transition: background-color 0.2s ease-in-out;
	}

	.toggle-switch.active {
		background-color: var(--accent-color, #e74c3c);
	}

	.toggle-knob {
		position: absolute;
		width: 1.1vh;
		height: 1.1vh;
		border-radius: 50%;
		background: white;
		top: 0.2vh;
		left: 0.2vh;
		transition: transform 0.2s ease-in-out;
	}

	.toggle-switch.active .toggle-knob {
		transform: translateX(1.45vh);
	}

	.menu-footer {
		background: rgba(0, 0, 0, 0.8);
		color: white;
		font-size: 1.1vh;
		padding: 0.9vh;
		display: flex;
		justify-content: space-between;
		align-items: center;
	}

	.keybind-indicator {
		color: var(--accent-color, #e74c3c);
		font-size: 1vh;
		margin-right: 0.5vh;
		opacity: 0.8;
		transition: opacity 0.2s ease, transform 0.2s ease;
	}

	.menu-item:hover .keybind-indicator {
		opacity: 1;
		transform: scale(1.1);
	}

	/* Range Slider Styles */
	.range {
		width: 10.5vh;
		height: 1.2vh;
		background: transparent;
		margin: 0;
		outline: none;
		border: 0;
	}

	.range,
	.range::-webkit-slider-thumb {
		-webkit-appearance: none;
	}

	.range::-moz-range-thumb {
		border: none;
	}

	.range::-webkit-slider-runnable-track {
		height: 0.55vh;
		border-radius: 0.75vh;
		background: linear-gradient(
			to right,
			rgba(255,255,255,0.95) 0 var(--pct),
			rgb(46, 46, 46) var(--pct) 100%
		);
	}

	.range::-webkit-slider-thumb {
		width: 1.1vh;
		height: 1.1vh;
		margin-top: -0.3vh;
		border-radius: 50%;
		background: #fff;
		cursor: pointer;
	}

	.range:active::-webkit-slider-thumb {
		transform: scale(1.03);
	}

	/* Scroll display styles */
	.scroll-display {
		position: relative;
		display: flex;
		align-items: center;
		gap: 0.5vh;
		font-size: 1.2vh;
		color: white;
		background: rgba(0, 0, 0, 0.4);
		border: 0.1vh solid #555;
		border-radius: 0.5vh;
		padding: 0.5vh 0.8vh;
		min-width: 12vh;
		max-width: 15vh;
	}

	.scroll-arrows {
		color: var(--accent-color, #e74c3c);
		font-size: 1vh;
		opacity: 0.7;
	}

	.scroll-content {
		flex: 1;
		text-align: center;
		white-space: nowrap;
		overflow: hidden;
		text-overflow: ellipsis;
	}
</style>

<!-- Image Display Component -->
{#if showImage}
	<div class="image-display-container" transition:fade={{ duration: 200 }} style="--accent-color: {bannerColor};">
		<div class="image-display-title">
			{#if currentImageType === 'weapon'}
				Weapon Preview
			{:else if currentImageType === 'model'}
				Model Preview
			{:else}
				Preview
			{/if}
		</div>
		<img 
			src="{currentImageUrl}" 
			alt="{currentImageLabel}"
			class="image-display-img"
			on:error={() => hideImageDisplay()}
		/>
		<div class="image-display-label">{currentImageLabel}</div>
	</div>
{/if}

<!-- Keybind Setup Dialog -->
{#if showKeybindSetup}
	<div class="keybind-setup-container" transition:fade={{ duration: 200 }}>
		<div class="keybind-setup-box" style="--accent-color: {bannerColor};">
			<div class="keybind-setup-title">{keybindSetupTitle}</div>
			<div class="keybind-setup-feature">Feature: {keybindFeatureName}</div>
			<div class="keybind-setup-instruction">{keybindInstruction}</div>
			
			<div class="keybind-setup-display">
				{#if keybindSelectedKeyName}
					<span class="keybind-selected-key">{keybindSelectedKeyName}</span>
				{:else}
					<span class="keybind-waiting-text">Waiting for key press...</span>
				{/if}
			</div>

			{#if keybindSelectedKeyName}
				<div class="keybind-setup-confirm">Press ENTER to confirm or ESC to cancel</div>
			{:else}
				<div class="keybind-setup-hint">{keybindHint}</div>
			{/if}
		</div>
	</div>
{/if}

<!-- Key Selection Dialog -->
{#if showKeySelection}
	<div class="key-selection-container" transition:fade={{ duration: 200 }}>
		<div class="key-selection-box" style="--accent-color: {bannerColor};">
			<div class="key-selection-title">{keySelectionTitle}</div>
			<div class="key-selection-instruction">{keySelectionInstruction}</div>
			
			<div class="key-selection-display">
				{#if selectedKeyName}
					<span class="selected-key">{selectedKeyName}</span>
				{:else}
					<span class="waiting-text">Waiting for key press...</span>
				{/if}
			</div>

			{#if selectedKeyName}
				<div class="key-selection-confirm">Press ENTER to confirm or ESC to cancel</div>
			{:else}
				<div class="key-selection-hint">{keySelectionHint}</div>
			{/if}
		</div>
	</div>
{/if}

<!-- Text Input Dialog -->
{#if showTextInput}
	<div class="text-input-container" transition:fade={{ duration: 200 }}>
		<div class="text-input-box" style="--accent-color: {bannerColor};">
			<div class="text-input-title">{textInputQuestion}</div>
			
			<div class="text-input-field">
				{#if textInputValue}
					<span class="text-content">{textInputValue}</span><span class="cursor"></span>
				{:else}
					<span class="placeholder-text">{textInputPlaceholder}</span><span class="cursor"></span>
				{/if}
			</div>

			<div class="text-input-info">
				<span>{textInputValue.length}/{textInputMaxLength}</span>
				<span>Type: {textInputType}</span>
			</div>
			
			<div class="text-input-hint">
				ENTER to confirm • ESC to cancel • CTRL+V to paste
			</div>
		</div>
	</div>
{/if}

<!-- Notification UI -->
<div class="notification-container">
	{#each notifications as { id, message, type } (id)}
		<div class="notification {type}" transition:fade style="--accent-color: {bannerColor};">{message}</div>
	{/each}
</div>

<!-- Main Menu UI -->
{#if showing && !isDialogActive}
	<div class="menu-container" in:fade={{ duration: 200 }} out:fade={{ duration: 200 }}
		 style="--accent-color: {bannerColor};">
		<div class="menu-header">
			<img src="{bannerLink}" alt="Menu Banner" />
		</div>
		
		<div class="breadcrumb-container">
			<span class="breadcrumb-text">{breadcrumb}</span>
		</div>
		
		{#if bannerSpacing > 0}
			<div class="banner-spacing" style="--spacing: {bannerSpacing}px;"></div>
		{/if}
		
		<div class="menu-content hide-scrollbar" bind:this={menuContentElement}
			 on:scroll={updatePositions}>
			<!-- Dynamic Highlight Bar -->
			<div class="menu-highlight-bar" 
				 style="top: {highlightTop}px; height: {highlightHeight}px; --accent-color: {bannerColor};"></div>
			
			<!-- Custom Scrollbar Indicator -->
			{#if scrollbarIndicatorHeight > 0}
				<div class="scrollbar-indicator" style="height: {scrollbarIndicatorHeight}px; --accent-color: {bannerColor};">
					<div class="scrollbar-thumb-indicator" 
						 style="top: {scrollbarThumbTop}px; height: {scrollbarThumbHeight}px;"></div>
				</div>
			{/if}
			
			{#each activeMenu as option, index (option.label)}
				{@const icon = option.icon}
				{@const isActive = index == current}

				<div
					bind:this={itemRefs[index]}
					class="menu-item {isActive ? 'active' : ''}"
				>
					{#if icon}
						<i class={`ph ${icon} menu-icon`}></i>
					{/if}

					<span class="menu-label">{option.label}</span>

					{#if option.canBind}
						<i class="ph ph-keyboard keybind-indicator" 
						   title="This feature can be bound to a key (Press F9)"></i>
					{/if}

					{#if option.type == 'submenu'}
						<i class="ph ph-caret-right" style="font-size: 1.2vh;"></i>
					{:else if option.type == 'checkbox'}
						<div class="toggle-switch {option.checked ? 'active' : ''}">
							<div class="toggle-knob"></div>
						</div>
					{:else if option.type == 'slider'}
						<input 
							type="range" 
							class="range" 
							bind:value={option.value} 
							min={option.min} 
							max={option.max} 
							step={option.step} 
							style="--pct:{option.value / option.max * 100}%;" 
						/>
					{:else if option.type == 'scroll'}
						<div class="scroll-display">
							<span class="scroll-arrows">←</span>
							<span class="scroll-content">
								{#if option.options && option.options[option.selected] && option.options[option.selected].label}
									{option.options[option.selected].label}
								{:else if option.options && option.options[option.selected]}
									{option.options[option.selected]}
								{:else}
									No options
								{/if}
							</span>
							<span class="scroll-arrows">→</span>
						</div>
					{:else if option.type == 'input'}
						<div style="
							background: rgba(0, 0, 0, 0.4);
							border: 0.1vh solid #555;
							border-radius: 0.5vh;
							padding: 0.8vh 1.2vh;
							font-size: 1.2vh;
							max-width: 12vh;
							overflow: hidden;
							text-overflow: ellipsis;
							white-space: nowrap;
							display: flex;
							align-items: center;
							gap: 0.5vh;
						">
							<i class="ph ph-keyboard" style="font-size: 1vh;"></i>
							<span style="color: {option.value ? 'white' : '#888'}; font-style: {option.value ? 'normal' : 'italic'};">
								{option.value || option.placeholder || 'Enter text...'}
							</span>
						</div>
					{/if}
				</div>
			{/each}
		</div>
		<div class="menu-footer">
			Version: Dev | Nexus Private <span style="font-weight: 400;">{current + 1}/{activeMenu.length}</span>
		</div>
	</div>
{/if}
