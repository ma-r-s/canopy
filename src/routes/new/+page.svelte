<script>
	import { onMount } from 'svelte';

	// Initialize the game variables
	let currentQuestion = '';
	let currentValue = 50000000; // Start value
	let minValue = 0;
	let maxValue = 100000000;
	let threshold = 100000; // Precision threshold for binary search
	let currentPath = '';
	let branches = []; // Store the drawn branches
	let finalMessage = '';

	// Random angles for left and right branches
	let leftAngleDegrees = Math.floor(Math.random() * 30) + 10;
	let rightAngleDegrees = Math.floor(Math.random() * 30) + 5;
	let leftAngle = leftAngleDegrees * (Math.PI / 180);
	let rightAngle = rightAngleDegrees * (Math.PI / 180);

	// Create an initial branch structure
	onMount(() => {
		generateTree();
		askNextQuestion();
	});

	// Draw a branch based on the player's path
	function drawBranch(x1, y1, angle, length, depth, path) {
		const x2 = x1 + length * Math.cos(angle);
		const y2 = y1 + length * Math.sin(angle);
		branches.push({ x1, y1, x2, y2, depth, path });
		return { x2, y2 };
	}

	// Generate the tree based on the current path
	function generateTree() {
		let x1 = 0,
			y1 = 40;
		let length = 20;
		let angle = -Math.PI / 2;
		let depth = 1;
		let path = '';
		let newCoords = drawBranch(x1, y1, angle, length, depth, path);

		for (const direction of currentPath) {
			length *= 0.8;
			depth += 1;
			angle += direction === 'L' ? -leftAngle : rightAngle;
			newCoords = drawBranch(newCoords.x2, newCoords.y2, angle, length, depth, path + direction);
			path += direction;
		}
	}

	// Ask the next question based on the current range of values
	function askNextQuestion() {
		currentQuestion = `Would you be willing to lose an arm for ${currentValue.toLocaleString()} pesos?`;
	}

	// Handle the binary search response
	function nextQuestion(choice) {
		if (choice === 'L') {
			// If the player is willing, lower the max value
			maxValue = currentValue;
			currentPath += 'L';
		} else {
			// If not willing, raise the min value
			minValue = currentValue;
			currentPath += 'R';
		}

		// Redraw the tree after each choice

		// Narrow down the value and ask the next question
		currentValue = Math.floor((maxValue + minValue) / 2);

		// Check if the range between max and min is small enough to stop
		if (Math.abs(maxValue - minValue) <= threshold) {
			finalMessage = `You would lose an arm for approximately ${currentValue.toLocaleString()} pesos.`;
		} else {
			askNextQuestion();
		}
		generateTree();
	}

	// Reset the game
	function resetGame() {
		currentPath = '';
		minValue = 0;
		maxValue = 100000000;
		currentValue = maxValue;
		finalMessage = '';
		branches = [];
		generateTree();
		askNextQuestion();
	}
	generateTree();
</script>

<!-- Main Container -->
<div
	class="flex h-screen flex-col items-center justify-center bg-gradient-to-b from-indigo-500 to-purple-700 p-6 font-mono text-white"
>
	<h1 class="mb-8 text-4xl font-bold">Binary Decision Game</h1>

	<!-- Display the tree visualization -->
	<div class="relative mb-8 w-full max-w-screen-md grow overflow-hidden">
		<svg viewBox="-50 -50 100 100" class="absolute inset-0 h-full w-full">
			{#each branches as { x1, y1, x2, y2, depth, path }}
				<line
					{x1}
					{y1}
					{x2}
					{y2}
					stroke={path === currentPath ? 'black' : 'white'}
					stroke-linecap="round"
					stroke-width={2 / depth}
				/>
			{/each}
		</svg>
	</div>

	<!-- Display the current question or final message -->
	{#if finalMessage === ''}
		<div class="mb-4 text-center text-xl">
			<p>{currentQuestion}</p>
		</div>

		<!-- Buttons for yes/no (binary decision) -->
		<div class="flex gap-4">
			<button
				on:click={() => nextQuestion('L')}
				class="rounded-lg bg-green-500 px-4 py-2 hover:bg-green-600"
			>
				Yes
			</button>
			<button
				on:click={() => nextQuestion('R')}
				class="rounded-lg bg-red-500 px-4 py-2 hover:bg-red-600"
			>
				No
			</button>
		</div>
	{:else}
		<div class="text-center text-2xl">
			<p>{finalMessage}</p>
			<button on:click={resetGame} class="mt-4 rounded-lg bg-blue-500 px-4 py-2 hover:bg-blue-600">
				Start Again
			</button>
		</div>
	{/if}
</div>
